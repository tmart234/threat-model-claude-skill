# Centric methods — choosing the entry point

> All models are wrong; some are useful. Threat modeling methods are abstractions of reality; what differs between them is the entry point.

This file is about the **entry point** to threat generation — the question of "what do I look at first to enumerate threats?" — not about which methodology (STRIDE / LINDDUN / PASTA) you apply once you're enumerating. Those are separate decisions:

```
ENTRY POINT (centric method)        +    LENS (categorization)
──────────────────────────────             ────────────────────
asset-centric                              STRIDE
flow-centric                               LINDDUN
process-centric                            CAPEC
user-needs-centric                         ATT&CK
attacker-centric                           kill chain
code-centric                               CWE
                                           CVSS
```

For example, "STRIDE-Per-Element" is really *flow-centric generation + STRIDE characterization*. STRIDE is doing the prompt work; the DFD is doing the coverage work. Confusing the two leads people to think STRIDE alone is a complete approach. It isn't — STRIDE is a categorization lens that needs an entry point to be applied against.

### Mapping to Shostack's three approaches

Shostack's *Threat Modeling: Designing for Security* (Ch. 2) frames structured threat modeling as a choice between three approaches: **focusing on assets, focusing on attackers, or focusing on software** — and recommends focusing on software. His "focus on software" maps onto a cluster of entry points in the more granular taxonomy used here: flow-centric, process-centric, and user-needs-centric all model the software (or system) directly via DFDs, workflow descriptions, or user stories. This skill's default — flow-centric generation with STRIDE-Per-Element — is squarely inside Shostack's "focus on software" recommendation.

User-needs-centric and code-centric entries below are extensions beyond Shostack's three categories — they capture modes of work (abuse-case inversion, implementation review) that have become more visible in the threat-modeling community since 2014.

## Generation vs. characterization

A subtle but important distinction:

- **Generative methods** *produce* threats. The DFD + asset list + user stories are generative — they create the surface area you enumerate against.
- **Characterization layers** *organize* threats. STRIDE, CAPEC, CWE, ATT&CK, CVSS all categorize threats once you have them.

STRIDE is unusual in that it can be used both ways: as a per-element prompt (generative — "what could go wrong at this element across these six axes?") and as a post-hoc bucket ("this finding is a Tampering issue"). Be explicit about which way you're using it. This skill defaults to using STRIDE generatively (as per-element prompts) inside a flow-centric or asset-centric entry point.

## What a threat model produces

Output of a threat model is **a set of hypothesized attack scenarios, not confirmed vulnerabilities**. A threat is something an attacker might attempt; a vulnerability is a confirmed weakness on this specific system; a risk is what materializes when a threat exploits a vulnerability. Threat modeling produces hypotheses about what *could* go wrong, which then need validation (typically code review, testing, or operations) to determine whether the vulnerability actually exists in the implementation.

This is why code-centric review is a validation layer, not a generation method (see below) — and it's why threat models go stale: the system's implementation drifts and the hypotheses need re-checking.

## The entry points

### Asset-centric

**Start by**: enumerating "crown jewel" assets and working backwards. For each asset, ask "how could this be compromised?" and walk STRIDE.

**Strengths**:
- FAIR-style thinking (impact-anchored, ties naturally to business risk).
- Maps cleanly to DFD elements — assets are usually processes or data stores.
- Pairs naturally with STRIDE-Per-Element.

**Weaknesses**:
- Coverage is bounded by the asset inventory, and people are notoriously bad at asset inventory. Forgotten assets = forgotten threats.
- Tends to under-weight assets that aren't crown jewels but are reachable stepping stones (the dev VM, the build server, the metrics endpoint).

**Use when**: the system has a small number of obvious high-value assets (PHI database, signing key, patient safety control loop). Less useful when "everything is sensitive."

**Shostack's critique** (Ch. 2): "There's no direct line from assets to threats, and no prescriptive set of steps. Essentially, effort put into enumerating assets is effort you're not spending finding or fixing threats." Asset enumeration tends to bog down in arguments about what *counts* as an asset (he distinguishes "things attackers want," "things you want to protect," and "stepping stones," which mostly overlap). The output of all that work is a list of things to look for in your software model — at which point you might as well have started with the software model. Asset thinking is most useful for *prioritizing* threats once you've found them, not for finding them in the first place.

### Flow-centric

**Start by**: enumerating data — what data exists, where it lives, where it goes — then ask "how does the data move?" Draw the processes and connections that handle each flow, then enumerate threats per element with STRIDE.

**Strengths**:
- Pairs with STRIDE-Per-Interaction, which is the most thorough STRIDE variant.
- Wide coverage — every flow gets attention.
- Forces you to surface trust boundaries (every boundary crossing is interesting).

**Weaknesses**:
- Doesn't capture emergent behavior — flows don't tell you what happens when two unrelated processes both have a race condition with the same store.
- Flattens time to a single flow. "What happens after the attacker is in" is hard to express.

**Use when**: this is the default entry point for most engineering-team threat modeling. It's what OWASP teaches and what this skill defaults to.

### Process-centric

**Start by**: enumerating operational workflows — incident response, deployment, on-call escalation, key rotation, account provisioning, support handoffs — and asking "how would an attacker exploit each step?"

**Strengths**:
- Threats are temporal, not structural. Captures attacks that depend on *when* something happens (a window during deploy, a race during rotation, a gap during handoff).
- Surfaces operational and human-factors threats that flow-centric misses entirely.

**Weaknesses**:
- Depends on having process documentation — or someone with deep operational knowledge — that's accurate. Many orgs don't.
- Easy to miss processes that exist but aren't named.

**Use when**: the system has significant operational surface (regular deploys, on-call rotations, key/credential rotations, manual approval steps). Pair with flow-centric, don't substitute.

### User-needs-centric (abuse-case inversion)

**Start by**: take a user story or a stated user need, then apply an inversion lens — for each "user can do X", ask "what if a different user / unauthorized user / malicious user does X?" and "what if this user does X to a target they shouldn't?". Generate threats from the abuse of intended functionality.

**Strengths**:
- Highly specific threat language ("a clinician can view another clinician's drafts" is more useful than "broken access control").
- Feature-complete — easier to avoid missing a feature, since every user story gets inverted.
- Surfaces business-logic flaws that STRIDE-Per-Element will not catch.

**Weaknesses**:
- Often lacks asset context — when inverting "user can export a report", you sometimes don't know what data the report actually contains. Needs flow-centric or asset-centric context layered in.
- Coverage depends on the user-story inventory being current.

**Use when**: the system has rich business logic, multiple user roles, or workflow complexity. Especially valuable for clinical software, multi-tenant SaaS, and anything with delegation or approval flows.

### Attacker-centric (persona non grata, attack trees)

**Start by**: characterize the attacker's capabilities (e.g. APT28, an insider, a script kiddie). From their TTPs, enumerate possible attacks against your system.

**Strengths**:
- Sophisticated, intel-driven. Useful for layered detection design.
- Surfaces attacks specific to a known adversary's capabilities.

**Weaknesses**:
- Requires the adversary to be known, prioritized, and capability-assessed (ATT&CK groups, threat intel feeds). Most teams don't have this.
- Tends to overweight the named adversary and miss everyone else.
- Models the *attacker's* perspective rather than your system's threat surface.

**Use when**: your environment is one with named, characterized adversaries — government / classified work, nation-state-target finance, defense contractors. **Generally avoid as a primary method for medical device work** — most medical device threat scenarios are commodity attackers, opportunistic ransomware, or insiders, not characterized APTs. If you do reach for an attack tree in medical-device work, reserve it for a single high-priority threat where adversarial reasoning genuinely adds value (e.g. "how could someone forge a firmware signature") and only after you can name and characterize the adversary you're modeling.

**Shostack's critique** (Ch. 2): even with detailed attacker personas, "attacker lists or even personas are not enough structure for most people to figure out what those people will do. Engineers may subconsciously project their own biases or approaches into what an attacker might do." Talking about human attackers can be useful for making threats feel concrete to non-security stakeholders ("a spy might do X") — but that's a *communication* benefit, not a methodological one, and it can backfire if it leads to "no one would ever do that" dismissals. Shostack recommends against attacker-centric modeling as the primary approach.

### Code-centric (validation only)

**Start by**: read the actual implementation. Use code review (or LLM-assisted review) to extract assumptions and find facts the model didn't capture.

**Crucial framing**: code-centric is a **validation layer, not a generative method**. It's not "model-based" — it's implementation-based. It produces ground truth, which means it can confirm or *invalidate* assumptions made by any of the model-based methods above. If the model says "the database is on a private network" and the code reveals a hardcoded public connection string, the model was wrong.

**Strengths**:
- The only approach grounded in what actually exists.
- Maps best to CWEs (since CWE is a code-level taxonomy).
- Can reveal things nobody documented — undisclosed assets, unwritten processes, latent abuse cases.

**Weaknesses**:
- Bottom-up. Produces findings, not a coherent narrative. You'll know "function `parse_dicom_pdu()` doesn't validate length" but not "this is the worst threat to the system."
- Needs a model-layer lens (one of the methods above) to give findings their full context.

**Use when**: validating an existing model, or when the model is suspiciously clean and you want to challenge it. Don't use code-centric *as* the threat model.

## Output characterization layers (these are not entry points)

A common confusion: people sometimes treat CAPEC, CWE, ATT&CK, CVSS, and even STRIDE as if they were entry-point methods. They're not. They're characterization / categorization layers applied to outputs:

- **STRIDE** — design-level threat categories (Spoofing/Tampering/...).
- **CAPEC** — design-level attack pattern catalog.
- **CWE** — code-level weakness taxonomy.
- **ATT&CK** — post-deployment / detection-layer adversary technique catalog.
- **CVSS** — vulnerability severity scoring.
- **Cyber Kill Chain** — adversary lifecycle stages.

You pick a centric method to *generate* threats and then optionally use one of these to *organize, communicate, or score* them. ATT&CK, in particular, is most useful for operational work (detection coverage, threat hunting, IR planning) — it's not the right primary lens for design-time modeling, even though it's sometimes positioned that way.

## What about supply chain and deployment threats?

These get treated as if they need their own methodology. They don't — they're inputs to the methods above that people just forget to enumerate.

- An Artifactory server is **an asset** — threat model it asset-centric.
- A CI/CD pipeline is **a process** — threat model it process-centric.
- A signed-firmware distribution flow is **a flow** — threat model it flow-centric.
- A "build engineer can promote a release" capability is **a user need** — invert it user-needs-centric.

The gap, when it exists, is an inventory problem (you didn't include the build server in your DFD), not a methodological one. The fix is to expand the inventory, not to add a "supply chain threat model" alongside.

## Choosing an entry point

Default for this skill: **flow-centric, with STRIDE-Per-Element as the lens**. It's the one most engineering teams know, OWASP teaches, and that pairs naturally with the DFD output this skill produces.

Supplement with one or more of:

- **Asset-centric** — when you can name a small set of crown jewels. Quick to do alongside flow-centric.
- **User-needs-centric** — when the system has rich business logic. Catches what STRIDE misses.
- **Process-centric** — when operational surface is significant.
- **Code-centric** — at the end, as validation. Always optional but always valuable.

Generally avoid attacker-centric as the primary method outside of nation-state-target contexts. Use it surgically (one attack tree on one high-value target) when it adds value.

The Manifesto's "Multiple representations" pattern means it's *fine* to combine these — and often correct. Just don't produce more documentation than the team will read.
