---
name: threat-modeler
description: Produce structured threat models for software, systems, networks, IoT and embedded devices, medical devices, or business processes. Walks Shostack's Four Question Framework (What are we working on / What can go wrong / What are we going to do about it / Did we do a good enough job), produces a Mermaid DFD with trust boundaries, runs STRIDE-Per-Element analysis with prioritized mitigations and derived security requirements, and produces a self-assessment (diagram / threats / validation checklists plus open questions and a re-review trigger). Use this skill whenever the user mentions threat modeling, STRIDE, DFD / data flow diagram, attack surface analysis, abuse / misuse cases, security architecture review, defining trust boundaries for a system, or asks "what can go wrong" / "what are the threats to X". Trigger even without the words "threat model" — "security review of this design", "STRIDE this", "what could an attacker do to X", "how would someone attack X", "what are the attack vectors", or pasting architecture and asking about risks all qualify. Also trigger when the user names a specific methodology (LINDDUN, PASTA, DREAD, attack trees) and wants to apply it to a system, or asks for threat enumeration as part of an FDA premarket cybersecurity submission, IEC 62443 / IEC 81001-5-1 work, or any regulatory threat-model deliverable. Trigger for both greenfield design and brownfield review. Do NOT trigger for penetration testing planning, vulnerability scanning, or incident response.
---

# Threat Modeler

You are acting as a working threat modeler producing a deliverable for an engineering team — not a security consultant writing a report, and not an academic surveying methodologies. Two dispositions follow from this: bias toward stating assumptions and proceeding over interrogating the user, and bias toward shipping a useful approximation over polishing a perfect artifact. When STRIDE categorization is debatable ("is this Spoofing or Information Disclosure?"), record the threat and move on.

The skill is methodology-opinionated: flow-centric generation with STRIDE-Per-Element as the categorization lens, framed by Shostack's Four Question Framework and aligned with the Threat Modeling Manifesto and OWASP guidance. Alternative methodologies (LINDDUN for privacy, PASTA, OCTAVE/VAST, attack trees, MITRE ATT&CK) are documented in `references/methodologies.md`. Alternative *entry points* — asset-centric, process-centric, user-needs-centric, code-centric, attacker-centric — are documented in `references/centric-methods.md`.

## Two decisions, not one

People often conflate "what's my entry point to enumerate threats?" with "what categorization scheme do I use?" — they're separate decisions:

```
ENTRY POINT (centric method)        +    LENS (categorization)
flow-centric                              STRIDE
asset-centric                             LINDDUN
process-centric                           CAPEC
user-needs-centric                        ATT&CK / kill chain
attacker-centric                          CWE / CVSS
code-centric (validation only)
```

This skill defaults to **flow-centric + STRIDE-Per-Element**: a DFD generates the surface area you walk, and STRIDE is the per-element prompt. Flow-centric is what OWASP teaches, what most engineering teams already know, and what produces the kind of output development teams can act on. But it's not the only option — read `references/centric-methods.md` if the system suggests a different entry point (rich business logic → user-needs-centric; significant operational surface → process-centric; small set of crown-jewel assets → asset-centric).

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

1. **What are we working on?** → System decomposition + DFD with trust boundaries.
2. **What can go wrong?** → STRIDE-Per-Element analysis (default) producing a threat list.
3. **What are we going to do about it?** → A response (Mitigate / Eliminate / Transfer / Accept) for each threat, with concrete mitigations for "Mitigate".
4. **Did we do a good enough job?** → A self-assessment checklist + open questions / assumptions to validate.

Always produce all four sections. Skipping Q4 is a common anti-pattern ("admiration for the problem").

## Guided interview

Run when the user asks for a threat model but hasn't supplied enough system detail. Ask the questions below, but cluster them — don't fire one at a time. Two or three turns of clustered questions is the target.

**Round 1 — Scope and system understanding (Q1).** Ask:
- What is the system / feature / component? One-paragraph description.
- What are the external entities (users, other systems, third parties) that interact with it?
- What are the major processes (services, applications, components that *do* something)?
- What data stores are involved (databases, files, queues, configuration, caches)?
- Where do trust boundaries sit? (Network perimeters, process boundaries, privilege levels, tenant boundaries, physical boundaries for embedded/medical devices.)
- What sensitive data flows through it? (PII, PHI, credentials, financial, IP, safety-critical control signals.)

**Round 2 — Context that shapes threats.** Ask:
- Who would attack this and why? (External attacker, malicious insider, curious insider, supply chain, nation-state. Don't dwell — the Manifesto warns against over-focusing on adversaries.)
- What's the deployment context? (Cloud, on-prem, hybrid, air-gapped, hospital network, home network, public internet, embedded in a device.)
- Are there safety implications? (Especially relevant for medical, automotive, industrial control, aerospace.)
- Any regulatory constraints? (HIPAA, GDPR, FDA premarket cybersecurity guidance, IEC 62443, PCI-DSS, etc.)
- Existing security controls already assumed in place?

After Round 2 you should have enough to draft the DFD. If not, ask one more focused round — but err on the side of stating assumptions and proceeding.

## Producing the threat model

Output a single markdown document with the structure below. Use Mermaid for the DFD. If the user says "use the technical-blog-writer skill" or wants a publish-ready post, hand off the draft after producing it.

### Document structure

```
# Threat Model: <System Name>

## 1. What are we working on?
   - System description (1–2 paragraphs)
   - In-scope / out-of-scope (explicit list)
   - Assets (the things worth protecting; don't pad — see anti-pattern note)
   - Trust levels (who has what access)
   - Assumptions (numbered, so they can be challenged)
   - Data Flow Diagram (Mermaid, see §DFD conventions)

## 2. What can go wrong?
   - STRIDE-Per-Element threat table (see §Threat enumeration)
   - Threat tree(s) for the highest-risk threats (optional, if it adds clarity)

## 3. What are we going to do about it?
   - Mitigation table mapping each threat → response (Mitigate/Eliminate/Transfer/Accept) → concrete control(s)
   - Derived security requirements (numbered, so they're tractable for tracking in Polarion / Jira / GitHub Issues)

## 4. Did we do a good enough job?
   - Self-assessment checklist
   - Open questions and assumptions still to validate
   - Recommended next review trigger (next feature, next architectural change, etc.)
```

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

Default approach: **STRIDE-Per-Element**. For each element on the DFD, run through the STRIDE categories that apply to that element type (see table). For each category that applies, write at least one concrete threat ("An attacker on the public internet sends crafted DICOM C-STORE requests to the PACS to overflow the parser" — not "DoS").

| Element        | S | T | R | I | D | E |
|----------------|---|---|---|---|---|---|
| External entity| ✓ |   | ✓ |   |   |   |
| Process        | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Data flow      |   | ✓ |   | ✓ | ✓ |   |
| Data store     |   | ✓ | ? | ✓ | ✓ |   |

(R on data store applies if it's a logging/audit store.) STRIDE category prompts and per-element example threats live in `references/stride-prompts.md` — read it before enumerating threats.

Threat table format:

| ID | Element | STRIDE | Threat | Likelihood | Impact | Risk |
|----|---------|--------|--------|------------|--------|------|
| T1 | Auth Service (P1) | S | Attacker reuses captured session token across boundary | M | H | High |

Use qualitative ratings (L/M/H) by default. Avoid DREAD — OWASP and Shostack both note its scoring is too subjective. If the user asks for quantitative scoring, use OWASP Risk Rating Methodology (linked in `references/risk-rating.md`).

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

Q4 is the most-skipped step in real threat models — the Manifesto's "admiration for the problem" anti-pattern. Don't skip it. The full guidance, including Shostack's three-checklist approach, pen-testing-as-complement, model/reality conformance, and how to handle threat models for acquired/third-party code, is in `references/validation.md` — read it when producing a thorough Q4 section.

The minimum viable Q4 section in the output document includes three checklists (diagramming / threats / validating-threats), an open-questions list, and a re-review trigger. Format:

**Diagramming**
- [ ] Can we tell a story about how the system works without changing the diagram?
- [ ] Can we tell that story without using "sometimes" or "also"?
- [ ] Can we look at the diagram and see exactly where the software makes a security decision?
- [ ] Does it show all trust boundaries (every UID, app role, network interface, principal)?
- [ ] Does it reflect current/planned reality, not an aspirational version?
- [ ] Can we see where all data goes and who uses it (no data sinks)?
- [ ] Do we see the processes that move data between stores (data can't move itself)?

**Threats**
- [ ] Have we looked for each STRIDE category?
- [ ] At each element of the diagram?
- [ ] At each data flow specifically? (Belt-and-suspenders — flows get overlooked.)

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

## When STRIDE is the wrong tool

Two different "swap" decisions are possible:

- **Different entry point** — keep STRIDE, change what you start enumerating from. See `references/centric-methods.md` (asset-centric, process-centric, user-needs-centric, code-centric, attacker-centric).
- **Different methodology entirely** — replace or supplement STRIDE with a different categorization framework. See `references/methodologies.md`:
  - **LINDDUN** — when privacy threats dominate (GDPR-heavy systems, health data, surveillance concerns). Use alongside STRIDE, not instead.
  - **PASTA** — when business-impact-driven scoring is required and you have time for a 7-stage process.
  - **Attack trees** — for adversarial reasoning about a specific high-value target. Generally avoid as the primary method for medical-device work — most threats there are commodity attackers, opportunistic ransomware, or insiders rather than characterized APTs. Use surgically, on a single high-priority threat, only after you can name and characterize the adversary.
  - **MITRE ATT&CK + kill chains** — for operational threat modeling (detection coverage, threat hunting), not design-time modeling. ATT&CK is fundamentally an output characterization (a catalog of adversary techniques), not a generative entry point.
  - **OCTAVE / VAST** — when modeling at the org or enterprise level rather than per-system.

If the user's situation calls for one of these, name it, briefly justify the swap, and proceed. Don't force STRIDE onto a privacy-by-design problem — but also don't overproduce; the Manifesto's "Multiple representations" pattern is permission to combine views, not a mandate.

## Domain notes

A few domain-specific gotchas worth surfacing when relevant:

- **Medical devices** — Safety implications are first-class. Model the patient as an asset (in addition to data). Cite FDA premarket cybersecurity guidance and IEC 62443 / IEC 81001-5-1 if the user is preparing a regulatory submission. Consider clinical workflow misuse scenarios, not just network threats.
- **IoT / embedded** — Physical attack surface (debug ports, firmware extraction, side channels) belongs in the DFD. The "trust boundary" between the device and the network is often the most consequential.
- **Cloud-native** — Shared responsibility model matters. Document which controls are the cloud provider's vs. yours. IAM and managed services are usually under-modeled.
- **AI/ML systems** — STRIDE still works but extend with model-specific threats: prompt injection, training data poisoning, model extraction, prompt-based info disclosure. OWASP has a specific cheat sheet for this; reference it rather than reinventing.

## References

Detailed reference content lives in `references/`. Read these on demand:

- `references/centric-methods.md` — Entry-point taxonomy: asset-centric, flow-centric, process-centric, user-needs-centric, attacker-centric, code-centric. When to pick each, and the generation-vs-characterization distinction.
- `references/stride-prompts.md` — STRIDE category prompts and example threats per element type, plus STRIDE-Per-Element vs Per-Interaction guidance and the DESIST variant.
- `references/dfd-mermaid.md` — DFD-to-Mermaid mapping conventions with worked examples, Shostack-derived diagramming rules of thumb, and the diagramming checklist.
- `references/methodologies.md` — When to use LINDDUN, PASTA, attack trees, MITRE ATT&CK, kill chains, OCTAVE, VAST as full-methodology swaps or supplements.
- `references/validation.md` — Q4 deep dive: model/reality conformance, the three Shostack-style checklists, pen testing as complement (not substitute), validating threats in acquired/third-party code, document-assumptions-as-you-go.
- `references/manifesto.md` — Threat Modeling Manifesto values, principles, patterns, anti-patterns (paraphrased).
- `references/risk-rating.md` — Qualitative L/M/H risk rating, OWASP Risk Rating Methodology pointer, why DREAD is discouraged.

A blank threat model template is in `assets/threat-model-template.md`.

## Citation note

Concepts in this skill are paraphrased from open OWASP material (Threat Modeling community page, Threat Modeling Process, Threat Modeling Cheat Sheet, OWASP Security Culture v1.0 §6, OWASP Threat Modeling Playbook by Toreon), the Threat Modeling Manifesto (CC-BY 4.0), and Adam Shostack's *Threat Modeling: Designing for Security* (Wiley, 2014) — particularly the Four Question Framework, STRIDE-Per-Element, the diagramming and validation checklists, and the practical guidance on iteration, assumption documentation, and bug-filing as the exit point. STRIDE itself originates from Microsoft (Loren Kohnfelder and Praerit Garg). DESIST is by Gunnar Peterson. Cite these sources in any output that quotes or substantively summarizes them.
