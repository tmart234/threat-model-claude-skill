# Threat Model: <System Name>

**Version**: 0.1
**Date**: <YYYY-MM-DD>
**Author(s)**:
**Reviewer(s)**:
**Status**: Draft / Reviewed / Approved
**Next review trigger**: <e.g. "next architectural change", "before v2.0 release", "annual">

> Section 2 below sketches three strata as a default layout (Contextual, Operational, Strategic). **Delete subsections you don't need — no stubs required.** The three-stratum layout is convenience scaffolding, not a coverage requirement. The system-type matrix in `references/methodologies.md` § "Decision matrix" suggests which contextual supplements and which strategic references typically fit each system type.

---

## 1. What are we working on?

### System description

<1–2 paragraphs. What does it do, who uses it, where does it run.>

### Scope

**In scope**:
-

**Out of scope**:
-

### Assets

> Use `AS` prefix for assets to keep them distinct from supplementary-pass findings (`V`-prefixed) in §2.1.

| ID | Asset | Description | Why it matters |
|----|-------|-------------|----------------|
| AS1 |      |             |                |

### Trust levels

| ID | Trust level | Description |
|----|-------------|-------------|
| TL1 | Anonymous external | |
| TL2 | Authenticated user | |
| TL3 | Privileged user / admin | |
| TL4 | Service / machine identity | |

### Assumptions

> Use the `ASM` prefix to keep these distinct from `AS#` asset IDs (`A1` looks too much like `AS1` on first read and breaks grep). Phrase each one falsifiably — "the load balancer terminates TLS before traffic reaches the app server" is testable; "the network is secure" is not.

- **ASM1**:
- **ASM2**:
- **ASM3**:

### Technology stack and environment

> Fill this in before drawing the DFD. See `references/environments.md` for the per-environment trust-boundary patterns this drives, and SKILL.md § "Round 1.5" for what to capture. If any field is unclear, record an assumption above and proceed.

- **Protocols on each flow**: <cite the actual protocol per flow — generic labels like "data" hide threats>
- **Runtimes / hosts per process**: <e.g. service on AWS Fargate / Node 20; embedded firmware on Cortex-M33 / FreeRTOS>
- **Identity / secrets**: <e.g. Okta SSO + AWS IAM; AD-integrated; device certificate from internal CA>
- **Environment types in scope** (from `environments.md` taxonomy): <cloud / on-prem enterprise / embedded / OT/ICS / mobile / hybrid>
- **Ownership per zone**: <who owns each environment in the DFD; e.g. "Vendor owns the cloud account; Customer IT owns the on-prem network">
- **Physical / operational context**: <where the device/host lives; tamper protections; exposed ports; network exposure>

### Data Flow Diagram

> Subgraph label convention: `subgraph ID["<owner> | <env-type> | <trust>"]` — see `references/dfd-mermaid.md` § "Subgraph labeling convention". The per-environment boundary patterns in `references/environments.md` should drive *which* subgraphs you draw.

```mermaid
flowchart LR
    subgraph ZoneA["<owner> | <env-type> | <trust>"]
        P1(Process 1)
        DS1[(Data Store 1)]
    end

    subgraph ZoneB["<owner> | <env-type> | <trust>"]
        EE1[External Entity 1]
    end

    EE1 -- "<protocol + auth>" --> P1
    P1 -- "<protocol + auth>" --> DS1

    classDef tb fill:none,stroke:#888,stroke-dasharray: 5 5
    class ZoneA,ZoneB tb
```

> The `classDef tb` + `class ... tb` lines above render every trust-boundary subgraph with a dashed border — required for every DFD this skill produces. Don't delete them. See `references/dfd-mermaid.md` § "Rendering trust boundaries as dashed subgraphs".

### Trust boundaries

| Boundary | Owner (left) | Owner (right) | What crosses | Mediating control |
|---|---|---|---|---|
| ZoneA ↔ ZoneB | | | | |

### Data of interest (only if §2.1 includes a data-centric pass)

> Delete this subsection if no data-centric pass is run. Workflow: `references/data-centric.md`.

- **Data class**: <e.g. signing key; refresh token; payment card>
- **Custodian**:
- **Security objectives in scope**: C / I / A — justify each drop
- **Authorized locations** (storage / transmission / execution / input / output): list with `L1`, `L2`, ... — reference the corresponding DFD elements where they exist; mark lifecycle-only locations explicitly

> **For STPA-SafeSec analyses**, copy the Losses / Hazards / Constraints / Control-layer / Component-layer scaffolding from `references/stpa-safesec.md` § "Section template (copy when in scope)" into this section.

---

## 2. What can go wrong?

> Three subsections below are the default layout. Add content where you have something to say; delete subsections you don't need.

### 2.1 Contextual stratum (system-specific)

#### 2.1.a Flow-centric STRIDE-Per-Element

| ID | Element | STRIDE | Threat | Likelihood | Impact | Risk |
|----|---------|--------|--------|------------|--------|------|
| T1 |         |        |        |            |        |      |
| T2 |         |        |        |            |        |      |

#### 2.1.b Supplementary entry-point pass (add when warranted)

> Pick from the system-type matrix in `references/methodologies.md`. Most common: data-centric (PHI / signing keys / tokens), asset-centric (crown jewels), user-needs-centric (rich business logic), process-centric (ops-heavy), code-centric (validation, if code is available). Delete this subsection if no supplementary pass is needed.

**Pass type**: <data-centric / asset-centric / user-needs-centric / process-centric / code-centric>

| ID | Location / Asset / Need / Process | Category | Threat / Vector | Likelihood | Impact | Risk |
|----|-----------------------------------|----------|-----------------|------------|--------|------|
| V1 |                                   |          |                 |            |        |      |
| V2 |                                   |          |                 |            |        |      |

> Cross-reference to flow-centric IDs where the same finding surfaced there: `V3 ↔ T7` (don't duplicate; cross-reference).

#### 2.1.c Privacy / AI-specific pass (only if applicable)

> Add LINDDUN if PII/PHI is in scope (Linking / Identifying / Non-repudiation / Detecting / Data disclosure / Unawareness / Non-compliance). Add an AI/ML threat list if ML components are present (prompt injection, model extraction, training-data poisoning, adversarial examples — see OWASP LLM Top 10 / OWASP ML Security Top 10). Delete this subsection if neither applies. Use `PR` prefix to keep these IDs distinct from DFD process labels (`P1`, `P2` …).

| ID | Element / Data | Category | Threat | Likelihood | Impact | Risk |
|----|----------------|----------|--------|------------|--------|------|
| PR1 |               |          |        |            |        |      |

#### 2.1.d Threat tree(s) for top 1–2 highest-value threats (optional)

```mermaid
flowchart TD
    G[Goal: <attacker goal>] --> A[<sub-goal>]
    G --> B[<sub-goal>]
```

### 2.2 Operational / Tactical stratum (only if produced)

> Include this stratum when the team will use it (handoff to SOC, detection coverage, IR roadmap). If included, add CAPEC and CWE alongside ATT&CK so derived requirements (`SR-###`) are traceable to a known weakness class. The CAPEC ID itself is what users care about — **the modeler picks the abstraction level for them** by SDLC stage (Meta = early architecture, Standard = design review, Detailed = component-level — see `references/capec.md`); leave the level off the row unless the choice was forced (no Detailed pattern exists for a domain-specific protocol — e.g. DICOM, HL7, ICS), in which case footnote the row with `(closest pattern; no Detailed available)`. Delete this section if the team won't use it.

| Threat ID | STRIDE | CAPEC | CWE(s) | ATT&CK | Kill chain | CVE / CVSS | Detection / handoff notes |
|-----------|--------|-------|--------|--------|------------|------------|---------------------------|
| T1 (example) | S | CAPEC-151 — Identity Spoofing | CWE-287, CWE-290 | T1078 (Valid Accounts) | Exploitation | — | mTLS + cert pinning; SOC: alert on cert mismatch |
| T2 (example, no Detailed)¹ | T | CAPEC-272 — Protocol Manipulation | CWE-345 | — | — | — | DICOM-specific; closest pattern (Meta) — no Detailed exists for DICOM PDU |
|           |        |       |        |        |            |            |                           |

¹ Include the "no Detailed available" footnote only when forced — most rows should not need it. See `references/capec.md` § "Honest about CAPEC coverage".

### 2.3 Strategic stratum (only if it shapes decisions)

> Include when sector landscape, regulators, or named-adversary context shape the threat picture. Delete this section if it doesn't.

- **Sector ISAC / threat-intel context**: <list relevant advisories — full ISAC list in `methodologies.md` § "Decision matrix">
- **Regulatory framing**: <FDA / IEC / HIPAA / GDPR / PCI / EU AI Act / NIST AI RMF — whichever apply>
- **Named-adversary context** (only if applicable): <e.g. ransomware groups targeting the sector, state actors — cite source>
- **Business-impact framing** (PASTA-borrowed, only if executive sign-off is required): <link threats to revenue / regulatory fine exposure / reputational scenarios>

---

## 3. What are we going to do about it?

### Mitigation table — single prioritized list across present strata

| Threat ID(s) | Cross-refs (CAPEC / CWE / ATT&CK / sector) | Risk | Response | Control / mitigation | Owner |
|--------------|-------------------------------------------|------|----------|----------------------|-------|
| T1           | CAPEC-151, CWE-287, ATT&CK T1078 | High | Mitigate |                      |       |
| T2           |                                  | Medium | Accept | (rationale)          |       |

### Derived security requirements

> Cite the CWE the requirement closes — that's what makes the requirement traceable to a known weakness class rather than to a free-text threat sentence. The CAPEC → CWE step in §2.2 supplies the CWE ID where §2.2 is produced.

- **SR-001**: The system SHALL <testable requirement>.
  Mitigates: T1, T3, V2 — closes CWE-287, CWE-290.
- **SR-002**: The system SHALL <testable requirement>.
  Mitigates: V1 — closes CWE-89.

---

## 4. Did we do a good enough job?

### Self-assessment checklist

**Diagram and setup**
- [ ] DFD reflects the system as actually built or specified, not an aspirational version
- [ ] Technology stack and environment section is filled in (protocols on each flow, runtimes, identity/secrets, environment types, owners, physical/operational context)
- [ ] Every subgraph carries an owner | env-type | trust label and ownership is reflected in the trust-boundary table
- [ ] Per-environment boundary patterns from `references/environments.md` checked for the in-scope environment types
- [ ] Flows are labeled with concrete protocols and authentication, not generic terms like "data"
- [ ] Assumptions are listed and falsifiable; out-of-scope items are explicit

**Threats and responses**
- [ ] Every applicable STRIDE category was walked at every applicable element
- [ ] Threats are cross-referenced across present strata, not duplicated
- [ ] All threats share one ID space and one risk-rating scale
- [ ] Every threat has a response (Mitigate / Eliminate / Transfer / Accept)
- [ ] Every "Mitigate" decision has a concrete, testable control
- [ ] Top risks have an owner identified
- [ ] No section that is present is empty or stubbed; subsections that don't apply are deleted, not stubbed

**Review**
- [ ] At least one stakeholder beyond the threat modeler has reviewed (note who below)

> **STPA-SafeSec users**: add the additional Q4 checks from `references/stpa-safesec.md` § "Q4 — Validation additions" (loss → hazard → constraint coverage; UCA → system-flaw traceability; hazard-scenario tree leaf-to-root mitigation).

### Reviewers

-

### Open questions / to validate

-

### Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1     |      |        | Initial draft |
