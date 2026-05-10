# Medical-device threat modeling — domain notes

> **Last verified**: 2026-05. FDA premarket cybersecurity guidance, IEC 81001-5-1, IEC 62443, IEC 62304, MDR/IVDR, HIPAA, and the H-ISAC sector reference are all subject to revision; re-confirm against the FDA CDRH guidance index, IEC store, and the EU OJEU before relying on a specific edition. The 2023 FDA *Cybersecurity in Medical Devices* final guidance is the most-recently-updated load-bearing reference here. Re-verify on every model that goes near a regulatory submission.
> **Sources paraphrased**: US FDA guidance (public domain, US government work); IEC 81001-5-1 / 62443 / 62304 (proprietary IEC standards — paraphrase only, do not quote); HIPAA Security Rule (US government work); DICOM Standard (NEMA, freely available); HL7 v2 / FHIR (HL7 International, freely available); H-ISAC (membership-restricted advisories). No substantive direct quotes from any of these in this file.

> **Related**: ← `SKILL.md` § "Picking supplements" • `methodologies.md` § "Decision matrix" (medical-device row) • `data-centric.md` (worked DICOM example) • `stpa-safesec.md` (safety-critical control loops) • `environments.md` § "Multi-device shared-space modeling" (IoMT room composition) • `capec.md` § "Domain-specific subset" (medical CAPEC working set).

Load this when the system is a medical device, a PACS / RIS / VNA, a DICOM- or HL7-adjacent service, or a clinical environment with multiple co-located devices (Internet of Medical Things — IoMT). The skill defaults stay the same; this file collects the medical-specific defaults, regulators, and gotchas in one place.

## Default layer mix for medical / PACS / DICOM

For a medical device or PACS the default hybrid pulls in:

- **Contextual core**: flow-centric DFD + STRIDE-Per-Element.
- **Contextual supplements** — at least one is usually warranted:
  - **Data-centric on PHI / device data** (NIST SP 800-154 workflow + worked DICOM example in `data-centric.md`). HIPAA / GDPR framing makes the data class the natural organizing principle.
  - **LINDDUN privacy pass** — PHI is the default reason; Linking and Identifying are the categories STRIDE most under-covers.
  - **Asset-centric** on signing / device keys, software-update trust roots, encrypted-channel keys.
  - **Attack tree** on the safety-critical path (one tree per top hazard) where one exists.
- **Operational stratum**: ATT&CK technique IDs on top threats; the STRIDE → CAPEC → CWE → mitigation chain when §2.2 is produced (medical CAPEC working set: `capec.md` § "Domain-specific subset"). CISA medical-device advisories where applicable.
- **Strategic stratum**: H-ISAC threat intel, FDA premarket cybersecurity, IEC 62443, IEC 81001-5-1, IEC 62304, HIPAA, the safety-bump rule from `risk-rating.md`.

## Patient as an asset

Medical-device threat models list the **patient** alongside data assets — not as a stakeholder but as the thing the safety case ultimately protects. Loss of confidentiality of PHI is harm to the patient; safety failure is harm to the patient. List the patient explicitly in §1's asset table (`AS#`) so the trust boundary that "the patient is in the room with the device" is visible in the DFD as an external entity, not a hidden assumption.

For devices the patient is *connected to* (infusion pumps, ventilators, dialysis machines, implantables), the patient sits *inside* a trust zone with the device — the connection itself is the trust boundary.

## Safety-critical control loops — STPA-SafeSec swap-vs-supplement

For medical devices with safety-critical control loops (infusion pumps, ventilators, robotic surgery, dialysis machines, ECMO, smart anesthesia), STPA-SafeSec belongs in the model. Two modes (full mode-selection guidance: `stpa-safesec.md` § "Two modes"):

- **Swap mode** — STPA-SafeSec replaces the flow-centric DFD + STRIDE for the contextual core. Use when regulators require the joint safety+security artifact (FDA premarket cybersec, IEC 62304, IEC 81001-5-1, IEC 61508). One artifact, traced from losses through hazards through constraints to mitigations.
- **Supplement mode** — STPA-SafeSec runs alongside flow-centric + STRIDE, adding hazard scenarios that cross-reference threats and capture non-adversarial safety failures STRIDE misses. Use when there's an existing STRIDE practice or substantial non-control attack surface (telemetry, OTA, audit, cloud connectivity).

When in doubt for an FDA premarket submission with a control loop: swap mode. When in doubt for an existing product with a service review or feature update: supplement.

## Clinical workflow misuse

Network threats are usually well-modeled; **clinical workflow misuse** is usually under-modeled. Run an explicit pass on:

- Operator-as-attacker scenarios (curious clinician, disgruntled staff, social engineering against credential workflows).
- "Help me, I forgot my password" backdoors documented in service manuals — common, often vendor-shared.
- Workflows that bypass safety interlocks "to get the patient treated" (e.g. forced medication-administration overrides). Each override is a documented control failure.
- Service-mode access (extended privileges granted to field engineers) — what's exposed when service mode is on, and how is its activation gated?
- Shared-account practices on shared workstations (one nurse logs in, the whole shift uses the session). Maps to STRIDE Repudiation as much as Spoofing.
- Cross-tenant misuse where a single physical device is used by multiple care teams or patients sequentially.

LINDDUN's Unawareness category is the right home for several of these.

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

The medical CAPEC working set (`capec.md` § "Domain-specific subset for medical-device / DICOM / embedded / ICS") covers the typical mappings. Note: no DICOM- or HL7-specific Detailed CAPEC patterns exist; cite the closest Standard or Meta pattern and say so explicitly in the row (`capec.md` § "Honest about CAPEC coverage").

## IoMT — multi-device shared physical space

When several devices share one physical space — patient room with infusion pump + monitor + Wi-Fi AP + nurse-call + clinician phone, OR + multiple anesthesia / robotic devices, NICU bay with multiple monitors — element-by-element STRIDE often misses multi-hop cyber-physical attack paths where each individual device looks fine but their composition over P2/P3 hops reaches a safety-critical target.

The between-device surface taxonomy is in `environments.md` § "Multi-device shared-space modeling":

- **P1** — direct contact (USB, JTAG, attached cable).
- **P2** — proximity (BLE/NFC reach, line-of-sight IR, audible-range acoustic, near-field EM).
- **P3** — shared physical medium without proximity (the 2.4 GHz ISM band shared across Wi-Fi / BLE / Zigbee in a clinical room, where a rogue BLE peripheral can DoS a Wi-Fi-attached infusion pump without ever associating with it; shared power; shared HVAC for acoustic).

Use the risk-prioritized cyber-physical attack-path construction in `methodologies.md` § "Risk-prioritized cyber-physical attack paths" alongside per-element STRIDE for these systems. Target-rooted walks from a safety-critical asset (an infusion pump's drug-flow setpoint, a ventilator's oxygen ratio, a defibrillator's shock command) outward through P1/P2/P3 hops surface paths a single-device pass misses.

## Regulators and threat-intel sources

Cite at the strategic stratum (§2.3) when applicable:

- **FDA premarket cybersecurity guidance** (US, "Content of Premarket Submissions for Management of Cybersecurity in Medical Devices" — 2014, 2018 draft, 2023 final). Includes SBOM, vulnerability handling, and threat-modeling expectations.
- **IEC 81001-5-1** — health software security activities in the product lifecycle (the medical-device-specific layer over IEC 62443-4-1).
- **IEC 62443** — industrial automation security (the foundation IEC 81001-5-1 builds on).
- **IEC 62304** — medical-device software lifecycle (safety-classified). The joint STPA-SafeSec artifact is what aligns IEC 62304 with security work.
- **HIPAA Security Rule** (US PHI custody).
- **GDPR** (EU PHI / patient data outside the US).
- **MDR / IVDR** (EU medical devices).
- **H-ISAC** — Health-ISAC threat intel; advisories are a useful named-adversary source for sector strategic context.
- **CISA medical-device advisories** — federal advisories for ICS-CERT-style disclosures of medical-device CVEs.

## Cross-references

- DICOM data-centric worked example: `data-centric.md` § "Worked example".
- DFD nested-subgraph pattern (e.g. dual-OS pump with Linux comms side and RTOS pump-control side): `dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries".
- STPA-SafeSec workflow (losses → hazards → constraints → control/component layers → UCAs → system flaws → hazard scenarios): `stpa-safesec.md`.
- Cyber-physical attack-path construction (Stellios target-rooted walk, P1/P2/P3 surface taxonomy, CVV scoring): `methodologies.md` § "Risk-prioritized cyber-physical attack paths".
- Safety-bump rule for impact ratings: `risk-rating.md` § "Safety bump".

## Citations

- US FDA, *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023).
- IEC 81001-5-1:2021 — *Health software and health IT systems safety, effectiveness and security — Part 5-1: Security — Activities in the product life cycle*.
- IEC 62443 series — *Security for industrial automation and control systems*.
- IEC 62304:2006/AMD 1:2015 — *Medical device software — software life cycle processes*.
- DICOM Standard, NEMA, https://www.dicomstandard.org/.
- HL7 v2 / FHIR specifications, HL7 International, https://www.hl7.org/.
- Health-ISAC, https://h-isac.org/.
