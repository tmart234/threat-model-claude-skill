# Data-centric threat modeling — workflow and caveats

> **Last verified**: 2026-05. NIST SP 800-154 has been a Draft since 2016 and may have been finalized, withdrawn, or superseded since this file was last reviewed; re-check csrc.nist.gov before citing the publication status. The four-step workflow itself has been stable since the 2016 draft.
> **Sources paraphrased**: NIST SP 800-154 (Draft, Souppaya & Scarfone, 2016, US public domain). HIPAA / GDPR / PCI / FDA / ITAR/EAR framings (public regulatory text, paraphrase only).

> **Related**: ← `SKILL.md` • `centric-methods.md` § Data-centric (entry-point overview and "when do I pick this") • `methodologies.md` § "The hybrid layout" (where this layer fits in the hybrid output) • `stride-prompts.md` (STRIDE applies per location).

This file is the deep-dive on the data-centric entry point.

Source: NIST SP 800-154 (Draft), *Guide to Data-Centric System Threat Modeling*, Souppaya & Scarfone, NIST, March 2016. Public-domain U.S. government work.

## The four NIST SP 800-154 steps

1. **Identify and characterize the system and data of interest.** Pick the data class. Name the security objectives that matter for it — explicitly drop the ones that don't. Enumerate the authorized locations the data lives in (storage / transmission / execution / input / output).
2. **Identify and select attack vectors.** Per location, enumerate vectors against the in-scope objectives. Include cross-location vectors (data leaking from an authorized location into an unauthorized one).
3. **Characterize security controls.** Existing and proposed, mapped to vectors.
4. **Analyze the threat model.** Residual risk, gaps, what to mitigate, what's accepted.

## Acquiring the data scope

The skill cannot model data it doesn't know about. There are three modes for acquiring that scope; pick whichever matches what the user gave you.

### Mode A — User volunteers the data class

The user names the data: *"I want to model customer records in this pipeline"*, *"the firmware signing key"*, *"refresh tokens"*. Direct path. Confirm:

- The data class and its custodian.
- Lifecycle scope: greenfield (entire lifecycle) or scoped (one phase, e.g. only "in transit").
- Security objectives in scope (see §"Narrowing security objectives" below).
- Whether neighboring data classes are explicitly out of scope or queued for a later pass.

### Mode B — Skill enumerates candidates

The user describes a system but hasn't named the data of interest. Walk these prompts and produce 2–4 candidate data classes:

- **What's the most regulated data class here?** (HIPAA-PHI, GDPR-personal-data, PCI-CHD, ITAR/EAR-controlled, FDA-device-data.)
- **What's the highest-impact loss-or-tampering target?** (Signing keys, root credentials, audit logs, control-loop setpoints, ML model weights.)
- **What dataset, if exfiltrated, makes the news?** (Customer records, internal financials, design IP.)
- **What dataset, if tampered with, hurts someone or something?** (Pricing rules, order records, billing records, inventory counts, crypto-key material.)

Present the candidates. The user picks one (or two — see callout). If the user can't pick, default to the most regulated class — it's the easiest to defend the choice for.

### Mode C — Skill infers from artifact

The user pastes a spec, schema, OpenAPI doc, DFD, code, or component list. Extract data classes by scanning for nouns that describe data ("customer record", "credential", "telemetry packet", "firmware image", "audit event"). Rank by sensitivity and regulatory weight. Then proceed as Mode B (present candidates, user picks).

## Per-data-class scoping (callout)

> **One data class per pass — this is the most-likely misapplication.**
>
> Trying to model "all PHI plus all credentials plus all device configs" in a single data-centric pass defeats the methodology. The point of NIST 800-154 is that picking *one* data class lets you narrow the security objectives and the authorized locations sharply. Multiple data classes blur both.
>
> For multiple data classes: run multiple passes (one per class), or accept that you're really doing asset-centric in disguise and switch entry points.
>
> The one exception: if two data classes share a lifecycle so tightly that they always co-occur (a record and the encryption keys protecting it; a JWT and its signing key), you can model them together — but say so explicitly and accept that the security-objective narrowing won't be as clean.

## Narrowing security objectives

NIST 800-154 explicitly endorses dropping objectives that don't apply for the modeling pass. This is the discipline that makes data-centric work and is the most-skipped step in practice.

For each objective, decide in-scope or out-of-scope and *justify*:

| Objective | Common reasons in scope | Common reasons out of scope |
|-----------|-------------------------|-----------------------------|
| Confidentiality | PHI, PCI, GDPR, secrets, IP | Public data; data already published |
| Integrity | Dose, control, audit, financial, signed artifacts | Ephemeral telemetry; cached read-replica |
| Availability | Safety-critical control loop; SLA-bound; revenue-bearing | Covered by the flow-centric pass; offline-tolerant |

A typical narrow scope for a regulated personal-data class: **Confidentiality + Integrity in scope; Availability deferred to the flow-centric pass.** Don't double-cover availability across two layers — pick the layer that owns it and say so.

## STRIDE applied to data locations (caveat)

STRIDE works as the per-location lens for data-centric, but the **STRIDE-Per-Element applicability table** in `stride-prompts.md` is DFD-specific — it assumes the four DFD element types (External Entity / Process / Data Flow / Data Store). NIST 800-154's authorized locations are *not* DFD elements and the applicability rules don't transfer 1:1:

- A "Storage" location can have **spoofing** threats (the storage backend is mis-identified — e.g. an attacker stands up a rogue iSCSI target or NFS mount that the host trusts) even though the DFD-element table doesn't mark S for Data Store.
- A "Transmission" location can have **repudiation** issues (no signed transit, sender deniable) even though Data Flow doesn't have R in the DFD-element table.
- An "Input" or "Output" location can have **all six** — these are choke points where the data is being acted on by a process, so they inherit Process-level threat surface.
- An "Execution / Processing" location is essentially "Process from the data's point of view" and also gets all six.

**Practical rule**: when running STRIDE against data-locations rather than DFD elements, treat all six STRIDE categories as *candidates* at every location and let the in-scope security objectives (above) do the filtering. Do not import the DFD-element applicability table.

## Hybrid: how data-centric and flow-centric layer together

In a hybrid output (the skill's default — see `methodologies.md`), the contextual stratum runs flow-centric *and* data-centric together, plus other entry points where applicable. The two connect like this:

- The **flow-centric DFD shows the system**. Elements are EE / Process / Data Flow / Data Store, identified `EE1`, `P1`, `DS1`, etc.
- The **data-centric pass picks one data class and walks its lifecycle through that system**. Locations are `L1`, `L2`, etc.
- Each data-centric location should reference the DFD element(s) it corresponds to. Convention: *"L1 (Storage — primary record DB) ≈ DS2 on the DFD"*; *"L3 (Transmission — API write from client to service) ≈ flow EE1→P1"*.
- Where data-centric finds a **lifecycle location the DFD doesn't render natively** — process memory, core dumps, debug logs, swap, the operator's clipboard — call it out: *"L6 (Output — record fields in debug log) — not on the DFD; lifecycle location."* This is the value-add of layering data-centric on flow-centric. The flow-centric pass would have missed it.
- **STRIDE coverage is broad in the flow-centric pass** (every applicable element × category). **STRIDE coverage is narrow in the data-centric pass** (one data class, only in-scope objectives) **but deep at locations the DFD doesn't show**.

ID conventions to avoid collision:
- Flow-centric pass produces threats `T1`, `T2`, ...
- Data-centric pass produces vectors `V1`, `V2`, ...
- A finding that surfaces in both gets a cross-reference (`V3 ↔ T7`) rather than two separate IDs.

A worked end-to-end data-centric example — a regulated record walked through a small system, including the lifecycle-only locations a flow-centric pass misses — lives with each regulated industry's pack under `industries/<industry>/`.
