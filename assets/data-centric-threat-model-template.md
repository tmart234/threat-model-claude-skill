# Data-Centric Threat Model: <Data Type>

**Methodology**: NIST SP 800-154 (Draft) — *Guide to Data-Centric System Threat Modeling*
**Version**: 0.1
**Date**: <YYYY-MM-DD>
**Author(s)**:
**Status**: Draft / Reviewed / Approved
**Next review trigger**: <e.g. "next data-flow change", "before v2.0 release", "annual">

> Use this template when one specific data type dominates the threat picture (PHI/DICOM study, signing key, refresh token, firmware image) or when regulatory framing is data-typed (HIPAA / GDPR / PCI / FDA). For broader system reviews, use `threat-model-template.md` (flow-centric).

---

## 1. Data of interest (NIST 800-154 Step 1)

### Data description

<One paragraph: what the data is, who owns it, what creates / consumes it.>

### Sensitivity & classification

<e.g. PHI under HIPAA; export-controlled under ITAR; PCI cardholder data; internal-only.>

### Security objectives in scope (narrow deliberately)

| Objective | In scope? | Why / why not |
|-----------|-----------|---------------|
| Confidentiality | Y/N | |
| Integrity       | Y/N | |
| Availability    | Y/N | |

> NIST 800-154 explicitly endorses dropping objectives that don't apply for this pass. Justify each drop.

### Assumptions

1.
2.

---

## 2. Authorized data locations (NIST 800-154 Step 1, continued)

For each location the data is authorized to exist in, name *where* concretely. Locations the data is **not** authorized to exist in (but might leak to) belong in §3 as attack vectors.

| ID | Location type | Concrete instance | Custodian | Notes |
|----|---------------|-------------------|-----------|-------|
| L1 | Storage       | <e.g. PACS image DB volume> | | |
| L2 | Storage       | <e.g. nightly backup to object store> | | |
| L3 | Transmission  | <e.g. DICOM C-STORE between modality and PACS> | | |
| L4 | Transmission  | <e.g. HL7 feed to EHR> | | |
| L5 | Execution     | <e.g. PACS server process memory during render> | | |
| L6 | Input         | <e.g. modality DICOM upload> | | |
| L7 | Output        | <e.g. clinician viewer render / export to USB> | | |

---

## 3. Attack vectors per location (NIST 800-154 Step 2)

For each location, enumerate vectors. Restrict to in-scope security objectives. STRIDE works as a per-location lens; LINDDUN is an alternative when the data is privacy-sensitive.

| ID | Location | Objective | Vector | Likelihood | Impact | Risk |
|----|----------|-----------|--------|------------|--------|------|
| V1 | L1       | C         | <e.g. offline theft of unencrypted backup volume> | M | H | High |
| V2 | L3       | C, I      | <e.g. MITM on un-TLS'd DICOM segment, modify pixel data> | | | |
| V3 | L5       | C         | <e.g. core dump containing decrypted PHI written to disk> | | | |
| V4 | L7       | C         | <e.g. PHI leaked into application debug log> | | | |

> Cross-location vectors (data leaking from an authorized location into an unauthorized one) belong here too — call them out explicitly.

---

## 4. Controls (NIST 800-154 Step 3)

| Vector ID | Existing controls | Proposed controls | Response (Mitigate / Eliminate / Transfer / Accept) | Owner |
|-----------|-------------------|-------------------|------------------------------------------------------|-------|
| V1        |                   |                   |                                                      |       |
| V2        |                   |                   |                                                      |       |

### Derived security requirements

- **SR-001**: The system SHALL <testable requirement>. Mitigates: V1, V2
- **SR-002**: The system SHALL <testable requirement>. Mitigates: V3

---

## 5. Analysis (NIST 800-154 Step 4) — Did we do a good enough job?

### Coverage checklist

- [ ] Every authorized location enumerated (storage / transmission / execution / input / output)
- [ ] In-scope security objectives covered at every location (where applicable)
- [ ] Cross-location leakage vectors considered (authorized → unauthorized location)
- [ ] Every vector has a response decision
- [ ] Every "Mitigate" decision has a concrete, testable control
- [ ] Dropped objectives (out-of-scope CIA components) are justified, not just omitted
- [ ] Assumptions are listed and falsifiable

### Open questions / to validate

-

### Gaps not covered by this model

> Data-centric is bounded by the chosen data scope. List system-level threats (DoS unrelated to this data, supply-chain on tooling, threats to other data types) that need a separate pass — flow-centric, asset-centric, or another data-centric pass on a different data type.

-

### Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1     |      |        | Initial draft |
