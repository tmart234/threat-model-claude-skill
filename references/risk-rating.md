# Risk rating

> **Last verified**: 2026-05. OWASP Risk Rating Methodology and CVSS specification versions both update; re-confirm against owasp.org and first.org/cvss before citing factor lists or version numbers in a deliverable.
> **Sources paraphrased**: OWASP Risk Rating Methodology (CC-BY 4.0); CVSS specification (FIRST.org, public); Stellios, Kotzanikolaou & Grigoriadis (Computers & Security 107, 2021) — CVV / pruning math (paraphrase, see methodologies.md for full citation); FMEA conventions (public); Adam Shostack and OWASP critiques of DREAD (paraphrase).

> **Related**: ← `SKILL.md` • `methodologies.md` (the L/M/H scale spans all three strata in the hybrid) • `validation.md` (Q4 cross-stratum check: one risk-rating scale across the whole model).

This skill defaults to **qualitative L/M/H ratings** for likelihood and impact. Risk is L/M/H, derived from the 3×3 matrix below. That's enough for most threat models and avoids the false-precision trap of numeric models.

## Default: 3×3 qualitative matrix (L/M/H output)

|              | Impact: Low | Impact: Medium | Impact: High |
|--------------|-------------|----------------|--------------|
| Likelihood: High   | Medium      | High           | High         |
| Likelihood: Medium | Low         | Medium         | High         |
| Likelihood: Low    | Low         | Low            | Medium       |

The risk column in the threat table takes one of `Low / Medium / High` — same scale across §2.1, §2.2, §2.3, and §3 so the prioritized list sorts cleanly.

**Adding a fourth `Critical` tier (optional).** Some teams or regulators want a separate top tier for "drop-everything-now" findings (safety-critical control loops, near-certain catastrophic loss). When that's needed, promote `High + High` to `Critical` and document the promotion rule in §1 prose so the rating stays auditable. Don't introduce `Critical` silently — it inflates everything if not anchored to a concrete promotion rule.

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

### Safety bump

For safety-critical systems (medical devices, automotive, industrial control, aerospace, robotics), bump the impact rating any time patient / operator / public safety is in play. Don't rate physical-harm scenarios as "Medium impact"; that's almost always High. The bump applies whether the harm is intentional (adversarial) or non-adversarial (sensor failure cascading to harm) — the safety case doesn't care which.

## When to use OWASP Risk Rating Methodology instead

If the user wants a more structured numeric model — typically for regulatory submissions, FMEA-style analysis, or alignment with an existing risk register — use OWASP Risk Rating Methodology (OWASP-RR).

OWASP-RR scores threat agent factors (skill, motive, opportunity, size), vulnerability factors (ease of discovery, ease of exploit, awareness, intrusion detection), technical impact factors (loss of confidentiality, integrity, availability, accountability), and business impact factors (financial damage, reputation, non-compliance, privacy violation). Each on a 0–9 scale. Average to get likelihood and impact, combine for risk.

Reference: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology

For medical device submissions specifically, FMEA-style scoring (severity × occurrence × detection) is often expected by quality / regulatory teams. If that's the constraint, mirror the FMEA scale rather than inventing a parallel scoring scheme.

## CVSS-based attack-path risk (cyber-physical IoT)

Use this **only** when the threat being rated is a composed *path* across several devices in one physical space (multi-device IoMT room, plant-floor cabinet, robotics cell, smart building zone) and the composition is what makes it dangerous — no single hop scores high under per-element OWASP-RR, so OWASP-RR under-rates the chain. This is a path-scoring supplement, not a third numeric option for ordinary single-element threats; for those, stick with L/M/H or OWASP-RR.

Each hop carries a CVSS base score on the standard 0.0–10.0 scale (or a CVV — Cyber-physical Vulnerability Vector — extending CVSS with physical-interaction parameters per Stellios et al. 2021). The path's per-hop scores are normalized to the unit interval (`s_i = CVSS_i / 10`) and the **path score is the product of per-hop normalized scores**: `S_path = ∏ (CVSS_i / 10)`. Lower products mean less feasible composed paths.

**Default pruning threshold**: drop any path whose product falls below `0.05` *before* expanding the next hop. This is the value Stellios et al. demonstrate keeps the working set small enough for human review on IoMT-room-scale graphs (single-digit / low-tens of paths rather than thousands). Adjust upward (e.g. `0.10`) when the analyst wants only the most-feasible chains, or downward (e.g. `0.01`) when the system has high consequence and the team is willing to review more paths. **State the threshold explicitly in §1 prose** so the artifact is auditable — e.g. "Path-risk pruning threshold: `S_path ≥ 0.05` (Stellios default)."

**Worked example — three-hop IoMT path** (rogue BLE peripheral → Wi-Fi AP → infusion pump):

| Hop | Surface | CVSS | Normalized |
|---|---|---|---|
| H1 | P3 — rogue BLE peripheral DoSing the 2.4 GHz band shared with the AP | 6.5 | 0.65 |
| H2 | P2 — degraded AP retransmits expose pump's mTLS-less management VLAN | 7.5 | 0.75 |
| H3 | P1 — exposed serial console on the pump accepts unauthenticated commands | 8.8 | 0.88 |

`S_path = 0.65 × 0.75 × 0.88 ≈ 0.43`. Above the `0.05` default threshold by an order of magnitude — keep the path and triage it.

A second candidate path that scores `0.65 × 0.30 × 0.20 ≈ 0.039` falls below the default threshold and is dropped before further expansion (any deeper hops can only make the product smaller, since each `s_i ≤ 1`). The pruning argument is exactly that: the product is monotone-non-increasing in path length, so a sub-threshold prefix never produces a super-threshold extension.

The per-element OWASP-RR table still lists the individual flaws; the path-risk table lists the composed attack chains and is what gets triaged. Construction technique: `methodologies.md` § "Risk-prioritized cyber-physical attack paths".

## Why DREAD is discouraged

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) was a common scoring model. Both OWASP and Shostack note that DREAD scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. The qualitative L/M/H matrix above is at least honest about being qualitative.

If a stakeholder insists on DREAD, deliver it but flag the limitation in the model.

## A note on prioritization

Risk rating is for triage, not for replacing engineering judgment. The Manifesto-aligned approach: "ranking should be based on the mathematical product of an identified threat's likelihood and its impact" but in practice "include the work to fix a problem" in the prioritization. A High-risk threat with a $50k fix and a Medium-risk threat with a 1-line fix should usually both get done.

Don't let scoring become the goal. The goal is fixing things.
