# Worked examples — minimal end-to-end threat models

> **Last verified**: 2026-05. The system descriptions here are illustrative; CWE / CAPEC / ATT&CK IDs were spot-checked at that date — re-confirm against the live catalogs before reusing in a deliverable.
> **Sources paraphrased**: MITRE CWE / CAPEC / ATT&CK (public); OWASP Top 10 framing (CC-BY 4.0); generic SaaS/cloud architecture conventions (no single attributable source).

> **Related**: ← `SKILL.md` § "Producing the threat model" • `dfd-mermaid.md` (DFD conventions used here) • `methodologies.md` § "Decision matrix" (these examples instantiate two of the matrix rows) • `data-centric.md` § "Worked example" (the medical / DICOM end-to-end example) • `environments.md` § "AI / ML — domain notes" (the LLM DFD pattern these examples reference for the AI/ML row)

This file collects minimal end-to-end worked examples for the **non-medical** rows of the system-type matrix. The medical / DICOM end-to-end example lives in `data-centric.md` § "Worked example" + `dfd-mermaid.md` § "Worked example: small clinical PACS" — together those cover the medical row. This file covers the rest.

The point of these is not exhaustive coverage. Each example shows a complete shape — §1 + DFD + §2.1 STRIDE + §3 mitigations + §4 self-check — so a user can see what the skill produces on a real system before producing one themselves.

## Example A — Generic web app / SaaS (multi-tenant)

System type matrix row: "Generic web app" / "Cloud-native multi-tenant SaaS".

### §1 — What are we working on?

**System description.** A multi-tenant SaaS expense-reporting product. Web frontend, REST API, Postgres, S3 for receipts, Cognito for end-user identity. Tenants are organizations; each user belongs to exactly one tenant. Per-tenant KMS keys protect receipt blobs at rest.

**Scope.**

- In: web frontend, API service, Postgres, S3 receipt bucket, Cognito user pool, the per-tenant KMS keys.
- Out: corporate SSO (federated to Cognito, but the IdP is the customer's), AWS managed-service internals, the customer's billing-export integrations.

**Assets.**

| ID | Asset | Description | Why it matters |
|----|-------|-------------|----------------|
| AS1 | Tenant receipt blobs | Image / PDF receipts, per-tenant in S3, encrypted with per-tenant KMS keys | PII (cardholder names, addresses); regulatory liability under state privacy laws |
| AS2 | Expense rows + approval state | Postgres tables; describe spend amounts, vendors, approver chains | Financial integrity; audit basis |
| AS3 | Per-tenant KMS keys | Tenant isolation backbone | Compromise breaks tenant isolation across all assets |
| AS4 | Audit log | Append-only Postgres table of approve/edit/export events | Repudiation defense; compliance reporting |

**Trust levels.** TL1 anonymous external; TL2 authenticated end-user; TL3 tenant admin; TL4 service / machine identity (the API service role).

**Assumptions.**

- ASM1: AWS-managed Cognito + KMS internals are out of scope; we trust the AWS shared-responsibility line. Threats below it are noted but not enumerated.
- ASM2: Customer IdP federation tokens are validated by Cognito; we treat token issuance as authoritative.
- ASM3: All inter-service traffic inside the VPC stays inside the VPC (security-group enforced).

**Technology stack and environment.**

- Protocols: HTTPS+TLS 1.3 user→frontend; HTTPS+mTLS frontend→API; HTTPS+SigV4 API→S3; TLS Postgres on internal VPC; HTTPS+SigV4 API→KMS.
- Runtimes: frontend on CloudFront + S3 static hosting; API on ECS Fargate / Node 20; Postgres RDS; receipts on S3.
- Identity: Cognito for end-users (with optional customer-IdP federation); IAM roles for service identity; per-tenant KMS keys.
- Environments: cloud (AWS, single prod account).
- Ownership: vendor owns the prod AWS account; AWS owns provider control plane; customer owns their IdP; end-user owns device + passcode.
- Physical: no on-prem footprint. No physical attack surface in scope.

### Data Flow Diagram

```mermaid
flowchart LR
    subgraph User["End-user | public internet | untrusted"]
        Browser[Browser]
        IDP[Customer IdP]
    end

    subgraph Prod["Vendor | cloud (AWS prod account) | high trust"]
        FE(Frontend / CloudFront)
        API(API Service - Fargate)
        DB[(Postgres - expense rows + audit (AS2, AS4))]
        S3[(S3 receipts (AS1))]
        Cognito(Cognito User Pool)
        KMS(Per-tenant KMS keys (AS3))
    end

    subgraph CSP["AWS | cloud provider control plane | out-of-scope owner"]
        ControlPlane(AWS managed services)
    end

    Browser -- "HTTPS / TLS 1.3" --> FE
    Browser -- "OIDC redirect" --> Cognito
    Cognito -- "SAML/OIDC federation" --> IDP
    FE -- "HTTPS + mTLS" --> API
    API -- "TLS Postgres" --> DB
    API -- "SigV4 + KMS:Decrypt" --> KMS
    API -- "SigV4 + SSE-KMS" --> S3
    KMS -. "tenant key handles only" .-> API
    API -. "managed-service control plane" .-> ControlPlane

    classDef tb fill:none,stroke:#888,stroke-dasharray: 5 5
    class User,Prod,CSP tb
```

### Trust boundaries

| Boundary | Owner (left) | Owner (right) | What crosses | Mediating control |
|---|---|---|---|---|
| User ↔ Prod | End-user / Customer | Vendor | All traffic in/out | TLS 1.3, OIDC, mTLS internal |
| Prod ↔ CSP | Vendor | AWS | Managed-service API calls | IAM + SigV4 |
| Per-tenant boundary (within Prod) | Tenant A | Tenant B | None at runtime; isolated by row-level + KMS-key | RLS + per-tenant KMS key + IAM scope |

### §2.1 — Threats (flow-centric STRIDE)

| ID | Element | STRIDE | Threat | L | I | Risk |
|----|---------|--------|--------|---|---|------|
| T1 | API (P) | S | Attacker uses replayed Cognito token after logout because session revocation isn't enforced server-side | M | M | Medium |
| T2 | API (P) | E | Tenant-scoped query missing in one endpoint allows cross-tenant read (IDOR) | M | H | High |
| T3 | DB (DS, audit-only) | R | Operator with DB-write access edits or deletes audit rows after a privileged action | L | H | Medium |
| T4 | S3 (DS) | I | S3 bucket policy regression makes one tenant's prefix readable by IAM-anonymous | L | H | Medium |
| T5 | API → KMS (flow) | E | API-service IAM role over-broad (kms:* not kms:Decrypt-with-tenant-context); compromised pod can decrypt any tenant | L | H | Medium |
| T6 | API (P) | T | SQL injection in a new search endpoint; ORM bypassed for performance | L | H | Medium |
| T7 | Browser → FE (flow) | T | XSS in receipt-thumbnail rendering inserts JS into the SPA | M | M | Medium |
| T8 | API (P) | D | Aggregation endpoint allows unbounded row scan; one tenant's heavy report DoSes shared API | M | M | Medium |

### §2.1.b — User-needs-centric supplement (multi-tenant business logic)

| ID | Need | Threat | L | I | Risk |
|----|------|--------|---|---|------|
| V1 | "Approver can approve expenses up to a limit" | Approver edits approval limit on their own profile via a profile endpoint not gated by RBAC | L | H | Medium |
| V2 | "User can export their expense history" | Export endpoint accepts `tenant_id` parameter and returns another tenant's data | L | H | Medium ↔ T2 |

### §2.2 — Operational stratum (top threats only)

| Threat ID | STRIDE | CAPEC | CWE(s) | ATT&CK |
|-----------|--------|-------|--------|--------|
| T2 | E | CAPEC-1 — Accessing Functionality Not Properly Constrained by ACLs | CWE-639, CWE-285 | T1078 (Valid Accounts) |
| T6 | T | CAPEC-66 — SQL Injection | CWE-89 | T1190 (Exploit Public-Facing Application) |
| T7 | T | CAPEC-63 — Cross-Site Scripting | CWE-79 | T1059.007 (JavaScript) |
| T5 | E | CAPEC-122 — Privilege Abuse | CWE-269, CWE-732 | T1078.004 (Cloud Accounts) |

(No `(closest pattern; no Detailed available)` footnotes needed — every row has Detailed coverage.)

### §3 — Mitigations

| Threat ID(s) | Risk | Response | Control / mitigation | Owner |
|---|---|---|---|---|
| T2, V2 | High | Mitigate | Tenant scope enforced in middleware, not per-handler. Add cross-tenant access tests in CI for every authenticated endpoint | API team |
| T6 | Medium | Mitigate | Parameterized queries only; lint rule rejects raw SQL in handlers; quarterly SAST scan | API team |
| T7 | Medium | Mitigate | Render receipt thumbnails server-side as sanitized PNG; CSP header on the SPA; trusted-types | Frontend team |
| T5 | Medium | Mitigate | Scope the API IAM role to `kms:Decrypt` with `kms:EncryptionContext` matching tenant ID; alert on context mismatch | Platform team |
| T1 | Medium | Mitigate | Track session in Redis; on logout, invalidate; reject tokens whose session was invalidated | API team |
| T3 | Medium | Mitigate | Audit table is append-only via DB role; nightly hash-chain export to a write-once bucket | Platform team |
| T4 | Medium | Mitigate | IaC-managed bucket policy; AWS Config rule alerts on drift; daily access-logging review | Platform team |
| T8 | Medium | Mitigate | Per-tenant rate limit on aggregation endpoints; pagination required; cost cap | API team |

**Derived requirements (sample):**

- **SR-001**: The API SHALL enforce tenant scope as middleware on every authenticated route, validated by a per-route automated test that asserts a request from tenant A cannot read tenant B's data. *Mitigates: T2, V2 — closes CWE-639, CWE-285.*
- **SR-002**: The API service IAM role SHALL be scoped to `kms:Decrypt` with `kms:EncryptionContext` requiring the request's `tenant_id`, and CloudTrail SHALL alert on any decrypt without matching context. *Mitigates: T5 — closes CWE-732.*
- **SR-003**: Audit table writes SHALL be append-only via DB role, and the table contents SHALL be hash-chain-exported nightly to an object-locked bucket. *Mitigates: T3 — closes CWE-117, CWE-778.*

### §4 — Did we do a good enough job?

- DFD reflects the actual deployment (verified with one platform engineer).
- Every STRIDE category walked at every applicable element.
- All threats have a response.
- Open questions: cross-tenant test coverage on the new `/v2/reports/*` routes — flagged for next sprint.
- Next review trigger: before the customer-IdP federation feature ships, or annually, whichever comes first.

## Example B — Cloud-hosted LLM chatbot with tools

System type matrix row: "AI / ML system" (with the LLM DFD pattern from `environments.md` § "AI / ML — domain notes").

### §1 — What are we working on?

**System description.** A customer-facing support chatbot. End-users chat through a web widget; the backend assembles a prompt from a fixed system prompt, the user message, recent chat history, and top-k chunks retrieved from a public-knowledge-base vector store. The LLM (third-party provider) can call two tools — an HTTP fetch tool restricted to a hostname allowlist, and an internal-DB read tool restricted to one read-only view.

**Scope.**

- In: web widget, app backend, system prompt, chat history store, RAG vector store, the two tools, the LLM-provider API surface.
- Out: LLM-provider internals (managed); the underlying LLM weights (vendor-hosted).

**Assets.**

| ID | Asset | Description | Why it matters |
|----|-------|-------------|----------------|
| AS1 | System prompt + tool descriptions | The instruction surface that defines bot behavior, tool semantics, refusal rules | Exfiltration leaks competitive design and tells attackers exactly which tools to target |
| AS2 | Chat history | Per-user persisted history; may contain user-disclosed PII | PII / state-privacy-law exposure; cross-session memory poisoning surface |
| AS3 | RAG vector store | Embeddings + chunks of public-KB content | Indirect-prompt-injection ingress if the KB ingests attacker-controlled web pages |
| AS4 | DB read-only view | The tool's authorized data scope | Out-of-scope reads are a tool-call hijack outcome |

**Assumptions.**

- ASM1: The LLM provider is third-party; we trust their API surface but not what we send across it. Customer data sent in the prompt is governed by our DPA with the provider.
- ASM2: The public-KB ingestion pipeline scrubs raw HTML but does not filter prompt-injection payloads embedded in plain text.
- ASM3: HTTP tool fetches are restricted to a hostname allowlist and use a per-call signed URL; the tool process does not have outbound internet beyond the allowlist.

### Data Flow Diagram

> Per the canonical LLM DFD pattern in `environments.md` § "AI / ML — domain notes":

```mermaid
flowchart LR
    subgraph User["End-user | public internet | untrusted"]
        U[User]
    end

    subgraph App["Vendor | cloud (app account) | medium trust"]
        Web(Web App / API)
        SysPrompt[(System Prompt + tool descriptions (AS1))]
        ChatStore[(Chat History (AS2))]
        VectorDB[(RAG vector store (AS3))]
    end

    subgraph LLM["LLM Provider | cloud (provider account) | out-of-scope owner"]
        Model("Frontier LLM API")
    end

    subgraph Tools["Vendor | cloud (tool sandboxes) | low trust"]
        ToolHTTP("HTTP fetch tool (allowlist)")
        ToolDB("DB read-only view tool (AS4)")
    end

    U -- "HTTPS chat" --> Web
    Web -- "read system prompt" --> SysPrompt
    Web -- "read recent turns" --> ChatStore
    Web -- "RAG top-k retrieval" --> VectorDB
    Web -- "prompt = sys + history + retrieved + user (HTTPS, mTLS)" --> Model
    Model -- "tool call" --> Web
    Web -- "dispatch (signed)" --> ToolHTTP
    Web -- "dispatch (parameterized)" --> ToolDB
    ToolHTTP -. "untrusted fetch result" .-> Web
    ToolDB -- "row results" --> Web
    Web -- "tool result back to model" --> Model
    Model -- "assistant message" --> Web
    Web -- "render (HTTPS)" --> U

    classDef tb fill:none,stroke:#888,stroke-dasharray: 5 5
    class User,App,LLM,Tools tb
```

### §2.1 — Threats

Combined flow-centric STRIDE + AI/ML supplement (the supplement is where most of the value lives for this system).

| ID | Element / flow | Category | Threat | L | I | Risk |
|----|----------------|----------|--------|---|---|------|
| T1 | U → Web → Model | E (prompt injection) | Direct prompt injection: user smuggles instructions to call `ToolDB` for out-of-scope rows | H | M | High |
| T2 | VectorDB → Web → Model | E (indirect prompt injection) | Attacker plants instructions in a public-KB page that's ingested; instructions execute when the page is retrieved as a chunk | M | H | High |
| T3 | Model → Web → ToolDB | E (tool-call hijack) | Model induced to call `ToolDB` with parameters outside the read-only view's intent (e.g. constructing a row-key the prompt named) | M | H | High |
| T4 | Model → Web → U | I (system-prompt exfil) | User asks the model to recite system prompt; jailbreak-class request reveals tool descriptions and refusal rules (AS1) | M | M | Medium |
| T5 | ToolHTTP → Web → Model | E (tool-result smuggling) | Allowlisted host returns content with embedded prompt-injection; injected instructions execute in the model's next turn | M | M | Medium ↔ T2 |
| T6 | U ↔ Model API | I (model extraction / membership inference) | Many small queries probe boundary behavior to extract training-set membership or distill the system prompt | L | M | Low |
| PR1 | ChatStore writes | LINDDUN — Linking | Cross-session memory persists across users on a shared device; one user's queries linkable to another's | L | M | Low |
| PR2 | Prompt assembly | LINDDUN — Data disclosure | User-pasted PII flows into the LLM provider's processing scope; DPA covers it but not all customer policies allow it | M | M | Medium |

### §2.2 — Operational stratum (top threats)

| Threat ID | CAPEC | CWE(s) | OWASP LLM Top 10 |
|-----------|-------|--------|-------------------|
| T1 | CAPEC-242 — Code Injection (closest available; no Detailed pattern for prompt injection) | CWE-1427 (Improper Neutralization of Input Used for LLM Prompting) | LLM01 — Prompt Injection |
| T2 | CAPEC-548 — Contaminate Resource (closest pattern; no Detailed available for indirect prompt injection) | CWE-1427 | LLM01 (indirect) |
| T3 | CAPEC-122 — Privilege Abuse | CWE-269, CWE-862 | LLM07 — Insecure Plugin Design |
| T4 | CAPEC-118 — Collect and Analyze Information | CWE-200 | LLM06 — Sensitive Information Disclosure |

### §3 — Mitigations

| Threat ID(s) | Risk | Response | Control / mitigation | Owner |
|---|---|---|---|---|
| T1, T2, T5 | High | Mitigate | Treat all retrieved/fetched/user content as untrusted data, never instructions: structure the prompt with explicit role-tagged boundaries, strip instruction-shaped patterns from retrieved chunks, and require tool-call arguments to come from a fixed schema (no free-text passthrough) | App team |
| T3 | High | Mitigate | Tool authorization is server-enforced, not prompt-enforced: `ToolDB` accepts only parameter values from a per-user, per-session whitelist; the LLM cannot widen the view | App team |
| T4 | Medium | Mitigate | System-prompt-exfil refusal rule + post-response filter that detects and blocks system-prompt regurgitation; treat as defense in depth, not a guarantee | App team |
| T5 | Medium | Mitigate | Strip control characters and instruction-shaped boilerplate from HTTP-tool responses before re-inserting into the model context; cap response size | App team |
| T6 | Low | Accept | Per-user rate limit; documented residual risk in the DPA addendum | Platform team |
| PR2 | Medium | Mitigate | Pre-prompt PII redaction on user input (best-effort regex + entity tagging); user-facing notice that prompts are processed by the LLM provider | App team |
| PR1 | Low | Mitigate | Memory keyed by authenticated session ID, not device | App team |

**Derived requirements (sample):**

- **SR-001**: Tool calls SHALL be authorized server-side by mapping the model's structured tool-arg schema to a per-session allowlist; the LLM provider's argument values SHALL NOT be the sole source of authorization. *Mitigates: T3 — closes CWE-862, CWE-269.*
- **SR-002**: Retrieved content (RAG chunks, HTTP-tool responses) SHALL be passed to the model with an explicit "data, not instruction" role-tag; prompt template tests SHALL verify the model does not follow instructions originating in tool-output role-tags. *Mitigates: T2, T5 — closes CWE-1427.*

### §4 — Did we do a good enough job?

- DFD reflects the actual app (verified with the platform lead).
- AI/ML supplement run; OWASP LLM Top 10 mapped on top threats.
- Every threat has a response.
- Open questions: how to test SR-002 reliably (prompt-injection eval suite needed); how often to re-ingest the public KB.
- Next review trigger: before adding any third tool, before enabling cross-session memory, or annually.
