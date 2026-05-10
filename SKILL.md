---
name: threat-modeler
description: Use when the user asks for a threat model, security architecture review, attack-surface analysis, or "what can go wrong with this system" against a paste of an architecture, design, spec, or component list — or when they explicitly name a methodology (STRIDE, LINDDUN, PASTA, STPA-SafeSec) for design-time analysis. Also use when the user mentions a regulatory threat-model deliverable (FDA premarket cybersecurity, IEC 81001-5-1, IEC 62443, ISO 26262 cyber). Do not use for penetration test planning, vulnerability scanning, incident response, or passing references to STRIDE/DFD that aren't a request to produce a model.
---

# Threat Modeler

Produce a working threat model for an engineering team. State assumptions and proceed rather than interrogating the user. Ship a useful approximation rather than polishing a perfect artifact. When STRIDE categorization is debatable, record the threat and move on. Avoid two anti-patterns: **Admiration for the problem** (analyzing without producing solutions — every threat gets a response) and **Asset rabbit-holing** (padding the asset list). Full Manifesto values, principles, and pattern catalog: `references/manifesto.md`.

## Pick a mode

- **Guided interview** — user said "help me threat model X" without a system description. Run §"Guided interview" to scope, then produce the model.
- **Fast-path** — user pasted architecture, a DFD, a component list, a spec, code, or a description detailed enough to identify external entities, processes, data stores, and trust boundaries. Skip the interview, but **before drawing the DFD** pin down the four things from Round 1.5 (technology stack, environment type, environment ownership, physical/operational context). If any aren't in the artifact, state your inferred answer as a numbered assumption ("A3: assumed customer IT owns the on-prem network; correct if otherwise") and proceed.
- **Update / extend** — user pasted an existing threat model. Treat their document as input: preserve existing IDs, only renumber on collision, mark new entries clearly, and run a Q4 model/reality conformance check first per `references/validation.md`. Match their structure rather than rewriting it.

When in doubt, state assumptions and proceed.

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

ID conventions to keep cross-stratum references unambiguous (one prefix per namespace):

- Assets in §1: `AS1`, `AS2`, …
- Flow-centric STRIDE threats: `T1`, `T2`, …
- Supplementary entry-point findings: `V1`, `V2`, … (data-centric / asset-centric / etc. — use the `V` prefix regardless of pass type, with the pass type recorded in the row)
- Privacy / LINDDUN / AI-ML findings: `PR1`, `PR2`, …
- Mitigations / derived requirements: `SR-001`, `SR-002`, …
- ATT&CK / CAPEC / CWE / CVE: cite the upstream ID directly

## DFD

Use Mermaid `flowchart LR` (or `TD`). Map elements as: external entity → rectangle (`EE[User]`); process → rounded rectangle or circle (`P1(Auth Service)`); data store → cylinder (`DS[(User DB)]`); data flow → labeled arrow (`EE -- "credentials" --> P1`); trust boundary → Mermaid `subgraph`. Render trust-boundary subgraphs with a dashed border and label each subgraph with owner, environment type, and trust level — full conventions, the dashed-stroke pattern, the labeling format, and worked examples in `references/dfd-mermaid.md`. The per-environment patterns that should drive *which* subgraphs you draw live in `references/environments.md`.

Three rules, applied in order:

1. **Every element belongs to exactly one zone subgraph.** A floating element with no zone is a missing trust boundary. If an entity is genuinely outside any controlled zone (an attacker on the public internet), put it in an explicit `Untrusted` subgraph.
2. **Nest subgraphs for trust-boundaries-within-trust-boundaries.** Cloud account / VPC / subnet, Windows host / VM / container, embedded device / secure element, mobile device / app sandbox / hardware keystore. Pattern: `references/dfd-mermaid.md` § "Nested subgraphs".
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

If you produce §2.2 (operational stratum), include CAPEC and CWE alongside ATT&CK. The CAPEC → CWE bridge is what makes derived requirements (`SR-###`) traceable to a known weakness class rather than a free-text threat sentence. Format, abstraction-level rule of thumb, and STRIDE→CAPEC mapping: `references/capec.md`.

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
| STPA-SafeSec | Safety-critical control loops where worst case is physical harm (medical devices, ICS, automotive, aerospace, robotics); swap or supplement mode | `references/stpa-safesec.md` |
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

Blank template: `assets/threat-model-template.md`.

## Citations

Concepts in this skill paraphrase from:

- **OWASP** — Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, Security Culture v1.0 §6, Threat Modeling Playbook (Toreon).
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
