# Threat Modeling Manifesto (paraphrased reference)

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
- **Multiple representations** (added by the Manifesto's pattern catalogue, central to this skill's hybrid output) — multiple imperfect views beat one perfect view; context DFD + decomposed DFDs are fine and good.

## Anti-patterns (avoid these)

Definitions from the Manifesto, with the skill-output gloss:

- **Hero threat modeler** — assuming only one person with a special mindset can do this. (Use plain language; assume the development team will read this.)
- **Admiration for the problem** — analyzing endlessly without producing solutions. (Every threat gets a response: Mitigate / Eliminate / Transfer / Accept.)
- **Tendency to overfocus** — too much attention on one adversary, asset, or technique. (Cover the whole DFD before drilling down.)
- **Perfect representation** — refusing to ship until the model is "perfect"; multiple imperfect views beat one perfect view. (Ship a useful approximation; iterate.)
- **Asset rabbit-holing** (skill-specific extension) — long asset lists are usually a distraction. Keep the list to things actually worth protecting; don't pad with abstractions like "company reputation".

## Authors

Zoe Braiterman, Adam Shostack, Jonathan Marcil, Stephen de Vries, Irene Michlin, Kim Wuyts, Robert Hurlbut, Brook S.E. Schoenfield, Fraser Scott, Matthew Coles, Chris Romeo, Alyssa Miller, Izar Tarandach, Avi Douglen, Marc French.

## Citation

If quoting, attribute to "Threat Modeling Manifesto" with the URL https://www.threatmodelingmanifesto.org/. Licensed CC-BY 4.0 — paraphrasing for this skill is fine; substantive direct quotes need attribution.
