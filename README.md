# threat-modeler

A Claude skill that produces structured hybrid threat models (contextual / operational / strategic) for software, systems, IoT/embedded, medical devices, and business processes.

See `SKILL.md` for the full skill body, `references/` for on-demand methodology details, and `assets/threat-model-template.md` for a blank template.

## File structure

```
threat-modeler/
├── LICENSE                              # Apache 2.0 license
├── README.md                            # This file
├── SKILL.md                             # Skill body (frontmatter + instructions); entry point loaded by Claude
├── assets/
│   └── threat-model-template.md         # Blank threat model template
└── references/
    ├── capec.md                         # CAPEC attack pattern catalog guidance
    ├── centric-methods.md               # Entry-point taxonomy (asset / data / flow / process / etc.)
    ├── data-centric.md                  # Data-centric threat modeling (NIST SP 800-154)
    ├── dfd-mermaid.md                   # Data flow diagrams in Mermaid syntax
    ├── environments.md                  # Per-environment trust-boundary patterns + domain notes (embedded / cloud-native / AI-ML / third-party)
    ├── manifesto.md                     # Threat Modeling Manifesto principles
    ├── medical.md                       # Medical-device / PACS / DICOM / IoMT domain notes (FDA, IEC 81001-5-1, HL7)
    ├── methodologies.md                 # Hybrid framing, system-type matrix; STRIDE, PASTA, LINDDUN, OCTAVE, VAST, etc.
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
