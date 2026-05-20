# STRIDE per element

STRIDE is six categories of threat, used here as a *prompt* to find threats — not as a taxonomy to file them into. For each element of the DFD, walk the categories that apply and ask: "how could an attacker do this against this element?"

| Letter | Threat | Violates |
|--------|--------|----------|
| **S** | Spoofing | Authentication |
| **T** | Tampering | Integrity |
| **R** | Repudiation | Non-repudiation / accountability |
| **I** | Information disclosure | Confidentiality |
| **D** | Denial of service | Availability |
| **E** | Elevation of privilege | Authorization |

(STRIDE was created by Loren Kohnfelder and Praerit Garg at Microsoft. A variant, DESIST — Gunnar Peterson — uses the same six threats with a non-repeating mnemonic. The categories are identical; the choice doesn't matter.)

## Applicability table

Not every category applies to every element type. Walk the marked cells:

| Element | S | T | R | I | D | E |
|----------------|---|---|---|---|---|---|
| External entity| ✓ |   | ✓ |   |   |   |
| Process        | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Data flow      |   | ✓ |   | ✓ | ✓ |   |
| Data store     |   | ✓ | ?* | ✓ | ✓ |   |

\* R on a data store applies only if it is an audit/logging store — otherwise a store just holds what it's told to hold.

This is **STRIDE-Per-Element** (Shostack), the default here. The table is a guide, not a contract: it tells you which prompts to walk so none gets skipped. Adapt it — for example, "information disclosure by an external entity" is unmarked but is a real privacy concern; if privacy is in scope, expand the table or run a LINDDUN pass (`methods.md`).

A useful exit criterion: one threat per marked cell means you're doing reasonably well. Circling back to threaten your own *mitigations* means you're doing well.

**The element in the table is the victim, not the attacker.** Tampering against a data store means the store is what gets tampered with. "Spoofing by tampering with the network" is really a spoof of an endpoint — the endpoint is the victim, wherever the attack technically happens.

## Category prompts

For each, the question is "how could an attacker accomplish this violation against this element?"

### Spoofing — Authentication

- Could an attacker pretend to be this external entity? (stolen credentials, replayed token, forged certificate)
- Could an attacker pretend to be this process to others? (DNS/ARP spoofing, container impersonation, BGP hijack)
- Is identity asserted but never verified? Is mutual auth used at trust boundaries, or only one-way?
- Are service and machine identities protected as well as user identities?

### Tampering — Integrity

- Can data in transit be modified? (no TLS, downgradeable TLS, no MAC, MITM-able)
- Can data at rest be modified? (writable by the wrong principal, no integrity check on read)
- Can the running process be tampered with? (library injection, container escape)
- Can configuration, policy, code, or firmware be modified? (unsigned binaries, missing secure boot, config drift)

### Repudiation — Accountability

- Can a user perform an action and later credibly deny it?
- Are logs written, tamper-evident, time-synced, and correlatable?
- Is there enough identity context to distinguish who did what?
- Can logs be deleted or edited by someone with operational access?
- For shared / service / break-glass accounts, can actions be tied back to an individual?

### Information disclosure — Confidentiality

- Can data in transit be read? (no TLS, weak TLS, sniffable wireless)
- Can data at rest be read by the wrong principal? (broken file perms, public bucket, unencrypted backup)
- Does the process leak via error messages, debug output, logs, or telemetry?
- Are secrets in code, config, environment variables, or command lines?
- Are caches, swap, core dumps protected? Side channels (timing, power, cache)?

### Denial of service — Availability

- Can the process be crashed by malformed input?
- Can a resource be exhausted? (memory, CPU, file handles, connection/thread pool, disk)
- Can a data store be filled, slow-queried, or locked?
- Can a flow be flooded, blocked, or jammed?
- Are dependencies (DNS, identity provider, key vault) single points of failure?
- For safety-critical systems, does the failure mode put a person or asset at risk?

### Elevation of privilege — Authorization

- Can a low-privileged user do high-privileged things? (broken access control, IDOR, missing boundary checks)
- Can untrusted input become code? (injection, deserialization, template injection, prompt injection)
- Can a process gain more privilege than it needs? (sudo misconfig, capabilities, setuid, container privilege)
- Can privilege escalate horizontally, between tenants or peers?
- Is there a path from "I have a shell" to "I have the keys to the kingdom"?

## Per-element example threats

Illustrative seeds, not a checklist.

**External entity** (e.g. a user)
- S: Attacker phishes the user's credentials and authenticates as them.
- R: User performs a sensitive action, then claims they didn't.

**Process** (e.g. an API service)
- S: Attacker spoofs an upstream service the API trusts.
- T: Attacker exploits a parser flaw to alter execution.
- R: Attacker erases service-local logs after acting.
- I: Attacker reads secrets from an error response or debug log.
- D: Attacker exhausts the connection pool with slow requests.
- E: Attacker turns input into code and runs as the service account, then pivots.

**Data flow** (e.g. service-to-database traffic)
- T: Attacker on the segment modifies data in transit.
- I: Attacker on the segment captures sensitive data.
- D: Attacker floods the segment to delay legitimate traffic.

**Data store** (e.g. a database)
- T: Attacker with write access modifies historical records.
- I: Attacker with read access exfiltrates the dataset.
- D: Attacker fills the volume to halt writes.
- R (audit store only): Attacker deletes log rows covering their activity.

## STRIDE → security property → typical mitigations

| STRIDE | Property | Typical mitigations |
|--------|----------|---------------------|
| Spoofing | Authentication | mTLS, MFA, signed tokens, certificate pinning, hardware-backed identity |
| Tampering | Integrity | hashes, MACs, signatures, append-only logs, secure boot, code signing |
| Repudiation | Non-repudiation | signed audit logs, synced timestamps, per-identity attribution, log forwarding to an isolated collector |
| Information disclosure | Confidentiality | encryption at rest and in transit, key management, RBAC/ABAC, secret hygiene |
| Denial of service | Availability | rate limiting, quotas, backpressure, redundancy, autoscaling, graceful degradation |
| Elevation of privilege | Authorization | least privilege, sandboxing, input validation, output encoding, RBAC/ABAC, separation of duties |

## When STRIDE-Per-Element misses things

STRIDE on a DFD is element-centric and design-time. It under-covers some things — but the fix differs:

**Inventory gaps — fix by expanding the model, not the methodology.**
- *Supply-chain threats* — a compromised library, build pipeline, or signing key. A build server is an asset; a CI/CD pipeline is a process; a signed-artifact flow is a flow. If they're in the DFD, STRIDE works on them. If not, add them.
- *Operational threats* — credential leakage from a laptop, on-call account abuse. Same story — processes and assets that weren't enumerated.

**Methodology gaps — fix by supplementing the lens (`methods.md`).**
- *Privacy* — "Information disclosure" is a coarse bucket next to LINDDUN's seven privacy categories.
- *Business logic* — STRIDE won't surface "a user can refund themselves twice"; abuse cases will.
- *Adversarial ML* — for ML components, add model-specific threats (prompt injection, poisoning, extraction).

If any of these feels under-covered after the STRIDE pass, run a focused supplementary pass and add a short "Beyond STRIDE" subsection.
