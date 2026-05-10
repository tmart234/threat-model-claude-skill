# Risk rating

> **Last verified**: 2026-05. OWASP Risk Rating Methodology, CVSS specification, ISO 14971 (current edition 14971:2019), and AAMI TIR57 (current edition 2023) all update; re-confirm against owasp.org, first.org/cvss, the ISO store, and the AAMI catalog before citing factor lists or version numbers in a regulatory deliverable.
> **Sources paraphrased**: OWASP Risk Rating Methodology (CC-BY 4.0); CVSS specification (FIRST.org, public); Stellios, Kotzanikolaou & Grigoriadis (Computers & Security 107, 2021) — CVV / pruning math (paraphrase, see methodologies.md for full citation); FMEA conventions (public); Adam Shostack and OWASP critiques of DREAD (paraphrase); ISO 14971:2019 — *Medical devices — Application of risk management to medical devices* (proprietary ISO standard, paraphrase only — severity / probability decomposition referenced, no direct quote); AAMI TIR57:2023 — *Principles for medical device security — Risk management* (proprietary AAMI technical information report, paraphrase only — `P1 × P2` cyber-to-safety bridge referenced, no direct quote); US FDA *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023, US government work — joint cyber/safety risk-model expectation cited).

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

### ISO 14971 / AAMI TIR57 mapping for medical-device submissions

For FDA premarket cybersecurity submissions, IEC 81001-5-1, IEC 62304, and MDR / IVDR submissions, the safety bump above is the right *instinct* but not the right *artifact*. Regulators expect cybersecurity risk to be expressed in the same language as patient-safety risk — specifically the **ISO 14971** risk-management framework (severity-of-harm × probability-of-occurrence-of-harm), with **AAMI TIR57** as the bridge that maps cyber threats into the ISO 14971 model. The 2023 FDA *Cybersecurity in Medical Devices* final guidance is explicit that the joint cyber/safety risk model is what reviewers will look for.

Use this mapping to translate the skill's L/M/H ratings into ISO 14971 / TIR57 terms. The mapping is what closes the loop from a STRIDE threat to an ISO 14971 unacceptable-risk finding.

**Severity of harm** (ISO 14971 severity scale — `S1..S5`):

| Markdown impact | ISO 14971 severity | When to pick (clinical / safety lens) |
|---|---|---|
| L | **S1 — Negligible** | Inconvenience, no clinical consequence (e.g. delayed non-urgent report) |
| L (very low) | (use S1) | — |
| M | **S2 — Minor** | Temporary discomfort, no lasting harm, no intervention beyond standard care |
| M (towards H) | **S3 — Serious** | Reversible injury requiring medical intervention; non-permanent harm |
| H | **S4 — Critical** | Permanent injury, life-threatening, major intervention required |
| H (catastrophic) | **S5 — Catastrophic** | Death, permanent disabling injury, multiple-patient harm |

**Probability of occurrence of harm** under TIR57 decomposes into two stages — `P1 × P2`:

- **P1** — probability that the cybersecurity threat materializes (a hazardous *situation* arises). This is roughly the L/M/H *Likelihood* column from the matrix above. A network-reachable, weakly-authenticated, easily-automated attack scores P1 high; a physical-access-only attack on a tamper-resistant device scores P1 low.
- **P2** — probability that, given the hazardous situation, harm to the patient *actually occurs*. This is what the safety case calculates — interlocks, alarms, clinician oversight, mechanical limits. A dosing-setpoint tamper with a hard mechanical flow limit and an alarm scores P2 low even when P1 is high; the same tamper on a device whose only check is software-based scores P2 high.

Total probability of harm `P = P1 × P2`. The TIR57 framing makes the safety case's mitigations (interlocks, alarms, clinician-in-the-loop) visible as P2-reducers — which is what a reviewer will look for when they ask "if the attacker wins the cyber argument, does the safety case still hold?"

| Probability range | TIR57 / ISO 14971 band | Skill-side interpretation |
|---|---|---|
| Frequent | High `P1` × high `P2` | Safety case has no working barrier against a likely attack — unacceptable on its face |
| Probable / Occasional | Mixed | The triage band: safety mitigations exist but may be defeatable; argue case-by-case |
| Remote / Improbable | Low `P1` × low `P2` | Safety case demonstrably reduces residual risk; document the barrier explicitly |

ISO 14971 risk = `severity × probability`; an "unacceptable risk" classification typically falls out of the upper-right region (high severity × frequent / probable probability). The exact band thresholds are an organization-level / submission-level decision, not a skill default — record the team's band rules in §1 prose.

**Worked example — Tampering threat on a dosing setpoint.**

A Tampering threat on an infusion pump's dosing-setpoint flow over the management VLAN. Skill-side rating: `Likelihood: H` (network-reachable, no mTLS, public exploit for a similar pump exists), `Impact: H (catastrophic — overdose risk)`.

Translated to ISO 14971 / TIR57:

- **Severity**: S5 (Catastrophic) — overdose can cause death.
- **P1**: high — the cyber threat is feasible and demonstrated.
- **P2**: depends on the safety case. If the pump has a hard mechanical flow limit independent of the software setpoint, and the dose-rate alarm is reliably annunciated to clinical staff, P2 is *low* — the safety case absorbs the cyber compromise. If neither barrier is present (software-only enforcement), P2 is *high* — the cyber compromise reaches the patient.
- **Combined**: with safety-case barriers, `S5 × low-P` lands in the `Probable / Occasional` × `S5` band — *still requires mitigation* under most submission frameworks, but the residual is bounded. Without safety-case barriers, `S5 × high-P` is unacceptable on its face.

The point: the cybersecurity-risk score (`Likelihood: H, Impact: H, Risk: High`) doesn't tell the regulator whether the device is acceptable — the ISO 14971 view does. **For medical-device submissions, every safety-bumped threat in §3 should carry both ratings**: the skill's L/M/H for the engineering-team triage view, and the `severity × P1 × P2` for the regulator-facing safety-case view. Worked alongside the Risk register in §3, this is the single artifact FDA reviewers thumb through to confirm cybersecurity has been integrated with the ISO 14971 risk file rather than bolted on.

**Cross-references**: `references/medical.md` § "Regulators and threat-intel sources" lists the underlying standards. `references/stpa.md` is the right home when the *control loop itself* is the analysis target — STPA generates losses → hazards → constraints in a form that maps directly into ISO 14971 hazardous situations. The combination of STPA hazards + TIR57 cyber-to-safety mapping is what the joint IEC 62304 / IEC 81001-5-1 artifact actually contains.

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

## L/M/H ↔ TM-BOM enums

The OWASP Threat Model Library schema (used as this skill's TM-BOM — see `SKILL.md` § "Producing the TM-BOM") uses 5-level enums for `risks[].likelihood` and `risks[].impact`, a 6-level enum for `risks[].level`, and an integer 0–25 `risks[].score`. The skill's L/M/H markdown values must translate to these schema enums when emitting the TM-BOM. Use this table — it's the canonical mapping; don't improvise.

### Likelihood (markdown L/M/H → schema enum)

| Markdown | Schema `likelihood` | When to pick |
|---|---|---|
| L | `unlikely` | Default for L; needs unusual conditions / specific access |
| L (very low) | `rare` | Only when the prerequisites are extreme (physical access to a secured facility, nation-state-only capability) |
| M | `possible` | Default for M; achievable by a moderately skilled attacker |
| H | `likely` | Default for H; commodity attack, internet-facing surface, no auth or weak auth |
| H (near-certain) | `certain` | Reserve for "this is happening today" — observed in production logs, public PoC against this version |

### Impact (markdown L/M/H → schema enum)

| Markdown | Schema `impact` | When to pick |
|---|---|---|
| L | `minor` | Default for L; nuisance, recoverable without external coordination |
| L (very low) | `negligible` | Only for losses with no business / regulatory / safety consequence |
| M | `moderate` | Default for M; one-team-day to recover, no regulatory notification |
| H | `major` | Default for H; PHI/PII breach, multi-team-day recovery, regulator notification |
| H (catastrophic) | `severe` | Reserve for safety-critical outcomes (patient harm, operator injury), unrecoverable data loss, or business-existential reputational hits |

The "very low" and "near-certain" / "catastrophic" sub-buckets are how the 3-level skill scale projects onto the schema's 5-level scale. Most threats land squarely in the default mapping; reserve the edge enums for genuine outliers and document the choice in §1 prose.

### Risk level (matrix output → schema `level`)

| Matrix output | Schema `level` |
|---|---|
| Low | `low` (or `very_low` if both likelihood and impact are at the bottom of the 5-level scale) |
| Medium | `medium` |
| High | `high` (or `very_high` if both likelihood and impact land in the top two enum buckets, e.g. `likely + major`) |
| Critical (per the §1 promotion rule) | `critical` |

### Risk score (integer 0–25)

The schema's `score` is `likelihood_index × impact_index` where each index is 1..5 in the order shown above (`rare=1, unlikely=2, possible=3, likely=4, certain=5`; `negligible=1, minor=2, moderate=3, major=4, severe=5`). Compute and emit:

| Likelihood × Impact | Score |
|---|---|
| `unlikely × moderate` | 2 × 3 = 6 |
| `possible × major` | 3 × 4 = 12 |
| `likely × major` | 4 × 4 = 16 |
| `likely × severe` | 4 × 5 = 20 |
| `certain × severe` | 5 × 5 = 25 |

The score is a derived field — recompute it from likelihood/impact rather than asking the user. Validate it against `level` (e.g. `score ≥ 16` should typically have `level: high` or higher).

### Worked translation

A flow-centric threat rated `Likelihood: M`, `Impact: H` in the markdown table → matrix output `High` → TM-BOM:

```json
{
  "symbolic_name": "t-1",
  "likelihood": "possible",
  "impact": "major",
  "score": 12,
  "level": "high"
}
```

A safety-bumped control-loop threat rated `Likelihood: M`, `Impact: H` (catastrophic — patient harm) → matrix output `High` (or `Critical` if the §1 promotion rule applies) → TM-BOM:

```json
{
  "symbolic_name": "t-7",
  "likelihood": "possible",
  "impact": "severe",
  "score": 15,
  "level": "very_high"
}
```

## Why DREAD is discouraged

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) was a common scoring model. Both OWASP and Shostack note that DREAD scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. The qualitative L/M/H matrix above is at least honest about being qualitative.

If a stakeholder insists on DREAD, deliver it but flag the limitation in the model.

## A note on prioritization

Risk rating is for triage, not for replacing engineering judgment. The Manifesto-aligned approach: "ranking should be based on the mathematical product of an identified threat's likelihood and its impact" but in practice "include the work to fix a problem" in the prioritization. A High-risk threat with a $50k fix and a Medium-risk threat with a 1-line fix should usually both get done.

Don't let scoring become the goal. The goal is fixing things.
