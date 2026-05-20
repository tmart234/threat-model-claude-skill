---
name: threat-modeler
description: 'Use when the user asks the assistant to *produce* a threat model, security architecture review, attack-surface analysis, or "what can go wrong with this system" — typically against a paste of an architecture, design, spec, code, or component list — or when they name a methodology (STRIDE, LINDDUN, PASTA, STPA) and ask for design-time analysis of *their* system. Also use for regulatory threat-model deliverables (FDA premarket cybersecurity, IEC 81001-5-1, IEC 62443, ISO 26262 cyber). The trigger is the request to *produce an artifact*, not the mention of a term. Do **not** use for educational queries, definitions, methodology comparisons not tied to a specific system, coursework, pen-test planning, vulnerability scanning, incident response, SBOM generation, threat intel, CVE triage, or passing references to STRIDE / DFD / "attack surface" in unrelated discussions. When in doubt: is the user asking me to produce an artifact about a system they''ve described? If yes, use the skill; if no, answer directly.'
---

# Threat Modeler

Produce a working threat model for an engineering team. State assumptions and proceed rather than interrogating. Ship a useful approximation rather than polishing a perfect artifact. When STRIDE categorization is debatable, record the threat and move on. Before shipping, scan the draft against `references/manifesto.md` § "Failure-mode catalog" (padded asset lists, single-threat-per-element, generic mitigations like "apply RBAC", threat cells that aren't a title-plus-description, mitigation table that doesn't match the threat table) — a model with those shapes is worse than no model because it implies threats were considered when they weren't. Manifesto values, principles, and anti-patterns: `references/manifesto.md`.

**Minimum viable threat model.** For low-stakes systems (internal tool, no sector adversary, no regulator, no SOC handoff), the complete deliverable is: §1 with a DFD + a few assumptions, §2.1 STRIDE-Per-Element, §3 with one response per threat, and the three-check Q4. That's a finished threat model — not a single-method deficient one. The three-stratum layout, TM-BOM JSON emission, persona/event/source columns, and CAPEC → CWE chain are escalations the *system* pulls in (regulated, multi-tenant, safety-critical, sector-targeted), not defaults the skill imposes. Match output to stakes; the skill is a menu, not a coverage quota.

## Pick a mode

- **Guided interview** — user said "help me threat model X" without a system description. Run §"Guided interview" to scope, then produce the model.
- **Fast-path** — user pasted architecture, a DFD, a component list, a spec, code, or a description detailed enough to identify external entities, processes, data stores, and trust boundaries. Skip the interview, but **before drawing the DFD** pin down the four things from Round 1.5 (technology stack, environment type, environment ownership, physical/operational context). If any aren't in the artifact, state your inferred answer as a numbered assumption ("ASM3: assumed customer IT owns the on-prem network; correct if otherwise") and proceed.
- **Update / extend** — user pasted an existing threat model. Treat their document as input: preserve existing IDs, only renumber on collision, mark new entries clearly (e.g. `T17 [new]`, `SR-014 [new]`), and run a Q4 model/reality conformance check first per `references/validation.md`. **Branch on the existing structure** — most existing models won't use this skill's three-stratum hybrid layout, and silently imposing it is rude:
  - **If the existing document uses §1 / §2 / §3 / §4 (Four Question Framework) but not the three-stratum §2.1 / §2.2 / §2.3 split** (most common shape): preserve their §2 structure. Add new threats inline in their existing tables. If the user explicitly asks for the operational or strategic stratum, add `§2.x` subsections rather than restructuring. State this once: "Keeping your existing §2 structure; new threats appended below."
  - **If the existing document doesn't use the Four Question Framework at all** (e.g. a vendor-specific template, a Microsoft TMT export, a Polarion-imported table): match their structure. Don't reorganize. Add new entries in the same shape they used. If their structure makes Q4 hard to find, add a Q4 self-assessment in their style at the end and say so.
  - **If the existing document already uses the three-stratum hybrid**: extend it in place; that's what the defaults are for.
  - **Default to ASK before restructuring.** If you're tempted to convert their model to the three-stratum layout because it would be cleaner, surface that as an explicit offer ("Your model uses §1/§2/§3/§4 without strata; want me to keep that, or convert to the hybrid layout?") rather than rewriting silently. The user's existing structure is usually load-bearing — it ties to a tracker, a regulator template, or a team convention.
  - **ID-prefix conventions stay flexible in update mode.** If the existing model uses `A1, A2, ...` for assets (instead of `AS1, AS2, ...`) or `RR-001` for requirements (instead of `SR-001`), match their prefixes. The convention serves the team, not the skill.

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
- External entities (users, other systems, third parties). Capture for each: actor type from this fixed enum — `system / user / power_user / administrator / engineer / third_party` (used directly in the OWASP TML TM-BOM; if you're unsure, default to `user` for end-users, `engineer` for internal devs, `third_party` for vendor integrations).
- Major processes (services / applications / components that *do* something).
- Data stores (databases, files, queues, caches, configuration). For each, capture the storage type from this fixed enum — `sql / key_value / document / object / graph / time_series` (used directly in the TM-BOM).
- Where trust boundaries sit (network, process, privilege, tenant, physical).
- Sensitive data flowing through it. Tag each class against the TM-BOM `data_sensitivity` enum: `pii / phi / fin / ip / cred / biz / gov / pci / op` (multiple allowed). If one or two data classes dominate the threat picture, name them — they seed the data-centric supplement (`references/data-centric.md`).
- **Scope classification (required for TM-BOM emission, useful for prioritization in any case).** Pick one value per field:
  - `business_criticality`: `minimal / low / moderate / high / maximal` — how much the business depends on this system.
  - `exposure`: `internal / external` — reachable from the public internet, or only from inside an org boundary.
  - `tier`: `mission_critical / business_critical / important / non_critical` — operational impact tier.

  These four (the three above plus `data_sensitivity`) are the OWASP TML schema's required `scope` fields. If the user doesn't volunteer them, infer plausible defaults from the system description and record as numbered assumptions (`ASM#`).

- **Medical-device intake (conditional).** If the system is a medical device, a PACS / RIS / VNA, a DICOM- or HL7-adjacent service, or anything heading for FDA premarket / IEC 81001-5-1 / IEC 62304 / MDR / IVDR review, also collect the structured intake from `references/medical.md` § "Medical-device §1 intake template" — device class, intended use (verbatim), period of expected use, invasiveness, core use case, core technology, user roles. These shape the safety-case framing the threat model has to compose with; skip them for non-medical systems and the standard Round 1 fields above are sufficient.

**Round 1.5 — Technology, environment, and ownership (Q1, before the DFD).** This round is what turns a generic DFD into one that reflects this system in its environment. Skipping it is how trust boundaries get missed.

- **Protocols on each flow and runtimes per process.** Cite the actual protocol per flow ("HTTPS + mTLS", "MQTT over TLS", "BLE GATT") — generic labels like "data" hide threats. Identify each process's runtime/host, the identity provider, the secrets store, and the crypto modules. Per-environment protocol catalog: `references/environments.md`.
- **Environment type per zone** from a fixed taxonomy: cloud, on-prem enterprise, embedded / IoT, OT / ICS, mobile, or hybrid (pick the dominant one and note the bridge). Boundary patterns differ sharply by type — `references/environments.md` is the catalog.
- **Ownership per zone.** Who owns the underlying network, host, account, or device? Common owners: vendor, customer, end-user, cloud provider, mobile-OS vendor, third-party SaaS. Ownership ≠ data custody and ≠ legal accountability — it's "who can change configuration, run the patch cycle, control IAM." Mark each zone owner explicitly in the DFD subgraph label. If an owner isn't named, record an assumption and proceed.
- **Physical and operational context.** Where does the device/host live (controlled facility, public space, user's pocket, attacker's hands)? Tamper protections (secure boot, secure element / TEE)? Debug ports, USB, removable media exposed? Network air-gapped, segmented, DMZ-fronted, or directly internet-exposed? These shape both what trust boundaries exist and what threats cross them.
- **Multi-device shared physical space.** When several devices share one physical space (smart building zone, plant-floor cabinet, vehicle cabin, robotics cell, clinical room), one device's RF / acoustic / power surface can reach another. If this applies, plan a multi-device cyber-physical attack-path pass — see `references/environments.md` and `references/methodologies.md` § "Risk-prioritized cyber-physical attack paths".

**Round 2 — Context that shapes threats.**
- Who would attack this and why? Capture **at least one threat persona** (multiple if the system faces distinct adversaries — a malicious insider and an external attacker are different personas). Each persona needs:
  - A short title (e.g. *External attacker*, *Malicious insider*, *Curious user*, *Nation-state*).
  - `is_person` (boolean — `false` for automated bots, supply-chain compromise, etc.).
  - `skill_level` from this enum: `script_kid / insider / engineer / expert_engineer / oc_sponsored / state_sponsored`.
  - `access_level` from: `anonymous / user / admin`.
  - `malicious_intent` (boolean — `false` for accidental-misuse personas; threats from `human_error` sources still need a non-malicious persona).
  - `applicability_to_org`: `minimal / low / moderate / high / maximal` — how relevant this persona is to *this* organization in particular.

  Even a one-line answer seeds ATT&CK mapping — don't dwell, but don't skip either; the TM-BOM's `threats[].threat_persona` field is required and references back to one of these. Default starter set when the user gives nothing: `external-anonymous` (anonymous + script_kid + malicious + moderate applicability) and `internal-user` (user + insider + non-malicious + moderate applicability).
- Safety implications? If yes, the system needs an STPA hazard analysis (`references/stpa.md`) — the threat row's `Impact` column carries only `C/I/A` flags, so physical-harm severity lives in STPA hazards (`H-#`) cross-referenced from threat rows.
- Regulatory constraints?
- What sector? Determines which sector ISAC and regulator framing apply (full lists in `references/methodologies.md`).
- Existing security controls already assumed in place?

If something's still missing after Round 2, ask one focused round — but err on stating assumptions and proceeding. After Round 2 you should have enough to draft the DFD and pick supplements from §"Picking supplements" below. Domain-specific guidance loads on demand: medical / PACS / DICOM → `references/medical.md`; embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes".

## Producing the threat model

Output a single markdown document. Default to a single document; only split into linked documents if the user asks. Use Mermaid for the DFD. **Copy `assets/threat-model-template.md` and fill it in** rather than rebuilding the skeleton — the template carries the full structure (header metadata, scope-classification table, DFD element catalog, trust-boundary table, threat-personas table, the §2.1 / §2.2 / §2.3 tables, mitigation table, risk register, derived requirements, Q4 checklist, changelog).

**Artifact taxonomy (MITRE Threat Modeling Playbook §2.4.6).** A complete threat-modeling deliverable is composed of *four* artifact types, not one. Pick where each finding lives based on who needs to act on it — don't try to push everything into the markdown:

1. **Process documentation** — what's in scope, what's out, what assumptions are taken as given, how threats were identified, when the model is re-reviewed. The markdown produced by this skill (the `assets/threat-model-template.md` instance) is the process doc.
2. **Ticketing system** — every threat becomes a tracked work item in the team's bug tracker (Jira, Polarion, GitHub Issues, ServiceNow). This is the *exit point* from threat modeling per `references/stride-prompts.md` § "Enumeration tactics" — once threats are bugs, normal triage and engineering machinery takes over. The markdown's threat IDs (`T#`, `V#`, `PR#`, `SR-###`) are the cross-reference back from the tracker.
3. **Diagrams** — DFDs (`references/dfd-mermaid.md`), sequence / swim-lane / state diagrams (`references/non-dfd-models.md`), attack trees (`references/methodologies.md`). Embed inline in the markdown when small; link out to a dedicated diagram source when large or maintained separately (e.g. a Threat Dragon `.tdt` file or pytm script in the repo).
4. **Threat table / register** — the §2 / §3 tables. Default home is inside the markdown; for large systems with hundreds of threats, link a spreadsheet or database export (the TM-BOM JSON described in § "Producing the TM-BOM" below is the structured form of this register).

The placement choice is consequential: a threat that lives only in the markdown but never reaches the tracker won't be triaged; a threat that lives only in the tracker but isn't reflected in the markdown won't survive the next model revision. Cross-reference between the artifacts explicitly — ID conventions below exist precisely to make these cross-references unambiguous.

The four top-level sections are fixed (Q1 / Q2 / Q3 / Q4). §2 can carry up to three subsections — §2.1 Contextual / §2.2 Operational / §2.3 Strategic; §2.1 is always present, and §2.2 / §2.3 are added only when the system gives you content for them.

**Default scaffold rule.** Add content to a §2 subsection when you have something to say there. Omit a subsection entirely when you don't. No "not applicable" stubs. The three-stratum layout is *available scaffolding*, not a coverage quota — it comes from Tatam et al. (2021), one paper, not a practitioner consensus. Background and the system-type matrix that suggests which supplements to add: `references/methodologies.md` § "The hybrid layout".

**Threats span strata, mitigations don't.** Threats live in §2.1 / §2.2 / §2.3 with one shared ID space and one risk scale. §3 produces a single prioritized mitigation table over all of them — never duplicate a threat across strata, cross-reference instead.

ID conventions to keep cross-stratum references unambiguous (one prefix per namespace, chosen so single-letter prefixes can't be misread as truncations of multi-letter ones):

- Assets in §1: `AS1`, `AS2`, …
- Assumptions in §1: `ASM1`, `ASM2`, … (avoid bare `A1` — it's visually confusable with `AS1` and breaks grep). The §1 Assumptions block is the model's **assumptions register** — the MITRE Threat Modeling Playbook §2.4.6 deliverable that survives across iterations. Place it early in §1 (immediately after Scope, before the asset and trust-level tables) so a reviewer scanning §1 sees what's been taken as given before they see what's been claimed as fact. Every `ASM#` referenced in §2 / §3 prose must resolve to this register; new or revised assumptions across revisions are reflected in the §4 changelog.
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

For each DFD element, walk the STRIDE categories that apply. Threats follow data flow, not control flow, and cluster around trust boundaries *and* complex parsers (deserializers, image decoders, structured-document parsers). Write each threat cell as `**Title**: description.` — a ≤6-word noun-phrase title (e.g. *Session token replay*; not the bare STRIDE category, which is already in the STRIDE column), then 1–2 sentences naming who does what, against which element, with what effect. No paragraphs, no unstructured sentences with no title. Same format for §2.1.b's `Threat / Vector` and §2.1.c's `Threat`. Per-element seeds: `references/stride-prompts.md` § "Per-element example threats".

Threat table format:

| ID | Element | STRIDE | Threat | CAPEC | CWE | Persona | Event | Source | AV | PR | AC | Impact |
|----|---------|--------|--------|-------|-----|---------|-------|--------|----|----|----|--------|
| T1 | Auth Service (P1) | S | **Session token replay**: An attacker replays a captured session token across the trust boundary to take over an authenticated session. | CAPEC-60 (Reusing Session IDs) | CWE-294 | external-anonymous | session takeover | adversary | N | N | L | C, I |

Per-row schema (every column required; populates the TM-BOM):
- **CAPEC + CWE** — every row carries an attack-pattern ID and the weakness it exploits. The CAPEC → CWE link is 1:1 (each CAPEC pattern declares its CWEs); pick the CAPEC, the CWE follows. Pick CAPEC abstraction by SDLC stage (Meta = early architecture, Standard = design review, Detailed = component-level — `references/capec.md`). When no Detailed pattern exists for a domain-specific protocol (DICOM, HL7, ICS), cite the closest Standard or Meta and note `(closest pattern; no Detailed available)`.
- **Persona** — symbolic name from the Round 2 personas table (`threats[].threat_persona`).
- **Event** — ≤6-word verb phrase for *what happens* (`threats[].event`). Example: *"session takeover"*, *"PHI exfiltration"*, *"unauthenticated firmware flash"*.
- **Source** — fixed enum: `adversary / human_error / failure / events_beyond_org_control` (`threats[].sources`). Most threats are `adversary`; STPA non-adversarial scenarios are `human_error` or `failure`.
- **AV / PR / AC** — CVSS 3.1 intrinsic exposure: Attack Vector `N/A/L/P` (Network/Adjacent/Local/Physical), Privileges Required `N/L/H` (None/Low/High), Attack Complexity `L/H` (Low/High; `H` only when conditions are beyond the attacker's control — race window, specific network position, victim interaction beyond a click). Definitions and the §3 sort heuristic: `references/risk-rating.md`.
- **Impact** — comma-separated subset of `C` / `I` / `A` (Confidentiality / Integrity / Availability). At least one — a row with no impact isn't a threat. Safety hazards (physical-harm impact) carry severity in the STPA section (`references/stpa.md`); cross-reference a hazard ID (`H-#`) in the `Element` cell when a threat triggers a hazard.

§3 sorts threats most-exposed first (AV:N > A > L > P; PR:N > L > H; AC:L > H), then by Impact-set size — no standalone "Risk" column. Avoid DREAD. The OWASP TML schema's required `risks[].likelihood / impact / level / score` fields are derived from these row values at TM-BOM emission time per the translation in `references/risk-rating.md` § "Translation to TM-BOM enums".

Iteration tactics: walk across trust boundaries first (highest-value threats live there); start with whatever crosses the outermost boundary; don't skip a threat because it's not the category you're walking right now (write it down — redundancy is fine); every recorded threat ends up as a tracked work item in the team's bug tracker. Full tactics catalog: `references/stride-prompts.md` § "Enumeration tactics".

For the highest-risk handful of threats (top 3–5), consider drawing a threat tree showing how the threat could be realized. One Mermaid tree per top threat is plenty.

## Mitigations and Q4

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a **layered stack of concrete controls** — typically 2–6 per threat, scaled to stakes; a single-control row on a threat that crosses a trust boundary with `C,I,A` impact is a finding (`references/manifesto.md` § "Failure-mode catalog"). **Order the stack Protect first** (prevent the threat), then Detect / Respond / Recover as compensating layers that catch the Protect failure — not four equal partners. Map each control to the violated property — STRIDE → security property → typical mitigations table in `references/stride-prompts.md`. **Prefer mitigations built into the in-scope technology** (platform IAM, KMS / HSM, managed mTLS, K8s NetworkPolicy, browser CSP / SOP / SameSite, framework CSRF tokens, ORM parameterization, OS sandboxing, secure boot / TPM / TEE, memory-safe languages) over custom code — the built-in already carries the vendor's threat model. **When a control is cryptographic, name the suite, not the class** (TLS 1.3 / mTLS, AES-256-GCM or ChaCha20-Poly1305, Ed25519 / ECDSA P-256+, HKDF, Argon2id; FIPS 140-3 module when regulated) — *"use encryption"* is the "Generic mitigations" anti-pattern. Each Mitigate row **may** carry a NIST SP 800-53 r5 control-family tag (AC / AU / IA / SC / SI / CM / CP / IR / SR) alongside its CSF function for traceability to compliance baselines. Full guidance: `references/stride-prompts.md` § "Choosing the right primitive" and § "NIST SP 800-53 r5 control-family linkage".

**Avoid silent transference and silent acceptance.** The single most-common anti-pattern in real threat models — and the central concern of the MITRE Threat Modeling Playbook §2.5 — is closing a threat by handing the residual risk to someone else (the deployer, the operator, the customer, the HDO) without naming the someone, the threats covered, or the controls they're now responsible for. *"Install on a secure network"* and *"assumes a hardened environment"* are silent transfers. Every `Transfer` row names the receiving party, the threats transferred (`T#` IDs), and the concrete control the receiver must implement; every `Accept` row records `{rationale, decision_maker, decided_at}`. Defensive assumptions about another party's environment go in the §1 assumptions register as `ASM#` with explicit residual-risk language, not in the mitigation column. See `references/manifesto.md` § "Anti-patterns" → "Silent risk transference / silent risk acceptance".

**§2.2 operational stratum** — when produced, layers ATT&CK technique IDs, kill-chain phase, CVE / CVSS, and detection / handoff notes onto threats already enumerated in §2.1. CAPEC and CWE come from the upstream §2.1 row; don't duplicate them here. STRIDE → CAPEC mapping and the abstraction-level rule of thumb: `references/capec.md`.

Close the loop on the defensive side with **MITRE D3FEND**: when a threat row carries an ATT&CK technique ID, the corresponding mitigation row carries the matching D3FEND defensive technique ID (`capec.md` § "Closing the loop with MITRE D3FEND on the defensive side"). The full chain becomes STRIDE → CAPEC → CWE → ATT&CK → D3FEND → SR-###. Skip D3FEND for minimum-viable models with no §2.2 — its value comes from the ATT&CK mapping.

**Tag every Mitigate row with its NIST CSF function** — Protect / Detect / Respond / Recover. The STRIDE → property → mitigation table answers *what kind of control*; the CSF function answers *what role it plays in defense in depth*. A §3 where every control is `Protect` with no `Detect` / `Respond` / `Recover` is itself a finding — log the gap as a derived requirement. Two axes, both populated: see `references/stride-prompts.md` § "NIST CSF function — Protect / Detect / Respond / Recover".

Derive security requirements from mitigations. Each one is testable, stable enough to import into a tracker, and **names how it's verified** — a test ID, an integration suite, an audit step, a manual checklist, or a runbook drill. An SR with no verification activity is an unverified claim; QE / regulator-facing reviewers will flag it:

```
SR-001: The system SHALL authenticate all service-to-service API calls using mutual TLS with X.509 certificates issued by <CA>.
   Mitigates: T3, T7
   Verification: integration test `auth/mTLS_handshake_test.py::test_invalid_cert_rejected`; quarterly SOC drill (runbook RB-04)
```

**Q4 — minimum viable.** Three checks (full Shostack-style checklists in `references/validation.md`):

1. **Model/reality conformance** — does the diagram match what was actually built or planned?
2. **Every threat has a response** — Mitigate / Eliminate / Transfer / Accept, each tracked in the team's bug system.
3. **Every section that's present is populated** — no empty subsections, no stratum stubs.

Plus: open questions / assumptions still to validate, and a **next review trigger** — pick at least one concrete trigger from this candidate list (a bare "annually" alone is weak):

- **Architecture / design change** above a threshold (new external integration, new data class, new trust boundary, change of identity provider).
- **New dependency** above a criticality threshold (new SDK, vendor, SaaS).
- **Security incident** in this system or in the sector (CVE in a load-bearing dependency; sector ISAC advisory matching this system's profile).
- **Time-elapsed** (no shorter than 12 months for low-stakes systems; quarterly for safety-critical or regulated).
- **Regulatory update** affecting the framing (new FDA guidance, new IEC revision, new GDPR opinion).

Record the chosen trigger(s) in §4 *and* in the template header so the TM is bound to a re-review event, not a date guess. Q4 is the most-skipped step in real threat models. Don't skip it.

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
| Sequence / swim-lane diagram | Ordered multi-actor workflow, two-person rule, service-mode access, replay/reorder threats are plausible | `references/non-dfd-models.md` § Sequence diagrams |
| State diagram | Device or service has named operational modes with documented transitions; auth/interlock checks are gated on state, not on requests | `references/non-dfd-models.md` § State diagrams |
| AI/ML threat list | ML components in scope (prompt injection, training-data poisoning, model extraction) | `references/methodologies.md` § ML/AI; OWASP LLM Top 10 |
| PASTA framing | Borrow the business-impact stage when executive sign-off is needed | `references/methodologies.md` § PASTA |
| STPA | Load whenever the system has **safety impact** — any plausible worst-case outcome involves physical harm to a person, equipment, or the environment (medical devices, ICS, automotive, aerospace, robotics, consumer IoT that controls something dangerous). Swap or supplement mode; default to supplement. | `references/stpa.md` |
| OCTAVE / VAST | Org/portfolio-level only, not per-system | `references/methodologies.md` |

Domain pointers: medical / PACS / DICOM / IoMT → also load `references/medical.md`. Embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes" has each domain's specifics.

## References

Read on demand:

- `methodologies.md` — Hybrid framing, system-type decision matrix, layer catalog, single-doc vs linked-doc shapes, pruning rules. When to use LINDDUN / PASTA / attack trees / ATT&CK / kill chain / OCTAVE / VAST. Cyber-physical attack-path construction. Sector ISAC and regulator lists.
- `environments.md` — Per-environment trust-boundary patterns (cloud, on-prem enterprise, embedded / IoT, OT / ICS, mobile) + ownership taxonomy + domain notes (embedded specifics, cloud-native, AI/ML, third-party code). Read when drawing the DFD.
- `medical.md` — Default layer mix for medical / PACS / DICOM / IoMT, patient-as-asset framing, clinical workflow misuse, regulatory list (FDA / IEC 62304 / IEC 81001-5-1 / HIPAA / H-ISAC), DICOM/HL7 STRIDE specifics, the Four Question ↔ TIR57 joint risk-management workflow mapping. Load only when the system is medical.
- `centric-methods.md` — Entry-point taxonomy (asset, data, flow, process, user-needs, attacker, code) and the generation-vs-characterization distinction.
- `data-centric.md` — NIST SP 800-154 workflow.
- `capec.md` — Operational-stratum reference: STRIDE → CAPEC → CWE → ATT&CK → D3FEND → mitigation chain, three abstraction levels (Meta / Standard / Detailed), CAPEC-1000 and CAPEC-3000 views, coverage gaps.
- `stride-prompts.md` — STRIDE category prompts, per-element example threats, Per-Element vs Per-Interaction, DESIST variant, enumeration tactics, NIST CSF Protect/Detect/Respond/Recover mitigation-function axis.
- `dfd-mermaid.md` — DFD-to-Mermaid mapping with worked examples (Level 0 + Level 1), dashed-subgraph pattern, subgraph labeling convention, nested-subgraph pattern, diagramming checklist, the six high-value-dataflow categories that pull a Level-2 sequence/state view (auth, programming, updates, PHI egress, alarms, backup restore). Also documents the skill's deliberate one-way-arrow deviation from DFD3 and the MITRE Playbook "complete shape" rule for trust boundaries.
- `non-dfd-models.md` — When the DFD isn't enough: swim-lane / sequence diagrams (Mermaid `sequenceDiagram`) for ordered multi-actor workflows, and state diagrams (Mermaid `stateDiagram-v2`) for devices with named operational modes. Both are *supplements* to the DFD, not replacements. Load whenever the system has clinical workflow, two-person rules, service-mode access, or a documented state machine — typical for medical devices, OT/ICS, and multi-step approval flows.
- `validation.md` — Q4 deep dive: model/reality conformance, the three Shostack-style checklists, pen testing as complement, validating threats in third-party code.
- `manifesto.md` — Threat Modeling Manifesto values, principles, patterns, anti-patterns (paraphrased), including the silent risk transference / silent risk acceptance anti-pattern and the artifact-level failure-mode catalog.
- `risk-rating.md` — AV / PR / AC + CIA scheme (CVSS 3.1 intrinsic enums), §3 sort heuristic, translation to OWASP TML's `risks[].likelihood/impact/level/score` enums, MITRE CVSS Rubric for Medical Devices pointer, ISO 14971 / TIR57 mapping (driven from STPA hazards + AV/PR/AC, not a row-level safety bump), Stellios cyber-physical attack-path scoring, why DREAD is discouraged.
- `stpa.md` — Safety + security joint hazard analysis. Umbrella term covering Leveson's STPA, Young & Leveson's STPA-Sec, and Friedberg et al.'s STPA-SafeSec; the workflow uses the integrated form with the component-layer mapping. Two modes (swap or supplement; default to supplement). Load whenever the system has safety impact — physical harm to people, equipment, or the environment is the trigger, not control-loop shape or regulator presence.
- `examples.md` — Pointer to the OWASP Threat Model Library (https://github.com/OWASP/www-project-threat-model-library) as the source for end-to-end worked examples by system type (web-applications, ai-ml-systems, infrastructure, third-party-integrations). The library's JSON schema also defines the TM-BOM contract — see § "Producing the TM-BOM" above. The medical / DICOM end-to-end example is the exception, kept inline at `data-centric.md` § "Worked example" + `dfd-mermaid.md` § "Worked example: small clinical PACS".
- `tm-bom.md` — TM-BOM emission procedure: OWASP TML field-by-field schema mapping, symbolic-name derivation, the Mitigate/Eliminate/Transfer/Accept → `control.status` mapping, and the pre-ship validation checklist. Load only when emitting a TM-BOM (opt-in — see § "Producing the TM-BOM").

Blank template: `assets/threat-model-template.md`.

## Producing the TM-BOM (machine-readable artifact)

The TM-BOM (Threat Model Bill of Materials) is a structured JSON companion to the markdown — the system, threats, controls, and risks in machine-readable form, emitted against the **OWASP Threat Model Library** JSON schema (`threat-model.schema.json` v1.0.2, MIT-licensed).

**Emission is opt-in.** The markdown is the always-shipped, canonical artifact for humans. Emit the TM-BOM only when the user asks for tracker import (Polarion, Jira, GitHub Issues, ServiceNow), tooling-pipeline integration, or upstream contribution to the OWASP TML library. Never emit it unprompted, and never emit only the JSON. When you do emit one, validate it against the OWASP TML schema before handing it over.

Full procedure — field-by-field schema mapping, symbolic-name derivation, the Mitigate/Eliminate/Transfer/Accept → `control.status` mapping, the top-level `extensions` pattern, and the pre-ship validation checklist (with the `check-jsonschema` command): `references/tm-bom.md`. Starter file: `assets/tm-bom-example.json`.

## Citations

Concepts in this skill paraphrase from:

- **OWASP** — Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, Security Culture v1.0 §6, **Threat Model Library** (https://github.com/OWASP/www-project-threat-model-library, MIT-licensed; TM-BOM JSON schema and the curated example-models repository the skill defers to for worked examples).
- **MITRE Threat Modeling Playbook** (Toreon, OWASP) — §2.3.1.1 DFD3 conformance items (trust boundary as a complete shape; data-flow arrow convention noted as a deliberate deviation in `references/dfd-mermaid.md`); §2.3.2 sequence / swim-lane diagrams; §2.3.3 state diagrams and high-value dataflow categories that pull a Level-2 view (`references/dfd-mermaid.md` § "High-value dataflows that should always pull a Level-2 view"); §2.4.6 Assumptions register as a first-class deliverable; §2.5 the silent risk transference / silent risk acceptance anti-pattern (central organizing concern of "What are we going to do about it?"; see `references/manifesto.md`); §2.5.2 NIST CSF Protect/Detect/Respond/Recover function mapping over mitigations and MITRE D3FEND as the defensive counterpart to ATT&CK; §2.5.6.2 MITRE CVSS Rubric for Medical Devices.
- **Threat Modeling Manifesto** — values, principles, patterns, anti-patterns (CC-BY 4.0).
- **Adam Shostack**, *Threat Modeling: Designing for Security* (Wiley, 2014) — Four Question Framework, STRIDE-Per-Element, diagramming and validation checklists, iteration tactics, bug-filing as exit point.
- **NIST SP 800-154** (Draft, Souppaya & Scarfone, 2016) — data-centric methodology.
- **Tatam, Shanmugam, Azam & Kannoorpatti**, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021, CC-BY 4.0) — three-stratum hybrid framing; CAPEC → CWE design-time payoff.
- **MITRE** — ATT&CK, CAPEC, CWE, **D3FEND** (https://d3fend.mitre.org/ — defensive-technique knowledge graph mapped to ATT&CK techniques; closes the offensive→defensive loop in the operational stratum), **CVSS Rubric for Medical Devices** (github.com/mitre/md-cvss-rubric-tools — branching decision-tree *methods* for CVSS scoring in clinical context; apply against current CVSS 3.1 / 4.0 calculators on NVD or first.org; see `references/risk-rating.md` § "Clinical-context CVSS scoring").
- **NIST Cybersecurity Framework v2.0** — Protect / Detect / Respond / Recover function taxonomy applied as the second axis on §3 mitigation rows alongside STRIDE → property → control (https://www.nist.gov/cyberframework).
- **Lockheed Martin** — Cyber Kill Chain.
- **STRIDE** — Loren Kohnfelder & Praerit Garg (Microsoft). **DESIST** — Gunnar Peterson.
- **STPA** (umbrella term in this skill; lineage: Leveson, *Engineering a Safer World*, MIT Press, 2011 → STPA-Sec, Young & Leveson, 2014 → STPA-SafeSec, Friedberg et al., 2017, CC-BY — the workflow draws from across the lineage; cite the specific paper when the distinction matters).
- **Risk-prioritized cyber-physical attack paths** — Stellios, Kotzanikolaou & Grigoriadis (2021).
- **ISO 14971:2019** — *Medical devices — Application of risk management to medical devices* (severity-of-harm × probability-of-occurrence-of-harm decomposition; the safety-risk language regulators expect cybersecurity risk to be expressed in for medical-device submissions).
- **AAMI TIR57:2023** — *Principles for medical device security — Risk management* (the `P1 × P2` cyber-to-safety bridge into ISO 14971; mapping table in `references/risk-rating.md` § "ISO 14971 / AAMI TIR57 mapping for medical-device submissions").
- **US FDA** — *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023 final guidance; sets the joint cyber/safety risk-model expectation for premarket review).

Cite these in any output that quotes or substantively summarizes them.
