# threat-modeler

A Claude skill that produces structured hybrid threat models (contextual / operational / strategic) for software, systems, IoT/embedded, medical devices, and business processes.

See `SKILL.md` for the full skill body, `references/` for on-demand methodology details, `assets/threat-model-template.md` for a blank template, and `assets/tm-bom-example.json` for a worked TM-BOM JSON example aligned to the OWASP Threat Model Library schema.

## Intended use

The skill is general-purpose — it scales down to a low-stakes internal web app (DFD + STRIDE-Per-Element + a response per threat is a complete deliverable) and up to regulated safety-critical work. The deeper machinery is aimed at:

- **Medical devices and clinical systems** — FDA premarket cybersecurity submissions, IEC 81001-5-1, IEC 62304, EU MDR / IVDR, HIPAA, H-ISAC sector framing; PACS / RIS / VNA, DICOM- and HL7-adjacent services, IoMT / multi-device clinical rooms. Includes a §1 intake template aligned to regulatory submissions, the AAMI TIR57 `P1 × P2` cyber-to-safety bridge into ISO 14971, and the FDA-aligned CVSS rubric for medical devices. See `references/medical.md`.
- **Safety-critical systems** — anything where a worst-case outcome is physical harm to people, equipment, or the environment: medical, ICS / SCADA / OT (IEC 62443), automotive (ISO 26262 cyber), aerospace, robotics, consumer IoT that controls something dangerous. STPA / STPA-Sec joint hazard analysis lives in `references/stpa.md`; the safety-bump risk-rating rule lives in `references/risk-rating.md`.
- **Regulated and sector-targeted software** — multi-tenant SaaS with compliance scope (PCI, GDPR, HIPAA), AI / ML systems, third-party-integrated systems, anything with a SOC handoff or sector ISAC adversary model.
- **General software and architecture review** — web apps, cloud-native, embedded / IoT, mobile, on-prem enterprise. Per-environment trust-boundary patterns and domain notes in `references/environments.md`.

The trigger is a request to *produce an artifact* (§1 system description / §2 threats / §3 responses / §4 validation), not the mention of a methodology term — see `SKILL.md` frontmatter for the full activation rules.

## Adapting to other industries

The skill is structured so industry depth lives in a single file under `references/` and loads on demand. `references/medical.md` is the worked example. To add another vertical (automotive, aerospace, financial, energy / utilities, rail, defense, pharma manufacturing, etc.), create `references/<industry>.md` mirroring `medical.md`'s structure:

- A **§1 intake template** — the structured fields a regulator or domain reviewer expects to see in the system description (device class equivalent, intended use, operational envelope, criticality tier, user roles).
- **Default layer mix** — which strata (contextual / operational / strategic) and which methodologies (STRIDE, LINDDUN, STPA, data-centric) load by default for this domain.
- **Domain-specific STRIDE / threat notes** — protocols and abuse patterns the generic STRIDE prompts don't surface (DICOM C-STORE for medical, CAN bus / UDS for automotive, Modbus / DNP3 for ICS, ISO 20022 for payments, etc.).
- **Regulatory and sector list** — the standards, regulators, and ISACs whose framing the threat model must compose with.
- **Asset / actor framing** — the domain's "patient-as-asset" equivalent (driver and bystander for automotive, grid stability for utilities, market integrity for finance).

Then add the domain pointer to `SKILL.md` § "Picking supplements" and § "Reference index" so the skill loads it on the right triggers, and (if relevant) extend `references/environments.md` § "Domain notes" with anything that's really an environment pattern rather than an industry one. No core skill changes are required — the contextual / operational / strategic scaffolding is industry-agnostic by design.

## File structure

```
threat-modeler/
├── LICENSE                              # Apache 2.0 license
├── README.md                            # This file
├── SKILL.md                             # Skill body (frontmatter + instructions); entry point loaded by Claude
├── assets/
│   ├── threat-model-template.md         # Blank threat model template
│   └── tm-bom-example.json              # Worked TM-BOM JSON example (OWASP TML schema)
└── references/
    ├── capec.md                         # CAPEC attack pattern catalog guidance
    ├── centric-methods.md               # Entry-point taxonomy (asset / data / flow / process / etc.)
    ├── data-centric.md                  # Data-centric threat modeling (NIST SP 800-154)
    ├── dfd-mermaid.md                   # Data flow diagrams in Mermaid syntax
    ├── environments.md                  # Per-environment trust-boundary patterns + domain notes (embedded / cloud-native / AI-ML / third-party)
    ├── examples.md                      # Pointers to worked threat-model examples (defers to OWASP Threat Model Library)
    ├── manifesto.md                     # Threat Modeling Manifesto principles
    ├── medical.md                       # Medical-device / PACS / DICOM / IoMT domain notes (FDA, IEC 81001-5-1, HL7)
    ├── methodologies.md                 # Hybrid framing, system-type matrix; STRIDE, PASTA, LINDDUN, OCTAVE, VAST, etc.
    ├── non-dfd-models.md                # Sequence / swim-lane and state diagrams in Mermaid (DFD supplements)
    ├── risk-rating.md                   # Risk rating frameworks (DREAD, CVSS, OWASP RR)
    ├── stpa.md                          # STPA — safety + security joint hazard analysis (umbrella for STPA / STPA-Sec / STPA-SafeSec lineage)
    ├── stride-prompts.md                # STRIDE per-element prompts and checklists
    └── validation.md                    # Threat model validation and review guidance
```

All files must be present at these paths for the skill to function correctly. `SKILL.md` references the `assets/` and `references/` files by relative path.

## Install

The skill name (in `SKILL.md` frontmatter) is `threat-modeler`. Anthropic's convention is that the skill directory name matches the skill name, so install this repository under that directory:

```sh
# user-scoped
git clone https://github.com/tmart234/threat-model-claude-skill ~/.claude/skills/threat-modeler

# project-scoped
git clone https://github.com/tmart234/threat-model-claude-skill .claude/skills/threat-modeler
```

If you cloned with the default directory name (`threat-model-claude-skill/`), rename it to `threat-modeler/` so the skill loader picks it up.

## License

Apache 2.0 — see `LICENSE`.
