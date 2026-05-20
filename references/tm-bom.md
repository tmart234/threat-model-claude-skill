# Producing the TM-BOM (machine-readable artifact)

> **Related**: ← `SKILL.md` § "Producing the TM-BOM" (the opt-in trigger and the markdown-is-canonical rule) • `risk-rating.md` § "Translation to TM-BOM enums" (the `risks[]` field derivation) • `examples.md` (OWASP Threat Model Library — the schema source and the worked-example repository).

This file is the **canonical home** for the TM-BOM emission procedure: the OWASP TML field-by-field mapping, symbolic-name derivation, the Mitigate/Eliminate/Transfer/Accept → `control.status` mapping, and the pre-ship validation checklist. `SKILL.md` keeps only the opt-in trigger — emit a TM-BOM only when the user asks for it.

> **What "TM-BOM" means here**: a Threat Model Bill of Materials — a structured, machine-readable description of the system, threats, controls, and risks, analogous to an SBOM for software components. The concrete schema this skill emits against is the **OWASP Threat Model Library** JSON schema (`threat-model.schema.json` v1.0.2, MIT-licensed), which itself aligns with the pending CycloneDX TM-BOM specification. Tools that consume CycloneDX BOMs (SBOM, SaaSBOM, HBOM today; TM-BOM when published) will be the consumers; OWASP TML is the working contract today.

**TM-BOM emission is opt-in.** The markdown is the always-shipped artifact; the TM-BOM (JSON) is a derived view, emitted only when the user asks for tracker import (Polarion, Jira, GitHub Issues, ServiceNow), tooling-pipeline integration, or upstream contribution to the OWASP TML library. Don't emit it unprompted, and never emit only the JSON — the markdown is the canonical artifact for humans, the TM-BOM is the machine-readable companion. When you do emit one, validate against the OWASP TML schema before handing it over (the schema is the contract downstream tools already implement against; don't invent your own — command in § "Validation checklist" below).

## Symbolic-name derivation (every TM-BOM ID must match `^[0-9a-z-]+$`)

The schema requires every `symbolic_name` to be lowercase alphanumeric plus hyphens. The skill's markdown IDs (`AS1`, `T1`, `V1`, `PR1`, `SR-001`, `TB1`, `ASM1`) are uppercase for human readability — derive the schema-valid symbolic name by lowercasing and inserting a hyphen between the letter prefix and any digits *only when no hyphen is already present*:

| Markdown ID | TM-BOM `symbolic_name` |
|---|---|
| `AS1` | `as-1` |
| `T1` | `t-1` |
| `V1` | `v-1` |
| `PR1` | `pr-1` |
| `SR-001` | `sr-001` (already hyphenated; just lowercase) |
| `TB1` | `tb-1` |
| `ASM1` | `asm-1` |

Trust-zone subgraph IDs from the DFD (e.g. `CorpNet`, `Cloud`, `Internet`, `ProdAcct`) become trust_zone symbolic_names by lowercasing and replacing camelCase or spaces with hyphens (`corp-net`, `cloud`, `internet`, `prod-acct`). Cross-references in the JSON (`threats[].threat_persona`, `controls[].threats`, `risks[].threats`, `data_sets[].placements[].data_store`, `trust_boundaries[].trust_zone_a/b`, `data_flows[].source/destination.object`) use the derived symbolic names — keep them consistent across the file.

## Field-by-field mapping

| OWASP TML field (required unless noted) | Source in this skill | Notes / enum values |
|---|---|---|
| `version` (top-level) | Set to `"1.0.0"` for the first emission, bump per the skill's changelog | Must match `^\d+(\.\d+)*$` |
| `scope.title` | §1 system name | |
| `scope.description` | §1 system description (1–2 paragraphs) | |
| `scope.business_criticality` | Round 1 elicitation | enum: `minimal / low / moderate / high / maximal` |
| `scope.data_sensitivity` | Round 1 elicitation (data classes) | array enum: `pii / phi / fin / ip / cred / biz / gov / pci / op` |
| `scope.exposure` | Round 1 elicitation | enum: `internal / external` |
| `scope.tier` | Round 1 elicitation | enum: `mission_critical / business_critical / important / non_critical` |
| `description` (top-level, optional) | Free-form supplemental description | |
| `repo_link` (optional) | If the user names a repo URL | |
| `released_at` / `reviewed_at` (optional) | Template metadata block | ISO-8601 date or date-time |
| `trust_zones[]` | DFD subgraphs (one zone per subgraph) | `symbolic_name` (derived from subgraph ID), `title`, `description` (the `<owner> \| <env-type> \| <trust>` triplet works as the description) |
| `trust_boundaries[]` | §1 trust-boundary table | `trust_zone_a` / `trust_zone_b` are symbolic refs; `access_control_methods` array enum: `none / acl / rbac / mac / dac / abac`; `authentication_methods` array enum: `none / password / otp / challenge_response / public_key / token / biometrics / sso / social` |
| `actors[]` | §1 external entities | `type` enum: `system / user / power_user / administrator / engineer / third_party`; `trust_zone` is symbolic ref |
| `components[]` | DFD processes (rounded boxes / circles) | `trust_zone` is symbolic ref; `parent_component` (optional) for nested subgraphs |
| `data_stores[]` | DFD data stores (cylinders) | `type` enum: `sql / key_value / document / object / graph / time_series` (elicited in Round 1); `vendor` / `product` optional |
| `data_sets[]` | Data classes from data-centric pass (if run) | `placements[]` is `[{data_store: <symbolic_name>, encrypted: bool}, …]`; `data_sensitivity` array enum same as `scope.data_sensitivity` |
| `data_flows[]` | DFD arrows | `source` / `destination` are `{type: "actor"|"component"|"data_store", object: <symbolic_name>}`; `has_sensitive_data` and `encrypted` are booleans (defaults: `false` if no PII/PHI/cred in the flow's data class; `true` if the flow label includes mTLS / TLS / signed) |
| `diagrams[]` (optional) | The Mermaid DFD itself | `type: "mermaid"`, `source: <full Mermaid block as a string>`, `title: "Data Flow Diagram"` |
| `assumptions[]` (optional) | §1 assumptions table | `validity` enum: `unconfirmed / confirmed / rejected` (default `unconfirmed`); `topics` (optional) is an array of symbolic refs to other entities the assumption is about |
| `threat_personas[]` | Round 2 personas (always at least one) | `skill_level` enum: `script_kid / insider / engineer / expert_engineer / oc_sponsored / state_sponsored`; `access_level` enum: `anonymous / user / admin`; `applicability_to_org` enum: `minimal / low / moderate / high / maximal` |
| `threats[]` | §2.1 / §2.2 / §2.3 threat tables (one row per `T#` / `V#` / `PR#`) | `threat_persona` is symbolic ref; `event` is the verb-phrase event column; `sources` array enum: `adversary / human_error / failure / events_beyond_org_control`; `attack_mechanisms[]` is `[{capec_id: int, capec_title: string}, …]`; `weaknesses[]` is `[{cwe_id: int, cwe_title: string}, …]`; `components_affected[]` is symbolic refs |
| `controls[]` | §3 mitigation rows | `threats` is array of threat symbolic refs; `status` enum: `assumed / active / suggested / under_review / approved / scheduled / retired / wont_do` (default for new mitigations: `suggested`; for already-in-place: `active`); `priority` enum: `none / low / medium / high / critical` (translate from §3 risk: Low→low, Medium→medium, High→high, Critical→critical); `trust_boundary` (optional) is `{trust_zone_a, trust_zone_b}` |
| `risks[]` | Derived at TM-BOM emission from §2.1 row's `AV / PR / AC / Impact` columns, one risk per threat (or per cluster of cross-referenced threats) | `threats` array of refs; `likelihood` derived from AV+PR+AC; `impact` derived from CIA-impact set; `score` is integer 0–25; `level` enum: `very_low / low / medium / high / very_high / critical`. Translation table: `risk-rating.md` § "Translation to TM-BOM enums" |
| Skill-specific fields with no schema home (STRIDE category, stratum tag, Stellios path-product score, Mitigate/Eliminate/Transfer/Accept response distinct from `control.status`, Accept-rationale + decision-maker for threats with no `control` row) | **Top-level `extensions` only** — not per-object (every object def in the schema has `additionalProperties: false`, so per-object extensions fail validation). Key per-object data by the object's `symbolic_name`. | Extension keys must match the schema's regex: `<domain-with-hyphens-allowed>.<letters-only-TLD>/<path>`. Use the namespace `threat-modeler.tmskill/...` — the natural-looking `tmskill.threat-modeler/...` is **rejected** by the schema because the TLD label can't contain hyphens. Recommended paths: `threat-modeler.tmskill/stride-by-threat`, `threat-modeler.tmskill/stratum-by-threat`, `threat-modeler.tmskill/path-score-by-threat`, `threat-modeler.tmskill/response-by-control`, `threat-modeler.tmskill/accept-rationale-by-threat`. |

**Top-level header.** Every emitted TM-BOM starts with these two lines so the file declares which schema version it was built against:

```json
{
  "$schema": "https://github.com/OWASP/www-project-threat-model-library/blob/v1.0.2/threat-model.schema.json",
  "version": "1.0.0",
  ...
}
```

Bump `$schema` when the OWASP TML schema version moves; bump `version` per the model's own changelog (it's a free string matching `^\d+(\.\d+)*$`, not the schema version).

**Per-object extension example.** STRIDE category, response, stratum tag — all are per-object data the schema doesn't carry. Top-level extensions look like this:

```json
"extensions": {
  "threat-modeler.tmskill/stride-by-threat": {
    "t-1": "S",
    "t-2": "T"
  },
  "threat-modeler.tmskill/stratum-by-threat": {
    "t-1": "contextual",
    "v-1": "contextual"
  },
  "threat-modeler.tmskill/response-by-control": {
    "sr-001": "Mitigate",
    "sr-002": "Eliminate"
  }
}
```

## Mapping the Mitigate/Eliminate/Transfer/Accept response

The skill's response taxonomy doesn't map 1:1 onto `control.status`. Use this rule:

| Skill response | `control.status` | When to use which |
|---|---|---|
| `Mitigate` (control planned/in place) | `suggested` (new) or `active` (existing) | A new control gets `suggested`; if the user confirms the control is already deployed, use `active` |
| `Eliminate` (remove the threat surface) | `approved` if the design change is approved; `suggested` otherwise | The "control" is the design change itself |
| `Transfer` (insurance, vendor SLA, customer responsibility) | `assumed` | The control exists outside this system |
| `Accept` (no control) | Don't emit a `control` row at all | Record rationale + decision-maker in the top-level `extensions["threat-modeler.tmskill/accept-rationale-by-threat"]` block, keyed by the threat's symbolic name (`{rationale, decision_maker, decided_at}`); also note rationale in the threat's `description`. An unjustified `Accept` is indistinguishable from a missed threat — failure-mode catalog. |

Also record the original response verbatim in the top-level `extensions["threat-modeler.tmskill/response-by-control"]` (and `…/accept-rationale-by-threat` for Accept rows) so the markdown ↔ JSON round-trip is lossless.

Example Accept-rationale extension:

```json
"extensions": {
  "threat-modeler.tmskill/accept-rationale-by-threat": {
    "t-12": {
      "rationale": "Mitigation cost (~$200k re-architecting the legacy importer) exceeds expected loss; risk re-evaluated annually.",
      "decision_maker": "A. Smith, Engineering Director",
      "decided_at": "2026-05-10"
    }
  }
}
```

## Validation checklist before shipping the TM-BOM

- [ ] Top-level `$schema` and `version` are present.
- [ ] Every `symbolic_name` matches `^[0-9a-z-]+$` (run a regex over the file).
- [ ] Every cross-ref (`threats[].threat_persona`, `controls[].threats`, `risks[].threats`, `data_sets[].placements[].data_store`, `trust_boundaries[].trust_zone_a/b`, `data_flows[].source/destination.object`, `actors[].trust_zone`, `components[].trust_zone`, `data_stores[].trust_zone`) resolves to a defined symbolic name in the same file.
- [ ] All nine top-level required fields are present: `version`, `scope`, `trust_zones`, `trust_boundaries`, `actors`, `components`, `data_stores`, `data_sets` (may be empty array if no data-centric pass run), `data_flows`.
- [ ] All four required `scope` enums are present (`business_criticality`, `data_sensitivity`, `exposure`, `tier`).
- [ ] At least one `threat_persona` is defined and every threat references one.
- [ ] `threats[].event` is non-empty for every threat.
- [ ] `threats[].sources` is non-empty for every threat.
- [ ] `risks[].score` is in `[0, 25]` and is consistent with `risks[].level`.
- [ ] No per-object `extensions` (every object has `additionalProperties: false` — extensions only at top level).
- [ ] Extension keys match the schema's regex (TLD must be letters-only — `threat-modeler.tmskill/<path>`, **not** `tmskill.threat-modeler/<path>`).
- [ ] Schema validation exits 0. The v1.0.2 schema is bundled at `assets/threat-model.schema.json` — validate against it directly:

  ```
  check-jsonschema --schemafile assets/threat-model.schema.json <your-tm-bom>.json
  ```

  To refresh the bundled copy when the OWASP TML schema version moves:

  ```
  curl -sSf -o assets/threat-model.schema.json \
    https://raw.githubusercontent.com/OWASP/www-project-threat-model-library/main/threat-model.schema.json
  ```

Starters: `assets/tm-bom-example.json` (minimal, exercises every required field and the top-level `extensions` pattern, validates against v1.0.2 with `check-jsonschema` exit 0); the OWASP TML library at https://github.com/OWASP/www-project-threat-model-library/tree/main/threat-models for fuller worked examples (`ai-ml-systems/husky-ai-threat-model.json` is the most complete). Open the closest example before producing a TM-BOM; mirror its shape and depth.
