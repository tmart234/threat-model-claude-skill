# Medical-device vertical pack

This folder is the **medical-device industry pack** for the `threat-modeler` skill. It collects every medical-specific default, regulatory mapping, protocol detail, and worked example in one place, separate from the industry-agnostic methodology in `references/`.

It is **optional and self-contained**. Delete `industries/medical/` and the core skill still works as a general-purpose threat modeler — it just loses medical depth. Nothing in `references/` or `SKILL.md` depends on this folder; the dependency runs one way (this folder references `references/`, never the reverse).

## When the skill loads this

`SKILL.md` points here on demand when the system is a medical device, a PACS / RIS / VNA, a DICOM- or HL7-adjacent service, an IoMT clinical environment, or anything heading for FDA premarket / IEC 81001-5-1 / IEC 62304 / MDR / IVDR review. For non-medical systems the skill never loads this folder.

## Files

| File | What it carries |
|---|---|
| `domain-notes.md` | The §1 medical-device intake template; the Four Questions ↔ TIR57 / ISO 14971 joint-workflow mapping; default layer mix and the medical decision-matrix row; patient-as-asset framing; STPA swap-vs-supplement guidance; attack-trees-in-medical-work caution; clinical workflow misuse; IoMT multi-device modeling; regulators and threat-intel sources. **Start here.** |
| `dicom-hl7.md` | DICOM / HL7 v2 / FHIR protocol-level STRIDE specifics; medical STRIDE seed prompts; the medical-device CAPEC subset; a worked CAPEC → CWE → ATT&CK → D3FEND chain. |
| `risk-rating.md` | The ISO 14971 / AAMI TIR57 `severity × P1 × P2` mapping for regulatory submissions; FMEA-style scoring note; clinical-context CVSS scoring (MITRE rubric methods). |
| `worked-examples.md` | End-to-end medical worked examples: the small clinical PACS DFD; per-element STRIDE threats; the data-centric pass on a DICOM study; the medication-administration sequence diagram; the smart infusion-pump state machine. |

## How this folder is a template for other industries

This pack is the worked example referenced by the repo `README.md` § "Adapting to other industries". To build another vertical (automotive, aerospace, finance, energy / utilities, rail, defense, pharma manufacturing, etc.), create `industries/<industry>/` mirroring this structure:

- A **§1 intake template** — the structured fields a regulator or domain reviewer expects in the system description (device-class equivalent, intended use, operational envelope, criticality tier, user roles).
- **Default layer mix** — which strata and methodologies (STRIDE, LINDDUN, STPA, data-centric) load by default for this domain, plus the domain's decision-matrix row.
- **Domain-specific STRIDE / CAPEC notes** — protocols and abuse patterns the generic prompts don't surface (DICOM C-STORE for medical; CAN bus / UDS for automotive; Modbus / DNP3 for ICS; ISO 20022 for payments).
- **Regulatory and sector list** — the standards, regulators, and ISACs the threat model must compose with.
- **Asset / actor framing** — the domain's "patient-as-asset" equivalent (driver and bystander for automotive; grid stability for utilities; market integrity for finance).
- **Worked examples** — at least one end-to-end model in the domain.

Then point `SKILL.md` § "Picking supplements" and § "References" at the new folder so the skill loads it on the right triggers. No core skill or `references/` changes are required — the contextual / operational / strategic scaffolding is industry-agnostic by design.
