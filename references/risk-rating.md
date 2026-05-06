# Risk rating

This skill defaults to **qualitative L/M/H ratings** for likelihood and impact, with risk = function(likelihood, impact). That's enough for most threat models and avoids the false-precision trap of numeric models.

## Default: 3×3 qualitative matrix

|              | Impact: Low | Impact: Medium | Impact: High |
|--------------|-------------|----------------|--------------|
| Likelihood: High   | Medium      | High           | Critical     |
| Likelihood: Medium | Low         | Medium         | High         |
| Likelihood: Low    | Low         | Low            | Medium       |

### Likelihood prompts

- Can the attack be performed remotely?
- Does the attacker need authentication?
- Can the attack be automated?
- What skill / resources does the attacker need?
- How exposed is the attack surface (internet, internal network, physical only)?
- Are there existing partial mitigations that reduce likelihood?

### Impact prompts

- Can the attacker take over the system fully? Get admin?
- Can the attacker access sensitive data (PII, PHI, secrets, IP)?
- How many records / users / devices are affected?
- Can the attack cause physical or safety harm? (Especially relevant for medical, automotive, ICS.)
- Is there reputational, legal, or regulatory impact?
- Can the attacker pivot to other systems?

For safety-critical systems (medical devices, automotive, industrial control), bump the impact rating any time patient/operator/public safety is in play. Don't rate patient harm as "Medium impact"; that's almost always High.

## When to use OWASP Risk Rating Methodology instead

If the user wants a more structured numeric model — typically for regulatory submissions, FMEA-style analysis, or alignment with an existing risk register — use OWASP Risk Rating Methodology (OWASP-RR).

OWASP-RR scores threat agent factors (skill, motive, opportunity, size), vulnerability factors (ease of discovery, ease of exploit, awareness, intrusion detection), technical impact factors (loss of confidentiality, integrity, availability, accountability), and business impact factors (financial damage, reputation, non-compliance, privacy violation). Each on a 0–9 scale. Average to get likelihood and impact, combine for risk.

Reference: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology

For medical device submissions specifically, FMEA-style scoring (severity × occurrence × detection) is often expected by quality / regulatory teams. If that's the constraint, mirror the FMEA scale rather than inventing a parallel scoring scheme.

## Why DREAD is discouraged

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) was a common scoring model. Both OWASP and Shostack note that DREAD scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. The qualitative L/M/H matrix above is at least honest about being qualitative.

If a stakeholder insists on DREAD, deliver it but flag the limitation in the model.

## A note on prioritization

Risk rating is for triage, not for replacing engineering judgment. The Manifesto-aligned approach: "ranking should be based on the mathematical product of an identified threat's likelihood and its impact" but in practice "include the work to fix a problem" in the prioritization. A High-risk threat with a $50k fix and a Medium-risk threat with a 1-line fix should usually both get done.

Don't let scoring become the goal. The goal is fixing things.
