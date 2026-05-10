# CAPEC — using the attack-pattern catalog in a hybrid threat model

> **Related**: ← `SKILL.md` • `stride-prompts.md` (STRIDE is the input to the CAPEC chain) • `centric-methods.md` (CAPEC is a characterization layer, not an entry point) • `methodologies.md` § "Hybrid as default" (CAPEC's place in the operational stratum).

This file is the working reference for CAPEC (Common Attack Pattern Enumeration and Classification, MITRE) inside this skill's hybrid output. CAPEC is one of the operational-stratum characterization layers (see `methodologies.md` § "Hybrid as default"). It is **not** an entry point — you don't enumerate threats from CAPEC; you map already-enumerated threats to CAPEC patterns to make them concrete and traceable.

CAPEC's distinctive design-time payoff over MITRE ATT&CK: each CAPEC pattern references the **CWE** weaknesses it exploits, which gives you a chain from the abstract STRIDE category to a concrete weakness class to a known mitigation. ATT&CK doesn't provide that chain at design time (it's a TTP catalog for ops). Surface and use the chain explicitly.

CAPEC site: https://capec.mitre.org/

## The chain that makes CAPEC worth citing

The threat-table column order in this skill's hybrid output is:

```
STRIDE  →  CAPEC pattern  →  CWE(s) the pattern exploits  →  Mitigation
```

Without the CAPEC and CWE columns, mitigations are derived ad-hoc from the threat sentence. With them, each mitigation traces to a known weakness class. This is what makes derived security requirements (`SR-001`, `SR-002`, ...) traceable to a known weakness class rather than just to a free-text threat description.

Worked chain:

| STRIDE | Threat | CAPEC | CWE(s) | Mitigation |
|--------|--------|-------|--------|------------|
| Spoofing | Attacker spoofs DICOM AE Title to deliver malicious imagery | CAPEC-151 (Identity Spoofing); child CAPEC-21 (Exploitation of Trusted Identifiers) | CWE-287 (Improper Authentication), CWE-290 (Authentication Bypass by Spoofing) | mTLS for DICOM associations; pin AE Title to certificate |
| Elevation | Attacker injects SQL via management interface to escalate | CAPEC-66 (SQL Injection) | CWE-89 (SQL Injection) | Parameterized queries; ORM with bound parameters; least-privilege DB role |
| Tampering | Attacker overflows DICOM PDU parser to modify execution | CAPEC-100 (Overflow Buffers); CAPEC-46 (Overflow Variables and Tags) | CWE-119 (Improper Restriction of Operations within Bounds), CWE-787 (Out-of-bounds Write) | Bounds-checked parsing; memory-safe language for parser; fuzz-test the parser |

The columns to the right of each CAPEC ID come from CAPEC itself (each pattern lists its CWEs and typical mitigations). Don't reinvent them; cite them.

## Three CAPEC abstraction levels — which to cite when

CAPEC patterns come at three abstraction levels. Citing the wrong level is a common mistake: too abstract makes the citation decorative; too detailed claims more knowledge of the implementation than the threat model has.

| Level | What it is | Cite when… | Examples |
|-------|------------|------------|----------|
| **Meta** | Broad, technology-independent attack patterns | Early architecture review; you don't yet know the implementation; you want stratum-level coverage | CAPEC-156 (Engage in Deceptive Interactions), CAPEC-118 (Collect and Analyze Information), CAPEC-225 (Subvert Access Control) |
| **Standard** | More specific; technology-aware but not technology-locked | Mid-design; protocol or component class is known; multiple implementations possible | CAPEC-151 (Identity Spoofing), CAPEC-66 (SQL Injection), CAPEC-100 (Overflow Buffers), CAPEC-176 (Configuration/Environment Manipulation) |
| **Detailed** | Concrete, technology-specific or step-specific | Component-level review; you know the parser / protocol / library; implementation details exist | CAPEC-7 (Blind SQL Injection), CAPEC-46 (Overflow Variables and Tags), CAPEC-470 (Expanding Control over the Operating System from the Database) |

**Rule of thumb tied to SDLC stage**:

- **Pre-architecture / requirements review** → Meta only. The model can't support more.
- **Architecture / design review** → Standard. This is where most threat models live; the system's components and data flows are known but implementation isn't locked.
- **Pre-implementation / component review** → Standard + Detailed where the implementation is specified.
- **Code review (validation pass, see `centric-methods.md` § Code-centric)** → Detailed. By this point you can see the parser, the framework, the DB driver.

If unsure, **cite the Standard pattern and link to the Detailed children**. That's honest about what the threat model knows while leaving the trail for component-level follow-up.

## Two CAPEC views — which to enter from

CAPEC organizes patterns under two top-level views. For threat modeling, enter from the right one for the question you're answering.

- **CAPEC-1000: Mechanisms of Attack.** Groups patterns by *what the attacker does*. This is the entry point for STRIDE-driven threat modeling — the categories below map roughly to STRIDE (see the mapping in the next section).
- **CAPEC-3000: Domains of Attack.** Groups patterns by *where the attack happens*: Software (CAPEC-513), Hardware (CAPEC-515), Communications (CAPEC-512), Supply Chain (CAPEC-437), Social Engineering (CAPEC-403), Physical Security (CAPEC-514). Useful for medical-device / embedded / ICS modeling that crosses multiple domains.

Practical pattern: walk Mechanisms first (STRIDE-aligned, finds the bulk of patterns), then walk Domains for any domain not covered by the contextual core (almost always: Hardware, Supply Chain, Physical Security for embedded/medical/ICS).

## STRIDE → CAPEC mapping

These are the most-cited parent patterns per STRIDE category. Walk down to children as the implementation detail allows. **Most threat models will cite Standard-level parents**; the children listed are illustrative starting points.

### Spoofing (Authentication violation)

Mechanisms-of-Attack root: **CAPEC-156 (Engage in Deceptive Interactions)**.

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-151 (Identity Spoofing) | Standard | Impersonating an external entity, AE Title spoofing, certificate forgery |
| CAPEC-148 (Content Spoofing) | Meta | Tampering with content the user/process trusts (DICOM tag manipulation, manipulated images) |
| CAPEC-21 (Exploitation of Trusted Identifiers) | Standard | Replayed session tokens, reused API keys, AE Title without TLS pinning |
| CAPEC-194 (Fake the Source of Data) | Standard | Forged sensor readings, spoofed telemetry, fake firmware-update server |
| CAPEC-89 (Pharming) | Detailed | DNS / hosts manipulation to redirect to attacker server |

CWEs commonly chained: CWE-287 (Improper Authentication), CWE-290 (Auth Bypass by Spoofing), CWE-291 (Reliance on IP Address for Authentication), CWE-294 (Auth Bypass by Capture-replay).

### Tampering (Integrity violation)

Mechanisms roots: **CAPEC-255 (Manipulate Data Structures)**, **CAPEC-262 (Manipulate System Resources)**, **CAPEC-152 (Inject Unexpected Items)**.

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-176 (Configuration/Environment Manipulation) | Standard | Device config tampering, env-var override, plist/registry tampering |
| CAPEC-153 (Input Data Manipulation) | Standard | Modified DICOM pixel data in transit, HL7 field manipulation |
| CAPEC-100 (Overflow Buffers) | Standard | Protocol parser overflow, image decoder overflow (DICOM, JPEG2000, NIfTI) |
| CAPEC-46 (Overflow Variables and Tags) | Detailed | Specific tag/length overflow in TLV-style protocols (DICOM PDU) |
| CAPEC-441 (Malicious Logic Insertion) | Standard | Firmware compromise, OTA-update payload tampering, signed binary substitution |
| CAPEC-438 (Modification During Manufacture) | Standard | Supply-chain tampering of devices before delivery |
| CAPEC-548 (Contaminate Resource) | Standard | Poisoning a shared store (training data, signed-package cache, DNS cache) |
| CAPEC-271 (Schema Poisoning) | Detailed | Schema/contract tampering (XSD, OpenAPI, GraphQL introspection abuse) |

CWEs: CWE-345 (Insufficient Verification of Data Authenticity), CWE-353 (Missing Support for Integrity Check), CWE-494 (Download of Code Without Integrity Check), CWE-119/-787 (bounds errors).

### Repudiation (Non-repudiation / accountability violation)

CAPEC coverage of repudiation specifically is thin — most repudiation findings cite **Tampering** patterns against logs:

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-93 (Log Injection-Tampering-Forging) | Standard | Forged or altered audit log entries |
| CAPEC-81 (Web Logs Tampering) | Detailed | Specifically web-server log manipulation |
| CAPEC-571 (Block Logging to Central Repository) | Detailed | Disrupting log forwarding to deny correlation |
| CAPEC-268 (Audit Log Manipulation) | Standard | General audit-log tampering |

CWEs: CWE-117 (Improper Output Neutralization for Logs), CWE-778 (Insufficient Logging), CWE-779 (Logging of Excessive Data), CWE-223 (Omission of Security-Relevant Information).

### Information Disclosure (Confidentiality violation)

Mechanisms root: **CAPEC-118 (Collect and Analyze Information)**.

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-117 (Interception) | Standard | Eavesdropping on DICOM C-STORE, sniffing un-TLS'd traffic, wireless capture |
| CAPEC-150 (Collect Data from Common Resource Locations) | Standard | PHI in logs / cores / temp files / debug output / telemetry |
| CAPEC-189 (Black Box Reverse Engineering) | Standard | Side-channel inference (timing, power), ML model extraction |
| CAPEC-188 (Reverse Engineering) | Meta | Firmware extraction from device, binary analysis to recover keys/secrets |
| CAPEC-545 (Pull Data from System Resources) | Standard | API enumeration, sequential-ID harvesting (IDOR-adjacent) |
| CAPEC-37 (Retrieve Embedded Sensitive Data) | Detailed | Hardcoded secrets in firmware/binaries/containers |

CWEs: CWE-200 (Exposure of Sensitive Information), CWE-319 (Cleartext Transmission), CWE-311 (Missing Encryption), CWE-532 (Insertion of Sensitive Info into Log File), CWE-798 (Use of Hard-coded Credentials).

### Denial of Service (Availability violation)

Mechanisms root: **CAPEC-262 (Manipulate System Resources)** for resource attacks; **CAPEC-227 (Sustained Client Engagement)** for protocol exhaustion.

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-119 (Resource Depletion) | Standard | Memory/handle/connection-pool exhaustion |
| CAPEC-125 (Flooding) | Standard | Network-layer flooding, DICOM-association flooding, syslog flooding |
| CAPEC-130 (Excessive Allocation) | Standard | Decompression bombs, billion laughs, large DICOM PDU |
| CAPEC-227 (Sustained Client Engagement) | Standard | Slowloris-style attacks against PACS / web frontends |
| CAPEC-2 (Inducing Account Lockout) | Detailed | Triggering lockout policies to deny legitimate access |

CWEs: CWE-400 (Uncontrolled Resource Consumption), CWE-770 (Allocation Without Limits), CWE-834 (Excessive Iteration), CWE-409 (Improper Handling of Highly Compressed Data).

### Elevation of Privilege (Authorization violation)

Mechanisms root: **CAPEC-225 (Subvert Access Control)**, plus **CAPEC-152 (Inject Unexpected Items)** for code-injection paths to EoP.

| CAPEC | Level | Use for |
|-------|-------|---------|
| CAPEC-233 (Privilege Escalation) | Meta | General EoP framing |
| CAPEC-122 (Privilege Abuse) | Standard | Using legitimate privileges in unintended ways |
| CAPEC-114 (Authentication Abuse) | Standard | Using auth in ways outside intended use |
| CAPEC-115 (Authentication Bypass) | Standard | Skipping authentication entirely (missing checks at boundaries) |
| CAPEC-1 (Accessing Functionality Not Properly Constrained by ACLs) | Standard | IDOR, missing authz checks, broken access control |
| CAPEC-242 (Code Injection) | Standard | RCE paths from input → execution |
| CAPEC-66 (SQL Injection) | Standard | DB injection → data extraction or RCE |
| CAPEC-88 (OS Command Injection) | Standard | Shelling out from web/management interface |
| CAPEC-549 (Local Execution of Code) | Detailed | Specifically local code execution after foothold |

CWEs: CWE-285 (Improper Authorization), CWE-269 (Improper Privilege Management), CWE-863 (Incorrect Authorization), CWE-732 (Incorrect Permission Assignment), CWE-22 (Path Traversal), CWE-94 (Improper Control of Generation of Code), CWE-89 (SQLi), CWE-78 (OS Command Injection).

## Domain-specific subset for medical-device / DICOM / embedded / ICS

For medical-device, embedded, and ICS work the relevant CAPEC subset is small enough to keep in one place. This is the working set; cite from here first and reach for the broader catalog only when none fit.

| Threat scenario | CAPEC | Notes |
|-----------------|-------|-------|
| AE Title spoofing on DICOM association | CAPEC-151, CAPEC-21 | mTLS + AE-Title-pinned cert mitigates both |
| DICOM tag manipulation (PatientName, PatientID swap) | CAPEC-148, CAPEC-153 | LINDDUN Linking applies too |
| DICOM PDU / pixel-data parser overflow | CAPEC-100, CAPEC-46 | Map to CWE-119/-787; fuzz the parser |
| DICOM / HL7 protocol manipulation (no Detailed pattern exists) | CAPEC-272 (Protocol Manipulation) | Closest available — see "coverage honesty" below |
| HL7 field injection via management interface | CAPEC-153, CAPEC-66/-88 (depending on sink) | Treat as injection family |
| Firmware modification (unsigned / weak-signed) | CAPEC-441, CAPEC-185 (Malicious Software Download) | CWE-494 chain |
| Supply-chain compromise of device pre-delivery | CAPEC-438 (Modification During Manufacture), CAPEC-439 (Manipulation During Distribution) | Rarely fixable post-deployment — emphasize provenance / SBOM |
| Management interface auth bypass | CAPEC-115, CAPEC-114 | Map to CWE-287/-285 |
| Management interface SQLi | CAPEC-66 | CWE-89; parameterized queries |
| Management interface command injection | CAPEC-88, CAPEC-242 | CWE-78/-94 |
| Device config tampering (operator panel, exposed serial console) | CAPEC-176 | Often paired with physical-access threats |
| PHI exfiltration via debug log / core dump / telemetry | CAPEC-150 | Maps to data-centric Output / Execution locations (`data-centric.md`) |
| Side-channel on cryptographic ops | CAPEC-189, CAPEC-188 | Power/timing/EM; relevant for embedded HSM-less devices |
| JTAG / debug-port exploitation | CAPEC-401 (Physically Hacking Hardware) | Domain: Hardware (CAPEC-515) |
| Hardware fault injection (glitching) | CAPEC-624 (Hardware Fault Injection) | Domain: Hardware |
| Wireless interception of telemetry / OTA traffic | CAPEC-117 | Often pair with CAPEC-194 (fake source) for active attacks |
| Decompression bomb / oversized study | CAPEC-130 | Availability + safety implication if it wedges acquisition |
| Operator credential phishing | CAPEC-98 (Phishing) | Domain: Social Engineering (CAPEC-403) |

## Honest about CAPEC coverage

CAPEC has thorough coverage of:
- Web-application attack patterns (injection family, broken access control, session attacks)
- General network-protocol attack patterns
- Cryptographic / authentication misuse patterns
- Supply-chain and physical-attack patterns at a generic level

CAPEC has patchier or no coverage of:
- **Medical-protocol-specific attacks** (DICOM, HL7, FHIR, IHE profiles). No Detailed pattern exists for "DICOM C-STORE PDU overflow"; the closest available is CAPEC-100 (Standard) or CAPEC-46 (Detailed but not DICOM-specific) plus CAPEC-272 (Protocol Manipulation, Meta).
- **ICS / OT protocols** (Modbus, OPC, S7, DNP3, IEC 61850). Domain-of-attack view (CAPEC-3000) has Communications (CAPEC-512) but the specific ICS protocols don't have dedicated patterns.
- **Embedded firmware specifics** beyond generic Hardware (CAPEC-515) coverage.
- **AI/ML attack patterns** — partial; supplement with OWASP LLM Top 10 and OWASP ML Security Top 10 (see `methodologies.md` § ML/AI).

**The honest framing for users**: when CAPEC has no Detailed pattern that matches your specific protocol or device class, **cite the closest Standard or Meta pattern and say so explicitly in the threat row**. That's still better than no CAPEC ID — the closest-fit pattern still points at the right CWE family and the right mitigation class. Don't waste time hunting for a pattern that doesn't exist.

Example threat-row note: *"CAPEC-272 (Protocol Manipulation, Meta) — no DICOM-specific pattern exists; cited as closest abstract match. Detailed mitigation derived from CWE-345 (Insufficient Verification of Data Authenticity) and DICOM-specific knowledge."*

## How CAPEC slots into the hybrid output

In a hybrid threat model with §2 split into three strata (see `methodologies.md` § "Three strata as default scaffolding"):

- **§2.1 Contextual stratum** — STRIDE/data-centric/etc. produce the threats. Don't add CAPEC IDs here yet; the contextual stratum is about what could go wrong, not its catalog ID.
- **§2.2 Operational stratum** — this is where the CAPEC column lives, alongside ATT&CK, CWE, CVSS. Each row: threat ID → CAPEC pattern (with abstraction level) → CWE(s) → ATT&CK mapping → kill-chain stage → detection notes.
- **§3 Mitigation table** — the CAPEC and CWE columns make the mitigation traceable. Derived security requirements (`SR-###`) cite both the threat ID and the CWE class so they're traceable to a known weakness.

Cross-stratum: a CAPEC pattern can be the bridge between a contextual threat (`T1`) and an operational ATT&CK technique. Don't duplicate — cross-reference: *"T1 (Spoofing of AE Title) ↔ CAPEC-151 ↔ CWE-287 ↔ ATT&CK T1078 (Valid Accounts)"*. One row in §3, four IDs across the chain.
