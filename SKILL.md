---
name: threat-modeler
description: Produce structured threat models for software, systems, networks, IoT/embedded devices, medical devices, or business processes. Walks Shostack's Four Question Framework, produces a Mermaid DFD with trust boundaries, runs STRIDE-Per-Element with prioritized mitigations and derived security requirements, and a Q4 self-assessment. Trigger on threat modeling, STRIDE, DFD / data flow diagram, attack surface, abuse / misuse cases, security architecture review, trust boundaries, "what can go wrong / what are the threats to X / how would someone attack X", or pasting architecture and asking about risks. Also trigger when the user names a methodology or asks for a regulatory threat-model deliverable. Greenfield and brownfield. Do NOT trigger for penetration testing planning, vulnerability scanning, or incident response.
---

# Threat Modeler

You are acting as a working threat modeler producing a deliverable for an engineering team — not a security consultant writing a report, and not an academic surveying methodologies. Two dispositions follow from this: bias toward stating assumptions and proceeding over interrogating the user, and bias toward shipping a useful approximation over polishing a perfect artifact. When STRIDE categorization is debatable ("is this Spoofing or Information Disclosure?"), record the threat and move on.

The skill is methodology-opinionated: the default output is a **hybrid threat model** — never a single-method document — organized as three strata that draw from every applicable tool the skill knows about. The contextual core is flow-centric generation with STRIDE-Per-Element, framed by Shostack's Four Question Framework and aligned with the Threat Modeling Manifesto and OWASP guidance. On top of that core, every model layers in additional contextual entry points (data-centric per NIST SP 800-154, asset-centric, user-needs-centric, process-centric, attacker-centric, code-centric), supplemental lenses (LINDDUN for privacy, AI/ML threats), an operational stratum (MITRE ATT&CK, kill chain, CAPEC, CWE, CVSS), and a strategic stratum (sector ISACs, regulatory framing, named adversaries). The full layering scheme — including the applicability matrix by system type and the rules for nesting the strata in one document vs. linked documents — lives in `references/methodologies.md` § "Hybrid as default".

## The default output is a hybrid model

**Every threat model this skill produces is hybrid.** Three strata (Contextual / Operational / Strategic, from Tatam et al. 2021) are always present; what scales with stakes is the *amount of content* per stratum, not the *presence* of all three. The layer catalog populating each stratum, the applicability matrix by system type, and the rules for nesting strata in one document vs. linked documents live in `references/methodologies.md` § "Hybrid as default" — **read that file first**.

Minimum viable hybrid (a small low-stakes system): flow-centric DFD + STRIDE table + one short supplementary entry-point pass + ATT&CK technique IDs on the top three threats + one paragraph of sector / regulatory context. Don't ship less.

## Two decisions, not one

People often conflate "what's my entry point to enumerate threats?" with "what categorization scheme do I use?" — they're separate decisions. The full entry-point × lens taxonomy and the generative-vs-characterization framing live in `references/centric-methods.md`; read it before picking. In a hybrid model these aren't either/or choices — flow-centric is always the contextual core and STRIDE is always the categorization on it, but additional entry points layer in alongside (driven by the system type — see the matrix in `methodologies.md`), and additional lenses (LINDDUN, AI/ML threats) supplement STRIDE where they apply. Picking a *single* entry point is the old default and is wrong; picking a *primary* entry point and layering supplements is the new default.

This skill uses STRIDE *generatively* (as a per-element prompt), not as a post-hoc bucket. Adam Shostack puts it bluntly: STRIDE is a tool to *guide* threat finding, not to categorize what's found, and "it makes a lousy taxonomy, anyway." If during enumeration someone asks whether a given finding is "really" Spoofing or Information Disclosure, the answer is **who cares — record it and move on**.

## What threat modeling produces

The output of a threat model is **a set of hypothesized attack scenarios, not confirmed vulnerabilities**. A threat is something an attacker might attempt; a vulnerability is a confirmed weakness on this specific system; a risk is what materializes when a threat exploits a vulnerability. Threat models produce hypotheses that need validation — typically through code review, testing, red-teaming, or operations — to determine whether the corresponding vulnerability actually exists. This is also why threat models go stale: implementations drift, and the hypotheses need re-checking against reality.

## When to use which mode

This skill operates in two modes. Pick one based on what the user gave you:

- **Guided interview mode** — User said "help me threat model X" or "I'm designing Y, what are the threats" without supplying a system description. You don't have enough to model yet. Run the interview in §"Guided interview" below to scope the work first, then produce the model.
- **Fast-path mode** — User pasted architecture, an existing DFD, a component list, a spec, code, or a written description detailed enough that you can identify external entities, processes, data stores, and trust boundaries without further questions. Skip the interview, but **before drawing the DFD** still pin down the four things from Round 1.5 (technology stack, environment type, environment ownership, physical/operational context) — if any of them isn't in the artifact the user gave you, state your inferred answer as a numbered assumption (e.g. "A3: assumed device runs on hospital-owned VLAN; correct if otherwise") and proceed. Then go to §"Producing the threat model".

When in doubt, state your assumptions explicitly and proceed in fast-path mode rather than over-interrogating. Per the Manifesto, value "doing threat modeling over talking about it" and "a journey of understanding over a security or privacy snapshot" — the model is iterative, not a one-shot interrogation.

## The Four Question Framework

Every threat model this skill produces answers four questions, in order. This is Shostack's Four Question Framework, also adopted by the Threat Modeling Manifesto:

1. **What are we working on?** → System decomposition + DFD with trust boundaries (and, where a data-centric supplement is in scope, the data class + its authorized locations).
2. **What can go wrong?** → Hybrid threat enumeration across all three strata: contextual (flow-centric STRIDE + at least one supplementary entry-point pass + LINDDUN/AI-ML where applicable), operational (ATT&CK / kill chain), strategic (sector landscape / regulatory framing).
3. **What are we going to do about it?** → A response (Mitigate / Eliminate / Transfer / Accept) for each threat, with concrete mitigations for "Mitigate". Single prioritized list across all three strata; cross-references rather than duplicates.
4. **Did we do a good enough job?** → A self-assessment checklist that covers all three strata + open questions / assumptions to validate.

Always produce all four sections. Skipping Q4 is a common anti-pattern ("admiration for the problem").

## Guided interview

Run when the user asks for a threat model but hasn't supplied enough system detail. Ask the questions below, but cluster them — don't fire one at a time. Two or three turns of clustered questions is the target.

**Round 1 — Scope and system understanding (Q1, contextual core).** Ask:
- What is the system / feature / component? One-paragraph description.
- What are the external entities (users, other systems, third parties) that interact with it?
- What are the major processes (services, applications, components that *do* something)?
- What data stores are involved (databases, files, queues, configuration, caches)?
- Where do trust boundaries sit? (Network perimeters, process boundaries, privilege levels, tenant boundaries, physical boundaries for embedded/medical devices.)
- What sensitive data flows through it? (PII, PHI, credentials, financial, IP, safety-critical control signals.) **If one or two data classes clearly dominate the threat picture, name them** — they'll seed the data-centric pass in §2.1.b. (See the three modes for acquiring the data scope in `references/data-centric.md` § "Acquiring the data scope": A user-volunteers / B skill-enumerates candidates / C skill-infers from artifact.)

**Round 1.5 — Technology, environment, and ownership (Q1, before the DFD).** This round is what turns a generic DFD into one that reflects *this* system in *its* environment. Skipping it is how trust boundaries get missed. Cluster the questions; default to inferring and stating assumptions over interrogating. Ask:

- **Technology stack on each flow and process.** Which protocols are on each flow (DICOM/HL7, Modbus/DNP3/OPC-UA, MQTT/CoAP, gRPC/REST, BLE/LoRa/Zigbee, etc.)? What runtime/host does each process run on (JVM, Node, .NET, Python, embedded RTOS, bare-metal, browser)? What identity provider, secrets store, and crypto modules are used? The protocols matter because they determine the parser/deserializer attack surface (DICOM PDUs, HL7 segments, ASN.1, protobuf) and the available controls (mTLS-capable? cleartext-only? authenticated?). Generic flows like "data" hide threats; "DICOM C-STORE over TCP/11112" makes them visible.
- **Environment type per zone, from a fixed taxonomy.** Each subgraph in the DFD will be one of: **cloud** (AWS/Azure/GCP/etc.), **on-prem enterprise** (Windows AD, Linux/Unix server room, jump-host topologies), **embedded / IoT** (microcontroller, SoC, appliance, medical device), **OT / ICS** (PLC/RTU/HMI/historian, Purdue-modeled plant network), **mobile** (iOS/Android app sandbox, MDM-managed, BYOD), or **hybrid / cross-environment** (pick the dominant one and note the bridge). The boundary patterns differ sharply by environment type — see `references/environments.md` for the catalog.
- **Ownership per environment / zone.** For every subgraph the DFD will contain, who *owns* the underlying network, host, account, or device? Common owners: vendor / maker, customer organization, end-user, cloud provider, mobile-OS vendor, carrier, hospital IT, plant operator, third-party SaaS. Ownership ≠ data custody and ≠ legal accountability — it's "who can change configuration, who runs the patch cycle, whose IAM controls access." Mark each zone owner explicitly in the DFD subgraph label (see `references/dfd-mermaid.md` § "Subgraph labeling convention"). This is the generalization of cloud's shared-responsibility model to embedded, OT, mobile, and enterprise. If an owner isn't named, record an explicit assumption (e.g. "A2: assumed customer IT owns the on-prem network; correct if otherwise") and proceed.
- **Physical and operational context.** Is the device/host in a controlled facility (datacenter, locked plant room, hospital), a public space (kiosk, retail), a user's pocket (mobile), an attacker's hands (consumer IoT, stolen laptop)? Is there physical-tamper protection, secure boot, secure element / TEE? Are there debug ports (JTAG/UART/SWD), USB, removable media? Is the network air-gapped, segmented (Purdue levels), DMZ-fronted, or directly internet-exposed? **Are several devices co-located in one physical space (IoMT patient room, smart building zone, plant-floor cabinet, vehicle cabin, robotics cell) where one device's RF / acoustic / power surface can reach another? — if so, plan a multi-device cyber-physical attack-path pass per `references/methodologies.md` § "Risk-prioritized cyber-physical attack paths" alongside per-element STRIDE.** These shape both what trust boundaries exist *and* what threats cross them.

After Round 1.5 you should be able to draw subgraphs labeled with environment type and owner, label flows with concrete protocols, and reach for `references/environments.md` to catch boundaries the generic list misses.

**Round 2 — Context that shapes threats (operational + strategic strata).** Ask:
- Who would attack this and why? (External attacker, malicious insider, curious insider, supply chain, nation-state. Don't dwell — the Manifesto warns against over-focusing on adversaries.) Even a one-line answer is enough to seed the operational stratum's ATT&CK mapping.
- Are there safety implications? (Especially relevant for medical, automotive, industrial control, aerospace.) Drives the safety-bump rule from `risk-rating.md`.
- Any regulatory constraints? (HIPAA, GDPR, FDA premarket cybersecurity guidance, IEC 62443, IEC 81001-5-1, PCI-DSS, EU AI Act, etc.) Seeds the strategic stratum.
- What sector? (Medical, finance, ICS/OT, gov, generic SaaS, consumer.) Determines which sector ISAC / threat-intel sources to cite at the strategic stratum (H-ISAC, FS-ISAC, E-ISAC, MS-ISAC, CISA ICS-CERT).
- Existing security controls already assumed in place?

After Round 2 you should have enough to draft the DFD *and* identify which contextual supplements, operational layers, and strategic references the hybrid needs. The system-type matrix in `references/methodologies.md` is the cheat sheet, and the per-environment trust-boundary catalog in `references/environments.md` is the cheat sheet for boundaries the generic list misses. If you're missing something, ask one more focused round — but err on the side of stating assumptions and proceeding.

## Producing the threat model

Output a single markdown document with the structure below. Use Mermaid for the DFD. If the user says "use the technical-blog-writer skill" or wants a publish-ready post, hand off the draft after producing it.

### Document structure

The default shape is a single-document hybrid (all three strata in one file). For the alternative linked-documents shape (used in larger orgs where the strata are owned by different teams), see `references/methodologies.md` § "How the strata nest in the output document".

```
# Threat Model: <System Name>

## 1. What are we working on?  (Contextual setup)
   - System description (1–2 paragraphs)
   - In-scope / out-of-scope (explicit list)
   - Assets (the things worth protecting; don't pad — see anti-pattern note)
   - Trust levels (who has what access)
   - Assumptions (numbered, so they can be challenged)
   - Data Flow Diagram (Mermaid, see §DFD conventions)
   - Data of interest, if a data-centric supplement is in the contextual stratum
     (the data class, security objectives in scope, authorized locations — see
     `references/data-centric.md`)

## 2. What can go wrong?  (Hybrid threat enumeration — three strata)

   ### 2.1 Contextual stratum (system-specific)
   - 2.1.a  Flow-centric STRIDE-Per-Element threat table (always — see §Threat enumeration)
   - 2.1.b  Supplementary entry-point pass(es): pick from data-centric / asset-centric /
            user-needs-centric / process-centric / code-centric per the system-type
            matrix in `methodologies.md` (always at least one)
   - 2.1.c  LINDDUN privacy table if PII/PHI is in scope; AI/ML-specific threat list
            if ML components are present
   - 2.1.d  Threat tree(s) for the top 1–2 highest-value threats (optional)

   ### 2.2 Operational / Tactical stratum (generic adversary techniques + design-time chain)
   - **STRIDE → CAPEC → CWE → mitigation chain on top threats (always — this is the design-time payoff)**.
     Each row of the operational table cites: a CAPEC pattern (with abstraction level — Meta for early
     architecture, Standard for design review, Detailed for component-level), the CWE(s) the pattern
     exploits, and the resulting mitigation class. The CAPEC → CWE bridge is what makes the
     derived requirements (`SR-###` in §3) traceable to a known weakness class rather than to free-text
     threat sentences. Full mapping by STRIDE category, abstraction-level rule of thumb, and
     medical-device subset: `references/capec.md`.
   - ATT&CK technique IDs on top threats (always — even if just the top 3)
   - Cyber Kill Chain mapping where it clarifies sequencing (optional)
   - CVSS for any threats that map to known CVEs (optional)

   ### 2.3 Strategic stratum (sector landscape)
   - Sector ISAC / threat-intel pointers (H-ISAC for medical, FS-ISAC for finance,
     E-ISAC for energy, MS-ISAC for state/local gov, ICS-CERT/CISA for ICS)
   - Regulatory framing (FDA premarket cybersecurity, IEC 62443, IEC 81001-5-1,
     HIPAA, GDPR, PCI, EU AI Act, NIST AI RMF — whatever applies)
   - Named-adversary context where one applies; PASTA business-impact framing
     if executive sign-off is needed
   - One paragraph minimum, even for low-stakes systems. If genuinely nothing
     to add, say so explicitly: "Strategic: not applicable; <reason>."

## 3. What are we going to do about it?
   - Mitigation table covering threats from all three strata in one prioritized
     list — sharing one ID space and one risk-rating scale (qualitative L/M/H
     by default; OWASP-RR or FMEA where the org requires — see `risk-rating.md`)
   - Each row: threat ID → response (Mitigate/Eliminate/Transfer/Accept) → concrete control(s)
   - Cross-stratum references where the same finding surfaced multiple times
     (V3 ↔ T7 ↔ ATT&CK T1119 ↔ H-ISAC 2023-07) — never duplicate threats; cross-reference
   - Derived security requirements (numbered, so they're tractable for tracking
     in Polarion / Jira / GitHub Issues)

## 4. Did we do a good enough job?
   - Self-assessment checklist (covers all three strata — see §Self-assessment)
   - Open questions and assumptions still to validate
   - Recommended next review trigger (next feature, next architectural change, etc.)
```

ID conventions (to keep cross-stratum references unambiguous — one prefix per namespace, no collisions):
- Assets in §1 (the things worth protecting): `AS1`, `AS2`, ...
- Flow-centric STRIDE threats: `T1`, `T2`, ...
- Data-centric vectors: `V1`, `V2`, ...
- Asset-centric findings: `A1`, `A2`, ...
- Privacy / LINDDUN / AI-ML findings: `PR1`, `PR2`, ...
- Mitigations / derived requirements: `SR-001`, `SR-002`, ...
- ATT&CK / CAPEC / CWE / CVE references: cite the upstream ID directly

### DFD conventions

Use Mermaid `flowchart LR` (or `TD`). Map DFD elements as follows; full details and a worked example in `references/dfd-mermaid.md`:

- **External entity** → rectangle: `EE[User]`
- **Process** → rounded rectangle (or circle via `(( ))`): `P1(Auth Service)`
- **Data store** → cylinder: `DS[(User DB)]`
- **Data flow** → labeled arrow: `EE -- "credentials" --> P1`
- **Trust boundary** → Mermaid `subgraph` with a **dashed border** (Mermaid's solid subgraph border reads as containment, not as a trust crossing — see `references/dfd-mermaid.md` § "Rendering trust boundaries as dashed subgraphs" for the `style ... stroke-dasharray` pattern that every diagram must use). Subgraphs carry a structured label that names environment owner, environment type, and trust level: `subgraph HospNet["Hospital IT | on-prem enterprise | moderate trust"]`. The per-environment patterns that should drive *which* subgraphs you draw (Purdue levels for OT, app sandbox / keychain for mobile, secure-boot / JTAG for embedded, account / VPC / IAM for cloud, AD forest / SSO / jump-host for enterprise) live in `references/environments.md` — read it after the system-type matrix when drawing the DFD.

Three rules that govern what goes in the DFD, applied in order:

1. **Every element belongs to exactly one zone.** External entities, processes, data stores, smart pendants, e-stop chains, physical actors (patients, operators) — all of them. A floating element with no zone label is a missing trust boundary. If an external entity is genuinely outside any controlled zone (a patient in the room, an attacker on the public internet), put it in an explicit `Untrusted` or `Unowned | <env-type> | untrusted` subgraph rather than leaving it floating.
2. **Use nested subgraphs for trust-boundaries-within-trust-boundaries.** Mermaid supports nesting; the skill requires it where the system has them. The canonical case: a single device that runs two operating systems with a documented privilege boundary between them (e.g. a smart infusion pump's Linux comms / HMI side ↔ RTOS pump-control side) is one outer subgraph containing two inner subgraphs, not one flat zone. Same for: cloud-account / VPC / subnet (three nests), Windows host / VM / container, embedded device / secure element, mobile device / app sandbox / hardware keystore. See `references/dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries" for the pattern.
3. **Reconcile the DFD against §1 before moving to threat enumeration.** Every asset listed in §1 and every trust boundary mentioned in §1 prose must appear in the DFD as a drawable element or as a subgraph border respectively. If §1 names something the DFD doesn't show, the DFD is incomplete — fix the DFD, don't quietly drop the element. This is the most common failure mode: details in §1 get cut on the way to the diagram.

Keep the DFD at the right level of abstraction. A single diagram with 5–15 elements is usually right; if the system needs more, produce a Level-0 (context) diagram and one or more Level-1 (decomposed) diagrams rather than one giant unreadable mess. Multiple representations is a Manifesto pattern, not a failure.

Symbol simplification follows Shostack's DFD3 convention (external entity, process, data store, data flow, trust boundary — drop the multi-process shape unless decomposition is genuinely needed).

### Threat enumeration

The contextual core of the hybrid (§2.1.a in the document structure above) is **STRIDE-Per-Element on the DFD**. The other contextual passes (data-centric, asset-centric, user-needs-centric, process-centric, code-centric — at least one is always added per the system-type matrix) plus the operational stratum (ATT&CK and friends) and the strategic stratum (sector landscape) build out from there. This subsection covers the contextual core; the rest is in `references/methodologies.md` and `references/data-centric.md`.

For each DFD element, run through the STRIDE categories that apply to that element type. The applicability table (which categories apply to each of External entity / Process / Data flow / Data store), the category prompts, per-element example threats, the **element-is-the-victim** framing rule, and the **Per-Element vs Per-Interaction** choice all live in `references/stride-prompts.md` — **read it before enumerating threats**. Write at least one concrete threat per applicable cell ("An attacker on the public internet sends crafted DICOM C-STORE requests to the PACS to overflow the parser" — not "DoS").

A reasonable exit criterion: at least one threat per check-marked cell in the applicability table. If you also circle back to consider threats against your *mitigations*, you're doing pretty well.

Threat table format:

| ID | Element | STRIDE | Threat | Likelihood | Impact | Risk |
|----|---------|--------|--------|------------|--------|------|
| T1 | Auth Service (P1) | S | Attacker reuses captured session token across boundary | M | H | High |

Use qualitative ratings (L/M/H) by default. For safety-critical systems (medical devices, automotive, industrial control), see `references/risk-rating.md` for the impact-bumping rule before assigning ratings — patient/operator/public safety in play almost always means High impact, regardless of likelihood. Avoid DREAD — OWASP and Shostack both note its scoring is too subjective. If the user asks for quantitative scoring, use OWASP Risk Rating Methodology (linked in `references/risk-rating.md`).

For the highest-risk handful of threats (top 3–5), consider drawing a threat tree showing how the threat could be realized (root cause analysis). One Mermaid tree per top threat is plenty.

### Tactics for enumerating threats efficiently

These come from Shostack's practical guidance and are worth following:

- **Top-down, breadth-first.** Start from the highest-level view of the whole system. Iterate breadth-first across elements rather than depth-first into one component. Bottom-up threat modeling — building per-feature models and trying to aggregate them — does not produce a coherent system view, and is a known failure mode.
- **Pick an iteration axis.** Three valid options: walk **across trust boundaries** (highest-value threats live there), walk **across diagram elements** (good when multiple sub-teams collaborate), or walk **across the threats** themselves ("where are all the spoofing threats?" — good for finding related threats). Pick one; don't let the choice become a straightjacket. If unsure, start at trust boundaries.
- **Start with external entities** (or with whatever crosses the outermost trust boundary). It's a natural anchor for "where could an attacker reach in?"
- **Never skip a threat because it's not what you're looking for right now.** If you're walking Spoofing and notice a Tampering issue, write it down. Redundancy is fine.
- **Focus on feasible threats.** "Someone might insert a backdoor at the chip factory" is a real possibility, but for most systems it's not a productive threat to model. Stay where the team can act.
- **Threats follow data flow, not control flow** — and they cluster around trust boundaries *and* complex parsing. The "trust boundary" rule is a starting point, not the limit; complex parsers (DICOM PDUs, image decoders, deserializers) are threat-rich even inside a trust zone.
- **Five threats per diagram element is a reasonable rule of thumb** when using DFD + STRIDE. Significantly fewer probably means an element was under-analyzed; significantly more probably means redundant threats that can be combined.
- **Document assumptions as you go**, not upfront. When someone says "I assume that…" — write it down right then, in falsifiable form. The assumption list then becomes test cases, not pedantry. (More in `references/validation.md`.)
- **Threats become bugs.** Every recorded threat should end up as a tracked work item in whatever bug/issue system the team uses. This is the exit point from threat modeling — once threats are bugs, normal triage and engineering machinery takes over.

### Mitigations and responses

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a concrete control mapped to the violated property — the STRIDE → security property → typical mitigations table lives in `references/stride-prompts.md` § "STRIDE → security property → typical mitigations".

Then derive security requirements from the mitigations. Each requirement should be testable. Format:

```
SR-001: The system SHALL authenticate all DICOM associations using mutual TLS with X.509 certificates issued by <CA>.
   Mitigates: T3, T7
```

Numbered IDs make these importable into requirements management tools.

### Self-assessment (Q4)

Q4 is the most-skipped step in real threat models — the Manifesto's "admiration for the problem" anti-pattern. Don't skip it. Q4 has three checks, not one:

1. **Model/reality conformance** — does the diagram match what was actually built or planned?
2. **Task and process completion** — does every threat have a recorded response (Mitigate / Eliminate / Transfer / Accept), and is each tracked in the team's bug tracker?
3. **Bug checking** — as the work proceeds, are the threat-derived bugs actually being closed with mitigations tested?

Full guidance — including Shostack's three-checklist approach, pen-testing-as-complement (black-box vs glass-box), model/reality conformance after architectural drift, and how to handle threat models for acquired/third-party code — is in `references/validation.md`. Read it when producing a thorough Q4 section.

The minimum viable Q4 section in the output document includes:

- **Diagramming checklist** — see `references/validation.md` § "Diagramming checklist".
- **Threats checklists** — by stratum (contextual / operational / strategic / cross-stratum) — see `references/validation.md` § "Threats checklists (hybrid)".
- **Validating-threats checklist** — see `references/validation.md` § "Validating-threats checklist".
- **Open questions / to validate** — anything unresolved, including assumptions that need testing.
- **Next review trigger** — the event that will require re-modeling (e.g. "before the cloud upload feature ships," "next architecture revamp," "annually," "if the deployment topology changes").

The blank template in `assets/threat-model-template.md` carries all three checklists pre-filled — copy from there.

## Manifesto patterns and anti-patterns

The Threat Modeling Manifesto's patterns and anti-patterns shape what this skill should and shouldn't produce. The full list (Systematic approach, Informed creativity, Varied viewpoints, Useful toolkit, Theory into practice, Multiple representations / Hero threat modeler, Admiration for the problem, Tendency to overfocus, Perfect representation, Asset rabbit-holing) and the skill-output gloss on each live in `references/manifesto.md`. Two are easy to slip into and worth keeping in mind throughout: **Admiration for the problem** (every threat must get a response) and **Asset rabbit-holing** (don't pad the asset list).

## When the contextual core needs supplementing

The "default = single-method, swap if needed" framing is gone. The question is no longer *"do I swap STRIDE / flow-centric for something else?"* — the answer is almost always no for the contextual core. The one documented exception is **STPA-SafeSec** for safety-critical control-loop systems where the worst case is physical harm (medical devices with control loops, ICS / SCADA / OT, automotive, aerospace, robotics) — and even there it has two modes (full swap when regulators require the joint safety+security artifact; supplement-alongside-STRIDE when there's an existing threat-model practice or substantial non-control attack surface). See `references/stpa-safesec.md` § "Two modes" for the mode-selection guidance. Otherwise, the right question is *"which additional layers does the system in front of me need in its hybrid?"*. Use the applicability matrix in `references/methodologies.md` § "Decision matrix — which layers, by system type" as the starting point.

Quick reference for the most common supplements:

- **Data-centric (NIST SP 800-154)** — add when one specific data type dominates or regulatory framing is data-typed (HIPAA / GDPR / PCI / FDA). Workflow and worked example: `references/data-centric.md`.
- **LINDDUN** — add when PII/PHI is in scope or privacy is a stated concern. Privacy-specific categories (Linking, Identifying, Detecting, Unawareness, Non-compliance) that STRIDE under-covers.
- **User-needs-centric (abuse-case inversion)** — add for rich business logic, multi-tenant SaaS, anything with delegation/approval flows.
- **Process-centric** — add when operational surface is significant (regular deploys, on-call rotations, key/credential rotations, manual approval steps).
- **Asset-centric** — add when there are clear crown-jewel assets (signing keys, KMS roots, control-loop setpoints).
- **Attack tree** — add on the top 1–2 highest-value threats, when adversarial reasoning about a specific target adds value. Generally not as a primary method — see `references/methodologies.md` § Attack trees.
- **AI/ML threat list** — add for any system with ML components (prompt injection, training-data poisoning, model extraction, etc.). OWASP LLM Top 10 / OWASP ML Security Top 10 are the references.
- **PASTA framing** — borrow the business-impact stage when executive sign-off is needed. Don't run the full 7-stage process unless that's the explicit ask.
- **OCTAVE / VAST** — only at org/portfolio level, not per-system.
- **STPA-SafeSec (swap or supplement)** — for safety-critical control-loop systems where the worst case is physical harm. Two modes: **swap** the contextual core (one joint safety+security artifact — use when regulators require it: FDA premarket cybersec, IEC 62304, IEC 81001-5-1, IEC 61508, ISO 26262), or run as a **supplement** alongside flow-centric + STRIDE (adds hazard scenarios that cross-reference threats and capture non-adversarial safety failures STRIDE misses — use when there's an existing threat-model artifact or substantial non-control attack surface). Mode-selection guidance and workflow: `references/stpa-safesec.md`.

The operational stratum (ATT&CK, kill chain) and strategic stratum (sector ISAC, regulatory framing) are also always added, not "swapped in"; they're sized to the system's stakes but always present. See the document structure above and the matrix in `methodologies.md`.

## Domain notes

A few domain-specific gotchas worth surfacing when relevant:

- **Medical devices** — Safety implications are first-class. Model the patient as an asset (in addition to data). The default hybrid for a medical device or PACS pulls in: flow-centric DFD + STRIDE (contextual core), **data-centric on PHI / device data** (workflow and worked example in `references/data-centric.md`), LINDDUN privacy pass, asset-centric on signing / device keys, optional attack tree on the safety-critical path; ATT&CK + CISA medical advisories at the operational stratum; H-ISAC + FDA premarket cybersecurity + IEC 62443 + IEC 81001-5-1 + HIPAA + safety-bump rule from `risk-rating.md` at the strategic stratum. Consider clinical workflow misuse scenarios, not just network threats. **For devices with safety-critical control loops (infusion pumps, ventilators, robotic surgery, dialysis machines), add STPA-SafeSec — either as a contextual-core swap (one joint FDA / IEC 62304 / IEC 81001-5-1 artifact) or as a supplement alongside STRIDE (adds safety-failure coverage and prioritizes which threats matter for safety outcomes). See `references/stpa-safesec.md` § "Two modes".** **For multi-device clinical environments — Internet of Medical Things (IoMT), e.g. a patient room with infusion pump + monitor + Wi-Fi AP + nurse-call peripheral + clinician phone — element-by-element STRIDE often misses multi-hop cyber-physical attack paths where each individual device looks fine but their composition over P2/P3 hops reaches a safety-critical target. Use the risk-prioritized attack-path construction in `references/methodologies.md` § "Risk-prioritized cyber-physical attack paths".** The matrix in `references/methodologies.md` § "Decision matrix" has the full list.
- **IoT / embedded** — Physical attack surface (debug ports, firmware extraction, side channels) belongs in the DFD. The "trust boundary" between the device and the network is often the most consequential. **When several devices coexist in one physical space, classify the *between-device* surface as P1 (direct contact — USB, JTAG, attached cable), P2 (proximity — BLE/NFC reach, line-of-sight IR, audible-range acoustic, near-field EM), or P3 (shared physical medium without proximity — e.g. the 2.4 GHz ISM band shared across Wi-Fi / BLE / Zigbee in a hospital room, where a rogue BLE peripheral can DoS a Wi-Fi-attached infusion pump without ever associating with it; or shared power, shared HVAC for acoustic). The P-hops drive the cyber-physical attack-path construction in `references/methodologies.md` § "Risk-prioritized cyber-physical attack paths".** For control-loop devices (automotive ECUs, robotics, drones, industrial actuators), add STPA-SafeSec — swap or supplement, depending on whether ISO 26262 / IEC 61508 require the joint artifact. See `references/stpa-safesec.md`.
- **Cloud-native** — Shared responsibility model matters. Document which controls are the cloud provider's vs. yours. IAM and managed services are usually under-modeled.
- **AI/ML systems** — STRIDE still works but extend with model-specific threats: prompt injection, training-data poisoning, model extraction, membership inference, model inversion, adversarial examples, supply-chain on pre-trained weights. The full list and OWASP LLM Top 10 / ML Security Top 10 pointers are in `references/methodologies.md` §ML/AI.
- **Acquired or third-party code** — when modeling components the team didn't write, see `references/validation.md` §"Validating threats addressed in third-party / acquired code" for the inside-out workflow (enumerate accounts, listening ports, admin interfaces, platform changes). Especially relevant for medical-device work where vendor components are ubiquitous.

## References

Detailed reference content lives in `references/`. Read these on demand:

- `references/methodologies.md` — **Hybrid as default**: the three strata (Contextual / Operational / Strategic), the layer catalog populating each, the system-type applicability matrix, single-document vs linked-documents output shapes, pruning rules. Also: when to use LINDDUN, PASTA, attack trees, MITRE ATT&CK, kill chains, OCTAVE, VAST. **Read this first** — it tells you which layers the system in front of you needs.
- `references/environments.md` — **Per-environment trust-boundary patterns**: the boundaries you should expect to draw for each of cloud, on-prem enterprise, embedded / IoT, OT / ICS, and mobile, plus an ownership taxonomy that generalizes cloud's shared-responsibility model. Read this when drawing the DFD — the generic list in `dfd-mermaid.md` § "Trust boundaries — what to include" is the floor; this is the ceiling.
- `references/centric-methods.md` — Entry-point taxonomy: asset-centric, data-centric (NIST SP 800-154), flow-centric, process-centric, user-needs-centric, attacker-centric, code-centric. When to pick each, and the generation-vs-characterization distinction.
- `references/data-centric.md` — Data-centric workflow deep-dive: the four NIST 800-154 steps, the three modes for acquiring the data scope (Mode A user-volunteers / Mode B skill-enumerates / Mode C skill-infers-from-artifact), the per-data-class scoping rule, the STRIDE-against-locations caveat, the hybrid bridge to flow-centric, and a worked DICOM example.
- `references/capec.md` — CAPEC working reference for the operational stratum: the **STRIDE → CAPEC → CWE → mitigation chain** that makes derived requirements traceable, the three abstraction levels (Meta / Standard / Detailed) and an SDLC-stage rule of thumb, the two CAPEC views (Mechanisms-of-Attack CAPEC-1000 vs Domains-of-Attack CAPEC-3000), STRIDE-to-CAPEC mapping tables with CWE chains, the medical-device / DICOM / embedded subset, and an honest note on CAPEC's coverage gaps for medical and ICS protocols.
- `references/stride-prompts.md` — STRIDE category prompts and example threats per element type, plus STRIDE-Per-Element vs Per-Interaction guidance and the DESIST variant.
- `references/dfd-mermaid.md` — DFD-to-Mermaid mapping conventions with worked examples, Shostack-derived diagramming rules of thumb, and the diagramming checklist.
- `references/validation.md` — Q4 deep dive: model/reality conformance, the three Shostack-style checklists, pen testing as complement (not substitute), validating threats in acquired/third-party code, document-assumptions-as-you-go.
- `references/manifesto.md` — Threat Modeling Manifesto values, principles, patterns, anti-patterns (paraphrased).
- `references/risk-rating.md` — Qualitative L/M/H risk rating (used across strata in the hybrid), OWASP Risk Rating Methodology pointer, why DREAD is discouraged.
- `references/stpa-safesec.md` — **STPA-SafeSec deep dive**: safety+security joint hazard analysis for control-loop systems. Two modes — **swap** the contextual core (joint artifact for FDA / IEC 62304 / IEC 81001-5-1 / IEC 61508 / ISO 26262) or **supplement** alongside flow-centric + STRIDE (cross-references threats, adds non-adversarial safety failures). Losses → hazards → constraints → control layer → component layer → UCAs → system flaws → hazard-scenario trees → mitigations + requirements. Use when the worst case is physical harm and the system is shaped as a control loop (medical devices, ICS / SCADA / OT, automotive, aerospace, robotics).

A single blank threat model template — laid out as a hybrid with the three strata as nested sections — is in `assets/threat-model-template.md`. There is intentionally only one template; data-centric, asset-centric, etc. are entry-point passes that populate sub-sections of this template, not separate documents.

## Citation note

Concepts in this skill are paraphrased from open OWASP material (Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, OWASP Security Culture v1.0 §6, OWASP Threat Modeling Playbook by Toreon), the Threat Modeling Manifesto (CC-BY 4.0), and Adam Shostack's *Threat Modeling: Designing for Security* (Wiley, 2014) — particularly the Four Question Framework, STRIDE-Per-Element, the diagramming and validation checklists, and the practical guidance on iteration, assumption documentation, and bug-filing as the exit point. The data-centric methodology and the authorized-data-location framing are from NIST SP 800-154 (Draft), *Guide to Data-Centric System Threat Modeling* (Murugiah Souppaya, Karen Scarfone; NIST, March 2016) — public-domain U.S. government work. The three-stratum hybrid framing (Contextual / Operational / Strategic) and the "no single approach covers all permutations" rationale are from Tatam, Shanmugam, Azam & Kannoorpatti, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021) — open-access, CC-BY 4.0. MITRE ATT&CK, CAPEC, and CWE are MITRE Corporation; CAPEC's three-abstraction-level structure (Meta / Standard / Detailed) and the two top-level views (Mechanisms of Attack CAPEC-1000, Domains of Attack CAPEC-3000) are from CAPEC's published schema. The point that CAPEC's CAPEC → CWE chain is the design-time payoff CAPEC offers over ATT&CK is paraphrased from Tatam et al. (above). The Cyber Kill Chain is Lockheed Martin. STRIDE itself originates from Microsoft (Loren Kohnfelder and Praerit Garg). DESIST is by Gunnar Peterson. The STPA-SafeSec methodology and the integration of safety and security into one workflow with a component-layer mapping are from Friedberg, McLaughlin, Smith, Laverty & Sezer, *STPA-SafeSec: Safety and security analysis for cyber-physical systems*, Journal of Information Security and Applications 34 (2017) — CC-BY. The underlying STPA-Sec framing is from Young & Leveson, *An Integrated Approach to Safety and Security Based on Systems Theory*, CACM 57(2) (2014). The classic STPA causal-factor taxonomy is from Leveson, *Engineering a Safer World* (MIT Press, 2011), and the state-space approach to UCA enumeration is from Thomas, *Extending and automating a systems-theoretic hazard analysis for requirements generation and analysis* (PhD thesis, MIT, 2013). The risk-prioritized cyber-physical attack-path construction — target-rooted source-driven walk, P1/P2/P3 between-device physical-surface taxonomy, CVV (CVSS-derived) scoring, and risk-based pruning before layer expansion — is from Stellios, Kotzanikolaou & Grigoriadis, *Assessing IoT-enabled cyber-physical attack paths against critical systems*, Computers & Security 107 (2021); the unpruned IoT attack-graph baseline it extends is from Agadakos, Chen, Campanelli, Anantharaman, Hasan, Copos, Lepoint, Locasto, Ciocarlie & Lindqvist, *Jumping the air gap: Modeling cyber-physical attack paths in the Internet-of-Things*, CPS-SPC '17 (2017). Cite these sources in any output that quotes or substantively summarizes them.
