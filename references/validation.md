# Q4 — validating the threat model

"Did we do a good enough job?" is the most-skipped of the four questions. Teams enumerate threats and never close the loop on whether the model is right, whether the responses landed, or whether the implementation matches the model. This file makes Q4 a real activity.

The frame is paraphrased from Shostack's *Threat Modeling: Designing for Security* (Ch. 1, "Checking Your Work"; Ch. 10, "Validating That Threats Are Addressed").

## Three things to check

1. **Model/reality conformance** — does the diagram match what was actually built?
2. **Task completion** — does every threat have a response, and has each response become a tracked work item?
3. **Bug checking** — as work proceeds, are the threat-derived issues actually being closed, with mitigations tested?

## Model/reality conformance

Models go stale fast. After a couple of architecture revamps, threats found against the old diagram may no longer apply, and threats against the *current* system were never considered.

- Walk the diagram with someone who built the system and ask: is this complete? accurate? does it cover the security decisions made? could we start the next version from this diagram unchanged? If everyone says yes, the model is good. If not, update it.
- Whenever threats led to redesign, re-check the diagram against the code at check-in, or the deployed system at deployment. Don't assume the planned mitigation was implemented as drawn.
- For deployed systems, the complexities of operations cause on-the-fly changes — firewall rules disabled in a pinch, IAM roles widened during an incident. If those happened, the model needs a revamp.

## Task completion

Every threat must have a recorded response. Skipping this is the signature of "admiration for the problem". For each threat:

- Was a response chosen — Mitigate / Eliminate / Transfer / Accept?
- For Mitigate — is there a concrete control, with an owner?
- For Accept — is the rationale recorded, attributed to a named decision-maker?
- For Transfer — does the receiving party know they're now on the hook?
- Is each one tracked in the team's normal issue tracker?

The tracker integration matters. Threats that live only in the threat-model document get lost; threats filed as issues get the team's normal triage. **That hand-off is the exit point — it lets you say "we're done threat modeling this."** A `threat-model` label makes the bug-checking query trivial.

## A closing checklist

Drop a short check into the document — enough to be honest, not a ritual. The core:

- [ ] The DFD reflects the system as actually built or planned, not an aspirational version.
- [ ] Every element was walked for its applicable STRIDE categories; data flows specifically were not skipped.
- [ ] Any supplementary pass the system needed (privacy, abuse cases, data-centric, AI/ML) was run.
- [ ] Every threat has a response decision.
- [ ] Every "Mitigate" has a concrete, testable control; high risks have an owner.
- [ ] Threats are filed where the team will act on them.
- [ ] Assumptions are listed and falsifiable.
- [ ] The next-review trigger is recorded.
- [ ] Someone other than the author has reviewed it.

Add items only when the system warrants them. Don't pad the checklist to look rigorous — a short checklist that gets honestly completed beats a long one that gets rubber-stamped.

## Penetration testing — complement, not substitute

Pen testing is not, by itself, the validation activity for a threat model. **You can't pen-test your way to a secure design** — pen testing tells you what's broken now; threat modeling tells you what should and shouldn't exist. They complement: give pen testers the threat model as a roadmap, and feed their findings back into model updates.

Glass-box testing (testers get the model, design, and code) targets the threats the model identified and reports which are actually exploitable — much higher value than black-box, which mostly tests how hard the system is to discover. If a test comes back "found only XSS and SQLi" and the team wanted architecture-level findings, the failure was scope alignment, not testing.

## Modeling third-party or acquired code

Most systems include code the team didn't write, often without diagrams or source. Shostack's inside-out approach — for each acquired component, enumerate:

- The **accounts and processes** it creates or runs as.
- The **listening ports** and IPC endpoints.
- The **administrative interfaces** — including account-recovery paths and service accounts (undocumented "forgot your password" backdoors are unfortunately common).
- The **platform changes** it makes — OS changes, firewall rules, permission changes, auto-updater behavior, unpatched vulnerabilities at install time.

`ps`, `netstat`, `find -newer` help on Linux; Sysinternals Autoruns on Windows. A component that ships with its own threat model and security guide is worth substantially more than one that doesn't — flag that in a buy/build decision.

## Document assumptions as you go

The common advice "document all assumptions" leads to teams enumerating every possible assumption upfront, which is unbounded and turns into pedantry. Better:

- During modeling, when someone says "I assume that…", write it down right then.
- Phrase it falsifiably: "the load balancer terminates TLS before the app server" is testable; "the network is secure" is not.
- For each, ask: could it ever not be true? what breaks if it's false?
- Hand the list to whoever tests — assumptions become test cases.

## When to re-validate

Q4 isn't done at ship time. Re-validate when:

- A new feature or major component lands.
- An architecture change affects trust boundaries, identity, or data flow.
- An incident reveals a gap the model missed.
- The deployment environment changes (cloud migration, new tenancy model).
- A calendar trigger fires (annually for stable systems, more often for fast-moving ones).

Record the next-review trigger in the document so the team has an answer to "when do we look at this again?"

---

## The Threat Modeling Manifesto

The Threat Modeling Manifesto (threatmodelingmanifesto.org, 2020, CC-BY 4.0) is methodology-agnostic — it doesn't mandate STRIDE — but its values shape what a good threat model looks like.

### Values

It values the items on the left more than those on the right:

- A **culture of finding and fixing** design issues, over checkbox compliance.
- **People and collaboration**, over processes, methodologies, and tools.
- **A journey of understanding**, over a security/privacy snapshot.
- **Doing** threat modeling, over talking about it.
- **Continuous refinement**, over a single delivery.

### Patterns (do these)

- **Systematic approach** — be thorough and reproducible. (STRIDE-Per-Element gives reproducibility.)
- **Informed creativity** — leave room for craft once the structured pass is exhausted (misuse cases, abuse stories, supply-chain angles).
- **Varied viewpoints** — bring in diverse expertise; call out where domain SMEs are needed.
- **Useful toolkit** — tools that increase productivity and repeatability.
- **Theory into practice** — field-tested techniques tailored to local context; prefer mitigations that map to real engineering work the team does.
- **Multiple representations** — multiple imperfect views beat one perfect view; a context diagram plus decomposed diagrams is good.

### Anti-patterns (avoid these)

- **Hero threat modeler** — assuming only one special person can do this. Use plain language; assume the development team will read the output.
- **Admiration for the problem** — analyzing endlessly without producing solutions. Every threat gets a response.
- **Tendency to overfocus** — too much attention on one adversary, asset, or technique. Cover the whole DFD before drilling down.
- **Perfect representation** — refusing to ship until the model is "perfect". Ship a useful approximation; iterate.
- **Asset rabbit-holing** — long asset lists are usually a distraction. Keep the list to things genuinely worth protecting; don't pad with abstractions like "company reputation".

Attribute substantive quotes to "Threat Modeling Manifesto", https://www.threatmodelingmanifesto.org/.
