# DICOM / HL7 / FHIR — protocol STRIDE specifics and CAPEC subset

> **Last verified**: 2026-05 against the live CAPEC catalog (capec.mitre.org). DICOM, HL7 v2, and FHIR are subject to revision; re-confirm protocol details and CAPEC pattern numbers before citing. Re-verify the CAPEC subset quarterly.
> **Sources paraphrased**: DICOM Standard (NEMA, freely available); HL7 v2 / FHIR (HL7 International, freely available); MITRE CAPEC and MITRE CWE (public domain). No substantive direct quotes. Cite the upstream CAPEC / CWE ID directly in any output that references it.

> **Related**: ← `industries/medical/README.md` • `domain-notes.md` (clinical workflow misuse, regulators) • `worked-examples.md` (PACS DFD / data-centric example). Generic chain: `references/capec.md` (STRIDE → CAPEC → CWE → ATT&CK → D3FEND, the three abstraction levels, coverage honesty) • `references/stride-prompts.md` (STRIDE category prompts).

Load this alongside the generic STRIDE and CAPEC references when the system speaks DICOM, HL7 v2, or FHIR. The generic methodology is unchanged; this file collects the protocol-level threats and CAPEC mappings that the generic STRIDE prompts don't surface without domain knowledge.

## DICOM / HL7 STRIDE specifics

A few protocol-level threats consistently surface and are easy to miss without domain knowledge.

**DICOM** (network protocol over TCP, typically port 11112; AE Title is the application-level identifier):

- AE Title is *not* an authenticator. Without TLS + cert pinning, AE Title can be spoofed by anyone on the network — many production deployments still run cleartext DICOM.
- DICOM PDU and pixel-data parsers are large, written in C/C++, and a recurring source of memory-corruption vulnerabilities. Large image attachments can be decompression bombs.
- DICOM TLS support exists but is unevenly deployed; an interoperability claim is not a security claim.
- Image-tag manipulation (PatientName, PatientID, StudyInstanceUID swap) attacks both integrity and privacy — a Linking/Identifying threat under LINDDUN as well as STRIDE Tampering.

**HL7 v2** (segment-and-pipe text protocol over MLLP; rarely authenticated by itself):

- MLLP framing has no authentication; deployments depend on network segmentation.
- Field separator escaping bugs are the classic injection path.
- HL7 v2 → FHIR or HL7 v2 → DB writes are the typical SQL/NoSQL injection sinks.

**HL7 FHIR** (REST/JSON over HTTPS; OAuth/SMART-on-FHIR is conventional):

- SMART-on-FHIR scope handling is the typical authorization weak point — overly broad scopes or scope-confusion on cross-tenant access.
- FHIR's `_search` permits powerful queries; rate-limiting and search-result caps prevent enumeration.

The medical CAPEC working set below covers the typical mappings. Note: no DICOM- or HL7-specific Detailed CAPEC patterns exist; cite the closest Standard or Meta pattern and say so explicitly in the row (`references/capec.md` § "Honest about CAPEC coverage").

## Medical STRIDE seed prompts

Domain-specific seeds to add to the generic STRIDE category prompts (`references/stride-prompts.md` § "Category prompts") when the system is a medical device, PACS, or clinical service. Use them as seeds, not a checklist.

**Spoofing** — Could an attacker spoof a trusted DICOM AE Title?

**Tampering** — Can DICOM image pixel data be modified in transit? Can dose calculations be tampered with?

**Repudiation** — Are clinical actions (dose changes, scan orders, override events) attributable to a specific user? Are audit logs HIPAA / IEC 62443-aligned? Is there a chain-of-custody for evidence?

**Information Disclosure** — Is PHI exposed in DICOM tags, log files, or debug output?

**Denial of Service** — Can a DoS on a clinical system delay treatment? (This is a *safety* event, not just availability.)

**Elevation of Privilege** — Can a low-privilege technician account access patient data? Can a network-side compromise reach the safety-critical clinical control loop?

## CAPEC subset for medical-device / DICOM / HL7

For medical-device and clinical-protocol work the relevant CAPEC subset is small enough to keep in one place. This is the working set; cite from here first and reach for the broader catalog (`references/capec.md`) only when none fit. CAPEC abstraction levels and the closest-pattern fallback rule: `references/capec.md`.

| Threat scenario | CAPEC | Notes |
|-----------------|-------|-------|
| AE Title spoofing on DICOM association | CAPEC-151, CAPEC-21 | mTLS + AE-Title-pinned cert mitigates both |
| DICOM tag manipulation (PatientName, PatientID swap) | CAPEC-148, CAPEC-153 | LINDDUN Linking applies too |
| DICOM PDU / pixel-data parser overflow | CAPEC-100, CAPEC-46 | Map to CWE-119/-787; fuzz the parser |
| DICOM / HL7 protocol manipulation (no Detailed pattern exists) | CAPEC-272 (Protocol Manipulation) | Closest available — see `references/capec.md` § "Honest about CAPEC coverage" |
| HL7 field injection via management interface | CAPEC-153, CAPEC-66/-88 (depending on sink) | Treat as injection family |
| PHI exfiltration via debug log / core dump / telemetry | CAPEC-150 | Maps to data-centric Output / Execution locations (`references/data-centric.md`) |
| Decompression bomb / oversized DICOM study | CAPEC-130 | Availability + safety implication if it wedges acquisition |
| Operator credential phishing | CAPEC-98 (Phishing) | Domain: Social Engineering (CAPEC-403) |

The closest-fit framing: when CAPEC has no Detailed pattern for a specific clinical protocol, **cite the closest Standard or Meta pattern and say so explicitly in the threat row** — e.g. *"CAPEC-272 (Protocol Manipulation, Meta) — no DICOM-specific pattern exists; cited as closest abstract match. Detailed mitigation derived from CWE-345 (Insufficient Verification of Data Authenticity) and DICOM-specific knowledge."*

## Worked CAPEC chain (DICOM example)

A worked instance of the `STRIDE → CAPEC → CWE → mitigation` chain (generic chain: `references/capec.md` § "The chain that makes CAPEC worth citing"):

| STRIDE | Threat | CAPEC | CWE(s) | Mitigation |
|--------|--------|-------|--------|------------|
| Spoofing | Attacker spoofs DICOM AE Title to deliver malicious imagery | CAPEC-151 (Identity Spoofing); child CAPEC-21 (Exploitation of Trusted Identifiers) | CWE-287 (Improper Authentication), CWE-290 (Authentication Bypass by Spoofing) | mTLS for DICOM associations; pin AE Title to certificate |
| Tampering | Attacker overflows DICOM PDU parser to modify execution | CAPEC-100 (Overflow Buffers); CAPEC-46 (Overflow Variables and Tags) | CWE-119 (Improper Restriction of Operations within Bounds), CWE-787 (Out-of-bounds Write) | Bounds-checked parsing; memory-safe language for parser; fuzz-test the parser |

Extended with the operational/defensive chain (`references/capec.md` § "Closing the loop with MITRE D3FEND"):

| Threat | CAPEC | CWE | ATT&CK | D3FEND | Mitigation |
|---|---|---|---|---|---|
| Spoofing of AE Title via reused session token | CAPEC-151 (Identity Spoofing) | CWE-287 (Improper Authentication) | T1078 (Valid Accounts) | D3-MFA (Multi-factor Authentication), D3-CH (Credential Hardening) | SR-001: mTLS + AE-Title pinning to certificate |
| Buffer overflow in DICOM PDU parser | CAPEC-100 (Overflow Buffers) | CWE-787 (Out-of-bounds Write) | T1203 (Exploitation for Client Execution) | D3-PSEP (Process Segment Execution Prevention), D3-EAL (Exception Handler Pointer Validation) | SR-014: bounds-checked parser; fuzz suite `dicom_parser_fuzz_test`; DEP/ASLR enforced |

**D3FEND coverage for clinical protocols.** D3FEND's mappings to ATT&CK are good for enterprise / network / endpoint techniques and patchier for embedded-firmware and clinical-protocol techniques (DICOM-protocol attacks, OTA-update attacks on MCU-class devices). Expect the closest-mapped D3FEND technique to be one or two abstraction levels up from the actual control; cite the closest match and note the gap in the row.
