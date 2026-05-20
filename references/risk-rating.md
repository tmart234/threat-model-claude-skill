# Risk rating

> **Last verified**: 2026-05. OWASP Risk Rating Methodology, CVSS specification, ISO 14971 (current edition 14971:2019), and AAMI TIR57 (current edition 2023) all update; re-confirm against owasp.org, first.org/cvss, the ISO store, and the AAMI catalog before citing factor lists or version numbers in a regulatory deliverable.
> **Sources paraphrased**: OWASP Risk Rating Methodology (CC-BY 4.0); CVSS specification (FIRST.org, public); MITRE CVSS Rubric for Medical Devices — methods only, applied to current-version CVSS (MITRE, public; github.com/mitre/md-cvss-rubric-tools); Stellios, Kotzanikolaou & Grigoriadis (Computers & Security 107, 2021) — CVV / pruning math (paraphrase, see methodologies.md for full citation); FMEA conventions (public); Adam Shostack and OWASP critiques of DREAD (paraphrase); ISO 14971:2019 — *Medical devices — Application of risk management to medical devices* (proprietary ISO standard, paraphrase only — severity / probability decomposition referenced, no direct quote); AAMI TIR57:2023 — *Principles for medical device security — Risk management* (proprietary AAMI technical information report, paraphrase only — `P1 × P2` cyber-to-safety bridge referenced, no direct quote); US FDA *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023, US government work — joint cyber/safety risk-model expectation cited).

> **Related**: ← `SKILL.md` § "Threat enumeration" (the row format these columns populate) • `methodologies.md` (the AV/PR/AC + CIA scheme spans all three strata) • `validation.md` (Q4 cross-stratum check: every row uses the same enums) • `stpa.md` (where physical-harm severity lives — not on the threat row).

This skill rates threats with **CVSS 3.1 intrinsic exposure (`AV / PR / AC`) plus a presence list of impacted security properties (`C / I / A`)** — three CVSS-derived enums that engineers already know, plus a flag set per row. No L/M/H, no compound "Risk" column, no row-level safety bump. Safety severity lives in the STPA section, cross-referenced from the threat row.

## Default scheme: AV / PR / AC + CIA presence

The `Threat` rows in §2.1 (and §2.2 / §2.3 if produced) carry these columns. Pick the values once per row; they're reused downstream — §3 sort, TM-BOM emission, and the CVSS calculator if a numeric score is required.

**Attack Vector (`AV`) — how the attacker reaches the element**

| Value | Meaning | Typical example |
|---|---|---|
| `N` | **Network** — reachable across an L3 boundary (Internet or routed VPN) | Public REST endpoint; cross-VPC service mesh; internet-exposed MQTT broker |
| `A` | **Adjacent** — reachable from the same L2 segment / Bluetooth Classic / facility-wide Wi-Fi | On-segment ARP spoofing; rogue AP within facility coverage; CAN bus on the same vehicle |
| `L` | **Local** — requires a shell / process on the host or short-range RF (BLE LE, NFC, inductive ≈ 10 ft) | Privilege escalation from an authenticated session; exploit via BLE in the patient's room |
| `P` | **Physical** — requires bodily contact with the device | Debug-port probe; chip-off; USB plugin requiring case-open |

The MITRE CVSS Rubric for Medical Devices' wireless-range distinction applies (§ "Clinical-context CVSS scoring" below): short-range BLE / NFC / Zigbee → `L`, facility-wide Wi-Fi / Bluetooth Classic → `A`.

**Privileges Required (`PR`) — what credentials/role the attacker needs *before* the attack**

| Value | Meaning |
|---|---|
| `N` | None — anonymous attacker |
| `L` | Low — any authenticated user (regular tenant, logged-in clinician, normal device user) |
| `H` | High — administrator / service account / privileged role |

**Attack Complexity (`AC`) — conditions beyond the attacker's control**

| Value | Meaning |
|---|---|
| `L` | Low — repeatable; the attacker can carry out the attack at will |
| `H` | High — requires preconditions the attacker doesn't control (race window, specific target configuration, victim interaction beyond a click, defender misstep) |

`AC:H` is the *exception*, not the default. If the attacker can run the attack on demand, it's `AC:L` even when the attack is technically sophisticated. CVSS-rubric guidance applies: don't credit obscurity of proprietary protocols, hard-coded keys, or undocumented service commands as `AC:H`.

**Impact — which CIA properties are violated**

`Impact` is a comma-separated subset of `C / I / A` (Confidentiality, Integrity, Availability). At least one — a row with no impact isn't a threat. Repudiation threats violate `I` (audit-trail integrity); Spoofing almost always includes `I` (the spoofed identity is now wrong); DoS is `A`.

**Safety hazards** (physical harm to people, equipment, or the environment) are *not* captured here. They live in the STPA section as `H-#` IDs with their own severity scale (`references/stpa.md`). Cross-reference the hazard ID in the threat row's `Element` column when a threat triggers a hazard, e.g. *"Auth Service (P1) → H-2"*.

## §3 sort heuristic — which threats triage first

Without a single Risk column, §3 prioritization sorts on the row dimensions in fixed order. Most-exposed → least-exposed first, then by impact breadth:

1. **`AV`** — `N > A > L > P` (network-reachable > on-segment > host-local > physical contact required)
2. **`PR`** — `N > L > H` (anonymous > authenticated user > admin)
3. **`AC`** — `L > H` (can run at will > needs a precondition)
4. **Impact-set size** — `{C,I,A}` > 2-hits > 1-hit
5. **STPA hazard linkage** — threats that reference an `H-#` from the STPA section sort above non-safety threats with otherwise identical exposure / impact (the hazard severity tie-breaks).

Ties beyond rule 5 are reviewer judgment — record the call in §3 prose so the order is auditable. Cost of fix is factored *after* the technical sort (a top-exposure threat with a $50k fix and a mid-exposure threat with a 1-line fix often both get done — see § "A note on prioritization" below).

## Translation to TM-BOM enums

The OWASP Threat Model Library schema (`tm-bom.md`) requires `risks[].likelihood / impact / level` enums and an integer 0–25 `score`. These are derived at TM-BOM emission from the row's `AV / PR / AC / Impact` — no separate per-row judgment.

**Likelihood — from `AV + PR + AC` points**

| Axis | Value → points |
|---|---|
| `AV` | `N`=4, `A`=3, `L`=2, `P`=1 |
| `PR` | `N`=3, `L`=2, `H`=1 |
| `AC` | `L`=2, `H`=1 |

| Sum | Schema `likelihood` |
|---|---|
| 9 | `certain` (`AV:N + PR:N + AC:L` — anonymous, network-reachable, repeatable) |
| 7–8 | `likely` |
| 5–6 | `possible` |
| 4 | `unlikely` |
| 3 | `rare` (`AV:P + PR:H + AC:H` — physical, admin, conditional) |

**Impact — from CIA-hit count + system criticality**

| CIA-hit count | Default schema `impact` | Promote to `major` / `severe` when … |
|---|---|---|
| 1 | `minor` | data class is `phi` / `pci` / `cred` / `gov`; `scope.tier` = `mission_critical`; STPA hazard cross-ref present |
| 2 | `moderate` | as above |
| 3 (`C, I, A` full set) | `major` | STPA hazard severity is `Catastrophic`, or `scope.business_criticality` = `maximal` → `severe` |

`negligible` is reserved for findings with no business / regulatory / safety consequence (rare).

**Level — combined band**

| `likelihood` × `impact` | Schema `level` |
|---|---|
| `rare` / `unlikely` × `negligible` / `minor` | `very_low` |
| `unlikely` × `moderate`; `possible` × `minor` | `low` |
| `possible` × `moderate`; `likely` × `minor` | `medium` |
| `possible` × `major`; `likely` × `moderate` | `high` |
| `likely` × `major`; `certain` × `moderate` / `major` | `very_high` |
| `likely` / `certain` × `severe` | `critical` |

**Score — integer 0–25**

`score = likelihood_index × impact_index` where each index is 1..5 in enum order (`rare=1 … certain=5`; `negligible=1 … severe=5`). Recompute at emission; validate consistency with `level` (e.g. `score ≥ 16` should typically have `level: high` or higher).

### Worked translation

A flow-centric threat with `AV:N / PR:N / AC:L / Impact: C, I` on a system where `scope.tier = business_critical`:

- `likelihood` sum = 4 + 3 + 2 = 9 → `certain`
- `impact` from 2 CIA hits, no tier promotion → `moderate`
- `level` = `certain × moderate` → `very_high`
- `score` = 5 × 3 = 15

```json
{
  "symbolic_name": "t-1",
  "likelihood": "certain",
  "impact": "moderate",
  "score": 15,
  "level": "very_high"
}
```

A control-loop threat with `AV:A / PR:N / AC:L / Impact: I, A` cross-referenced to STPA `H-2` (Catastrophic):

- `likelihood` sum = 3 + 3 + 2 = 8 → `likely`
- `impact` from 2 CIA hits + STPA Catastrophic hazard → `severe`
- `level` = `likely × severe` → `critical`
- `score` = 4 × 5 = 20

## Safety severity — STPA, not the threat row

Physical-harm severity isn't encoded on the threat row. The STPA section (`references/stpa.md`) carries the system's losses → hazards → constraints chain with severity at the hazard level. When a threat triggers a hazard, the threat row references the `H-#` in its `Element` column; the safety case is read by composing the threat's exposure (`AV / PR / AC`) with the hazard's severity, not by adding a Severity column to every row.

This separation is exactly what running STPA *and* STRIDE buys you: STPA produces hazards independent of attacker intent (a sensor failure causes the same hazard a tampering attack does), and the cyber row records only the cyber half. The composition into ISO 14971 / TIR57 risk language happens in the medical-device-submission view below.

## ISO 14971 / AAMI TIR57 mapping for medical-device submissions

For FDA premarket cybersecurity submissions, IEC 81001-5-1, IEC 62304, and MDR / IVDR submissions, regulators expect cybersecurity risk to be expressed in the same language as patient-safety risk — specifically the **ISO 14971** risk-management framework (severity-of-harm × probability-of-occurrence-of-harm), with **AAMI TIR57** as the bridge that maps cyber threats into the ISO 14971 model. The 2023 FDA *Cybersecurity in Medical Devices* final guidance is explicit that the joint cyber/safety risk model is what reviewers will look for.

Under the AV/PR/AC + STPA scheme, the mapping reads off the row + hazard pair — no row-level safety bump needed.

**Severity of harm** (ISO 14971 `S1..S5`) — comes from the STPA hazard the threat references (`H-#`). STPA hazards already carry severity in the STPA section; reuse the value, don't re-derive. Typical mapping (full guidance: `references/stpa.md`):

| ISO 14971 severity | When the hazard scope warrants it |
|---|---|
| **S1 — Negligible** | Inconvenience, no clinical consequence (e.g. delayed non-urgent report) |
| **S2 — Minor** | Temporary discomfort, no lasting harm, no intervention beyond standard care |
| **S3 — Serious** | Reversible injury requiring medical intervention; non-permanent harm |
| **S4 — Critical** | Permanent injury, life-threatening, major intervention required |
| **S5 — Catastrophic** | Death, permanent disabling injury, multiple-patient harm |

**Probability of occurrence of harm** decomposes per TIR57 into two stages — `P1 × P2`:

- **P1** — probability that the cybersecurity threat materializes (a hazardous *situation* arises). Read directly from the §2.1 row's `AV / PR / AC` translated to `likelihood` per the table above (`certain` → high P1; `rare` → low P1). A network-reachable, weakly-authenticated, easily-automated attack (`AV:N + PR:N + AC:L` → sum 9 → `certain`) scores P1 high; a physical-access-only attack on a tamper-resistant device (`AV:P + PR:H + AC:H` → sum 3 → `rare`) scores P1 low.
- **P2** — probability that, given the hazardous situation, harm to the patient *actually occurs*. Read from the STPA hazard's documented control-loop barriers (interlocks, alarms, clinician oversight, mechanical limits). A dosing-setpoint tamper (`H-2`) with a hard mechanical flow limit and an annunciated dose-rate alarm scores P2 low even when P1 is high.

Total probability of harm `P = P1 × P2`. The TIR57 framing makes the safety case's mitigations visible as P2-reducers — what a reviewer looks for when they ask "if the attacker wins the cyber argument, does the safety case still hold?"

| Probability range | TIR57 / ISO 14971 band | Skill-side interpretation |
|---|---|---|
| Frequent | high `P1` × high `P2` | Safety case has no working barrier against a likely attack — unacceptable on its face |
| Probable / Occasional | mixed | Triage band: safety mitigations exist but may be defeatable; argue case-by-case |
| Remote / Improbable | low `P1` × low `P2` | Safety case demonstrably reduces residual risk; document the barrier explicitly |

ISO 14971 risk = `severity × probability`; an "unacceptable risk" classification typically falls out of the upper-right region (high severity × frequent / probable probability). The exact band thresholds are an organization-level / submission-level decision — record the team's band rules in §1 prose.

**Worked example — Tampering threat on a dosing setpoint.** A Tampering threat on an infusion pump's dosing-setpoint flow over the management VLAN. §2.1 row: `AV:A / PR:N / AC:L / Impact: I, A` cross-referenced to STPA `H-2` (Catastrophic — overdose).

- **Severity** (from STPA `H-2`): S5 — Catastrophic.
- **P1**: row `likelihood` = `likely` (sum 3 + 3 + 2 = 8) → high P1.
- **P2**: depends on STPA control-loop barriers. With a hard mechanical flow limit independent of the software setpoint *and* a reliably-annunciated dose-rate alarm, P2 is *low* — the safety case absorbs the cyber compromise. With neither barrier (software-only enforcement), P2 is *high* — the cyber compromise reaches the patient.
- **Combined**: with barriers, `S5 × low-P` lands in `Probable / Occasional × S5` — *still requires mitigation* under most submission frameworks, but residual is bounded. Without barriers, `S5 × high-P` is unacceptable on its face.

The point: the cyber row alone (`AV:A / PR:N / AC:L / Impact: I, A`) doesn't tell the regulator whether the device is acceptable — the ISO 14971 view does. **For medical-device submissions, every threat that references an STPA `H-#` carries both readings**: the §2.1 row for the engineering-team triage view, and `severity × P1 × P2` for the regulator-facing safety-case view. Worked alongside the §3 risk register, this is the single artifact FDA reviewers thumb through to confirm cybersecurity has been integrated with the ISO 14971 risk file rather than bolted on.

**Table shape for §3.** Emit one row per `H-#`-linked threat alongside the §3 risk register:

| Threat | Hazard (STPA `H-#`) | Severity | P1 | P2 | Barriers reducing P2 | Combined band |
|---|---|---|---|---|---|---|
| T7 | H-2 [L-1] — dosing setpoint exceeds prescribed rate | S5 | High | Low | Mechanical flow limit + annunciated dose-rate alarm | Probable/Occasional × S5 — bounded; mitigation still required |

**Cross-references**: `references/medical.md` § "Regulators and threat-intel sources" lists the underlying standards. `references/stpa.md` is the right home when the *control loop itself* is the analysis target — STPA generates losses → hazards → constraints in a form that maps directly into ISO 14971 hazardous situations. The combination of STPA hazards + TIR57 cyber-to-safety mapping is what the joint IEC 62304 / IEC 81001-5-1 artifact actually contains.

## When to use OWASP Risk Rating Methodology instead

If the user wants a more structured numeric model — typically for regulatory submissions, FMEA-style analysis, or alignment with an existing risk register — use OWASP Risk Rating Methodology (OWASP-RR).

OWASP-RR scores threat agent factors (skill, motive, opportunity, size), vulnerability factors (ease of discovery, ease of exploit, awareness, intrusion detection), technical impact factors (loss of confidentiality, integrity, availability, accountability), and business impact factors (financial damage, reputation, non-compliance, privacy violation). Each on a 0–9 scale. Average to get likelihood and impact, combine for risk.

Reference: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology

For medical device submissions specifically, FMEA-style scoring (severity × occurrence × detection) is often expected by quality / regulatory teams. If that's the constraint, mirror the FMEA scale rather than inventing a parallel scoring scheme.

## Clinical-context CVSS scoring (rubric methods, current CVSS)

For medical-device CVSS scoring (premarket, post-market disclosure, CISA advisories, inter-vendor comparison), generic enterprise-IT scoring under-fits the clinical context. This skill adopts the methods of the MITRE *Rubric for Applying CVSS to Medical Devices* (github.com/mitre/md-cvss-rubric-tools) — its per-metric decision trees and clinical questions — and applies them at the CVSS version the team scores in (3.1 or 4.0). Calculators: NVD (`nvd.nist.gov/vuln-metrics/cvss/v3-calculator` or `/v4-calculator`) or FIRST (`first.org/cvss/calculator/3.1` or `/4.0`). Don't use the rubric's own calculator — it's locked to v3.0.

**The rubric's clinical-context contributions, distilled** (version-agnostic — these are what survive across 3.1 and 4.0):

| Metric | Rubric's clinical insight | Why generic CVSS misses it |
|---|---|---|
| **Attack Vector** | Wireless range ≈ 10 ft (BLE LE, NFC, Zigbee, inductive) → *Local*, not *Adjacent*; facility-wide Wi-Fi / Bluetooth Classic → *Adjacent* | Generic CVSS treats all wireless as *Adjacent*, conflating an attacker in the patient's room with one anywhere on campus |
| **Attack Vector** | Hospital-internal management VLAN is still *Network* per CVSS — record VLAN segmentation under **Modified Attack Vector (MAV)** to downgrade to *Adjacent*; segmentation is not a base-score discount | Generic scoring either ignores segmentation (over-scores) or silently downgrades the base (under-scores and breaks comparability) |
| **Attack Complexity** | Assume full attacker knowledge of proprietary protocols, hard-coded keys, service manuals — don't credit obscurity of DICOM PDU formats, vendor-specific BLE profiles, or undocumented service-port commands | Medical-device threat models routinely score *High* AC because "the protocol is proprietary"; the rubric rules that out as a base-score factor |
| **Privileges Required** | One-role-fits-all devices (most legacy / embedded clinical hardware) score `PR:N` — there is no privilege to require | Generic CVSS lets analysts claim `PR:L` for "the device has a login screen," even when the login is shared and posted on the device |
| **User Interaction** | A clinician confirmation dialog counts as `UI:R` only if it *meaningfully gates the attack* — a habituated click-through alert may not | Generic scoring credits any prompt as `UI:R`; the rubric forces the analyst to ask whether the prompt would actually stop the attack |
| **Scope (3.1) / SC-SI-SA (4.0)** | Programmer/monitor that can reflash an on-body device → *Scope:Changed* (3.1) or non-zero `SC/SI/SA` (4.0); a read-only home monitor → *Scope:Unchanged* unless it shares reprogramming code with a clinician programmer | Generic scoring marks all device-to-device flows scope-changed or none; the rubric's "vulnerable vs impacted component, distinct authorities" test resolves it |
| **CIA / VC-VI-VA** | Score against six clinical data categories (PHI/PII, diagnosis/monitoring, **therapy delivery**, clinical workflow, system/credential, other); worst category wins; *therapy delivery* High routinely means *Integrity:High* regardless of byte count | Generic CVSS asks "is data confidentiality affected" without distinguishing a tampered drug-library entry from a tampered log timestamp |
| **Environmental: CR / IR / AR** | Set *High* whenever loss of CIA could cause delayed therapy, incorrect therapy, or PHI breach | Generic scoring leaves CR/IR/AR at default (*Medium*) for medical devices |

The rubric's branch numbers are 3.0-specific; the *questions* are not. Walk the rubric document's decision trees, map to the metric names at your CVSS version, record the vector and a one-line rubric trace in §1 prose. Example: `"CVSS:3.1/AV:A/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H — 8.3 (High); AV from short-range BLE; CIA Highs from therapy-delivery category."`

**CVSS 4.0-specific — what the rubric couldn't say in 3.0**:

The rubric document flags v3.0 gaps that matter for medical devices: no direct expression of safety impact, no provider-urgency channel, no way to encode patient-population reach. CVSS 4.0's **Supplemental** group closes most of them — score these when using 4.0:

- **Safety (S)** — `Present` whenever the rubric's therapy-delivery, diagnosis, or monitoring I/A scored *High*. **MSI:S / MSA:S** carries the same signal for subsequent-system impact (programmer compromise reaching an on-body device).
- **Automatable (AU)** — `Yes` for network-reachable, no-UI-required attacks (matches rubric's `UI:N + AV:N` branches).
- **Recovery (R)** — `Irrecoverable` for firmware-bricking or implant-reprogramming; `User` for clinical re-provisioning; `Automatic` rarely applies.
- **Provider Urgency (U)** — TLP-style flag (Red / Amber / Green / Clear); the rubric's official-fix question maps here.

Supplemental metrics don't change the numeric score but travel with the vector — record them in §1 prose.

**Hybrid safety blend (STPA + TIR57 lens over the rubric's CIA questions)**:

The rubric scores *cybersecurity* severity. The patient-safety side belongs in the ISO 14971 / TIR57 mapping documented above (`severity × P1 × P2`) and, when the control loop is the analysis target, in the STPA losses → hazards → constraints chain (`references/stpa.md`). The three views compose:

| View | What it answers | Drives |
|---|---|---|
| Clinical-context CVSS (rubric methods) | How severe is the *cyber* weakness in the clinical deployment? | Advisory disclosure, premarket cyber severity, inter-vendor comparison |
| TIR57 `severity × P1 × P2` | If the attacker wins, does *harm* reach the patient? | ISO 14971 risk file, joint cyber/safety acceptability |
| STPA hazards → constraints | What *control-loop* failures (cyber or otherwise) produce the hazard? | Safety requirements covering both attack and non-adversarial failure |

For every threat that references an STPA `H-#`: score CVSS with the rubric's methods, carry `severity × P1 × P2` alongside, link to the STPA UCA / scenario where the control loop is in scope. Any *High* on the rubric's therapy-delivery, diagnosis, or PHI I/A questions is the cue to run TIR57 / STPA — not to stop at the CVSS number.

**When *not* to use the rubric's methods**: non-medical-device systems (clinical questions misfire on a generic web app — the row's `AV / PR / AC / Impact` is enough, optionally extended via OWASP-RR); minimum-viable threat models for medical-adjacent systems that aren't themselves regulated.

Rubric document (methods reference, v3.0-pinned, public): `github.com/mitre/md-cvss-rubric-tools`. CVSS calculators: NVD or FIRST.org at the version the team scores in.

## CVSS-based attack-path risk (cyber-physical IoT)

Use this **only** when the threat being rated is a composed *path* across several devices in one physical space (multi-device IoMT room, plant-floor cabinet, robotics cell, smart building zone) and the composition is what makes it dangerous — no single hop scores high under per-element rating, so the row-level `AV / PR / AC + CIA` under-rates the chain. This is a path-scoring supplement, not a third numeric option for ordinary single-element threats; for those, stick with the row-level scheme (or OWASP-RR if the team needs the extended factor list).

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

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) was a common scoring model. Both OWASP and Shostack note that DREAD scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. The CVSS-derived `AV / PR / AC` enums in the default scheme above are at least anchored to observable design properties of the system.

If a stakeholder insists on DREAD, deliver it but flag the limitation in the model.

## A note on prioritization

Risk rating is for triage, not for replacing engineering judgment. The Manifesto-aligned approach: "ranking should be based on the mathematical product of an identified threat's likelihood and its impact" but in practice "include the work to fix a problem" in the prioritization. A top-of-sort threat with a $50k fix and a mid-sort threat with a 1-line fix should usually both get done.

Don't let scoring become the goal. The goal is fixing things.
