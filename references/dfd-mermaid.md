# DFD → Mermaid mapping

> **Related**: ← `SKILL.md` • `environments.md` (per-environment boundary patterns + ownership taxonomy) • `centric-methods.md` (flow-centric entry point) • `stride-prompts.md` (enumerate threats once the DFD is drawn) • `validation.md` (canonical diagramming checklist)

Mermaid renders well in GitHub, GitLab, Markdown editors, and Polarion (with the Mermaid plugin). It's the practical default for a text-first threat modeling workflow.

Mermaid doesn't natively render trust boundaries the way a dedicated threat modeling tool does, so we use `subgraph` blocks to group elements by trust zone and a prose key to explain the convention.

## Element mapping

| DFD element | Mermaid syntax | Notes |
|---|---|---|
| External entity | `EE[Label]` (square brackets, rectangle) | Distinct visual from processes |
| Process | `P1(Label)` (parens, rounded) or `P1((Label))` (double parens, circle) | Pick one and be consistent |
| Multi-process | `MP[[Label]]` (subroutine shape) | Use only when decomposition is shown elsewhere |
| Data store | `DS[(Label)]` (cylinder) | Standard cylinder shape |
| Data flow | `A -- "label" --> B` | Always label what flows |
| Trust boundary | `subgraph Zone["<owner> \| <env-type> \| <trust>"]` ... `end` | Each subgraph is one trust zone — see § "Subgraph labeling convention" |

Per-edge styling (optional, for emphasis on highest-risk flows):

```
linkStyle 0 stroke:#d62728,stroke-width:2px
```

## Worked example: small clinical PACS

```mermaid
flowchart LR
    subgraph Hospital["Hospital IT | on-prem enterprise (clinical VLAN) | moderate trust"]
        Modality(CT Modality)
        Workstation[Clinician Workstation]
        PACS(PACS Server)
        ImageDB[(Image Database)]
        AuditDB[(Audit Log)]
    end

    subgraph Internet["Unowned | public internet | untrusted"]
        Vendor[Vendor Remote Support]
    end

    subgraph Cloud["Vendor | cloud (AWS prod account) | separate trust zone"]
        Archive(Long-term Archive Service)
        ArchiveStore[(Archive Object Store)]
    end

    Modality -- "DICOM C-STORE over TCP/11112 (PHI)" --> PACS
    Workstation -- "DICOM Q/R, view requests" --> PACS
    PACS -- "store images" --> ImageDB
    PACS -- "audit events" --> AuditDB
    PACS -- "tiered archive (mTLS)" --> Archive
    Archive -- "object PUT (SigV4)" --> ArchiveStore
    Vendor -- "remote support session (VPN + MFA)" --> PACS
```

Trust boundaries (the implicit "dotted lines"):

- `Hospital` ↔ `Internet` — vendor support is the riskiest crossing.
- `Hospital` ↔ `Cloud` — egress over TLS to cloud archive.
- Within `Hospital`, modality ↔ PACS may itself be a soft boundary if the imaging VLAN is separated; call this out in prose.

## Levels of decomposition

Don't pack everything into one diagram. Use a hierarchy:

- **Level 0 (context)** — the system as a black box plus all external entities and external data flows. One page, ~5–8 elements.
- **Level 1 (decomposed)** — the system's major internal processes and stores, with the same external entities. ~10–15 elements.
- **Level 2+ (focused)** — drill into one component when it warrants its own model (e.g. the DICOM parser, or the auth service). Reference back to the Level 1 element this expands.

If your diagram has more than ~15 elements or feels unreadable, you owe the reader a decomposition.

## Subgraph labeling convention

Every subgraph (= one trust zone) is labeled with three pipe-delimited fields, in this fixed order:

```
subgraph ID["<owner> | <env-type> | <trust>"]
```

Where:

- **`<owner>`** — who owns the underlying network, host, account, or device (and therefore who can change configuration / patch / control access). Pick from the ownership taxonomy in `environments.md` § "Ownership taxonomy". When ownership is unclear, write `Unknown` and record an explicit assumption — don't guess silently. When two owners share a zone, write the one with operational control (e.g. `Customer IT (vendor service-account exception — A3)`).
- **`<env-type>`** — the environment kind, from the fixed taxonomy: `cloud (<provider>)`, `on-prem enterprise (<sub-zone, e.g. AD Tier 0 / corp VLAN / DMZ>)`, `embedded (<sub-zone, e.g. application core / secure element / baseband>)`, `OT/ICS (<Purdue level or named zone>)`, `mobile (<sub-zone, e.g. app sandbox / keystore>)`, or `public internet`. Use the per-environment patterns in `environments.md` for sub-zones; collapsing a whole environment into one box loses most of the boundaries that matter.
- **`<trust>`** — qualitative trust level: `untrusted`, `low trust`, `medium trust`, `high trust`, `very high trust`, or domain-specific qualifiers like `safety-critical`, `out-of-scope owner`, `unknown`. Use the same scale across every diagram in one model so the prioritized §3 list is consistent.

Examples:

```
subgraph HospNet["Hospital IT | on-prem enterprise (clinical VLAN) | moderate trust"]
subgraph DeviceSE["Vendor | embedded (secure element) | very high trust"]
subgraph SIS["Plant Operator | OT/ICS (Safety Instrumented System) | safety-critical, isolated"]
subgraph Keystore["OS Vendor (Apple/Google) | mobile (hardware-backed keystore) | very high trust"]
subgraph CSPCtrl["AWS | cloud provider control plane | out-of-scope owner"]
```

Why three fields and not one: a subgraph label like `Hospital network — moderate trust` (the older convention) tells the reader the trust level but hides the *owner* and the *environment kind* — both of which drive which mitigations are even possible. The three-field label makes responsibility-for-mitigation legible from the diagram, which is what §3 of the threat model needs to answer.

When a system spans multiple environments (almost always the case), every subgraph gets its own owner / env-type / trust triplet. Don't share labels across environments.

## Conventions to keep things readable

- **Label every flow.** "DICOM C-STORE over TCP/11112 (PHI)" not "data". Concrete protocols make threats tractable: an attacker can craft DICOM PDUs against an open `11112` listener, but they can't attack a flow labeled "data".
- **Direction matters.** Use `-->` for one-way and `<-->` only when the flow truly is symmetric request/response with the same content. For RPC-style flows, draw two arrows with their actual content labels.
- **Group by trust zone, not by physical layout.** Trust zones are what STRIDE-Per-Element care about.
- **Color sparingly.** Use `style` only to highlight the riskiest crossings or the highest-risk elements. Don't decorate. (Note: ~1 in 12 people have some form of color blindness, so don't rely on color alone — always pair color with text labels.)
- **Stable element IDs.** Use short stable IDs (`P1`, `DS2`, `EE3`) so the threat table can reference them.

## Shostack's diagramming rules of thumb

These rules (paraphrased from *Threat Modeling: Designing for Security*, Ch. 1–2) are the single best test for whether a DFD is good enough to enumerate threats against. Apply them as you draw, and as a checklist at the end:

- **Focus on data flow, not control flow.** Threats follow data; control-flow diagrams hide where attackers can reach.
- **The "sometimes / also" test.** Anytime the team has to qualify a description with "sometimes we connect via TLS, but also fall back to HTTP" — that's two flows, not one. Draw both, and consider whether an attacker can force the fallback.
- **No data sinks.** Every piece of data written somewhere has a reader; show who reads it. If you can't name the reader, either the flow shouldn't exist or you've missed a process.
- **Data can't move itself between stores.** If the diagram shows a flow from one data store directly to another with no process in between, you've omitted the process that actually moves the data. Add it.
- **Tell a story.** The diagram should support the team telling a story about how the system works, end-to-end, while pointing at it. If telling that story requires editing the diagram or adding caveats, the diagram isn't done.
- **Don't draw an eye chart.** A diagram so dense you have to squint is no longer doing its job. Decompose.
- **Combine equivalent elements.** If two boxes are inside the same trust boundary, run on the same technology, and handle the same data — they're equivalent for threat modeling purposes. Combine them into one labeled element. (Don't lose information; do reduce clutter.)
- **The diagram is for thinking, not for showing off.** When considering whether to add detail or another sub-diagram, don't ask "is this the right way to do it?" Ask "does this help us think about what might go wrong?"

## Diagramming checklist

The end-of-diagramming checklist (Shostack-derived) is the canonical version in `validation.md` § "Diagramming checklist". Use it at the end of the diagramming phase — every box should be checkable before moving to threat enumeration. Don't duplicate it here.

## Trust boundaries — what to include

Trust boundaries are anywhere principals with different privileges interact. The list below is the **floor** — boundaries that apply to every environment. Once you know the environment type (cloud / on-prem enterprise / embedded / OT/ICS / mobile), use `references/environments.md` for the per-environment patterns that catch boundaries this generic list misses (Purdue levels for OT, app sandbox / keystore for mobile, secure-boot / JTAG / secure element for embedded, account / VPC / IAM / KMS for cloud, AD tier model / SSO federation / jump-host for enterprise). Both lists; not just one.

Generic boundaries that should always show up:

- Account / UID / SID boundaries (different OS users)
- Network interface boundaries (different segments, VLANs, VPCs)
- Different physical computers or hosts
- VM / container boundaries (especially when the host is shared)
- Organizational boundaries (your org ↔ vendor ↔ customer)
- Tenant boundaries in multi-tenant systems
- Ownership boundaries — wherever the *owner* of the underlying network/host/account changes (vendor → customer, customer → cloud provider, end-user → OS vendor). The shared-responsibility model applies in every environment, not just cloud — see `environments.md` § "Ownership taxonomy".
- Anywhere you can argue for different privileges

A useful tactic when you can't find boundaries: ask *does everything in this system have the same level of privilege and access to everything else? Is everything the system communicates with inside that same boundary?* If both answers are yes, draw a single trust boundary around everything. If either is no, you've found a missing boundary or a missing element.

A note on terminology: "trust boundary" and "attack surface" are closely related — an attack surface is a trust boundary plus the direction an attacker would come from. Many people use the terms interchangeably. This skill prefers "trust boundary" because it's directionally neutral and threats can flow either way across one.

A common piece of advice is that "trust boundaries should only cross data flows." That's good advice for a fully-decomposed model. If a boundary appears to cross a data store, that often indicates the store has different tables/rows with different trust levels — break it into two stores or add a sub-diagram. If a boundary crosses a process, the process probably has internal privilege separation that should be drawn explicitly.

Threats cluster around trust boundaries — but not *only* there. Complex parsing (DICOM PDU decoders, image format parsers, deserializers, regex engines on attacker input) is also threat-rich, even inside a single trust zone. Don't constrain enumeration to boundary crossings alone.

## Trust boundary prose template

After the diagram, include a short prose section that names each boundary, what crosses it, **and who owns each side** — ownership tells §3 who can implement the mitigation. Use a table for clarity once you have more than two boundaries.

Example (table form):

| Boundary | Owner (left) | Owner (right) | What crosses | Mediating control |
|---|---|---|---|---|
| Hospital ↔ Internet | Hospital IT | Unowned (public internet) | Vendor support sessions only | VPN concentrator + MFA |
| Hospital ↔ Cloud | Hospital IT | Vendor (AWS prod account) | Tiered archive PUTs (outbound only) | mTLS + cert pinning; no inbound |
| Imaging VLAN ↔ Clinical VLAN (within Hospital) | Hospital IT (imaging team) | Hospital IT (clinical team) | Modality → PACS only | L3 firewall ACLs; soft boundary |

Or in prose form:

> **Trust boundaries**
>
> - **Hospital ↔ Internet** (owners: Hospital IT ↔ unowned). Only vendor support sessions cross this boundary. Mediated by VPN concentrator and require MFA. Mitigation responsibility: Hospital IT.
> - **Hospital ↔ Cloud** (owners: Hospital IT ↔ Vendor on AWS). Outbound only, TLS 1.2+, mTLS via mutual cert pinning. No inbound from cloud to PACS. Mitigation responsibility split: Vendor for cloud-side controls, Hospital IT for egress firewall.
> - **Imaging VLAN ↔ Clinical VLAN** (within Hospital, both Hospital IT-owned). Soft boundary. Modalities can reach PACS but not the audit DB. Enforced at L3 firewall.

This section is what a reviewer actually reads when assessing whether the model is right. The diagram alone isn't enough. The Owner columns are what make §3 of the threat model assignable — without ownership, "implement mTLS" has no addressee.

## Anti-patterns to avoid in the diagram

- **Spaghetti**: too many flows, unreadable. Decompose.
- **Trust boundaries everywhere**: if every element is in its own subgraph, you've drawn a network diagram, not a threat model. Boundaries should be where *trust* changes.
- **Missing data stores**: people often draw the processes and forget the databases. Stores hold the data attackers want.
- **Aspirational**: drawing what the system *should* look like rather than what it *is*. Threat model the real system; if the real system needs to change, that's a finding.
