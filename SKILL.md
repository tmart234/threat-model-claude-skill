---
name: threat-modeler
description: 'Use when the user asks the assistant to *produce* a threat model, security architecture review, attack-surface analysis, or "what can go wrong with this system" — typically against a paste of an architecture, design, spec, code, or component list — or when they name a methodology (STRIDE, LINDDUN, PASTA, STPA) and ask for design-time analysis of *their* system. Also use for regulatory threat-model deliverables (FDA premarket cybersecurity, IEC 81001-5-1, IEC 62443, ISO 26262 cyber). The trigger is the request to *produce an artifact*, not the mention of a term. Do **not** use for educational queries, definitions, methodology comparisons not tied to a specific system, coursework, pen-test planning, vulnerability scanning, incident response, SBOM generation, threat intel, CVE triage, or passing references to STRIDE / DFD / "attack surface" in unrelated discussions. When in doubt: is the user asking me to produce an artifact about a system they''ve described? If yes, use the skill; if no, answer directly.'
---

# Threat Modeler

Produce a working threat model for an engineering team. State assumptions and proceed rather than interrogating. Ship a useful approximation rather than polishing a perfect artifact. When STRIDE categorization is debatable, record the threat and move on. Before shipping, scan the draft against `references/manifesto.md` § "Failure-mode catalog" (padded asset lists, single-threat-per-element, generic mitigations like "apply RBAC", threat cells that aren't a title-plus-description, mitigation table that doesn't match the threat table) — a model with those shapes is worse than no model because it implies threats were considered when they weren't. Manifesto values, principles, and anti-patterns: `references/manifesto.md`.

**Minimum viable threat model.** For low-stakes systems (internal tool, no sector adversary, no regulator, no SOC handoff), the complete deliverable is: §1 with a DFD + a few assumptions, §2.1 STRIDE-Per-Element using the **eight-column minimum-viable threat table** (§ "Threat enumeration"), §3 with one response per threat, and the three-check Q4. That's a finished threat model — not a single-method deficient one. The three-stratum layout, TM-BOM JSON emission, and the five escalation columns (`CAPEC / CWE / Persona / Event / Source`) are escalations the *system* pulls in (regulated, multi-tenant, safety-critical, sector-targeted), not defaults the skill imposes. Match output to stakes; the skill is a menu, not a coverage quota.

## Pick a mode

- **Guided interview** — user said "help me threat model X" without a system description. Run §"Guided interview" to scope, then produce the model.
- **Fast-path** — user pasted architecture, a DFD, a component list, a spec, code, or a description detailed enough to identify external entities, processes, data stores, and trust boundaries. Skip the interview, but **before drawing the DFD** pin down the four things from Round 1.5 (technology stack, environment type, environment ownership, physical/operational context). If any aren't in the artifact, state your inferred answer as a numbered assumption ("ASM3: assumed customer IT owns the on-prem network; correct if otherwise") and proceed.
- **Update / extend** — user pasted an existing threat model. Treat their document as input: preserve existing IDs, only renumber on collision, mark new entries clearly (`T17 [new]`, `SR-014 [new]`), and run a Q4 model/reality conformance check first (`references/validation.md`). **Branch on the existing structure; never silently restructure** — the user's existing layout is usually load-bearing (it ties to a tracker, a regulator template, or a team convention):
  - If it uses the Four Question Framework but not the three-stratum §2.1 / §2.2 / §2.3 split (most common): preserve their §2 structure, add new threats inline in their tables. Add `§2.x` subsections only if the user explicitly asks for the operational/strategic stratum. State once: "Keeping your existing §2 structure; new threats appended below."
  - If it doesn't use the Four Question Framework at all (vendor template, Microsoft TMT export, Polarion table): match their structure, add entries in the same shape. If Q4 is hard to find, append a Q4 self-assessment in their style and say so.
  - If it already uses the three-stratum hybrid: extend it in place.
  - Match their ID-prefix conventions (`A1` for assets, `RR-001` for requirements) — the convention serves the team. If converting to the hybrid layout would be cleaner, *offer* it explicitly ("want me to keep §1/§2/§3/§4, or convert to the hybrid layout?"); don't impose it.

## Handling sensitive input

Threat modeling invites users to paste architecture, identifiers, and sometimes credentials-adjacent material. Surface this guidance once at the start of the session — not as a gate, but so the user can decide what to share:

- **Redact before pasting** production credentials, API keys, customer PII / PHI, real patient identifiers, internal hostnames / IPs, signing-key fingerprints, internal-only URLs. The model doesn't need any of these — placeholders (`prod-db-1`, `<internal-host>`, `customer-X`) work just as well.
- **Suspicious-string check.** If the pasted artifact contains anything resembling a JWT, base64 secret, AWS key (`AKIA…`), GitHub token (`ghp_…`), a connection string with embedded credentials, `BEGIN PRIVATE KEY`, or a real customer name — **stop, flag it, and ask whether to redact or pause**. Don't silently echo the substring into output.
- **The produced threat model is itself sensitive** — it maps trust boundaries and unmitigated gaps. Default the template's `Status` to `Draft / Internal`, state the sensitivity in the header, and recommend storing it under the same access controls as the design docs it describes.
- **If the user names a third party**, don't speculate about that party's vulnerabilities beyond public advisories — stick to the system the user is responsible for.

This applies across every mode. State it once, then proceed; don't re-prompt unless new pasted material reintroduces the concern.

## The Four Question Framework

Every threat model answers four questions, in order (Shostack; adopted by the Threat Modeling Manifesto):

1. **What are we working on?** → System decomposition + DFD with trust boundaries.
2. **What can go wrong?** → Threat enumeration.
3. **What are we going to do about it?** → A response (Mitigate / Eliminate / Transfer / Accept) for each threat, with concrete controls for "Mitigate" and derived security requirements.
4. **Did we do a good enough job?** → A self-assessment plus open questions.

Treat the output of Q2 as **hypothesized attack scenarios, not confirmed vulnerabilities** — they need validation by code review, testing, or red-teaming. Don't skip Q4.

## Guided interview

Cluster questions; don't fire one at a time. Two or three turns of clustered questions is the target.

**Round 1 — Scope and system understanding (Q1).** Capture: a one-paragraph system description; external entities (users, other systems, third parties); major processes; data stores; where trust boundaries sit (network, process, privilege, tenant, physical); and the sensitive data flowing through. If one or two data classes dominate the threat picture, name them — they seed the data-centric supplement (`references/data-centric.md`).

The §1 intake fields in `assets/threat-model-template.md` are the structured form of this — including the actor-type, storage-type, `data_sensitivity`, and scope-classification (`business_criticality` / `exposure` / `tier`) enums that the OWASP TML TM-BOM requires and that are useful for prioritization in any case. Fill them from the description; where the user doesn't volunteer a value, infer a plausible default and record it as a numbered assumption (`ASM#`). Enum definitions: `references/tm-bom.md` § "Field-by-field mapping".

**Medical-device intake (conditional).** For a medical device, PACS / RIS / VNA, DICOM- or HL7-adjacent service, or anything heading for FDA premarket / IEC 81001-5-1 / IEC 62304 / MDR / IVDR review, also collect the structured intake from `references/medical.md` § "Medical-device §1 intake template". Skip for non-medical systems.

**Round 1.5 — Technology, environment, and ownership (Q1, before the DFD).** This round turns a generic DFD into one that reflects this system in its environment; skipping it is how trust boundaries get missed. Fill the template's "Technology stack and environment" section — four things, none optional:

- **Protocols and runtimes.** Cite the actual protocol per flow ("HTTPS + mTLS", "MQTT over TLS", "BLE GATT") — generic labels like "data" hide threats. Note each process's runtime/host, identity provider, and secrets store.
- **Environment type per zone** — cloud / on-prem enterprise / embedded-IoT / OT-ICS / mobile / hybrid. Boundary patterns differ sharply by type.
- **Ownership per zone** — who can change configuration, run the patch cycle, control IAM (vendor, customer, end-user, cloud provider, mobile-OS vendor, SaaS). Ownership ≠ data custody ≠ legal accountability. Mark it in each DFD subgraph label; if an owner isn't named, record an assumption and proceed.
- **Physical and operational context** — where the host lives, tamper protections (secure boot, SE / TEE), exposed debug / USB / removable-media ports, network exposure (air-gapped / segmented / DMZ / internet-facing).

Per-environment boundary and protocol catalog: `references/environments.md`. When several devices share one physical space (clinical room, plant-floor cabinet, vehicle cabin, robotics cell), one device's RF / acoustic / power surface can reach another — plan a cyber-physical attack-path pass (`references/methodologies.md` § "Risk-prioritized cyber-physical attack paths").

**Round 2 — Context that shapes threats.** Cover:

- **Threat personas** — who would attack this and why. Capture at least one (more if the system faces distinct adversaries — a malicious insider and an external attacker are different personas). The §1 personas table in the template lists the per-persona fields and their enums (`skill_level`, `access_level`, `applicability_to_org`, `is_person`, `malicious_intent`); a one-line answer per persona is enough. When the user gives nothing, default to the starter set `external-anonymous` + `internal-user` (both defined in the template).
- **Safety implications** — if any plausible worst case is physical harm, the system needs an STPA hazard analysis (`references/stpa.md`); physical-harm severity lives in STPA hazards (`H-#`) cross-referenced from threat rows, not in the row's `C/I/A` flags.
- **Regulatory constraints and sector** — the sector determines which ISAC and regulator framing apply (`references/methodologies.md`).
- **Existing security controls** already assumed in place.

If something's still missing after Round 2, ask one focused round — but err on stating assumptions and proceeding. After Round 2 you should have enough to draft the DFD and pick supplements from §"Picking supplements" below. Domain-specific guidance loads on demand: medical / PACS / DICOM → `references/medical.md`; embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes".

## Producing the threat model

Output a single markdown document. Default to a single document; only split into linked documents if the user asks. Use Mermaid for the DFD. **Copy `assets/threat-model-template.md` and fill it in** rather than rebuilding the skeleton — the template carries the full structure (header metadata, scope-classification table, DFD element catalog, trust-boundary table, threat-personas table, the §2.1 / §2.2 / §2.3 tables, mitigation table, risk register, derived requirements, Q4 checklist, changelog).

**Artifact taxonomy (MITRE Threat Modeling Playbook §2.4.6).** A complete deliverable is *four* artifact types, not one — place each finding where the people who act on it will see it:

1. **Process documentation** — the markdown this skill produces (scope, assumptions, how threats were found, re-review trigger).
2. **Ticketing system** — every threat becomes a tracked work item (Jira, Polarion, GitHub Issues, ServiceNow). This is the *exit point* from threat modeling; the markdown's `T#` / `V#` / `PR#` / `SR-###` IDs cross-reference back.
3. **Diagrams** — DFDs, sequence / state diagrams, attack trees; embed inline when small, link out when large or separately maintained.
4. **Threat table / register** — the §2 / §3 tables (the TM-BOM JSON is its structured form); link a spreadsheet / DB export for systems with hundreds of threats.

A threat that lives only in the markdown never gets triaged; one that lives only in the tracker won't survive the next model revision. Cross-reference explicitly — that's what the ID conventions below are for.

The four top-level sections are fixed (Q1 / Q2 / Q3 / Q4). §2 can carry up to three subsections — §2.1 Contextual / §2.2 Operational / §2.3 Strategic; §2.1 is always present, and §2.2 / §2.3 are added only when the system gives you content for them.

**Default scaffold rule.** Add content to a §2 subsection when you have something to say there. Omit a subsection entirely when you don't. No "not applicable" stubs. The three-stratum layout is *available scaffolding*, not a coverage quota — it comes from Tatam et al. (2021), one paper, not a practitioner consensus. Background and the system-type matrix that suggests which supplements to add: `references/methodologies.md` § "The hybrid layout".

**Threats span strata, mitigations don't.** Threats live in §2.1 / §2.2 / §2.3 with one shared ID space and one risk scale. §3 produces a single prioritized mitigation table over all of them — never duplicate a threat across strata, cross-reference instead.

ID conventions to keep cross-stratum references unambiguous (one prefix per namespace, chosen so single-letter prefixes can't be misread as truncations of multi-letter ones):

- Assets in §1: `AS1`, `AS2`, …
- Assumptions in §1: `ASM1`, `ASM2`, … (avoid bare `A1` — confusable with `AS1`, breaks grep). The §1 Assumptions block is the model's **assumptions register** (MITRE Playbook §2.4.6 deliverable) — place it early in §1, after Scope and before the asset/trust-level tables. Every `ASM#` in §2 / §3 prose must resolve here; revisions are reflected in the §4 changelog. The template carries the full register guidance.
- Flow-centric STRIDE threats: `T1`, `T2`, …
- Supplementary entry-point findings: `V1`, `V2`, … (data-centric / asset-centric / user-needs-centric / process-centric / code-centric — use the `V` prefix regardless of pass type, with the pass type recorded in the row; avoid `A#` for asset-centric findings since it collides with assets)
- Privacy / LINDDUN / AI-ML findings: `PR1`, `PR2`, …
- Mitigations / derived requirements: `SR-001`, `SR-002`, …
- ATT&CK / CAPEC / CWE / CVE: cite the upstream ID directly

## DFD

Use Mermaid `flowchart LR` (or `TD`). Map elements as: external entity → rectangle; process → rounded rectangle / circle; data store → cylinder; data flow → labeled arrow; trust boundary → `subgraph`. Render every trust-boundary subgraph with a dashed border (the `classDef tb` pattern — required on every diagram, not optional) and label it with owner, environment type, and trust level. Full conventions, the exact `classDef`/`class` lines, the per-subgraph `style` alternative for single-zone diagrams, the labeling format, and worked examples: `references/dfd-mermaid.md`. The per-environment patterns that should drive *which* subgraphs you draw: `references/environments.md`.

Three rules, applied in order:

1. **Every element belongs to exactly one zone subgraph.** A floating element with no zone is a missing trust boundary. If an entity is genuinely outside any controlled zone (an attacker on the public internet), put it in an explicit `Untrusted` subgraph.
2. **Nest subgraphs for trust-boundaries-within-trust-boundaries.** Cloud account / VPC / subnet, Windows host / VM / container, embedded device / secure element, mobile device / app sandbox / hardware keystore. Pattern: `references/dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries".
3. **Reconcile the DFD against §1 before threat enumeration.** Every asset and trust boundary mentioned in §1 prose must appear in the DFD. If §1 names something the DFD doesn't show, fix the DFD — don't quietly drop the element.

A single diagram with 5–15 elements is usually right; for larger systems produce a Level-0 (context) diagram and one or more Level-1 (decomposed) diagrams rather than one unreadable mess.

## Threat enumeration

The contextual core is **STRIDE-Per-Element on the DFD**. Use STRIDE generatively as a per-element prompt — don't argue about which cell a finding belongs in. Read `references/stride-prompts.md` before enumerating: it has the applicability table (which categories apply to External entity / Process / Data flow / Data store), the category prompts, the element-is-the-victim framing, and the Per-Element vs Per-Interaction choice.

For each DFD element, walk the STRIDE categories that apply. Threats follow data flow, not control flow, and cluster around trust boundaries *and* complex parsers (deserializers, image decoders, structured-document parsers). Write each threat cell as `**Title**: description.` — a ≤6-word noun-phrase title (e.g. *Session token replay*; not the bare STRIDE category, which is already in the STRIDE column), then 1–2 sentences naming who does what, against which element, with what effect. No paragraphs, no unstructured sentences with no title. Same format for §2.1.b's `Threat / Vector` and §2.1.c's `Threat`. Per-element seeds: `references/stride-prompts.md` § "Per-element example threats".

**Two table shapes — pick by stakes.** The threat table has a minimum-viable shape and an escalation shape. Use one shape consistently across §2.1.a / §2.1.b / §2.1.c — don't add columns the system doesn't need, don't drop columns it does.

**Minimum-viable table (default for low-stakes systems):**

| ID | Element | STRIDE | Threat | AV | PR | AC | Impact |
|----|---------|--------|--------|----|----|----|--------|
| T1 | Auth Service (P1) | S | **Session token replay**: An attacker replays a captured session token across the trust boundary to take over an authenticated session. | N | N | L | C, I |

- **AV / PR / AC** — CVSS 3.1 intrinsic exposure: Attack Vector `N/A/L/P` (Network/Adjacent/Local/Physical), Privileges Required `N/L/H` (None/Low/High), Attack Complexity `L/H` (`H` only when conditions are beyond the attacker's control — race window, specific network position, victim interaction beyond a click). Definitions and the §3 sort heuristic: `references/risk-rating.md`.
- **Impact** — comma-separated subset of `C` / `I` / `A` (Confidentiality / Integrity / Availability). At least one — a row with no impact isn't a threat. Safety hazards (physical-harm impact) carry severity in the STPA section (`references/stpa.md`); cross-reference a hazard ID (`H-#`) in the `Element` cell when a threat triggers a hazard.

**Escalation table — add `CAPEC | CWE | Persona | Event | Source` between `Threat` and `AV`** when the system warrants them: a TM-BOM will be emitted (all five populate the schema), §2.2 operational stratum is produced (the CAPEC → CWE → ATT&CK chain needs them), or the system is regulated / safety-critical / multi-tenant / sector-targeted.

| ID | Element | STRIDE | Threat | CAPEC | CWE | Persona | Event | Source | AV | PR | AC | Impact |
|----|---------|--------|--------|-------|-----|---------|-------|--------|----|----|----|--------|
| T1 | Auth Service (P1) | S | **Session token replay**: … | CAPEC-60 (Reusing Session IDs) | CWE-294 | external-anonymous | session takeover | adversary | N | N | L | C, I |

- **CAPEC + CWE** — the attack pattern and the weakness it exploits. CAPEC → CWE is a 1:1 lookup (each CAPEC pattern declares its CWEs); pick the CAPEC, the CWE follows. Abstraction levels by SDLC stage and the closest-pattern fallback for domain protocols (DICOM, HL7, ICS): `references/capec.md`.
- **Persona** — symbolic name from the §1 personas table (`threats[].threat_persona`).
- **Event** — ≤6-word verb phrase for *what happens* (`threats[].event`): *"session takeover"*, *"PHI exfiltration"*.
- **Source** — enum `adversary / human_error / failure / events_beyond_org_control` (`threats[].sources`). Most threats are `adversary`; STPA non-adversarial scenarios are `human_error` or `failure`.

§3 sorts threats most-exposed first (AV:N > A > L > P; PR:N > L > H; AC:L > H), then by Impact-set size — no standalone "Risk" column. Avoid DREAD. The OWASP TML schema's required `risks[].likelihood / impact / level / score` fields are derived from these row values at TM-BOM emission time per the translation in `references/risk-rating.md` § "Translation to TM-BOM enums".

Iteration tactics: walk across trust boundaries first (highest-value threats live there); start with whatever crosses the outermost boundary; don't skip a threat because it's not the category you're walking right now (write it down — redundancy is fine); every recorded threat ends up as a tracked work item in the team's bug tracker. Full tactics catalog: `references/stride-prompts.md` § "Enumeration tactics".

For the highest-risk handful of threats (top 3–5), consider drawing a threat tree showing how the threat could be realized. One Mermaid tree per top threat is plenty.

## Mitigations and Q4

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a **layered stack of concrete controls** — typically 2–6 per threat, scaled to stakes; a single-control row on a threat that crosses a trust boundary with `C,I,A` impact is a finding. **Order the stack Protect first** (prevent the threat), then Detect / Respond / Recover as compensating layers — not four equal partners. Three rules on control quality: map each control to the violated property (STRIDE → property → mitigations table in `references/stride-prompts.md`); **prefer built-in mitigations** from the in-scope technology (platform IAM, KMS / HSM, managed mTLS, framework CSRF, OS sandboxing, secure boot) over custom code; **name the cryptographic suite, not the class** (TLS 1.3 / mTLS, AES-256-GCM, Ed25519, HKDF, Argon2id; FIPS 140-3 when regulated) — *"use encryption"* is the "Generic mitigations" anti-pattern. A Mitigate row may also carry a NIST SP 800-53 r5 control-family tag. Full guidance — control primitives, crypto defaults, 800-53 linkage: `references/stride-prompts.md` § "Choosing the right primitive".

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

Plus: open questions / assumptions still to validate, and a **next review trigger** — pick at least one concrete trigger (an architecture/design change above a threshold, a new criticality-level dependency, a security incident in the system or sector, time-elapsed no shorter than 12 months, or a regulatory update; a bare "annually" alone is weak — the template header lists the candidates with thresholds). Record the trigger in §4 *and* the template header so the TM is bound to a re-review event, not a date guess. Q4 is the most-skipped step in real threat models. Don't skip it.

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

Read on demand. Each file carries its own `Related:` cross-links and a "canonical home" declaration — don't duplicate their content back into output.

- `methodologies.md` — hybrid three-stratum framing, system-type decision matrix, when to swap/supplement STRIDE (LINDDUN / PASTA / attack trees / ATT&CK / OCTAVE / VAST / AI-ML), cyber-physical attack-path construction, sector ISAC / regulator lists.
- `environments.md` — per-environment trust-boundary patterns (cloud / on-prem / embedded-IoT / OT-ICS / mobile), ownership taxonomy, domain notes. Read when drawing the DFD.
- `medical.md` — medical / PACS / DICOM / IoMT: layer mix, §1 intake template, patient-as-asset framing, regulatory list, DICOM/HL7 STRIDE specifics. Load only for medical systems.
- `centric-methods.md` — entry-point taxonomy (asset / data / flow / process / user-needs / attacker / code) and the generation-vs-characterization distinction.
- `data-centric.md` — NIST SP 800-154 data-centric workflow.
- `capec.md` — STRIDE → CAPEC → CWE → ATT&CK → D3FEND → mitigation chain, the three abstraction levels, coverage gaps.
- `stride-prompts.md` — STRIDE category prompts, per-element example threats, Per-Element vs Per-Interaction, enumeration tactics, mitigation tables, NIST CSF + 800-53 mitigation axes.
- `dfd-mermaid.md` — DFD-to-Mermaid mapping, dashed-subgraph and nested-subgraph patterns, labeling convention, worked examples, the high-value-dataflow categories that pull a Level-2 view.
- `non-dfd-models.md` — sequence / swim-lane and state diagrams as DFD supplements; load for clinical workflows, two-person rules, service-mode access, documented state machines.
- `validation.md` — Q4 deep dive: model/reality conformance, the three Shostack checklists, pen testing as complement, validating third-party-code threats.
- `manifesto.md` — Threat Modeling Manifesto values / principles / patterns / anti-patterns, the silent-transference anti-pattern, and the artifact-level failure-mode catalog.
- `risk-rating.md` — the AV/PR/AC + CIA scheme, §3 sort heuristic, translation to TM-BOM `risks[]` enums, clinical-context CVSS, ISO 14971 / TIR57 mapping, why DREAD is discouraged.
- `stpa.md` — safety + security joint hazard analysis (STPA / STPA-Sec / STPA-SafeSec lineage), two modes (default supplement). Load whenever a worst case is physical harm.
- `examples.md` — pointer to the OWASP Threat Model Library for end-to-end worked examples by system type.
- `tm-bom.md` — TM-BOM emission procedure (OWASP TML schema mapping, symbolic-name derivation, validation checklist). Load only when emitting a TM-BOM.

Assets: blank template `assets/threat-model-template.md`; TM-BOM starter `assets/tm-bom-example.json`; OWASP TML JSON schema (v1.0.2) for TM-BOM validation `assets/threat-model.schema.json`.

## Producing the TM-BOM (machine-readable artifact)

The TM-BOM (Threat Model Bill of Materials) is a structured JSON companion to the markdown — the system, threats, controls, and risks in machine-readable form, emitted against the **OWASP Threat Model Library** JSON schema (v1.0.2, MIT-licensed), bundled at `assets/threat-model.schema.json`.

**Emission is opt-in.** The markdown is the always-shipped, canonical artifact for humans. Emit the TM-BOM only when the user asks for tracker import (Polarion, Jira, GitHub Issues, ServiceNow), tooling-pipeline integration, or upstream contribution to the OWASP TML library. Never emit it unprompted, and never emit only the JSON. When you do emit one, validate it against the bundled schema (`check-jsonschema --schemafile assets/threat-model.schema.json <file>.json`) before handing it over.

Full procedure — field-by-field schema mapping, symbolic-name derivation, the Mitigate/Eliminate/Transfer/Accept → `control.status` mapping, the top-level `extensions` pattern, and the pre-ship validation checklist: `references/tm-bom.md`. Starter file: `assets/tm-bom-example.json`; bundled schema for validation: `assets/threat-model.schema.json`.

## Citations

Concepts in this skill paraphrase from the sources below; cite them in any output that quotes or substantively summarizes them. Fuller per-source detail (section numbers, licenses, what each contributes) lives in each reference file's "Sources paraphrased" header.

- **OWASP** — Threat Modeling community page, Process, Cheat Sheet, Security Culture v1.0; **Threat Model Library** (https://github.com/OWASP/www-project-threat-model-library, MIT-licensed — the TM-BOM JSON schema and worked-example repository).
- **MITRE Threat Modeling Playbook** (Toreon, OWASP) — DFD3 conformance, sequence / state diagrams, the assumptions register, the silent risk transference / acceptance anti-pattern, NIST CSF + D3FEND mapping, the CVSS Rubric for Medical Devices.
- **Threat Modeling Manifesto** — values, principles, patterns, anti-patterns (CC-BY 4.0).
- **Adam Shostack**, *Threat Modeling: Designing for Security* (Wiley, 2014) — Four Question Framework, STRIDE-Per-Element, diagramming / validation checklists, iteration tactics, bug-filing as exit point.
- **NIST** — SP 800-154 (data-centric methodology); SP 800-53 r5 (control families); Cybersecurity Framework v2.0 (Protect / Detect / Respond / Recover).
- **Tatam, Shanmugam, Azam & Kannoorpatti**, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021, CC-BY 4.0) — three-stratum hybrid framing; CAPEC → CWE design-time payoff.
- **MITRE** — ATT&CK, CAPEC, CWE, D3FEND (https://d3fend.mitre.org/); CVSS Rubric for Medical Devices (github.com/mitre/md-cvss-rubric-tools).
- **STRIDE** — Kohnfelder & Garg (Microsoft); **DESIST** — Gunnar Peterson; **Cyber Kill Chain** — Lockheed Martin.
- **STPA** — Leveson, *Engineering a Safer World* (MIT Press, 2011) → STPA-Sec, Young & Leveson (2014) → STPA-SafeSec, Friedberg et al. (2017, CC-BY); cite the specific paper when the distinction matters.
- **Risk-prioritized cyber-physical attack paths** — Stellios, Kotzanikolaou & Grigoriadis (2021).
- **ISO 14971:2019**, **AAMI TIR57:2023**, **US FDA** *Cybersecurity in Medical Devices* (2023 final guidance) — the joint cyber/safety risk model (severity × `P1 × P2`) expected for medical-device submissions.
