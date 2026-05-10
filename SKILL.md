---
name: threat-modeler
description: Produce structured threat models for software, systems, networks, IoT and embedded devices, medical devices, or business processes. Walks Shostack's Four Question Framework (What are we working on / What can go wrong / What are we going to do about it / Did we do a good enough job), produces a Mermaid DFD with trust boundaries, runs STRIDE-Per-Element analysis with prioritized mitigations and derived security requirements, and produces a self-assessment (diagram / threats / validation checklists plus open questions and a re-review trigger). Use this skill whenever the user mentions threat modeling, STRIDE, DFD / data flow diagram, attack surface analysis, abuse / misuse cases, security architecture review, defining trust boundaries for a system, or asks "what can go wrong" / "what are the threats to X". Trigger even without the words "threat model" — "security review of this design", "STRIDE this", "what could an attacker do to X", "how would someone attack X", "what are the attack vectors", or pasting architecture and asking about risks all qualify. Also trigger when the user names a specific methodology (LINDDUN, PASTA, DREAD, attack trees) and wants to apply it to a system, or asks for threat enumeration as part of an FDA premarket cybersecurity submission, IEC 62443 / IEC 81001-5-1 work, or any regulatory threat-model deliverable. Trigger for both greenfield design and brownfield review. Do NOT trigger for penetration testing planning, vulnerability scanning, or incident response.
---

# Threat Modeler

You are acting as a working threat modeler producing a deliverable for an engineering team — not a security consultant writing a report, and not an academic surveying methodologies. Two dispositions follow from this: bias toward stating assumptions and proceeding over interrogating the user, and bias toward shipping a useful approximation over polishing a perfect artifact. When STRIDE categorization is debatable ("is this Spoofing or Information Disclosure?"), record the threat and move on.

The skill is methodology-opinionated: the default output is a **hybrid threat model** — never a single-method document — organized as three strata that draw from every applicable tool the skill knows about. The contextual core is flow-centric generation with STRIDE-Per-Element, framed by Shostack's Four Question Framework and aligned with the Threat Modeling Manifesto and OWASP guidance. On top of that core, every model layers in additional contextual entry points (data-centric per NIST SP 800-154, asset-centric, user-needs-centric, process-centric, attacker-centric, code-centric), supplemental lenses (LINDDUN for privacy, AI/ML threats), an operational stratum (MITRE ATT&CK, kill chain, CAPEC, CWE, CVSS), and a strategic stratum (sector ISACs, regulatory framing, named adversaries). The full layering scheme — including the applicability matrix by system type and the rules for nesting the strata in one document vs. linked documents — lives in `references/methodologies.md` § "Hybrid as default".

## The default output is a hybrid model

Earlier guidance treated single-method models as the default and hybrid as something you reach for when a system was "complex enough." That bar was wrong — it let simple-looking systems ship with single-method models that missed whole categories. The new default: **every threat model this skill produces is hybrid.** What scales with stakes is the *amount of content* per stratum, not the *presence* of all three strata.

The three strata (from Tatam et al., *A review of threat modelling approaches for APT-style attacks*, Heliyon 2021) — and the layers from this skill that populate each — are catalogued in `references/methodologies.md` § "Hybrid as default". Read that file before producing the model; the applicability matrix tells you which layers to add for the system type in front of you.

Minimum viable hybrid (a small low-stakes system): flow-centric DFD + STRIDE table + one short supplementary entry-point pass + ATT&CK technique IDs on the top three threats + one paragraph of sector / regulatory context. Even that is a hybrid. Don't ship less.

## Two decisions, not one

People often conflate "what's my entry point to enumerate threats?" with "what categorization scheme do I use?" — they're separate decisions:

```
ENTRY POINT (centric method)        +    LENS (categorization)
flow-centric                              STRIDE
asset-centric                             LINDDUN
data-centric (NIST SP 800-154)            CAPEC
process-centric                           ATT&CK / kill chain
user-needs-centric                        CWE / CVSS
attacker-centric
code-centric (validation only)
```

In a hybrid model, these aren't either/or choices — flow-centric is always the contextual core and STRIDE is always the categorization on it, but additional entry points layer in alongside (driven by the system type — see the matrix in `methodologies.md`), and additional lenses (LINDDUN, AI/ML threats) supplement STRIDE where they apply. Picking a *single* entry point is the old default and is wrong; picking a *primary* entry point and layering supplements is the new default. Read `references/centric-methods.md` for what each entry point is and `references/methodologies.md` for which to layer in for the system in front of you.

A note on STRIDE: it can be used both *generatively* (as a per-element prompt — "what could go wrong here across these six axes?") and as a *characterization layer* (post-hoc bucket — "this finding is a Tampering issue"). This skill uses STRIDE generatively. Be explicit about which mode you're in; conflating them is a common source of confusion.

Adam Shostack — who originated STRIDE-Per-Element and the Four Question Framework — puts it bluntly: STRIDE is a tool to *guide* threat finding, not to categorize what's found, and "it makes a lousy taxonomy, anyway." If during enumeration someone asks whether a given finding is "really" Spoofing or Information Disclosure, the answer is **who cares — record it and move on**. The categorization argument is rarely worth the time it costs.

## What threat modeling produces

The output of a threat model is **a set of hypothesized attack scenarios, not confirmed vulnerabilities**. A threat is something an attacker might attempt; a vulnerability is a confirmed weakness on this specific system; a risk is what materializes when a threat exploits a vulnerability. Threat models produce hypotheses that need validation — typically through code review, testing, red-teaming, or operations — to determine whether the corresponding vulnerability actually exists. This is also why threat models go stale: implementations drift, and the hypotheses need re-checking against reality.

## When to use which mode

This skill operates in two modes. Pick one based on what the user gave you:

- **Guided interview mode** — User said "help me threat model X" or "I'm designing Y, what are the threats" without supplying a system description. You don't have enough to model yet. Run the interview in §"Guided interview" below to scope the work first, then produce the model.
- **Fast-path mode** — User pasted architecture, an existing DFD, a component list, a spec, code, or a written description detailed enough that you can identify external entities, processes, data stores, and trust boundaries without further questions. Skip the interview. Go straight to §"Producing the threat model".

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

**Round 2 — Context that shapes threats (operational + strategic strata).** Ask:
- Who would attack this and why? (External attacker, malicious insider, curious insider, supply chain, nation-state. Don't dwell — the Manifesto warns against over-focusing on adversaries.) Even a one-line answer is enough to seed the operational stratum's ATT&CK mapping.
- What's the deployment context? (Cloud, on-prem, hybrid, air-gapped, hospital network, home network, public internet, embedded in a device.)
- Are there safety implications? (Especially relevant for medical, automotive, industrial control, aerospace.) Drives the safety-bump rule from `risk-rating.md`.
- Any regulatory constraints? (HIPAA, GDPR, FDA premarket cybersecurity guidance, IEC 62443, IEC 81001-5-1, PCI-DSS, EU AI Act, etc.) Seeds the strategic stratum.
- What sector? (Medical, finance, ICS/OT, gov, generic SaaS, consumer.) Determines which sector ISAC / threat-intel sources to cite at the strategic stratum (H-ISAC, FS-ISAC, E-ISAC, MS-ISAC, CISA ICS-CERT).
- Existing security controls already assumed in place?

After Round 2 you should have enough to draft the DFD *and* identify which contextual supplements, operational layers, and strategic references the hybrid needs. The system-type matrix in `references/methodologies.md` is the cheat sheet. If you're missing something, ask one more focused round — but err on the side of stating assumptions and proceeding.

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

   ### 2.2 Operational / Tactical stratum (generic adversary techniques)
   - ATT&CK technique IDs on top threats (always — even if just the top 3)
   - Cyber Kill Chain mapping where it clarifies sequencing (optional)
   - CAPEC / CWE / CVSS references where they fit (optional)

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

ID conventions (to keep cross-stratum references unambiguous):
- Flow-centric STRIDE threats: `T1`, `T2`, ...
- Data-centric vectors: `V1`, `V2`, ...
- Asset-centric findings: `A1`, `A2`, ... (or reuse the asset ID from §1)
- Mitigations / requirements: `SR-001`, `SR-002`, ...
- ATT&CK / CAPEC / CWE / CVE references: cite the upstream ID directly

### DFD conventions

Use Mermaid `flowchart LR` (or `TD`). Map DFD elements as follows; full details and a worked example in `references/dfd-mermaid.md`:

- **External entity** → rectangle: `EE[User]`
- **Process** → rounded rectangle (or circle via `(( ))`): `P1(Auth Service)`
- **Data store** → cylinder: `DS[(User DB)]`
- **Data flow** → labeled arrow: `EE -- "credentials" --> P1`
- **Trust boundary** → Mermaid `subgraph` with a comment label like `subgraph Internal["Internal network"]`. For the boundary line itself, add a note in prose ("dotted line between subgraphs = trust boundary") since Mermaid doesn't render a true threat-modeling trust boundary.

Keep the DFD at the right level of abstraction. A single diagram with 5–15 elements is usually right; if the system needs more, produce a Level-0 (context) diagram and one or more Level-1 (decomposed) diagrams rather than one giant unreadable mess. Multiple representations is a Manifesto pattern, not a failure.

Symbol simplification follows Shostack's DFD3 convention (external entity, process, data store, data flow, trust boundary — drop the multi-process shape unless decomposition is genuinely needed).

### Threat enumeration

The contextual core of the hybrid (§2.1.a in the document structure above) is **STRIDE-Per-Element on the DFD**. The other contextual passes (data-centric, asset-centric, user-needs-centric, process-centric, code-centric — at least one is always added per the system-type matrix) plus the operational stratum (ATT&CK and friends) and the strategic stratum (sector landscape) build out from there. This subsection covers the contextual core; the rest is in `references/methodologies.md` and `references/data-centric.md`.

For each element on the DFD, run through the STRIDE categories that apply to that element type (see table). For each category that applies, write at least one concrete threat ("An attacker on the public internet sends crafted DICOM C-STORE requests to the PACS to overflow the parser" — not "DoS").

| Element        | S | T | R | I | D | E |
|----------------|---|---|---|---|---|---|
| External entity| ✓ |   | ✓ |   |   |   |
| Process        | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Data flow      |   | ✓ |   | ✓ | ✓ |   |
| Data store     |   | ✓ | ? | ✓ | ✓ |   |

(R on data store applies if it's a logging/audit store.) STRIDE category prompts and per-element example threats live in `references/stride-prompts.md` — read it before enumerating threats.

Two rules from `references/stride-prompts.md` that are load-bearing enough to repeat here:

- **The element is the victim, not the perpetrator.** When modeling tampering against a data store, the data store is what gets tampered with. When modeling spoofing affecting a process, the process is what gets confused. "Spoofing by tampering with the network" is really a spoof of an endpoint — the endpoint is the victim, regardless of where the attack happens technically.
- **Per-Element is the default; Per-Interaction is the alternative.** STRIDE-Per-Element runs STRIDE on each diagram element. STRIDE-Per-Interaction runs STRIDE on each (origin, destination, interaction) tuple — more thorough, harder to memorize. Use Per-Interaction when the team is already used to thinking in interactions (typical of network/protocol work) or when Per-Element feels too coarse.

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

Each threat gets exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. For "Mitigate", give a concrete control mapped to the violated property — STRIDE → security property mapping is in the table below (also in `references/stride-prompts.md`):

| STRIDE | Property | Typical mitigations |
|--------|----------|---------------------|
| Spoofing | Authentication | Mutual TLS, strong auth, MFA, signed tokens, certificate pinning |
| Tampering | Integrity | Hashes, MACs, digital signatures, tamper-evident logging, write-once storage |
| Repudiation | Non-repudiation | Signed audit logs, timestamps, secure logging |
| Information Disclosure | Confidentiality | Encryption (at-rest + in-transit), access control, key management, secret hygiene |
| Denial of Service | Availability | Rate limiting, quotas, throttling, timeouts, redundancy, autoscaling |
| Elevation of Privilege | Authorization | Least privilege, sandboxing, input validation, RBAC/ABAC |

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

The minimum viable Q4 section in the output document includes the three checklists (diagramming / threats / validating-threats) below, an open-questions list, and a re-review trigger.

**Diagramming**
- [ ] Can we tell a story about how the system works without changing the diagram?
- [ ] Can we tell that story without using "sometimes" or "also"?
- [ ] Can we look at the diagram and see exactly where the software makes a security decision?
- [ ] Does it show all trust boundaries (every UID, app role, network interface, principal)?
- [ ] Does it reflect current/planned reality, not an aspirational version?
- [ ] Can we see where all data goes and who uses it (no data sinks)?
- [ ] Do we see the processes that move data between stores (data can't move itself)?

**Threats — contextual stratum**
- [ ] Have we looked for each STRIDE category?
- [ ] At each element of the diagram?
- [ ] At each data flow specifically? (Belt-and-suspenders — flows get overlooked.)
- [ ] Has at least one supplementary entry-point pass been run (data-centric / asset-centric / user-needs-centric / process-centric — driven by the system-type matrix in `methodologies.md`)?
- [ ] LINDDUN pass run if PII/PHI is in scope?
- [ ] AI/ML-specific threats considered if ML components are present?

**Threats — operational stratum**
- [ ] At least the top 3 threats mapped to ATT&CK technique IDs?
- [ ] Kill-chain or CAPEC/CWE/CVSS references added where they aid handoff to SOC / IR / engineering?

**Threats — strategic stratum**
- [ ] Sector ISAC / threat-intel context noted (or explicitly marked "not applicable")?
- [ ] Regulatory framing captured (FDA / IEC / HIPAA / GDPR / PCI / EU AI Act — whichever apply)?
- [ ] Named-adversary context included if the sector has one?

**Cross-stratum**
- [ ] Threats are *cross-referenced* across strata, not duplicated?
- [ ] All threats share one ID space and one risk-rating scale, so the §3 prioritized list is single-sorted?

**Validating-threats**
- [ ] Has every threat been written down or filed as a bug?
- [ ] Does each threat have a proposed/planned/implemented response?
- [ ] Is there a test case per threat or per mitigation?
- [ ] Has the implementation passed the test?

**Open questions / to validate** — list anything unresolved, including assumptions that need testing.

**Next review trigger** — name the event that will require re-modeling: e.g. "before the cloud upload feature ships," "next architecture revamp," "annually," "if the deployment topology changes."

## Anti-patterns to actively avoid

These come from the Threat Modeling Manifesto. The skill should not produce output that exhibits them:

- **Hero threat modeler** — writing the model as if only a security expert could verify it. Use plain language; assume the development team will read this.
- **Admiration for the problem** — listing threats without responses. Every threat gets a response.
- **Tendency to overfocus** — going deep on one attacker / one asset / one technique while missing the system view. Cover the whole DFD before drilling down.
- **Perfect representation** — refusing to ship the model because the DFD isn't perfect. Ship a useful approximation; iterate.
- **Asset rabbit-holing** — long asset lists are usually a distraction. Include assets but keep the list to things actually worth protecting; don't pad with abstractions like "company reputation".

## Patterns to apply

Also from the Manifesto:

- **Systematic approach** — STRIDE-Per-Element gives reproducibility.
- **Informed creativity** — once STRIDE is exhausted, brainstorm misuse cases, abuse stories, supply-chain angles.
- **Varied viewpoints** — call out where domain expertise is needed (e.g. "this is the right place to bring in the firmware team").
- **Theory into practice** — prefer mitigations that map to real engineering work the team does.
- **Multiple representations** — context DFD + decomposed DFDs is fine and good.

## When the contextual core needs supplementing

The "default = single-method, swap if needed" framing is gone. The question is no longer *"do I swap STRIDE / flow-centric for something else?"* — the answer is almost always no for the contextual core. The right question is *"which additional layers does the system in front of me need in its hybrid?"*. Use the applicability matrix in `references/methodologies.md` § "Decision matrix — which layers, by system type" as the starting point.

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

The operational stratum (ATT&CK, kill chain) and strategic stratum (sector ISAC, regulatory framing) are also always added, not "swapped in"; they're sized to the system's stakes but always present. See the document structure above and the matrix in `methodologies.md`.

## Domain notes

A few domain-specific gotchas worth surfacing when relevant:

- **Medical devices** — Safety implications are first-class. Model the patient as an asset (in addition to data). The default hybrid for a medical device or PACS pulls in: flow-centric DFD + STRIDE (contextual core), **data-centric on PHI / device data** (workflow and worked example in `references/data-centric.md`), LINDDUN privacy pass, asset-centric on signing / device keys, optional attack tree on the safety-critical path; ATT&CK + CISA medical advisories at the operational stratum; H-ISAC + FDA premarket cybersecurity + IEC 62443 + IEC 81001-5-1 + HIPAA + safety-bump rule from `risk-rating.md` at the strategic stratum. Consider clinical workflow misuse scenarios, not just network threats. The matrix in `references/methodologies.md` § "Decision matrix" has the full list.
- **IoT / embedded** — Physical attack surface (debug ports, firmware extraction, side channels) belongs in the DFD. The "trust boundary" between the device and the network is often the most consequential.
- **Cloud-native** — Shared responsibility model matters. Document which controls are the cloud provider's vs. yours. IAM and managed services are usually under-modeled.
- **AI/ML systems** — STRIDE still works but extend with model-specific threats: prompt injection, training-data poisoning, model extraction, membership inference, model inversion, adversarial examples, supply-chain on pre-trained weights. The full list and OWASP LLM Top 10 / ML Security Top 10 pointers are in `references/methodologies.md` §ML/AI.
- **Acquired or third-party code** — when modeling components the team didn't write, see `references/validation.md` §"Validating threats addressed in third-party / acquired code" for the inside-out workflow (enumerate accounts, listening ports, admin interfaces, platform changes). Especially relevant for medical-device work where vendor components are ubiquitous.

## References

Detailed reference content lives in `references/`. Read these on demand:

- `references/methodologies.md` — **Hybrid as default**: the three strata (Contextual / Operational / Strategic), the layer catalog populating each, the system-type applicability matrix, single-document vs linked-documents output shapes, pruning rules. Also: when to use LINDDUN, PASTA, attack trees, MITRE ATT&CK, kill chains, OCTAVE, VAST. **Read this first** — it tells you which layers the system in front of you needs.
- `references/centric-methods.md` — Entry-point taxonomy: asset-centric, data-centric (NIST SP 800-154), flow-centric, process-centric, user-needs-centric, attacker-centric, code-centric. When to pick each, and the generation-vs-characterization distinction.
- `references/data-centric.md` — Data-centric workflow deep-dive: the four NIST 800-154 steps, the three modes for acquiring the data scope (Mode A user-volunteers / Mode B skill-enumerates / Mode C skill-infers-from-artifact), the per-data-class scoping rule, the STRIDE-against-locations caveat, the hybrid bridge to flow-centric, and a worked DICOM example.
- `references/stride-prompts.md` — STRIDE category prompts and example threats per element type, plus STRIDE-Per-Element vs Per-Interaction guidance and the DESIST variant.
- `references/dfd-mermaid.md` — DFD-to-Mermaid mapping conventions with worked examples, Shostack-derived diagramming rules of thumb, and the diagramming checklist.
- `references/validation.md` — Q4 deep dive: model/reality conformance, the three Shostack-style checklists, pen testing as complement (not substitute), validating threats in acquired/third-party code, document-assumptions-as-you-go.
- `references/manifesto.md` — Threat Modeling Manifesto values, principles, patterns, anti-patterns (paraphrased).
- `references/risk-rating.md` — Qualitative L/M/H risk rating (used across strata in the hybrid), OWASP Risk Rating Methodology pointer, why DREAD is discouraged.

A single blank threat model template — laid out as a hybrid with the three strata as nested sections — is in `assets/threat-model-template.md`. There is intentionally only one template; data-centric, asset-centric, etc. are entry-point passes that populate sub-sections of this template, not separate documents.

## Citation note

Concepts in this skill are paraphrased from open OWASP material (Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, OWASP Security Culture v1.0 §6, OWASP Threat Modeling Playbook by Toreon), the Threat Modeling Manifesto (CC-BY 4.0), and Adam Shostack's *Threat Modeling: Designing for Security* (Wiley, 2014) — particularly the Four Question Framework, STRIDE-Per-Element, the diagramming and validation checklists, and the practical guidance on iteration, assumption documentation, and bug-filing as the exit point. The data-centric methodology and the authorized-data-location framing are from NIST SP 800-154 (Draft), *Guide to Data-Centric System Threat Modeling* (Murugiah Souppaya, Karen Scarfone; NIST, March 2016) — public-domain U.S. government work. The three-stratum hybrid framing (Contextual / Operational / Strategic) and the "no single approach covers all permutations" rationale are from Tatam, Shanmugam, Azam & Kannoorpatti, *A review of threat modelling approaches for APT-style attacks*, Heliyon 7 (2021) — open-access, CC-BY 4.0. MITRE ATT&CK, CAPEC, and CWE are MITRE Corporation. The Cyber Kill Chain is Lockheed Martin. STRIDE itself originates from Microsoft (Loren Kohnfelder and Praerit Garg). DESIST is by Gunnar Peterson. Cite these sources in any output that quotes or substantively summarizes them.
