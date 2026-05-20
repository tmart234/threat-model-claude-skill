# Risk rating

> **Last verified**: 2026-05. OWASP Risk Rating Methodology and the CVSS specification update; re-confirm against owasp.org and first.org/cvss before citing factor lists or version numbers in a deliverable.
> **Sources paraphrased**: OWASP Risk Rating Methodology (CC-BY 4.0); CVSS specification (FIRST.org, public); Stellios, Kotzanikolaou & Grigoriadis (Computers & Security 107, 2021) — CVV / pruning math (paraphrase, see methodologies.md for full citation); FMEA conventions (public); Adam Shostack and OWASP critiques of DREAD (paraphrase).

> **Related**: ← `SKILL.md` § "Threat enumeration" (the row format these columns populate) • `methodologies.md` (the AV/PR/AC + CIA scheme spans all three strata) • `validation.md` (Q4 cross-stratum check: every row uses the same enums) • `stpa.md` (where physical-harm severity lives — not on the threat row). Regulated-industry extensions (e.g. medical-device ISO 14971 / TIR57 mapping and clinical-context CVSS) live in the relevant `industries/<industry>/` pack.

This skill rates threats with **CVSS 3.1 intrinsic exposure (`AV / PR / AC`) plus a presence list of impacted security properties (`C / I / A`)** — three CVSS-derived enums that engineers already know, plus a flag set per row. No L/M/H, no compound "Risk" column, no row-level safety bump. Safety severity lives in the STPA section, cross-referenced from the threat row.

## Default scheme: AV / PR / AC + CIA presence

The `Threat` rows in §2.1 (and §2.2 / §2.3 if produced) carry these columns. Pick the values once per row; they're reused downstream — §3 sort, TM-BOM emission, and the CVSS calculator if a numeric score is required.

**Attack Vector (`AV`) — how the attacker reaches the element**

| Value | Meaning | Typical example |
|---|---|---|
| `N` | **Network** — reachable across an L3 boundary (Internet or routed VPN) | Public REST endpoint; cross-VPC service mesh; internet-exposed MQTT broker |
| `A` | **Adjacent** — reachable from the same L2 segment / Bluetooth Classic / facility-wide Wi-Fi | On-segment ARP spoofing; rogue AP within facility coverage; CAN bus on the same vehicle |
| `L` | **Local** — requires a shell / process on the host or short-range RF (BLE LE, NFC, inductive ≈ 10 ft) | Privilege escalation from an authenticated session; exploit via BLE in physical proximity |
| `P` | **Physical** — requires bodily contact with the device | Debug-port probe; chip-off; USB plugin requiring case-open |

Wireless-range distinction: short-range BLE / NFC / Zigbee → `L`, facility-wide Wi-Fi / Bluetooth Classic → `A`.

**Privileges Required (`PR`) — what credentials/role the attacker needs *before* the attack**

| Value | Meaning |
|---|---|
| `N` | None — anonymous attacker |
| `L` | Low — any authenticated user (regular tenant, logged-in user, normal device user) |
| `H` | High — administrator / service account / privileged role |

**Attack Complexity (`AC`) — conditions beyond the attacker's control**

| Value | Meaning |
|---|---|
| `L` | Low — repeatable; the attacker can carry out the attack at will |
| `H` | High — requires preconditions the attacker doesn't control (race window, specific target configuration, victim interaction beyond a click, defender misstep) |

`AC:H` is the *exception*, not the default. If the attacker can run the attack on demand, it's `AC:L` even when the attack is technically sophisticated. Don't credit obscurity of proprietary protocols, hard-coded keys, or undocumented service commands as `AC:H`.

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

This separation is exactly what running STPA *and* STRIDE buys you: STPA produces hazards independent of attacker intent (a sensor failure causes the same hazard a tampering attack does), and the cyber row records only the cyber half. For regulated safety-critical industries, the composition of the cyber row with the hazard severity into a regulator-facing safety-case view is industry-specific — for medical devices, the ISO 14971 / AAMI TIR57 `severity × P1 × P2` mapping is in `industries/medical/risk-rating.md`.

## When to use OWASP Risk Rating Methodology instead

If the user wants a more structured numeric model — typically for regulatory submissions, FMEA-style analysis, or alignment with an existing risk register — use OWASP Risk Rating Methodology (OWASP-RR).

OWASP-RR scores threat agent factors (skill, motive, opportunity, size), vulnerability factors (ease of discovery, ease of exploit, awareness, intrusion detection), technical impact factors (loss of confidentiality, integrity, availability, accountability), and business impact factors (financial damage, reputation, non-compliance, privacy violation). Each on a 0–9 scale. Average to get likelihood and impact, combine for risk.

Reference: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology

For regulated submissions, FMEA-style scoring (severity × occurrence × detection) is often expected by quality / regulatory teams. If that's the constraint, mirror the FMEA scale rather than inventing a parallel scoring scheme. Industry-specific extensions (e.g. medical-device FMEA / ISO 14971 specifics) live in the relevant `industries/<industry>/` pack.

## CVSS-based attack-path risk (cyber-physical IoT)

Use this **only** when the threat being rated is a composed *path* across several devices in one physical space (multi-device IoMT room, plant-floor cabinet, robotics cell, smart building zone) and the composition is what makes it dangerous — no single hop scores high under per-element rating, so the row-level `AV / PR / AC + CIA` under-rates the chain. This is a path-scoring supplement, not a third numeric option for ordinary single-element threats; for those, stick with the row-level scheme (or OWASP-RR if the team needs the extended factor list).

Each hop carries a CVSS base score on the standard 0.0–10.0 scale (or a CVV — Cyber-physical Vulnerability Vector — extending CVSS with physical-interaction parameters per Stellios et al. 2021). The path's per-hop scores are normalized to the unit interval (`s_i = CVSS_i / 10`) and the **path score is the product of per-hop normalized scores**: `S_path = ∏ (CVSS_i / 10)`. Lower products mean less feasible composed paths.

**Default pruning threshold**: drop any path whose product falls below `0.05` *before* expanding the next hop. This is the value Stellios et al. demonstrate keeps the working set small enough for human review on room-scale graphs (single-digit / low-tens of paths rather than thousands). Adjust upward (e.g. `0.10`) when the analyst wants only the most-feasible chains, or downward (e.g. `0.01`) when the system has high consequence and the team is willing to review more paths. **State the threshold explicitly in §1 prose** so the artifact is auditable — e.g. "Path-risk pruning threshold: `S_path ≥ 0.05` (Stellios default)."

**Worked example — three-hop cyber-physical path** (rogue BLE peripheral → Wi-Fi AP → safety-critical controller):

| Hop | Surface | CVSS | Normalized |
|---|---|---|---|
| H1 | P3 — rogue BLE peripheral DoSing the 2.4 GHz band shared with the AP | 6.5 | 0.65 |
| H2 | P2 — degraded AP retransmits expose the controller's mTLS-less management VLAN | 7.5 | 0.75 |
| H3 | P1 — exposed serial console on the controller accepts unauthenticated commands | 8.8 | 0.88 |

`S_path = 0.65 × 0.75 × 0.88 ≈ 0.43`. Above the `0.05` default threshold by an order of magnitude — keep the path and triage it.

A second candidate path that scores `0.65 × 0.30 × 0.20 ≈ 0.039` falls below the default threshold and is dropped before further expansion (any deeper hops can only make the product smaller, since each `s_i ≤ 1`). The pruning argument is exactly that: the product is monotone-non-increasing in path length, so a sub-threshold prefix never produces a super-threshold extension.

The per-element OWASP-RR table still lists the individual flaws; the path-risk table lists the composed attack chains and is what gets triaged. Construction technique: `methodologies.md` § "Risk-prioritized cyber-physical attack paths".

## Why DREAD is discouraged

DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) was a common scoring model. Both OWASP and Shostack note that DREAD scores tend to be wildly inconsistent between raters — the per-factor definitions are too subjective, and averaging subjective numbers doesn't make them objective. The CVSS-derived `AV / PR / AC` enums in the default scheme above are at least anchored to observable design properties of the system.

If a stakeholder insists on DREAD, deliver it but flag the limitation in the model.

## A note on prioritization

Risk rating is for triage, not for replacing engineering judgment. The Manifesto-aligned approach: "ranking should be based on the mathematical product of an identified threat's likelihood and its impact" but in practice "include the work to fix a problem" in the prioritization. A top-of-sort threat with a $50k fix and a mid-sort threat with a 1-line fix should usually both get done.

Don't let scoring become the goal. The goal is fixing things.
