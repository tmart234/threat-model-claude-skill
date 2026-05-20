# Risk rating

The default here is **qualitative L/M/H** for likelihood and impact, with risk a function of the two. That's enough for most threat models and avoids the false precision of numeric models.

## Default: 3×3 qualitative matrix

|                    | Impact: Low | Impact: Medium | Impact: High |
|--------------------|-------------|----------------|--------------|
| Likelihood: High   | Medium      | High           | Critical     |
| Likelihood: Medium | Low         | Medium         | High         |
| Likelihood: Low    | Low         | Low            | Medium       |

### Likelihood prompts

- Can the attack be performed remotely?
- Does the attacker need authentication?
- Can it be automated?
- What skill or resources does it take?
- How exposed is the surface — internet, internal network, physical only?
- Do existing partial mitigations reduce likelihood?

### Impact prompts

- Can the attacker take over the system, or get admin?
- Can they reach sensitive data (PII, secrets, IP)?
- How many records, users, or devices are affected?
- Can the attack cause physical or safety harm?
- Is there reputational, legal, or regulatory impact?
- Can the attacker pivot to other systems?

### The safety bump

For systems where an attack can hurt a person — medical devices, automotive, industrial control, anything controlling a physical process — bump the impact rating whenever safety is in play. Don't rate physical harm to a person as "Medium"; that is almost always High, regardless of how unlikely the attack is.

## When to use OWASP Risk Rating instead

If the user wants a structured numeric model — typically for a regulatory submission, FMEA-style analysis, or alignment with an existing risk register — use the **OWASP Risk Rating Methodology**. It scores threat-agent factors, vulnerability factors, technical-impact factors, and business-impact factors, each 0–9, then combines them into likelihood and impact.

Reference: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology

For regulated submissions that already mandate FMEA-style scoring (severity × occurrence × detection), mirror that scale rather than inventing a parallel one.

## Why DREAD is discouraged

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. Both OWASP and Shostack note this. The qualitative L/M/H matrix is at least honest about being qualitative. If a stakeholder insists on DREAD, deliver it but flag the limitation.

## Prioritization is for triage, not for replacing judgment

Rank roughly by likelihood × impact, but factor in the cost to fix. A High-risk threat with a $50k fix and a Medium-risk threat with a one-line fix should usually both get done — do the cheap one immediately. Don't let scoring become the goal; the goal is fixing things.
