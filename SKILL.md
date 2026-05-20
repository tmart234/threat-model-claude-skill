---
name: threat-modeler
description: Produce a threat model for a system, feature, or architecture — a data-flow diagram, an enumeration of what can go wrong, and concrete mitigations. Use when the user asks to threat model something, asks how someone might attack a system or what could go wrong with it, or shares a design or architecture and wants its security risks assessed.
---

# Threat Modeler

Produce a threat model an engineering team can act on. You are doing the work — decomposing a system, finding what can go wrong, proposing fixes — not surveying methodologies or writing a consultant's report.

Two habits throughout:

- **State assumptions and proceed.** Don't interrogate the user for details you can reasonably infer. Write each inference down as a numbered assumption and keep moving.
- **Ship a useful approximation.** A one-page model that gets filed and acted on beats a perfect one that never ships. Threat modeling is iterative; a v1 is a success, not a draft to apologize for.

## The four questions

A threat model answers four questions, in order — Shostack's framework, adopted by the Threat Modeling Manifesto:

1. **What are we working on?** — Describe the system. Draw a data-flow diagram (DFD) with trust boundaries.
2. **What can go wrong?** — Enumerate threats against it.
3. **What are we going to do about it?** — Decide a response for every threat.
4. **Did we do a good enough job?** — Check the model against reality.

Answer all four. Q4 is the most-skipped and the cheapest to skip badly — don't.

A threat model produces *hypothesized* attack scenarios and *proposed* controls — not confirmed vulnerabilities and not shipped fixes. Both halves still need validation (code review, testing, red-teaming) and engineering work. Say so in the output, so no one mistakes the document for a security guarantee.

## Size the model to the system

This is the most important judgment call in the skill. Match the depth of the model to what the system is worth:

| System | A useful model is roughly… |
|---|---|
| Side project, internal tool, throwaway spike | A DFD + a STRIDE pass + a short mitigation list. One page. |
| Production service with real users or real data | The above + one extra angle (privacy, business-logic abuse, or operations) + a note on any compliance regime that applies. |
| Regulated, safety-critical, or high-value system | The above + the deeper methods the references describe — pick the ones that fit. |

Start with the smallest model that is genuinely useful, and add depth only when the system earns it. Do **not** pad a simple system with empty or stub sections to look thorough — that is the "checkbox compliance" the Manifesto explicitly warns against. A side project does not need a sector-threat-intelligence section; writing "not applicable" three times is worse than never raising it.

If you are unsure how deep to go, ask the user once (see *Scope it first*).

## Scope it first

Work out which situation you're in:

- **Enough detail already given** — the user pasted an architecture, a spec, code, a component list, or a description detailed enough to identify the external entities, processes, data stores, and trust boundaries. Go straight to Q1; fill gaps with stated assumptions.
- **Not enough detail** — the user asked for a threat model but didn't describe the system. Ask a compact, clustered set of questions (below), then proceed.
- **Existing model to extend** — the user pasted a threat model and wants it reviewed, updated, or extended. Treat their document as the input: keep their structure and IDs, mark what's new, and start with a Q4 check that the old model still matches reality.

When you do need to ask, cluster the questions — two or three turns, not a stream of single questions. Cover:

- **The system.** What is it, who uses it, what does it do? External entities, major processes, data stores.
- **Trust boundaries and data.** Where do privilege levels change (network, process, tenant, user/admin, physical)? What sensitive data flows through (PII, credentials, financial, health, IP, safety-critical signals)?
- **Environment and stack.** Where does it run (cloud, on-prem, mobile, embedded, OT/ICS)? What protocols are on the wire? Who owns each piece of infrastructure? Concrete protocols matter — "gRPC with mTLS" or "Modbus/TCP" exposes threats that a flow labelled "data" hides.
- **Stakes and audience.** Any safety implications or regulatory regime? Who will read this — engineers wanting testable requirements, a detection/red-team audience, compliance sign-off? The reader shapes what to include and how deep to go; if it isn't obvious, ask.

Don't block the model on missing answers. Infer, record the assumption, proceed.

## Q1 — What are we working on?

Decompose the system and draw a DFD.

Use Mermaid (`flowchart LR`). Map elements:

- External entity → rectangle: `EE[User]`
- Process → rounded rectangle: `P1(Auth Service)`
- Data store → cylinder: `DS[(User DB)]`
- Data flow → labelled arrow: `EE -- "credentials (HTTPS)" --> P1`
- Trust boundary → a `subgraph` around the elements on one side

Label every flow with its real protocol and authentication. Put every element inside a trust zone — a floating element usually means a missing boundary. Keep one diagram to roughly 5–15 elements; if it needs more, draw a context diagram plus one or more decomposed diagrams rather than a single unreadable one.

Trust boundaries are wherever principals with different privilege interact: network segments, process/account boundaries, tenant boundaries, user-vs-admin, physical boundaries. Conventions, a worked example, and per-environment boundary patterns are in `references/dfd.md` and `references/environments.md` — read them when drawing.

Also capture, briefly: in-scope / out-of-scope; the assets actually worth protecting (don't pad the list — a long asset inventory is a distraction, not thoroughness); trust levels; and numbered assumptions.

## Q2 — What can go wrong?

Default method: **STRIDE, applied per element of the DFD.**

For each element, walk the STRIDE categories that apply to it and ask "how could an attacker do this against this element?" — Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege. STRIDE is a *prompt to find threats*, not a filing system. When a threat is hard to categorize ("is this Spoofing or Information Disclosure?"), record it and move on — the category did its job by surfacing the threat.

- Write **concrete** threats: "an attacker on the shared network replays a captured session token to act as the user" — not "Spoofing".
- Roughly five threats per element is a healthy amount; far fewer suggests an element was skimmed.
- Walk threats top-down and breadth-first; start at the elements that cross the outermost trust boundary.
- Threats follow data flow and cluster around trust boundaries and complex parsers — but write down anything you notice, even out of category.

The applicability table, per-category prompts, and the STRIDE → security-property → mitigation mapping are in `references/stride.md`.

Give each threat a stable ID (`T1`, `T2`, …) so mitigations can reference it, and a likelihood and impact (L/M/H by default — matrix in `references/risk-rating.md`).

**When STRIDE alone isn't enough.** STRIDE on a DFD is the workhorse, but it misses some things. Reach into `references/methods.md` when the system calls for it:

- Privacy concerns beyond plain disclosure (PII, health, location data) → LINDDUN.
- Business-logic abuse (multi-tenant, delegation, approval flows) → abuse-case / user-needs analysis.
- One dominant sensitive data class, or data-typed regulation (HIPAA/GDPR/PCI) → data-centric (NIST SP 800-154).
- ML/AI components → an AI-specific threat pass (prompt injection, poisoning, model extraction).
- A high-value asset worth adversarial reasoning → an attack tree on that one target.
- A detection / SOC / IR audience → map threats to MITRE ATT&CK; trace STRIDE → CAPEC → CWE where requirement traceability is needed.
- Safety-critical control loops (medical devices, ICS, automotive, robotics) → STPA-SafeSec; see `references/safety-critical.md`.

These are options, not a checklist. Add the one or two that fit; ignore the rest.

## Q3 — What are we going to do about it?

Give every threat exactly one response: **Mitigate**, **Eliminate**, **Transfer**, or **Accept**. Enumerating endlessly without deciding — "admiration for the problem" — is the anti-pattern to avoid here.

For **Mitigate**, name a concrete control mapped to the violated property (the STRIDE → mitigations table in `references/stride.md`). For **Accept**, record the rationale and who accepted it.

Put the responses in one prioritized list, sorted by risk. Where the same finding surfaced more than once, cross-reference IDs rather than duplicating the threat.

If the team wants testable security requirements, derive them from the mitigations and number them — each should be something a test can check ("The system SHALL authenticate all X using Y"). This is what makes threats trackable: a threat becomes a tracked issue, and normal engineering triage takes over. That hand-off is the real exit point from threat modeling.

## Q4 — Did we do a good enough job?

A short, honest self-check — not a 25-box ritual. Cover:

- **Conformance** — does the DFD match what is actually built or planned?
- **Completion** — does every threat have a response, and is each captured somewhere the team will act on it (an issue tracker, not just this document)?
- **Open questions** — assumptions still to validate; anything you couldn't resolve.
- **Next review trigger** — the event that should prompt re-modeling (a new feature, an architecture change, a deployment-environment change, or a calendar interval).

Deeper validation guidance — model/reality conformance after drift, pen-testing as a complement, modeling third-party code — is in `references/validation.md`.

## Output

Produce one Markdown document organized by the four questions. A blank template is in `assets/threat-model-template.md` — copy from it.

Default to a single document. Split into linked documents only when genuinely different teams own and read different parts, and only if the user wants that; a single document is the safe default, and splitting later is cheap. Keep the structure proportional to the model (see *Size the model to the system*) — delete the template's optional sections the system doesn't need rather than filling them with "not applicable".

## References

Read these on demand — don't load them up front.

- `dfd.md` — drawing the DFD in Mermaid; trust boundaries; decomposition; worked example.
- `stride.md` — STRIDE per-element applicability table, category prompts, mitigation mapping.
- `methods.md` — when and how to go beyond STRIDE: entry points, LINDDUN, data-centric, attack trees, PASTA, ATT&CK/CAPEC/CWE, AI/ML, OCTAVE/VAST.
- `environments.md` — per-environment trust-boundary patterns (cloud, on-prem, mobile, embedded, OT/ICS) and the ownership taxonomy.
- `risk-rating.md` — qualitative L/M/H, OWASP Risk Rating, why DREAD is discouraged, the safety bump.
- `validation.md` — Q4 in depth; the Threat Modeling Manifesto's patterns and anti-patterns.
- `safety-critical.md` — STPA-SafeSec for control-loop systems; cyber-physical attack paths for multi-device environments.

## Sources

This skill paraphrases: the **Threat Modeling Manifesto** (CC-BY 4.0); **Adam Shostack**, *Threat Modeling: Designing for Security* (Wiley, 2014) — the Four Question Framework and STRIDE-Per-Element; **Microsoft** — STRIDE (Kohnfelder & Garg); **OWASP** threat-modeling material; **MITRE** — ATT&CK, CAPEC, CWE; **NIST SP 800-154** (data-centric modeling); plus the methodology-specific sources cited in each reference file. Attribute these in any output that quotes or substantively summarizes them.
