# threat-modeler

A Claude skill that produces a threat model — a data-flow diagram, an enumeration of what can go wrong, and concrete mitigations — for software, systems, IoT/embedded devices, and business processes.

The skill is a thin, imperative entry point. `SKILL.md` walks the four-question framework and sizes the model to the system; the `references/` files carry methodology depth that Claude reads only when a given system needs it.

## File structure

```
threat-modeler/
├── LICENSE                          # Apache 2.0
├── README.md                        # This file
├── SKILL.md                         # Skill body — entry point loaded by Claude
├── assets/
│   └── threat-model-template.md     # Blank threat-model template
└── references/                      # Read on demand, not up front
    ├── dfd.md                       # Drawing the data-flow diagram in Mermaid
    ├── stride.md                    # STRIDE per-element prompts and mitigation mapping
    ├── methods.md                   # Going beyond STRIDE: LINDDUN, data-centric, attack
    │                                #   trees, PASTA, ATT&CK/CAPEC/CWE, AI/ML
    ├── environments.md              # Per-environment trust-boundary patterns; ownership
    ├── risk-rating.md               # Qualitative L/M/H, OWASP Risk Rating
    ├── validation.md                # Q4 validation; the Threat Modeling Manifesto
    └── safety-critical.md           # STPA-SafeSec; cyber-physical attack paths
```

`SKILL.md` references the `assets/` and `references/` files by relative path, so all files must stay at these paths.

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
