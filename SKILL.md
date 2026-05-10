---
name: threat-modeler
description: Use when the user asks the assistant to *produce* a threat model, security architecture review, attack-surface analysis, or "what can go wrong with this system" — typically against a paste of an architecture, design, spec, code, or component list — or when they explicitly name a methodology (STRIDE, LINDDUN, PASTA, STPA) and ask for design-time analysis of *their* system. Also use when the user mentions a regulatory threat-model deliverable (FDA premarket cybersecurity, IEC 81001-5-1, IEC 62443, ISO 26262 cyber). The trigger is the request to *produce an artifact*, not the mention of a term. Do **not** use for: educational queries ("what does S in STRIDE stand for?", "explain DFD trust boundaries", "summarize LINDDUN", "what is PASTA?"); definitions, glossary lookups, or methodology comparisons that don't reference a specific system; coursework / certification study help; pen-test planning; vulnerability scanning; incident response; SBOM generation; threat intelligence summaries; CVE / advisory triage; or passing references to STRIDE / DFD / "attack surface" that appear in unrelated discussions (resume review, blog post drafting, code review of an unrelated change). When in doubt, the question to ask is "is the user asking me to produce a §1 / §2 / §3 / §4 artifact about a system they've described?" — if yes, use the skill; if no, answer the question directly without invoking it.
---

# Threat Modeler

Produce a working threat model for an engineering team. State assumptions and proceed rather than interrogating. Ship a useful approximation rather than polishing a perfect artifact. When STRIDE categorization is debatable, record the threat and move on. Before shipping, scan the draft against `references/manifesto.md` § "Failure-mode catalog" (padded asset lists, single-threat-per-element, generic mitigations like "apply RBAC", STRIDE labels with no concrete attack, mitigation table that doesn't match the threat table) — a model with those shapes is worse than no model because it implies threats were considered when they weren't. Manifesto values, principles, and anti-patterns: `references/manifesto.md`.

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
- Safety implications? Drives the safety-bump rule from `references/risk-rating.md`.
- Regulatory constraints?
- What sector? Determines which sector ISAC and regulator framing apply (full lists in `references/methodologies.md`).
- Existing security controls already assumed in place?

If something's still missing after Round 2, ask one focused round — but err on stating assumptions and proceeding. After Round 2 you should have enough to draft the DFD and pick supplements from §"Picking supplements" below. Domain-specific guidance loads on demand: medical / PACS / DICOM → `references/medical.md`; embedded / IoT / OT / cloud-native / AI-ML / third-party code → `references/environments.md` § "Domain notes".

## Producing the threat model

Output a single markdown document. Default to a single document; only split into linked documents if the user asks. Use Mermaid for the DFD. **Copy `assets/threat-model-template.md` and fill it in** rather than rebuilding the skeleton — the template carries the full structure (header metadata, scope-classification table, DFD element catalog, trust-boundary table, threat-personas table, the §2.1 / §2.2 / §2.3 tables, mitigation table, risk register, derived requirements, Q4 checklist, changelog).

The four top-level sections are fixed (Q1 / Q2 / Q3 / Q4). §2 has three subsections by default — Contextual / Operational / Strategic — populated when warranted.

**Default scaffold rule.** Add content to a §2 subsection when you have something to say there. Omit a subsection entirely when you don't. No "not applicable" stubs. The three-stratum layout is *available scaffolding*, not a coverage quota — it comes from Tatam et al. (2021), one paper, not a practitioner consensus. Background and the system-type matrix that suggests which supplements to add: `references/methodologies.md` § "Hybrid as default".

**Threats span strata, mitigations don't.** Threats live in §2.1 / §2.2 / §2.3 with one shared ID space and one risk scale. §3 produces a single prioritized mitigation table over all of them — never duplicate a threat across strata, cross-reference instead.

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

| ID | Element | STRIDE | Threat | Persona | Event | Source | Likelihood | Impact | Risk |
|----|---------|--------|--------|---------|-------|--------|------------|--------|------|
| T1 | Auth Service (P1) | S | Attacker reuses captured session token across boundary | external-anonymous | session takeover | adversary | M | H | High |

Per-row additions (required for the TM-BOM; useful narrative even without it):
- **Persona** — symbolic name of one of the personas captured in Round 2 (the OWASP TML schema's `threats[].threat_persona` field).
- **Event** — short verb-phrase summary (≤6 words) of *what happens* in the threat scenario; populates the schema's `threats[].event` field. Example: *"session takeover"*, *"PHI exfiltration"*, *"unauthenticated firmware flash"*.
- **Source** — one or more values from this fixed enum: `adversary / human_error / failure / events_beyond_org_control` (the schema's `threats[].sources` field). Most threats are `adversary`; STPA non-adversarial scenarios are `human_error` or `failure`.

Use qualitative L/M/H by default. For safety-critical systems, see `references/risk-rating.md` for the safety-bump rule before assigning ratings. Avoid DREAD. If quantitative scoring is required, use OWASP Risk Rating Methodology (see `references/risk-rating.md`).

Iteration tactics: walk across trust boundaries first (highest-value threats live there); start with whatever crosses the outermost boundary; don't skip a threat because it's not the category you're walking right now (write it down — redundancy is fine); every recorded threat ends up as a tracked work item in the team's bug tracker. Full tactics catalog: `references/stride-prompts.md` § "Enumeration tactics".

For the highest-risk handful of threats (top 3–5), consider drawing a threat tree showing how the threat could be realized. One Mermaid tree per top threat is plenty.

## Mitigations and Q4

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a concrete control mapped to the violated property — STRIDE → security property → typical mitigations table in `references/stride-prompts.md`.

If you produce §2.2 (operational stratum), include CAPEC and CWE alongside ATT&CK. The CAPEC → CWE bridge is what makes derived requirements (`SR-###`) traceable to a known weakness class rather than a free-text threat sentence. **Pick the CAPEC abstraction level for the user, by SDLC stage** (Meta for early architecture, Standard for design review, Detailed for component-level work) and don't expose the level in the row unless you were forced to use a higher abstraction because no Detailed pattern exists for the domain-specific protocol (DICOM, HL7, ICS). When forced, footnote `(closest pattern; no Detailed available)`. Format, the abstraction-level rule of thumb, and STRIDE→CAPEC mapping: `references/capec.md`.

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
| AI/ML threat list | ML components in scope (prompt injection, training-data poisoning, model extraction) | `references/methodologies.md` § ML/AI; OWASP LLM Top 10 |
| PASTA framing | Borrow the business-impact stage when executive sign-off is needed | `references/methodologies.md` § PASTA |
| STPA | Load whenever the system has **safety impact** — any plausible worst-case outcome involves physical harm to a person, equipment, or the environment (medical devices, ICS, automotive, aerospace, robotics, consumer IoT that controls something dangerous). Swap or supplement mode; default to supplement. | `references/stpa.md` |
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
- `stpa.md` — Safety + security joint hazard analysis. Umbrella term covering Leveson's STPA, Young & Leveson's STPA-Sec, and Friedberg et al.'s STPA-SafeSec; the workflow uses the integrated form with the component-layer mapping. Two modes (swap or supplement; default to supplement). Load whenever the system has safety impact — physical harm to people, equipment, or the environment is the trigger, not control-loop shape or regulator presence.
- `examples.md` — Pointer to the OWASP Threat Model Library (https://github.com/OWASP/www-project-threat-model-library) as the source for end-to-end worked examples by system type (web-applications, ai-ml-systems, infrastructure, third-party-integrations). The library's JSON schema also defines the TM-BOM contract — see § "Producing the TM-BOM" above. The medical / DICOM end-to-end example is the exception, kept inline at `data-centric.md` § "Worked example" + `dfd-mermaid.md` § "Worked example: small clinical PACS".

Blank template: `assets/threat-model-template.md`.

## Producing the TM-BOM (machine-readable artifact)

> **What "TM-BOM" means here**: a Threat Model Bill of Materials — a structured, machine-readable description of the system, threats, controls, and risks, analogous to an SBOM for software components. The concrete schema this skill emits against is the **OWASP Threat Model Library** JSON schema (`threat-model.schema.json` v1.0.2, MIT-licensed), which itself aligns with the pending CycloneDX TM-BOM specification. Tools that consume CycloneDX BOMs (SBOM, SaaSBOM, HBOM today; TM-BOM when published) will be the consumers; OWASP TML is the working contract today.

**TM-BOM emission is opt-in.** The markdown is the always-shipped artifact; the TM-BOM (JSON) is a derived view, emitted only when the user asks for tracker import (Polarion, Jira, GitHub Issues, ServiceNow), tooling-pipeline integration, or upstream contribution to the OWASP TML library. Don't emit it unprompted, and never emit only the JSON — the markdown is the canonical artifact for humans, the TM-BOM is the machine-readable companion. When you do emit one, validate against the OWASP TML schema before handing it over (the schema is the contract downstream tools already implement against; don't invent your own — command in § "Validation checklist" below).

### Symbolic-name derivation (every TM-BOM ID must match `^[0-9a-z-]+$`)

The schema requires every `symbolic_name` to be lowercase alphanumeric plus hyphens. The skill's markdown IDs (`AS1`, `T1`, `V1`, `PR1`, `SR-001`, `TB1`, `ASM1`) are uppercase for human readability — derive the schema-valid symbolic name by lowercasing and inserting a hyphen between the letter prefix and any digits *only when no hyphen is already present*:

| Markdown ID | TM-BOM `symbolic_name` |
|---|---|
| `AS1` | `as-1` |
| `T1` | `t-1` |
| `V1` | `v-1` |
| `PR1` | `pr-1` |
| `SR-001` | `sr-001` (already hyphenated; just lowercase) |
| `TB1` | `tb-1` |
| `ASM1` | `asm-1` |

Trust-zone subgraph IDs from the DFD (e.g. `Hospital`, `Cloud`, `Internet`, `ProdAcct`) become trust_zone symbolic_names by lowercasing and replacing camelCase or spaces with hyphens (`hospital`, `cloud`, `internet`, `prod-acct`). Cross-references in the JSON (`threats[].threat_persona`, `controls[].threats`, `risks[].threats`, `data_sets[].placements[].data_store`, `trust_boundaries[].trust_zone_a/b`, `data_flows[].source/destination.object`) use the derived symbolic names — keep them consistent across the file.

### Field-by-field mapping

| OWASP TML field (required unless noted) | Source in this skill | Notes / enum values |
|---|---|---|
| `version` (top-level) | Set to `"1.0.0"` for the first emission, bump per the skill's changelog | Must match `^\d+(\.\d+)*$` |
| `scope.title` | §1 system name | |
| `scope.description` | §1 system description (1–2 paragraphs) | |
| `scope.business_criticality` | Round 1 elicitation | enum: `minimal / low / moderate / high / maximal` |
| `scope.data_sensitivity` | Round 1 elicitation (data classes) | array enum: `pii / phi / fin / ip / cred / biz / gov / pci / op` |
| `scope.exposure` | Round 1 elicitation | enum: `internal / external` |
| `scope.tier` | Round 1 elicitation | enum: `mission_critical / business_critical / important / non_critical` |
| `description` (top-level, optional) | Free-form supplemental description | |
| `repo_link` (optional) | If the user names a repo URL | |
| `released_at` / `reviewed_at` (optional) | Template metadata block | ISO-8601 date or date-time |
| `trust_zones[]` | DFD subgraphs (one zone per subgraph) | `symbolic_name` (derived from subgraph ID), `title`, `description` (the `<owner> \| <env-type> \| <trust>` triplet works as the description) |
| `trust_boundaries[]` | §1 trust-boundary table | `trust_zone_a` / `trust_zone_b` are symbolic refs; `access_control_methods` array enum: `none / acl / rbac / mac / dac / abac`; `authentication_methods` array enum: `none / password / otp / challenge_response / public_key / token / biometrics / sso / social` |
| `actors[]` | §1 external entities | `type` enum: `system / user / power_user / administrator / engineer / third_party`; `trust_zone` is symbolic ref |
| `components[]` | DFD processes (rounded boxes / circles) | `trust_zone` is symbolic ref; `parent_component` (optional) for nested subgraphs |
| `data_stores[]` | DFD data stores (cylinders) | `type` enum: `sql / key_value / document / object / graph / time_series` (elicited in Round 1); `vendor` / `product` optional |
| `data_sets[]` | Data classes from data-centric pass (if run) | `placements[]` is `[{data_store: <symbolic_name>, encrypted: bool}, …]`; `data_sensitivity` array enum same as `scope.data_sensitivity` |
| `data_flows[]` | DFD arrows | `source` / `destination` are `{type: "actor"|"component"|"data_store", object: <symbolic_name>}`; `has_sensitive_data` and `encrypted` are booleans (defaults: `false` if no PII/PHI/cred in the flow's data class; `true` if the flow label includes mTLS / TLS / signed) |
| `diagrams[]` (optional) | The Mermaid DFD itself | `type: "mermaid"`, `source: <full Mermaid block as a string>`, `title: "Data Flow Diagram"` |
| `assumptions[]` (optional) | §1 assumptions table | `validity` enum: `unconfirmed / confirmed / rejected` (default `unconfirmed`); `topics` (optional) is an array of symbolic refs to other entities the assumption is about |
| `threat_personas[]` | Round 2 personas (always at least one) | `skill_level` enum: `script_kid / insider / engineer / expert_engineer / oc_sponsored / state_sponsored`; `access_level` enum: `anonymous / user / admin`; `applicability_to_org` enum: `minimal / low / moderate / high / maximal` |
| `threats[]` | §2.1 / §2.2 / §2.3 threat tables (one row per `T#` / `V#` / `PR#`) | `threat_persona` is symbolic ref; `event` is the verb-phrase event column; `sources` array enum: `adversary / human_error / failure / events_beyond_org_control`; `attack_mechanisms[]` is `[{capec_id: int, capec_title: string}, …]`; `weaknesses[]` is `[{cwe_id: int, cwe_title: string}, …]`; `components_affected[]` is symbolic refs |
| `controls[]` | §3 mitigation rows | `threats` is array of threat symbolic refs; `status` enum: `assumed / active / suggested / under_review / approved / scheduled / retired / wont_do` (default for new mitigations: `suggested`; for already-in-place: `active`); `priority` enum: `none / low / medium / high / critical` (translate from §3 risk: Low→low, Medium→medium, High→high, Critical→critical); `trust_boundary` (optional) is `{trust_zone_a, trust_zone_b}` |
| `risks[]` | §3 risk ratings, one risk per threat (or per cluster of cross-referenced threats) | `threats` array of refs; `likelihood` and `impact` use the schema enums (translation table in `references/risk-rating.md` § "L/M/H ↔ TM-BOM enums"); `score` is integer 0–25; `level` enum: `very_low / low / medium / high / very_high / critical` |
| Skill-specific fields with no schema home (STRIDE category, stratum tag, Stellios path-product score, Mitigate/Eliminate/Transfer/Accept response distinct from `control.status`, Accept-rationale + decision-maker for threats with no `control` row) | **Top-level `extensions` only** — not per-object (every object def in the schema has `additionalProperties: false`, so per-object extensions fail validation). Key per-object data by the object's `symbolic_name`. | Extension keys must match the schema's regex: `<domain-with-hyphens-allowed>.<letters-only-TLD>/<path>`. Use the namespace `threat-modeler.tmskill/...` — the natural-looking `tmskill.threat-modeler/...` is **rejected** by the schema because the TLD label can't contain hyphens. Recommended paths: `threat-modeler.tmskill/stride-by-threat`, `threat-modeler.tmskill/stratum-by-threat`, `threat-modeler.tmskill/path-score-by-threat`, `threat-modeler.tmskill/response-by-control`, `threat-modeler.tmskill/accept-rationale-by-threat`. |

**Top-level header.** Every emitted TM-BOM starts with these two lines so the file declares which schema version it was built against:

```json
{
  "$schema": "https://github.com/OWASP/www-project-threat-model-library/blob/v1.0.2/threat-model.schema.json",
  "version": "1.0.0",
  ...
}
```

Bump `$schema` when the OWASP TML schema version moves; bump `version` per the model's own changelog (it's a free string matching `^\d+(\.\d+)*$`, not the schema version).

**Per-object extension example.** STRIDE category, response, stratum tag — all are per-object data the schema doesn't carry. Top-level extensions look like this:

```json
"extensions": {
  "threat-modeler.tmskill/stride-by-threat": {
    "t-1": "S",
    "t-2": "T"
  },
  "threat-modeler.tmskill/stratum-by-threat": {
    "t-1": "contextual",
    "v-1": "contextual"
  },
  "threat-modeler.tmskill/response-by-control": {
    "sr-001": "Mitigate",
    "sr-002": "Eliminate"
  }
}
```

### Mapping the Mitigate/Eliminate/Transfer/Accept response

The skill's response taxonomy doesn't map 1:1 onto `control.status`. Use this rule:

| Skill response | `control.status` | When to use which |
|---|---|---|
| `Mitigate` (control planned/in place) | `suggested` (new) or `active` (existing) | A new control gets `suggested`; if the user confirms the control is already deployed, use `active` |
| `Eliminate` (remove the threat surface) | `approved` if the design change is approved; `suggested` otherwise | The "control" is the design change itself |
| `Transfer` (insurance, vendor SLA, customer responsibility) | `assumed` | The control exists outside this system |
| `Accept` (no control) | Don't emit a `control` row at all | Record rationale + decision-maker in the top-level `extensions["threat-modeler.tmskill/accept-rationale-by-threat"]` block, keyed by the threat's symbolic name (`{rationale, decision_maker, decided_at}`); also note rationale in the threat's `description`. An unjustified `Accept` is indistinguishable from a missed threat — failure-mode catalog. |

Also record the original response verbatim in the top-level `extensions["threat-modeler.tmskill/response-by-control"]` (and `…/accept-rationale-by-threat` for Accept rows) so the markdown ↔ JSON round-trip is lossless.

Example Accept-rationale extension:

```json
"extensions": {
  "threat-modeler.tmskill/accept-rationale-by-threat": {
    "t-12": {
      "rationale": "Mitigation cost (~$200k re-architecting the legacy importer) exceeds expected loss; risk re-evaluated annually.",
      "decision_maker": "A. Smith, Engineering Director",
      "decided_at": "2026-05-10"
    }
  }
}
```

### Validation checklist before shipping the TM-BOM

- [ ] Top-level `$schema` and `version` are present.
- [ ] Every `symbolic_name` matches `^[0-9a-z-]+$` (run a regex over the file).
- [ ] Every cross-ref (`threats[].threat_persona`, `controls[].threats`, `risks[].threats`, `data_sets[].placements[].data_store`, `trust_boundaries[].trust_zone_a/b`, `data_flows[].source/destination.object`, `actors[].trust_zone`, `components[].trust_zone`, `data_stores[].trust_zone`) resolves to a defined symbolic name in the same file.
- [ ] All nine top-level required fields are present: `version`, `scope`, `trust_zones`, `trust_boundaries`, `actors`, `components`, `data_stores`, `data_sets` (may be empty array if no data-centric pass run), `data_flows`.
- [ ] All four required `scope` enums are present (`business_criticality`, `data_sensitivity`, `exposure`, `tier`).
- [ ] At least one `threat_persona` is defined and every threat references one.
- [ ] `threats[].event` is non-empty for every threat.
- [ ] `threats[].sources` is non-empty for every threat.
- [ ] `risks[].score` is in `[0, 25]` and is consistent with `risks[].level`.
- [ ] No per-object `extensions` (every object has `additionalProperties: false` — extensions only at top level).
- [ ] Extension keys match the schema's regex (TLD must be letters-only — `threat-modeler.tmskill/<path>`, **not** `tmskill.threat-modeler/<path>`).
- [ ] Schema validation exits 0. Run, against the v1.0.2 schema:

  ```
  curl -sSf -o /tmp/threat-model.schema.json \
    https://raw.githubusercontent.com/OWASP/www-project-threat-model-library/main/threat-model.schema.json
  check-jsonschema --schemafile /tmp/threat-model.schema.json <your-tm-bom>.json
  ```

Starters: `assets/tm-bom-example.json` (minimal, exercises every required field and the top-level `extensions` pattern, validates against v1.0.2 with `check-jsonschema` exit 0); the OWASP TML library at https://github.com/OWASP/www-project-threat-model-library/tree/main/threat-models for fuller worked examples (`ai-ml-systems/husky-ai-threat-model.json` is the most complete). Open the closest example before producing a TM-BOM; mirror its shape and depth.

## Citations

Concepts in this skill paraphrase from:

- **OWASP** — Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, Security Culture v1.0 §6, Threat Modeling Playbook (Toreon), **Threat Model Library** (https://github.com/OWASP/www-project-threat-model-library, MIT-licensed; TM-BOM JSON schema and the curated example-models repository the skill defers to for worked examples).
- **Threat Modeling Manifesto** — values, principles, patterns, anti-patterns (CC-BY 4.0).
- **Adam Shostack**, *Threat Modeling: Designing for Security* (Wiley, 2014) — Four Question Framework, STRIDE-Per-Element, diagramming and validation checklists, iteration tactics, bug-filing as exit point.
- **NIST SP 800-154** (Draft, Souppaya & Scarfone, 2016) — data-centric methodology.
- **Tatam, Shanmugam, Azam & Kannoorpatti**, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021, CC-BY 4.0) — three-stratum hybrid framing; CAPEC → CWE design-time payoff.
- **MITRE** — ATT&CK, CAPEC, CWE.
- **Lockheed Martin** — Cyber Kill Chain.
- **STRIDE** — Loren Kohnfelder & Praerit Garg (Microsoft). **DESIST** — Gunnar Peterson.
- **STPA** (umbrella term in this skill; lineage: Leveson, *Engineering a Safer World*, MIT Press, 2011 → STPA-Sec, Young & Leveson, 2014 → STPA-SafeSec, Friedberg et al., 2017, CC-BY — the workflow draws from across the lineage; cite the specific paper when the distinction matters).
- **Risk-prioritized cyber-physical attack paths** — Stellios, Kotzanikolaou & Grigoriadis (2021).

Cite these in any output that quotes or substantively summarizes them.
