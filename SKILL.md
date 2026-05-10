---
name: threat-modeler
description: Use when the user asks the assistant to *produce* a threat model, security architecture review, attack-surface analysis, or "what can go wrong with this system" — typically against a paste of an architecture, design, spec, code, or component list — or when they explicitly name a methodology (STRIDE, LINDDUN, PASTA, STPA-SafeSec) and ask for design-time analysis of *their* system. Also use when the user mentions a regulatory threat-model deliverable (FDA premarket cybersecurity, IEC 81001-5-1, IEC 62443, ISO 26262 cyber). The trigger is the request to *produce an artifact*, not the mention of a term. Do **not** use for: educational queries ("what does S in STRIDE stand for?", "explain DFD trust boundaries", "summarize LINDDUN", "what is PASTA?"); definitions, glossary lookups, or methodology comparisons that don't reference a specific system; coursework / certification study help; pen-test planning; vulnerability scanning; incident response; SBOM generation; threat intelligence summaries; CVE / advisory triage; or passing references to STRIDE / DFD / "attack surface" that appear in unrelated discussions (resume review, blog post drafting, code review of an unrelated change). When in doubt, the question to ask is "is the user asking me to produce a §1 / §2 / §3 / §4 artifact about a system they've described?" — if yes, use the skill; if no, answer the question directly without invoking it.
---

# Threat Modeler

Produce a working threat model for an engineering team. State assumptions and proceed rather than interrogating the user. Ship a useful approximation rather than polishing a perfect artifact. When STRIDE categorization is debatable, record the threat and move on. Avoid two anti-patterns: **Admiration for the problem** (analyzing without producing solutions — every threat gets a response) and **Asset rabbit-holing** (padding the asset list). Full Manifesto values, principles, and pattern catalog: `references/manifesto.md`. Before shipping a draft, scan it against the failure-mode catalog in `references/manifesto.md` § "Failure-mode catalog — what bad output looks like" — padded asset lists, single-threat-per-element, generic mitigations ("apply RBAC"), STRIDE labels with no concrete attack, etc. If the draft has those shapes, fix them before sending; a model with those failure modes is worse than no model because it implies threats were considered when they weren't.

## Pick a mode

- **Guided interview** — user said "help me threat model X" without a system description. Run §"Guided interview" to scope, then produce the model.
- **Fast-path** — user pasted architecture, a DFD, a component list, a spec, code, or a description detailed enough to identify external entities, processes, data stores, and trust boundaries. Skip the interview, but **before drawing the DFD** pin down the four things from Round 1.5 (technology stack, environment type, environment ownership, physical/operational context). If any aren't in the artifact, state your inferred answer as a numbered assumption ("ASM3: assumed customer IT owns the on-prem network; correct if otherwise") and proceed.
- **Update / extend** — user pasted an existing threat model. Treat their document as input: preserve existing IDs, only renumber on collision, mark new entries clearly (e.g. `T17 [new]`, `SR-014 [new]`), and run a Q4 model/reality conformance check first per `references/validation.md`. **Branch on the existing structure** — most existing models won't use this skill's three-stratum hybrid layout, and silently imposing it is rude:
  - **If the existing document uses §1 / §2 / §3 / §4 (Four Question Framework) but not the three-stratum §2.1 / §2.2 / §2.3 split** (most common shape): preserve their §2 structure. Add new threats inline in their existing tables. If the user explicitly asks for the operational or strategic stratum, add `§2.x` subsections rather than restructuring. State this once: "Keeping your existing §2 structure; new threats appended below."
  - **If the existing document doesn't use the Four Question Framework at all** (e.g. a vendor-specific template, a Microsoft TMT export, a Polarion-imported table): match their structure. Don't reorganize. Add new entries in the same shape they used. If their structure makes Q4 hard to find, add a Q4 self-assessment in their style at the end and say so.
  - **If the existing document already uses the three-stratum hybrid**: extend it in place; that's what the defaults are for.
  - **Default to ASK before restructuring.** If you're tempted to convert their model to the three-stratum layout because it would be cleaner, surface that as an explicit offer ("Your model uses §1/§2/§3/§4 without strata; want me to keep that, or convert to the hybrid layout?") rather than rewriting silently. The user's existing structure is usually load-bearing — it ties to a tracker, a regulator template, or a team convention.
  - **ID-prefix conventions stay flexible in update mode.** If the existing model uses `A1, A2, ...` for assets (instead of `AS1, AS2, ...`) or `RR-001` for requirements (instead of `SR-001`), match their prefixes. The convention serves the team, not the skill.

When in doubt, state assumptions and proceed.

## Handling sensitive input

Threat modeling invites users to paste architecture, data-class names, identifiers, and sometimes credentials-adjacent material. Before processing user input, surface this guidance once at the start of the session — not as a gate, but so the user can decide what to share:

- **Redact before pasting**: production credentials, API keys, customer PII / PHI, real patient identifiers, internal hostnames, IPs of production systems, signing-key fingerprints, URLs of internal-only endpoints. The model doesn't need any of these to produce a useful threat model — placeholders (`prod-db-1`, `<internal-host>`, `customer-X`) work just as well, and a redacted DFD reads no worse than a literal one.
- **Suspicious-string check**: if the pasted artifact contains anything that looks like a JWT, base64 secret, AWS access key (`AKIA…`), GitHub token (`ghp_…`), connection string with embedded credentials, `BEGIN PRIVATE KEY`, or a real customer name, **stop, flag it to the user, and ask whether to proceed with redaction or pause**. Don't silently include the substring in the output.
- **The produced threat model is itself sensitive.** A complete threat model lists what the system protects, where the trust boundaries are, and what isn't yet mitigated — exactly the things an attacker would want. State this in the document's header and recommend the team store it under the same access controls as the design documents it describes (typically internal-only or SOC-restricted). Default the template's `Status` field to `Draft / Internal` and ask before recommending wider distribution.
- **If the user names a third party**: don't speculate about that third party's vulnerabilities or named-adversary exposure beyond what's in public advisories. Stick to the system the user is responsible for.

This guidance applies across every mode (guided / fast-path / update). State it once, then proceed; don't re-prompt unless new pasted material reintroduces the concern.

## The Four Question Framework

Every threat model answers four questions, in order (Shostack; adopted by the Threat Modeling Manifesto):

1. **What are we working on?** → System decomposition + DFD with trust boundaries.
2. **What can go wrong?** → Threat enumeration.
3. **What are we going to do about it?** → A response (Mitigate / Eliminate / Transfer / Accept) for each threat, with concrete controls for "Mitigate" and derived security requirements.
4. **Did we do a good enough job?** → A self-assessment plus open questions.

Treat the output of Q2 as **hypothesized attack scenarios, not confirmed vulnerabilities** — they need validation by code review, testing, or red-teaming. Don't skip Q4.

## Guided interview

Cluster questions; don't fire one at a time. Two or three turns of clustered questions is the target.

**Round 1 — Scope and system understanding (Q1).**
- One-paragraph system description.
- External entities (users, other systems, third parties).
- Major processes (services / applications / components that *do* something).
- Data stores (databases, files, queues, caches, configuration).
- Where trust boundaries sit (network, process, privilege, tenant, physical).
- Sensitive data flowing through it (PII, PHI, credentials, financial, IP, safety-critical control signals). If one or two data classes dominate the threat picture, name them — they seed the data-centric supplement (`references/data-centric.md`).

**Round 1.5 — Technology, environment, and ownership (Q1, before the DFD).** This round is what turns a generic DFD into one that reflects this system in its environment. Skipping it is how trust boundaries get missed.

- **Protocols on each flow and runtimes per process.** Cite the actual protocol per flow ("HTTPS + mTLS", "MQTT over TLS", "BLE GATT") — generic labels like "data" hide threats. Identify each process's runtime/host, the identity provider, the secrets store, and the crypto modules. Per-environment protocol catalog: `references/environments.md`.
- **Environment type per zone** from a fixed taxonomy: cloud, on-prem enterprise, embedded / IoT, OT / ICS, mobile, or hybrid (pick the dominant one and note the bridge). Boundary patterns differ sharply by type — `references/environments.md` is the catalog.
- **Ownership per zone.** Who owns the underlying network, host, account, or device? Common owners: vendor, customer, end-user, cloud provider, mobile-OS vendor, third-party SaaS. Ownership ≠ data custody and ≠ legal accountability — it's "who can change configuration, run the patch cycle, control IAM." Mark each zone owner explicitly in the DFD subgraph label. If an owner isn't named, record an assumption and proceed.
- **Physical and operational context.** Where does the device/host live (controlled facility, public space, user's pocket, attacker's hands)? Tamper protections (secure boot, secure element / TEE)? Debug ports, USB, removable media exposed? Network air-gapped, segmented, DMZ-fronted, or directly internet-exposed? These shape both what trust boundaries exist and what threats cross them.
- **Multi-device shared physical space.** When several devices share one physical space (smart building zone, plant-floor cabinet, vehicle cabin, robotics cell, clinical room), one device's RF / acoustic / power surface can reach another. If this applies, plan a multi-device cyber-physical attack-path pass — see `references/environments.md` and `references/methodologies.md` § "Risk-prioritized cyber-physical attack paths".

**Round 2 — Context that shapes threats.**
- Who would attack this and why? (External attacker, malicious insider, supply chain, nation-state.) Even a one-line answer seeds ATT&CK mapping — don't dwell.
- Safety implications? Drives the safety-bump rule from `references/risk-rating.md`.
- Regulatory constraints?
- What sector? Determines which sector ISAC and regulator framing apply (full lists in `references/methodologies.md`).
- Existing security controls already assumed in place?

If something's still missing after Round 2, ask one focused round — but err on stating assumptions and proceeding. After Round 2 you should have enough to draft the DFD and pick supplements from §"Picking supplements" below. Domain-specific guidance loads on demand: medical / PACS / DICOM → `references/medical.md`; embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes".

## Producing the threat model

Output a single markdown document. Default to a single document; only split into linked documents if the user asks. Use Mermaid for the DFD. Copy the skeleton from `assets/threat-model-template.md` rather than rebuilding it.

```
# Threat Model: <System Name>

## 1. What are we working on?  (Contextual setup)
   - System description (1–2 paragraphs)
   - In-scope / out-of-scope (explicit list)
   - Assets (the things worth protecting; don't pad)
   - Trust levels
   - Assumptions (numbered, so they can be challenged)
   - Technology stack and environment (Round 1.5 outputs)
   - Data Flow Diagram (Mermaid)
   - Trust boundary table

## 2. What can go wrong?
   ### 2.1 Contextual stratum (system-specific)
       - Flow-centric STRIDE-Per-Element threat table
       - Optional: a supplementary entry-point pass (data-centric, asset-centric,
         user-needs-centric, process-centric, code-centric) when one is warranted
         per the system-type matrix in `references/methodologies.md`
       - Optional: LINDDUN privacy table if PII/PHI is in scope; AI/ML-specific
         threat list if ML components are present
       - Optional: threat tree(s) for the top 1–2 highest-value threats

   ### 2.2 Operational / Tactical stratum (generic adversary techniques)
       - ATT&CK technique IDs on top threats
       - If included, the STRIDE → CAPEC → CWE → mitigation chain on top threats
         (the design-time payoff over plain ATT&CK; format and STRIDE→CAPEC mapping
         in `references/capec.md`)
       - Optional: Cyber Kill Chain mapping; CVSS where threats map to known CVEs

   ### 2.3 Strategic stratum (sector landscape)
       - Sector ISAC / threat-intel pointers (full lists in `references/methodologies.md`)
       - Regulatory framing
       - Named-adversary context where applicable; PASTA business-impact framing
         if executive sign-off is needed

## 3. What are we going to do about it?
   - Mitigation table covering threats from any stratum present, in one prioritized
     list — sharing one ID space and one risk-rating scale (qualitative L/M/H by
     default; OWASP-RR or FMEA where the org requires — see `references/risk-rating.md`)
   - Each row: threat ID → response (Mitigate/Eliminate/Transfer/Accept) → control(s)
   - Cross-stratum references where the same finding surfaced in multiple strata —
     never duplicate threats; cross-reference
   - Derived security requirements (numbered, testable, importable into a tracker)

## 4. Did we do a good enough job?
   - Self-assessment (see §"Mitigations and Q4")
   - Open questions and assumptions still to validate
   - Recommended next review trigger
```

**Default scaffold rule.** The default layout has three top-level subsections under §2 — Contextual, Operational, Strategic. Add content to a subsection when you have something to say there. Omit a subsection entirely when you don't. No "not applicable" stubs. The three-stratum layout is *available scaffolding*, not a coverage quota — it comes from Tatam et al. (2021), one paper, not a practitioner consensus. Background and the system-type matrix that suggests which supplements to add: `references/methodologies.md` § "Hybrid as default".

ID conventions to keep cross-stratum references unambiguous (one prefix per namespace, chosen so single-letter prefixes can't be misread as truncations of multi-letter ones):

- Assets in §1: `AS1`, `AS2`, …
- Assumptions in §1: `ASM1`, `ASM2`, … (avoid bare `A1` — it's visually confusable with `AS1` and breaks grep)
- Flow-centric STRIDE threats: `T1`, `T2`, …
- Supplementary entry-point findings: `V1`, `V2`, … (data-centric / asset-centric / user-needs-centric / process-centric / code-centric — use the `V` prefix regardless of pass type, with the pass type recorded in the row; avoid `A#` for asset-centric findings since it collides with assets)
- Privacy / LINDDUN / AI-ML findings: `PR1`, `PR2`, …
- Mitigations / derived requirements: `SR-001`, `SR-002`, …
- ATT&CK / CAPEC / CWE / CVE: cite the upstream ID directly

## DFD

Use Mermaid `flowchart LR` (or `TD`). Map elements as: external entity → rectangle (`EE[User]`); process → rounded rectangle or circle (`P1(Auth Service)`); data store → cylinder (`DS[(User DB)]`); data flow → labeled arrow (`EE -- "credentials" --> P1`); trust boundary → Mermaid `subgraph`. Render every trust-boundary subgraph with a dashed border using the `classDef tb fill:none,stroke:#888,stroke-dasharray: 5 5` + `class <Subgraph1>,<Subgraph2>,... tb` pair at the bottom of the diagram — apply this pattern to every diagram, not optional. Label each subgraph with owner, environment type, and trust level. Full conventions, the per-subgraph `style` alternative for single-zone diagrams, the labeling format, and worked examples in `references/dfd-mermaid.md`. The per-environment patterns that should drive *which* subgraphs you draw live in `references/environments.md`.

Three rules, applied in order:

1. **Every element belongs to exactly one zone subgraph.** A floating element with no zone is a missing trust boundary. If an entity is genuinely outside any controlled zone (an attacker on the public internet), put it in an explicit `Untrusted` subgraph.
2. **Nest subgraphs for trust-boundaries-within-trust-boundaries.** Cloud account / VPC / subnet, Windows host / VM / container, embedded device / secure element, mobile device / app sandbox / hardware keystore. Pattern: `references/dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries".
3. **Reconcile the DFD against §1 before threat enumeration.** Every asset and trust boundary mentioned in §1 prose must appear in the DFD. If §1 names something the DFD doesn't show, fix the DFD — don't quietly drop the element.

A single diagram with 5–15 elements is usually right; for larger systems produce a Level-0 (context) diagram and one or more Level-1 (decomposed) diagrams rather than one unreadable mess.

## Threat enumeration

The contextual core is **STRIDE-Per-Element on the DFD**. Use STRIDE generatively as a per-element prompt — don't argue about which cell a finding belongs in. Read `references/stride-prompts.md` before enumerating: it has the applicability table (which categories apply to External entity / Process / Data flow / Data store), the category prompts, the element-is-the-victim framing, and the Per-Element vs Per-Interaction choice.

For each DFD element, walk the STRIDE categories that apply. Write at least one concrete threat per applicable cell (e.g. "An attacker on the public internet sends crafted requests to the auth service to bypass token validation" — not "Spoofing"). Threats follow data flow, not control flow, and cluster around trust boundaries *and* complex parsers (deserializers, image decoders, structured-document parsers).

Threat table format:

| ID | Element | STRIDE | Threat | Likelihood | Impact | Risk |
|----|---------|--------|--------|------------|--------|------|
| T1 | Auth Service (P1) | S | Attacker reuses captured session token across boundary | M | H | High |

Use qualitative L/M/H by default. For safety-critical systems, see `references/risk-rating.md` for the safety-bump rule before assigning ratings. Avoid DREAD. If quantitative scoring is required, use OWASP Risk Rating Methodology (see `references/risk-rating.md`).

Iteration tactics: walk across trust boundaries first (highest-value threats live there); start with whatever crosses the outermost boundary; don't skip a threat because it's not the category you're walking right now (write it down — redundancy is fine); every recorded threat ends up as a tracked work item in the team's bug tracker. Full tactics catalog: `references/stride-prompts.md` § "Enumeration tactics".

For the highest-risk handful of threats (top 3–5), consider drawing a threat tree showing how the threat could be realized. One Mermaid tree per top threat is plenty.

## Mitigations and Q4

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a concrete control mapped to the violated property — STRIDE → security property → typical mitigations table in `references/stride-prompts.md`.

If you produce §2.2 (operational stratum), include CAPEC and CWE alongside ATT&CK. The CAPEC → CWE bridge is what makes derived requirements (`SR-###`) traceable to a known weakness class rather than a free-text threat sentence. **Pick the CAPEC abstraction level for the user, by SDLC stage** (Meta for early architecture, Standard for design review, Detailed for component-level work) and don't expose the level in the row unless you were forced to use a higher abstraction because no Detailed pattern exists for the domain-specific protocol (DICOM, HL7, ICS). When forced, footnote `(closest pattern; no Detailed available)`. Format, the abstraction-level rule of thumb, and STRIDE→CAPEC mapping: `references/capec.md`.

Derive security requirements from mitigations. Each one is testable and stable enough to import into a tracker:

```
SR-001: The system SHALL authenticate all service-to-service API calls using mutual TLS with X.509 certificates issued by <CA>.
   Mitigates: T3, T7
```

**Q4 — minimum viable.** Three checks (full Shostack-style checklists in `references/validation.md`):

1. **Model/reality conformance** — does the diagram match what was actually built or planned?
2. **Every threat has a response** — Mitigate / Eliminate / Transfer / Accept, each tracked in the team's bug system.
3. **Every section that's present is populated** — no empty subsections, no stratum stubs.

Plus: open questions / assumptions still to validate, and a **next review trigger** (e.g. "before the cloud upload feature ships," "next architecture revamp," "annually"). Q4 is the most-skipped step in real threat models. Don't skip it.

## Picking supplements

The contextual core (flow-centric DFD + STRIDE) almost never gets swapped. Add supplements when warranted; the system-type matrix in `references/methodologies.md` § "Decision matrix" is the cheat sheet. One-line summary:

| Supplement | When | Reference |
|---|---|---|
| Data-centric (NIST SP 800-154) | One data type dominates or framing is data-typed (PCI, GDPR, HIPAA, FDA) | `references/data-centric.md` |
| LINDDUN | PII/PHI is in scope or privacy is a stated concern | `references/methodologies.md` § LINDDUN |
| User-needs-centric (abuse-case inversion) | Rich business logic, multi-tenant SaaS, delegation/approval flows | `references/centric-methods.md` |
| Process-centric | Operational surface is significant (deploys, on-call, key rotations) | `references/centric-methods.md` |
| Asset-centric | Clear crown-jewel assets (signing keys, KMS roots, control-loop setpoints) | `references/centric-methods.md` |
| Code-centric | Source available; use as a validation pass after STRIDE | `references/centric-methods.md` |
| Attack tree | Top 1–2 highest-value threats where adversarial reasoning adds value | `references/methodologies.md` § Attack trees |
| AI/ML threat list | ML components in scope (prompt injection, training-data poisoning, model extraction) | `references/methodologies.md` § ML/AI; OWASP LLM Top 10 |
| PASTA framing | Borrow the business-impact stage when executive sign-off is needed | `references/methodologies.md` § PASTA |
| STPA-SafeSec **(opt-in only — do not load by default)** | Safety-critical control loops where worst case is physical harm (medical devices, ICS, automotive, aerospace, robotics); swap or supplement mode. Niche; ~14 KB of reference. Load `references/stpa-safesec.md` only when the system is a control loop *and* one of: regulator requires the joint safety+security artifact (FDA premarket cybersec for control-loop devices, IEC 62304/81001-5-1, IEC 61508, ISO 26262); team needs hazard scenarios alongside threats. Skip otherwise. | `references/stpa-safesec.md` |
| OCTAVE / VAST | Org/portfolio-level only, not per-system | `references/methodologies.md` |

Domain pointers: medical / PACS / DICOM / IoMT → also load `references/medical.md`. Embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes" has each domain's specifics.

## References

Read on demand:

- `methodologies.md` — Hybrid framing, system-type decision matrix, layer catalog, single-doc vs linked-doc shapes, pruning rules. When to use LINDDUN / PASTA / attack trees / ATT&CK / kill chain / OCTAVE / VAST. Cyber-physical attack-path construction. Sector ISAC and regulator lists.
- `environments.md` — Per-environment trust-boundary patterns (cloud, on-prem enterprise, embedded / IoT, OT / ICS, mobile) + ownership taxonomy + domain notes (embedded specifics, cloud-native, AI/ML, third-party code). Read when drawing the DFD.
- `medical.md` — Default layer mix for medical / PACS / DICOM / IoMT, patient-as-asset framing, clinical workflow misuse, regulatory list (FDA / IEC 62304 / IEC 81001-5-1 / HIPAA / H-ISAC), DICOM/HL7 STRIDE specifics. Load only when the system is medical.
- `centric-methods.md` — Entry-point taxonomy (asset, data, flow, process, user-needs, attacker, code) and the generation-vs-characterization distinction.
- `data-centric.md` — NIST SP 800-154 workflow.
- `capec.md` — Operational-stratum reference: STRIDE → CAPEC → CWE → mitigation chain, three abstraction levels (Meta / Standard / Detailed), CAPEC-1000 and CAPEC-3000 views, coverage gaps.
- `stride-prompts.md` — STRIDE category prompts, per-element example threats, Per-Element vs Per-Interaction, DESIST variant, enumeration tactics.
- `dfd-mermaid.md` — DFD-to-Mermaid mapping with worked examples, dashed-subgraph pattern, subgraph labeling convention, nested-subgraph pattern, diagramming checklist.
- `validation.md` — Q4 deep dive: model/reality conformance, the three Shostack-style checklists, pen testing as complement, validating threats in third-party code.
- `manifesto.md` — Threat Modeling Manifesto values, principles, patterns, anti-patterns (paraphrased).
- `risk-rating.md` — Qualitative L/M/H, OWASP-RR pointer, safety-bump rule, why DREAD is discouraged.
- `stpa-safesec.md` — Safety+security joint hazard analysis for control-loop systems. Two modes (swap or supplement). Use when the worst case is physical harm and the system is a control loop.
- `examples.md` — Pointer to the OWASP Threat Model Library (https://github.com/OWASP/www-project-threat-model-library) as the source for end-to-end worked examples by system type (web-applications, ai-ml-systems, infrastructure, third-party-integrations). The library's JSON schema also defines the structured-sidecar contract — see § "Machine-readable sidecar" above. The medical / DICOM end-to-end example is the exception, kept inline at `data-centric.md` § "Worked example" + `dfd-mermaid.md` § "Worked example: small clinical PACS".

Blank template: `assets/threat-model-template.md`.

## Machine-readable sidecar (for tracker import)

The default output is a markdown document. When the user is going to import findings into Polarion, Jira, GitHub Issues, ServiceNow, or any other tracker — or when they ask for a structured artifact for tooling — also emit a JSON sidecar alongside the markdown. **Use the OWASP Threat Model Library schema** (`threat-model.schema.json` from https://github.com/OWASP/www-project-threat-model-library, MIT-licensed; aligns with the pending CycloneDX TM-BOM specification). Don't invent a YAML schema — the upstream JSON schema is the contract downstream tools already implement against.

When the user asks for tracker import, asks "can I export this to Jira/GitHub/Polarion", or names a downstream tool, emit the sidecar by default. Otherwise mention it once and offer it. Validate the JSON output against the upstream schema before handing it over (e.g. `check-jsonschema --schemafile threat-model.schema.json <output>.json`).

The markdown remains the canonical artifact; the JSON sidecar is the derived view. Don't produce only the JSON.

**Mapping from this skill's IDs to OWASP TML schema fields** (the schema's symbolic-name fields take this skill's IDs verbatim — `AS1`, `T1`, `V1`, `PR1`, `SR-001`, `TB1` — so cross-references stay readable):

| This skill | OWASP TML field |
|---|---|
| §1 system description, scope, business criticality | `scope` |
| §1 assumptions (`ASM#`) | `assumptions` |
| §1 actors / external entities | `actors` |
| §1 processes + data stores (DFD nodes) | `components`, `data_stores` |
| §1 data classes (data-centric pass) | `data_sets` |
| §1 trust zones (DFD subgraphs) | `trust_zones` |
| §1 trust boundaries table | `trust_boundaries` |
| §1 DFD (Mermaid) | `diagrams` (Mermaid source inline) |
| §2 threats (`T#` / `V#` / `PR#`) | `threats` (CWE / CAPEC IDs in their own fields per the schema) |
| §2.2 STRIDE category, ATT&CK, kill chain | `threats[].extensions` (vendor-namespaced — see schema's `extensions` rules; recommended namespace `tmskill.threat-modeler/`) |
| §3 controls / mitigations | `controls` |
| §3 derived requirements (`SR-###`) | `controls` (with `requirement` extension) or `risks` cross-ref |
| §1 / §2 risk ratings (L/M/H, optional Critical) | `risks` |

Where the upstream schema doesn't carry a field this skill produces (STRIDE category labels, the Stratum tag, the Stellios path-product score), use `extensions` with reverse-domain naming (e.g. `tmskill.threat-modeler/stride`, `tmskill.threat-modeler/stratum`, `tmskill.threat-modeler/path-score`) — never invent top-level fields.

Worked example threat models in the OWASP TML JSON shape live in https://github.com/OWASP/www-project-threat-model-library/tree/main/threat-models, organized by system type: `ai-ml-systems/`, `web-applications/`, `infrastructure/`, `third-party-integrations/`. Reference one of those when producing a structured sidecar so the output matches what downstream consumers expect.

## Citations

Concepts in this skill paraphrase from:

- **OWASP** — Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, Security Culture v1.0 §6, Threat Modeling Playbook (Toreon), **Threat Model Library** (https://github.com/OWASP/www-project-threat-model-library, MIT-licensed; structured-sidecar JSON schema and the curated example-models repository the skill defers to for worked examples).
- **Threat Modeling Manifesto** — values, principles, patterns, anti-patterns (CC-BY 4.0).
- **Adam Shostack**, *Threat Modeling: Designing for Security* (Wiley, 2014) — Four Question Framework, STRIDE-Per-Element, diagramming and validation checklists, iteration tactics, bug-filing as exit point.
- **NIST SP 800-154** (Draft, Souppaya & Scarfone, 2016) — data-centric methodology.
- **Tatam, Shanmugam, Azam & Kannoorpatti**, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021, CC-BY 4.0) — three-stratum hybrid framing; CAPEC → CWE design-time payoff.
- **MITRE** — ATT&CK, CAPEC, CWE.
- **Lockheed Martin** — Cyber Kill Chain.
- **STRIDE** — Loren Kohnfelder & Praerit Garg (Microsoft). **DESIST** — Gunnar Peterson.
- **STPA-SafeSec** — Friedberg et al. (2017, CC-BY). **STPA-Sec** — Young & Leveson (2014). **STPA** — Leveson, *Engineering a Safer World* (MIT Press, 2011).
- **Risk-prioritized cyber-physical attack paths** — Stellios, Kotzanikolaou & Grigoriadis (2021).

Cite these in any output that quotes or substantively summarizes them.
