# Worked threat-model examples — defer to OWASP Threat Model Library

> **Last verified**: 2026-05.
> **Sources**: OWASP Threat Model Library (https://github.com/OWASP/www-project-threat-model-library, MIT-licensed). The library's JSON schema (`threat-model.schema.json`) aligns with the pending CycloneDX TM-BOM specification.

> **Related**: ← `SKILL.md` § "Producing the threat model" • `tm-bom.md` (the JSON schema this file's examples conform to) • `industries/<industry>/worked-examples.md` (end-to-end worked models for a given vertical — e.g. `industries/medical/worked-examples.md` for a DICOM PACS).

For end-to-end worked threat models, **defer to the OWASP Threat Model Library** (OWASP TML) rather than maintaining a parallel set in this skill. Their examples are peer-reviewed, schema-validated, and organized by system type — and they're the artifacts downstream tools already consume.

## Where to find examples by system type

The OWASP TML repo's [`/threat-models`](https://github.com/OWASP/www-project-threat-model-library/tree/main/threat-models) directory is organized by category. Map each system-type-matrix row from `methodologies.md` § "Decision matrix" to the matching subdirectory:

| System type (from `methodologies.md` matrix) | OWASP TML subdirectory |
|---|---|
| Generic web app | `threat-models/web-applications/` |
| Cloud-native multi-tenant SaaS | `threat-models/web-applications/` (and `infrastructure/` for the platform side) |
| Embedded / IoT | (none; nearest are `infrastructure/` for the deployment surface) |
| ICS / SCADA / OT | (none; build from `infrastructure/` plus this skill's STPA reference if a control loop is in scope) |
| AI / ML system | `threat-models/ai-ml-systems/` (e.g. `husky-ai-threat-model.json`) |
| Third-party-integrated systems | `threat-models/third-party-integrations/` |
| Medical device / PACS / DICOM | (none in OWASP TML; use `industries/medical/worked-examples.md`) |

Use the closest match as a structural reference for the TM-BOM; the markdown narrative this skill produces is independent and follows the §1 / §2 / §3 / §4 layout from `SKILL.md` § "Producing the threat model".

## How to use an OWASP TML example

1. **Open the example matching the system type.** Read its `scope`, `trust_zones`, `components`, `data_flows`, `threats`, `controls`, `risks` fields end-to-end.
2. **Mirror its shape in your TM-BOM.** When this skill emits a TM-BOM (per `tm-bom.md`), match the field structure of the closest example so downstream consumers see a familiar artifact.
3. **Validate before shipping.** Run `check-jsonschema --schemafile threat-model.schema.json <your-tm-bom>.json` (or any JSON-Schema validator) against the upstream schema. The skill's IDs (`AS1`, `T1`, `V1`, `PR1`, `SR-001`) populate the schema's symbolic-name fields directly.

## What this file deliberately does not include

This file used to host two custom worked examples (a multi-tenant SaaS web app and an LLM chatbot with tools) inline. Those have been removed because:

- The OWASP TML examples are peer-reviewed and updated by the community; in-skill copies would drift.
- The OWASP TML schema is what downstream tools consume; presenting custom markdown examples as the "canonical" output sidesteps that.
- The skill's *job* is to produce a model, not to ship reference models — those belong to the library.

If a system type isn't covered in OWASP TML, contribute upstream rather than copying into this file. Industry-specific worked examples — e.g. the medical / DICOM end-to-end example, which the OWASP TML library doesn't currently host — live with each vertical pack at `industries/<industry>/worked-examples.md`.
