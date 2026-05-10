# Threat Modeling Manifesto (paraphrased reference)

> **Last verified**: 2026-05 against threatmodelingmanifesto.org. The Manifesto is a dated document (2020) — its values, principles, patterns, and anti-patterns are stable, but re-check the live site for any post-2026 supplements before substantive citation.
> **Sources paraphrased**: Threat Modeling Manifesto (2020), https://www.threatmodelingmanifesto.org/, licensed CC-BY 4.0. Paraphrasing for this skill is allowed under the license. Direct quotes require attribution to "Threat Modeling Manifesto" with the URL.

> **Related**: ← `SKILL.md` (entry point — references the patterns and anti-patterns below) • `methodologies.md` ("Multiple representations" pattern as the basis for hybrid output) • `validation.md` ("Admiration for the problem" anti-pattern is the Q4 motivation).

This file is the **canonical home** for the Manifesto's patterns, anti-patterns, and values as they apply to this skill. SKILL.md cites them; don't duplicate.

The Threat Modeling Manifesto (2020, threatmodelingmanifesto.org, CC-BY 4.0) defines values, principles, patterns, and anti-patterns for the practice. It's methodology-agnostic — it doesn't tell you to use STRIDE — but the values shape how this skill produces output.

## What threat modeling is (per the Manifesto)

Analyzing representations of a system to surface security and privacy concerns. The four key questions:

1. What are we working on?
2. What can go wrong?
3. What are we going to do about it?
4. Did we do a good enough job?

These are Shostack's Four Question Framework, adopted as the Manifesto's structure.

## Values (paraphrased)

The Manifesto says that while the items on the right have value, it values the items on the left more:

- **Culture of finding and fixing design issues** > checkbox compliance.
- **People and collaboration** > processes, methodologies, and tools.
- **A journey of understanding** > a security/privacy snapshot.
- **Doing threat modeling** > talking about it.
- **Continuous refinement** > a single delivery.

How this shapes the skill:
- Output should help the team find and fix things, not just check a box.
- Recommend dialogue with stakeholders; don't pretend the model is final.
- Iterative output is fine. A "v1" model is better than no model.

## Principles

- The best use of threat modeling is to improve security and privacy through early and frequent analysis.
- Threat modeling must align with the organization's development practices and follow design changes in iterations.
- Outcomes are meaningful when they're valuable to stakeholders.
- Dialogue establishes shared understanding; documents record it and enable measurement.

How this shapes the skill:
- Suggest a re-review trigger in the Q4 section ("re-model when X changes").
- Make outputs actionable for the audience (devs, ops, regulators) — don't write only for security people.

## Patterns (do these)

Definitions from the Manifesto, with the skill-output gloss in parentheses:

- **Systematic approach** — be thorough and reproducible by applying knowledge in a structured way. (STRIDE-Per-Element gives the reproducibility.)
- **Informed creativity** — leave room for craft, not just process. (Once STRIDE is exhausted, brainstorm misuse cases, abuse stories, supply-chain angles.)
- **Varied viewpoints** — diverse SMEs and cross-functional collaboration. (Call out where domain expertise is needed — e.g. "this is the right place to bring in the firmware team".)
- **Useful toolkit** — tools that increase productivity and enable repeatability.
- **Theory into practice** — use field-tested techniques tailored to local context. (Prefer mitigations that map to real engineering work the team does.)
- **Multiple representations** (added by the Manifesto's pattern catalogue, central to this skill's hybrid output) — multiple imperfect views beat one perfect view; context DFD + decomposed DFDs are fine and good. The skill operationalizes this as DFDs (`dfd-mermaid.md`) plus, where the system warrants them, sequence / swim-lane diagrams and state diagrams (`non-dfd-models.md`) — three diagram families, one cross-referenced threat table.

## Anti-patterns (avoid these)

Definitions from the Manifesto, with the skill-output gloss:

- **Hero threat modeler** — assuming only one person with a special mindset can do this. (Use plain language; assume the development team will read this.)
- **Admiration for the problem** — analyzing endlessly without producing solutions. (Every threat gets a response: Mitigate / Eliminate / Transfer / Accept.)
- **Tendency to overfocus** — too much attention on one adversary, asset, or technique. (Cover the whole DFD before drilling down.)
- **Perfect representation** — refusing to ship until the model is "perfect"; multiple imperfect views beat one perfect view. (Ship a useful approximation; iterate.)
- **Asset rabbit-holing** (skill-specific extension) — long asset lists are usually a distraction. Keep the list to things actually worth protecting; don't pad with abstractions like "company reputation".

## Failure-mode catalog — what bad output looks like

The Manifesto's anti-patterns above describe *practitioner* failure modes ("admiration for the problem", "hero threat modeler"). What follows is the artifact-level analogue: failure modes that show up in the *output* of a threat model, regardless of what the practitioner did. Treat these as things to refuse to produce — when you notice them in your draft, fix them before shipping. None of these are theoretical; each has been observed in real threat-model deliverables.

- **Padded asset list.** Every internal table is in §1's asset list; "company reputation," "customer trust," "the business" appear as `AS#`. **Fix:** keep §1 to the things attackers actually want to compromise, plus the things whose loss would *change* the team's response. If removing an asset wouldn't change a single threat or mitigation, it's padding.
- **Single-threat-per-element.** Every DFD element gets exactly one row in the threat table — usually the most-obvious STRIDE category for that shape. A process gets one Spoofing threat; a data store gets one Information Disclosure threat. **Fix:** walk every applicable STRIDE cell at every applicable element. Five threats per diagram element is the rule-of-thumb floor; fewer suggests the element wasn't analyzed (`stride-prompts.md` § "Enumeration tactics").
- **Generic mitigations.** "Apply RBAC", "use encryption", "implement input validation", "follow least privilege". These describe a control *class*, not a *control*. **Fix:** name the actual mechanism, the actual control surface, the actual party who implements it. *"Tenant scope enforced in middleware on every authenticated route, with a per-route automated test that asserts a request from tenant A cannot read tenant B's data, owned by the API team"* is a mitigation. *"Apply RBAC"* is a placeholder for one.
- **STRIDE label without a concrete attack.** A row reads `T7 | Auth Service | S | Spoofing | M | H | High` — the threat column literally just says the STRIDE category. **Fix:** every threat sentence is one specific scenario: who, against which element, by what means, to what end. *"Attacker on the public internet sends a forged JWT with a manipulated `aud` claim to bypass the auth service's audience check"* — not *"Spoofing"*.
- **Asymmetric coverage by trust level.** The model has 30 threats against the public-internet boundary and 0 threats against the operator-console boundary, even though the operator console can do everything the public-internet attacker would want to do. **Fix:** if the model implicitly trusts a class of operator (admin, on-call, vendor service account), say so as an assumption (`ASM#`) and acknowledge the residual risk; don't silently omit operator-side threats.
- **Mitigation table that doesn't match the threat table.** §2 has 14 threats; §3 has 6 mitigation rows. The other 8 threats have no response listed at all (not even Accept). **Fix:** every threat in §2 has a row in §3, even when the row is just `Accept` with rationale. This is the most common form of "admiration for the problem".
- **Aspirational DFD.** The DFD shows the planned architecture (with the WAF, the bastion host, the KMS isolation that's in the next quarter's roadmap), not what's actually deployed. **Fix:** model the real system. If the team intends to change it, file the change as a finding (`SR-###`) — the threat model isn't the place to wish-cast.
- **Trust-boundary-everywhere DFD.** Every box is its own subgraph. The diagram becomes a network diagram with extra steps, and §2 enumerates threats at every internal flow as if the application were running on hostile internal hardware. **Fix:** boundaries are where *trust* changes. Two services running under the same identity, on the same VPC, owned by the same team are not on opposite sides of a trust boundary.
- **No response for "Accept".** A threat is marked `Accept` with no rationale, no decision-maker named, and no residual-risk language. **Fix:** every `Accept` row records *why* (cost, infeasibility, lower-priority-than-a-conflicting-loss) and *who* signed off. An unjustified `Accept` is indistinguishable from a missed threat.
- **CAPEC / CWE / ATT&CK without the chain.** §2.2 lists CAPEC IDs, CWE IDs, and ATT&CK IDs in separate columns with no connective tissue — no derived requirement closes the CWE, no mitigation cites the CAPEC, no detection note maps to the ATT&CK technique. **Fix:** if §2.2 is produced, every row's CAPEC → CWE → mitigation chain should land in a `SR-###` that closes the CWE. Otherwise the operational stratum is decorative (`capec.md` § "The chain that makes CAPEC worth citing").
- **Risk inflation by "Critical" creep.** Every safety-touched threat is marked `Critical` and the prioritization stops being useful. **Fix:** see `risk-rating.md` § "Default: 3×3 qualitative matrix" — `Critical` is for `High + High` *and* a documented promotion rule, not "anything that mentions safety". Apply the safety-bump rule to *impact*, not directly to risk.

If you find yourself producing any of these in a draft, the right move is to fix the draft before shipping — a threat model with these failure modes is worse than no threat model, because it gives the team the false impression that their threats have been considered.

## Authors

Zoe Braiterman, Adam Shostack, Jonathan Marcil, Stephen de Vries, Irene Michlin, Kim Wuyts, Robert Hurlbut, Brook S.E. Schoenfield, Fraser Scott, Matthew Coles, Chris Romeo, Alyssa Miller, Izar Tarandach, Avi Douglen, Marc French.

## Citation

If quoting, attribute to "Threat Modeling Manifesto" with the URL https://www.threatmodelingmanifesto.org/. Licensed CC-BY 4.0 — paraphrasing for this skill is fine; substantive direct quotes need attribution.
