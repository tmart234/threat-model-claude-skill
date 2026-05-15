# Medical-device threat modeling — domain notes

> **Last verified**: 2026-05. FDA premarket cybersecurity guidance, IEC 81001-5-1, IEC 62443, IEC 62304, MDR/IVDR, HIPAA, and the H-ISAC sector reference are all subject to revision; re-confirm against the FDA CDRH guidance index, IEC store, and the EU OJEU before relying on a specific edition. The 2023 FDA *Cybersecurity in Medical Devices* final guidance is the most-recently-updated load-bearing reference here. Re-verify on every model that goes near a regulatory submission.
> **Sources paraphrased**: US FDA guidance (public domain, US government work); IEC 81001-5-1 / 62443 / 62304 (proprietary IEC standards — paraphrase only, do not quote); HIPAA Security Rule (US government work); DICOM Standard (NEMA, freely available); HL7 v2 / FHIR (HL7 International, freely available); H-ISAC (membership-restricted advisories). No substantive direct quotes from any of these in this file.

> **Related**: ← `SKILL.md` § "Picking supplements" • `methodologies.md` § "Decision matrix" (medical-device row) • `data-centric.md` (worked DICOM example) • `stpa.md` (safety-critical control loops) • `environments.md` § "Multi-device shared-space modeling" (IoMT room composition) • `capec.md` § "Domain-specific subset" (medical CAPEC working set).

Load this when the system is a medical device, a PACS / RIS / VNA, a DICOM- or HL7-adjacent service, or a clinical environment with multiple co-located devices (Internet of Medical Things — IoMT). The skill defaults stay the same; this file collects the medical-specific defaults, regulators, and gotchas in one place.

## Medical-device §1 intake template

A regulated medical-device threat model expects a richer §1 system description than a generic web app's "what does it do, who uses it, where does it run." Reviewers (FDA, IEC 81001-5-1 auditors, MDR / IVDR notified bodies) read §1 for the same fields they look for in the device's regulatory submission — and finding them in the threat model is what tells the reviewer the threat model is the *same model of the device* as the rest of the file. Mismatched device descriptions across regulatory and security artifacts is a common audit finding.

Pull these structured fields into §1 in addition to the standard Round 1 questions when the system is a medical device. Many of these come straight from the device's intended-use statement and submission summary — copy them rather than paraphrasing, so the threat model stays consistent with the regulatory artifact:

- **Device class** — FDA Class I / II / III; EU MDR class I / IIa / IIb / III (or IVDR class A / B / C / D for in-vitro diagnostics). Class drives the depth of the safety case and the regulator's expected scrutiny.
- **Intended use** — one paragraph, *the same wording as the device's intended-use statement*. Don't paraphrase. The threat picture is shaped by what the device claims to do (a continuous glucose monitor that "informs treatment decisions" has a different threat surface than one that "is for trend information only"; a defibrillator that "automatically delivers shocks" carries threats that an "advisory" model doesn't).
- **Period of expected use** — single-use disposable / 24-hour wear / 30-day implant / multi-year implantable / capital equipment. Drives expectations for OTA support windows, key-rotation cadence, and physical-tamper threat plausibility.
- **Invasiveness** — non-contact (imaging at a distance) / surface (skin electrode, BP cuff) / partially invasive (catheter, IV) / implanted (pacemaker, neurostimulator). Invasive devices have higher severity ratings under ISO 14971 — the safety case is what the cyber risk has to compose with.
- **Core use case** — a narrative of the clinical workflow from order to result, in plain prose. This is what feeds the swim-lane diagram in `non-dfd-models.md` § "Sequence / swim-lane diagrams" if one is warranted (it usually is for clinical workflows). Naming the actors (clinician, nurse, pharmacist, patient, biomed, vendor field engineer) here surfaces who needs to be in the swim lane.
- **Core technology** — bulleted feature list of the security-relevant components: sensors (what they measure, sample rate, calibration model), actuators (what they control, mechanical / hydraulic / electrical limits), connectivity (BLE / Wi-Fi / cellular / wired Ethernet / proprietary RF), on-device compute (Linux SBC + RTOS partition / single-core MCU / hypervisor), storage (encrypted-at-rest, key custody), identity (device certificate, factory-installed key, derived from serial), update path (OTA over TLS, signed firmware, USB-only update, no field update). This is the input to `references/dfd-mermaid.md` element catalog.
- **User roles** — the populated set from this typical roster: patient (the subject), clinician (orders, administers), nurse (administers, monitors), pharmacist (verifies orders), biomed engineer (maintains the device, hospital-side), vendor field engineer (services the device, manufacturer-side), cloud / data operator (vendor cloud back-end), regulator / auditor (read-only, periodic). Drop the ones that don't apply; add domain-specific roles where needed (perfusionist for cardiopulmonary bypass, radiologist for imaging, prescriber-vs-administerer split for controlled substances).

These fields go into §1 of the threat-model document above the standard Scope / Assumptions / Asset-table block (the standard `assets/threat-model-template.md` skeleton continues from there). Keep the names of the fields, even when the value is short or "N/A" — a reviewer looking for "device class" finds it immediately rather than inferring it. For the `assets/threat-model-template.md` skeleton itself, leave it lean — the standard template doesn't carry medical-only fields; surface them through this file when the system warrants them.

This intake also feeds the ISO 14971 / TIR57 mapping in `references/risk-rating.md` § "ISO 14971 / AAMI TIR57 mapping for medical-device submissions" — invasiveness and intended-use directly inform the severity-of-harm scale, and the user-roles list informs the P2 (probability of harm given hazardous situation) calculation by making the in-the-loop clinical roles explicit.

## Four Questions ↔ TIR57 / ISO 14971 joint risk-management workflow

For FDA premarket cybersecurity submissions, IEC 81001-5-1, IEC 62304, and MDR / IVDR, the threat-model artifact has to compose with the device's safety risk file. Reviewers expect cybersecurity activities to map onto the **AAMI TIR57 / ISO 14971 joint risk-management workflow** — the same shape as Figure 16 of the MITRE Threat Modeling Playbook. The Four Question Framework that this skill's threat model is built on (`SKILL.md` § "The Four Question Framework") slots directly onto that workflow. Surface the mapping in §1 prose of any medical-device model so the reviewer can see the joint structure at a glance.

| Skill question | TIR57 / ISO 14971 activity | What lives here |
|---|---|---|
| **Q1 — What are we working on?** | **Security risk analysis — asset inventory.** Intended use, device class, period of expected use, invasiveness, core technology, user roles (per `medical.md` § "Medical-device §1 intake template"); the asset table (`AS#`); the DFD with trust zones and ownership; the assumptions register (`ASM#`). | §1 of the threat-model document. |
| **Q2 — What can go wrong?** | **Security risk analysis — threats and vulnerabilities.** STRIDE-Per-Element on the DFD, plus the supplements (data-centric, LINDDUN, attack trees, STPA when safety is in play); §2.2 operational stratum (CAPEC → CWE → ATT&CK → D3FEND); strategic stratum (FDA / IEC 62443 / IEC 81001-5-1 / H-ISAC). Hazardous-situation language (TIR57's `P1` × `P2` decomposition; `risk-rating.md` § "ISO 14971 / AAMI TIR57 mapping for medical-device submissions") translates each threat into the safety-case framing. | §2.1 / §2.2 / §2.3 of the threat-model document. |
| **Q3 — What are we going to do about it?** | **Security risk control + Evaluation of overall residual security risk acceptability.** Mitigate / Eliminate / Transfer / Accept response per threat; derived security requirements (`SR-###`); residual-risk register; `severity × P1 × P2` re-rating after mitigations applied. The "evaluation of overall residual acceptability" is the part that names *what was accepted on whose behalf* — silent transference (`manifesto.md` § "Anti-patterns" → "Silent risk transference / silent risk acceptance") fails the reviewer's read here. | §3 of the threat-model document. |
| **Q4 — Did we do a good enough job?** | **Security risk management report + Production and post-production information.** The Q4 self-assessment (model/reality conformance, every threat has a response, every present section is populated); next-review trigger; the assumptions register and what's still unconfirmed; the changelog binding the model version to the device-software version under change control. Production / post-production feedback (CISA advisories, H-ISAC alerts, vulnerability-disclosure intake, SBOM updates) is what re-triggers Q1 — the loop is what TIR57 calls "production and post-production information" feeding back into risk analysis. | §4 of the threat-model document, plus the next-review trigger that closes the loop. |

The lifecycle context (FDA / JSP Product Security Framework / IEC 81001-5-1 lifecycle activities — design, verification, release, post-market) is the outer loop that pulls the Q1 → Q4 cycle through the device's life. Each premarket submission, each significant design change, and each post-market intelligence event re-enters the cycle at the appropriate question. State this explicitly in §1 of any medical-device threat model:

> *This threat model is the security risk analysis (Q1–Q2) and security risk control (Q3) artifact for the joint TIR57 / ISO 14971 / IEC 81001-5-1 workflow. It composes with the device's ISO 14971 safety risk file via the `severity × P1 × P2` mapping in §3. Production and post-production information (CISA advisories, H-ISAC alerts, internal vulnerability disclosures, SBOM updates) re-triggers this model per the next-review trigger in §4.*

This single paragraph is what tells the reviewer the threat model is *integrated* with the safety risk file rather than bolted on — the most common premarket finding for cybersecurity submissions.

## Default layer mix for medical / PACS / DICOM

For a medical device or PACS the default hybrid pulls in:

- **Contextual core**: flow-centric DFD + STRIDE-Per-Element.
- **Contextual supplements** — at least one is usually warranted:
  - **Data-centric on PHI / device data** (NIST SP 800-154 workflow + worked DICOM example in `data-centric.md`). HIPAA / GDPR framing makes the data class the natural organizing principle.
  - **LINDDUN privacy pass** — PHI is the default reason; Linking and Identifying are the categories STRIDE most under-covers.
  - **Asset-centric** on signing / device keys, software-update trust roots, encrypted-channel keys.
  - **Attack tree** on the safety-critical path (one tree per top hazard) where one exists.
- **Operational stratum**: ATT&CK technique IDs on top threats; the STRIDE → CAPEC → CWE → mitigation chain when §2.2 is produced (medical CAPEC working set: `capec.md` § "Domain-specific subset"). CISA medical-device advisories where applicable.
- **Strategic stratum**: H-ISAC threat intel, FDA premarket cybersecurity, IEC 62443, IEC 81001-5-1, IEC 62304, HIPAA, the ISO 14971 / AAMI TIR57 mapping from `risk-rating.md` (severity from STPA `H-#` + `P1 × P2` from the row's `AV / PR / AC`).

## Patient as an asset

Medical-device threat models list the **patient** alongside data assets — not as a stakeholder but as the thing the safety case ultimately protects. Loss of confidentiality of PHI is harm to the patient; safety failure is harm to the patient. List the patient explicitly in §1's asset table (`AS#`) so the trust boundary that "the patient is in the room with the device" is visible in the DFD as an external entity, not a hidden assumption.

For devices the patient is *connected to* (infusion pumps, ventilators, dialysis machines, implantables), the patient sits *inside* a trust zone with the device — the connection itself is the trust boundary.

## Safety-critical control loops — STPA swap-vs-supplement

For medical devices with safety-critical control loops (infusion pumps, ventilators, robotic surgery, dialysis machines, ECMO, smart anesthesia), STPA belongs in the model. Two modes (full mode-selection guidance: `stpa.md` § "Two modes"):

- **Swap mode** — STPA replaces the flow-centric DFD + STRIDE for the contextual core. Use when regulators require the joint safety+security artifact (FDA premarket cybersec, IEC 62304, IEC 81001-5-1, IEC 61508). One artifact, traced from losses through hazards through constraints to mitigations.
- **Supplement mode** — STPA runs alongside flow-centric + STRIDE, adding hazard scenarios that cross-reference threats and capture non-adversarial safety failures STRIDE misses. Use when there's an existing STRIDE practice or substantial non-control attack surface (telemetry, OTA, audit, cloud connectivity).

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

**Data-sink violations specific to medical devices.** Shostack's "no data sinks" rule (`dfd-mermaid.md` § "Shostack's diagramming rules of thumb") has two consistently-overlooked forms in medical-device deployments: audit logs that are *written but never read* (PHI accumulating in a log store without lifecycle controls or retention policy) and device-local telemetry that's *stored on the device but never uploaded* (intended for vendor analytics but never wired up; PHI sitting on the device until decommissioning, often surviving "factory reset" via wear-leveled flash). Both are PHI risks even when no exfiltration happens — accumulation without a documented reader is a HIPAA / GDPR custody gap, not just a hygiene issue. When the DFD shows a write into a store, demand the reader; when no reader exists, the question is whether the data should be collected at all.

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
- **IEC 62304** — medical-device software lifecycle (safety-classified). The joint STPA artifact is what aligns IEC 62304 with security work.
- **HIPAA Security Rule** (US PHI custody).
- **GDPR** (EU PHI / patient data outside the US).
- **MDR / IVDR** (EU medical devices).
- **ISO 14971:2019** — application of risk management to medical devices (the safety-risk language regulators expect cybersecurity risk to be expressed in).
- **AAMI TIR57:2023** — principles for medical-device security risk management; the `P1 × P2` bridge from cybersecurity threat to ISO 14971 hazardous-situation-leading-to-harm. Use the mapping in `references/risk-rating.md` § "ISO 14971 / AAMI TIR57 mapping for medical-device submissions" to translate the §2.1 row's `AV / PR / AC / Impact` plus the cross-referenced STPA hazard severity into the regulator-facing safety-case view. The Four Question ↔ TIR57 joint-workflow mapping (`medical.md` § "Four Questions ↔ TIR57 / ISO 14971 joint risk-management workflow") is the spine of any medical-device threat model — surface it in §1 prose so a reviewer can see the joint structure on the first page.
- **H-ISAC** — Health-ISAC threat intel; advisories are a useful named-adversary source for sector strategic context.
- **CISA medical-device advisories** — federal advisories for ICS-CERT-style disclosures of medical-device CVEs.

## Cross-references

- DICOM data-centric worked example: `data-centric.md` § "Worked example".
- DFD nested-subgraph pattern (e.g. dual-OS pump with Linux comms side and RTOS pump-control side): `dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries".
- STPA workflow (losses → hazards → constraints → control/component layers → UCAs → system flaws → hazard scenarios): `stpa.md`.
- Cyber-physical attack-path construction (Stellios target-rooted walk, P1/P2/P3 surface taxonomy, CVV scoring): `methodologies.md` § "Risk-prioritized cyber-physical attack paths".
- Safety severity → STPA hazards (`stpa.md`); cyber row composes with hazard severity per `risk-rating.md` § "ISO 14971 / AAMI TIR57 mapping for medical-device submissions".

## Citations

- US FDA, *Cybersecurity in Medical Devices: Quality System Considerations and Content of Premarket Submissions* (2023).
- IEC 81001-5-1:2021 — *Health software and health IT systems safety, effectiveness and security — Part 5-1: Security — Activities in the product life cycle*.
- IEC 62443 series — *Security for industrial automation and control systems*.
- IEC 62304:2006/AMD 1:2015 — *Medical device software — software life cycle processes*.
- ISO 14971:2019 — *Medical devices — Application of risk management to medical devices*.
- AAMI TIR57:2023 — *Principles for medical device security — Risk management*.
- DICOM Standard, NEMA, https://www.dicomstandard.org/.
- HL7 v2 / FHIR specifications, HL7 International, https://www.hl7.org/.
- Health-ISAC, https://h-isac.org/.
