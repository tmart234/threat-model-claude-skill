# Medical-device risk rating — ISO 14971 / AAMI TIR57 and clinical-context CVSS

> **Last verified**: 2026-05. ISO 14971 (current edition 14971:2019), AAMI TIR57 (current edition 2023), the CVSS specification, and the MITRE CVSS Rubric for Medical Devices all update; re-confirm against the ISO store, the AAMI catalog, first.org/cvss, and github.com/mitre/md-cvss-rubric-tools before citing factor lists or version numbers in a regulatory deliverable.
> **Sources paraphrased**: ISO 14971:2019 — *Medical devices — Application of risk management to medical devices* (proprietary ISO standard, paraphrase only — severity / probability decomposition referenced, no direct quote); AAMI TIR57:2023 — *Principles for medical device security — Risk management* (proprietary AAMI technical information report, paraphrase only — `P1 × P2` cyber-to-safety bridge referenced, no direct quote); US FDA *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023, US government work — joint cyber/safety risk-model expectation cited); MITRE CVSS Rubric for Medical Devices — methods only, applied to current-version CVSS (MITRE, public; github.com/mitre/md-cvss-rubric-tools); CVSS specification (FIRST.org, public).

> **Related**: ← `industries/medical/README.md` • `domain-notes.md` (Four Questions ↔ TIR57 workflow, regulators) • `dicom-hl7.md` • `worked-examples.md`. Generic risk rating: `references/risk-rating.md` (the `AV / PR / AC + CIA` scheme, §3 sort heuristic, TM-BOM enum translation, OWASP-RR, cyber-physical attack-path scoring) • `references/stpa.md` (where physical-harm severity lives — STPA hazards `H-#`).

This file is the medical-device extension to the generic risk-rating scheme. The skill rates threats with **CVSS 3.1 intrinsic exposure (`AV / PR / AC`) plus a CIA presence list** per row (`references/risk-rating.md` § "Default scheme"). For medical-device submissions, regulators expect that scheme to *compose with* the device's patient-safety risk file. This file is that composition.

## ISO 14971 / AAMI TIR57 mapping for medical-device submissions

For FDA premarket cybersecurity submissions, IEC 81001-5-1, IEC 62304, and MDR / IVDR submissions, regulators expect cybersecurity risk to be expressed in the same language as patient-safety risk — specifically the **ISO 14971** risk-management framework (severity-of-harm × probability-of-occurrence-of-harm), with **AAMI TIR57** as the bridge that maps cyber threats into the ISO 14971 model. The 2023 FDA *Cybersecurity in Medical Devices* final guidance is explicit that the joint cyber/safety risk model is what reviewers will look for.

Under the `AV / PR / AC` + STPA scheme, the mapping reads off the row + hazard pair — no row-level safety bump needed.

**Severity of harm** (ISO 14971 `S1..S5`) — comes from the STPA hazard the threat references (`H-#`). STPA hazards already carry severity in the STPA section; reuse the value, don't re-derive. Typical mapping (full guidance: `references/stpa.md`):

| ISO 14971 severity | When the hazard scope warrants it |
|---|---|
| **S1 — Negligible** | Inconvenience, no clinical consequence (e.g. delayed non-urgent report) |
| **S2 — Minor** | Temporary discomfort, no lasting harm, no intervention beyond standard care |
| **S3 — Serious** | Reversible injury requiring medical intervention; non-permanent harm |
| **S4 — Critical** | Permanent injury, life-threatening, major intervention required |
| **S5 — Catastrophic** | Death, permanent disabling injury, multiple-patient harm |

**Probability of occurrence of harm** decomposes per TIR57 into two stages — `P1 × P2`:

- **P1** — probability that the cybersecurity threat materializes (a hazardous *situation* arises). Read directly from the §2.1 row's `AV / PR / AC` translated to `likelihood` per the generic translation table (`references/risk-rating.md` § "Translation to TM-BOM enums") — `certain` → high P1; `rare` → low P1. A network-reachable, weakly-authenticated, easily-automated attack (`AV:N + PR:N + AC:L` → sum 9 → `certain`) scores P1 high; a physical-access-only attack on a tamper-resistant device (`AV:P + PR:H + AC:H` → sum 3 → `rare`) scores P1 low.
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

**Cross-references**: `domain-notes.md` § "Regulators and threat-intel sources" lists the underlying standards. `references/stpa.md` is the right home when the *control loop itself* is the analysis target — STPA generates losses → hazards → constraints in a form that maps directly into ISO 14971 hazardous situations. The combination of STPA hazards + TIR57 cyber-to-safety mapping is what the joint IEC 62304 / IEC 81001-5-1 artifact actually contains.

## FMEA-style scoring

For medical-device submissions specifically, FMEA-style scoring (severity × occurrence × detection) is often expected by quality / regulatory teams. If that's the constraint, mirror the FMEA scale rather than inventing a parallel scoring scheme. OWASP Risk Rating Methodology (`references/risk-rating.md` § "When to use OWASP Risk Rating Methodology instead") is the alternative when a more structured numeric model is wanted but FMEA isn't mandated.

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
