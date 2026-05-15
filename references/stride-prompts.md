# STRIDE Prompts (per element type)

> **Last verified**: 2026-05. The STRIDE-Per-Element / Per-Interaction conventions are stable; re-confirm before citing if Microsoft updates its Threat Modeling Tool documentation.
> **Sources paraphrased**: Adam Shostack, *Threat Modeling: Designing for Security* (Wiley, 2014) — STRIDE-Per-Element framing, applicability table, "victim not perpetrator" rule, exit-criterion guidance, enumeration tactics (paraphrase only); Loren Kohnfelder & Praerit Garg (Microsoft) — original STRIDE; Larry Osterman / Douglas MacIver (Microsoft) — STRIDE-Per-Interaction; Gunnar Peterson — DESIST. Substantive direct quotes from Shostack 2014 require Wiley/Shostack attribution.

> **Related**: ← `SKILL.md` • `dfd-mermaid.md` (draw the DFD first) • `centric-methods.md` (STRIDE as generation vs characterization) • `methodologies.md` (when to swap or supplement STRIDE — LINDDUN, AI/ML, etc.) • `capec.md` (STRIDE → CAPEC → CWE chain) • `validation.md` (Q4 Threats checklist).

This file is the **canonical home** for the STRIDE-Per-Element applicability table and the STRIDE → security property → typical mitigations table. SKILL.md references both; don't duplicate.

When threat-enumerating a DFD, walk each element through the STRIDE categories that apply. The applicability table:

| Element        | S | T | R              | I | D | E |
|----------------|---|---|----------------|---|---|---|
| External entity| ✓ |   | ✓              |   |   |   |
| Process        | ✓ | ✓ | ✓              | ✓ | ✓ | ✓ |
| Data flow      |   | ✓ |                | ✓ | ✓ |   |
| Data store     |   | ✓ | ✓ (audit only) | ✓ | ✓ |   |

R on a data store applies **only when the store is an audit or logging store** — modifying or deleting log rows is a repudiation issue because it undermines accountability for prior actions. Non-audit data stores don't have a repudiation surface (they store what they're told to store), so leave R blank for them. If a single store carries both audit data and non-audit data, split it into two rows so the applicability is unambiguous.

This is **STRIDE-Per-Element** (Shostack), the default for this skill. There are two important alternatives:

- **STRIDE-Per-Interaction** (Larry Osterman / Douglas MacIver, Microsoft) — runs STRIDE on each (origin, destination, interaction) tuple instead of each element. It generates roughly the same number of threats as Per-Element but they may be easier to reason about for some teams. Per-Interaction is "too complex to use without a reference chart handy" — Per-Element is simple enough to memorize. Use Per-Interaction when Per-Element feels too coarse, or when the team is already used to thinking in interactions (typical of network/protocol work).
- **DESIST** (Gunnar Peterson) — same six categories with a different acronym (Dispute, Elevation of privilege, Spoofing, Information disclosure, Service denial, Tampering). The point of DESIST is that "STRIDE" has a repeated letter and is therefore harder to use as a true mnemonic. STRIDE remains dominant by inertia, but DESIST is defensible if starting from scratch. The threats covered are identical.

A subtle but important framing rule: **the element listed in the table is the victim, not the perpetrator.** If you're modeling tampering against a data store, the data store is what gets tampered with. If you're modeling spoofing affecting a process, the process is what gets confused. "Spoofing by tampering with the network" is really a spoof of an endpoint — the endpoint is the victim, regardless of where the attack happens technically.

## Note on the table itself

The applicability table above is essentially the Microsoft STRIDE-Per-Element chart. It's slightly opinionated — for example, "information disclosure by external entity" is unmarked, even though that's a real privacy issue. Microsoft handles privacy with a separate process; if privacy is in scope for your work, either expand the table or run a LINDDUN pass alongside (see `methodologies.md`). This is normal — adapt the table to your scenario rather than treating it as a fixed checklist.

A reasonable exit criterion (Shostack): when there's at least one threat per check-marked cell in the table, you're doing reasonably well. If you also circle back to consider threats against your *mitigations* (and ways to bypass them), you're doing pretty well.

## Empty STRIDE cells require an explicit rationale, not silent omission

STRIDE-Per-Element is a **gap-finding tool, not a coverage quota** (MITRE Threat Modeling Playbook §2.4.1: "STRIDE per Element approach can be helpful when trying to identify when to stop brainstorming"). The skill's framing is *generate, don't categorize* — but the playbook adds a rule that's worth elevating here: **if a check-marked applicability cell yields no threat for a given element, document why, don't leave it silently blank**.

The playbook's specific call-out is for Repudiation: "Repudiation threats can often be easy to skip over… If you have performed an initial pass… and have a process that does not include a Repudiation threat associated with it, it can be productive to spend additional time brainstorming to identify if that threat applies. That threat may not exist, in which case it may be helpful to document why it doesn't apply." The same logic generalizes to every applicability cell.

Concrete rule for the threat table:

- For every applicability cell marked `✓` in the table above (External entity S/R; Process S/T/R/I/D/E; Data flow T/I/D; audit-only Data store S/T/R/I/D), either record a concrete threat or record an explicit *no-threat row* with rationale. *"R / Process — no repudiation surface: this stateless validator emits no audit log because all actions are reflected in the downstream service's signed audit stream (T7's countermeasure)"* is a no-threat row. A blank cell is not.
- Silent omission is indistinguishable from "we forgot to walk this cell" — the same anti-pattern as silent acceptance / silent transference (`manifesto.md` § "Anti-patterns" → "Silent risk transference / silent risk acceptance"). A reviewer reading the model can't tell the difference.
- The "split a data store that holds both audit and non-audit data into two rows" rule above (line 19) is one instance of this rule — it makes R-applicability explicit per-row instead of letting it go silent.

In TM-BOM emission, no-threat rows do not become `threats[]` entries (the schema requires `event` to be non-empty); record them in the markdown only, or in the top-level `extensions` block as a per-element rationale map if downstream tooling needs to consume them.

## Category prompts

For each category, the question to ask is "How could an attacker accomplish [violation] against this element?"

### Spoofing — violates Authentication

Prompts:
- Could an attacker pretend to be this external entity? (stolen credentials, replayed token, forged certificate)
- Could an attacker pretend to be this process to other components? (DNS spoofing, ARP poisoning, container impersonation, BGP hijack)
- Are there places where identity is asserted but never verified?
- Is mutual authentication used at trust boundaries, or only one-way?
- Are service accounts and machine identities as well-protected as user identities?

Common in medical / IoT / embedded:
- Could an attacker spoof a trusted DICOM AE Title?
- Could an attacker spoof a sensor reading or telemetry from a peripheral device?
- Are firmware update servers authenticated to the device?

### Tampering — violates Integrity

Prompts:
- Can data in transit be modified? (no TLS, downgradeable TLS, no MAC, MITM-able)
- Can data at rest be modified? (writable by the wrong principal, no integrity check on read)
- Can the running process be tampered with? (DLL injection, library shimming, container escape, eBPF abuse)
- Can configuration / policy be modified? (file permissions, config drift)
- Can code or firmware be modified? (unsigned binaries, missing secure boot)

Common in medical / IoT / embedded:
- Can device firmware be flashed by an unauthenticated party?
- Can DICOM image pixel data be modified in transit?
- Can settings / dose calculations / control loops be tampered with?

### Repudiation — violates Non-repudiation / Accountability

Prompts:
- Can a user perform an action and later credibly deny it?
- Are logs written? Are they tamper-evident? Are they correlatable?
- Is there enough identity context in logs to distinguish who did what?
- Can logs be deleted or edited by someone with operational access?
- For shared / service / break-glass accounts, can individual humans be tied back to actions?

Common in medical / IoT / embedded:
- Are clinical actions (dose changes, scan orders, override events) attributable to a specific user?
- Are audit logs IEC 62443 / HIPAA-aligned?
- Is there a chain-of-custody for evidence?

### Information Disclosure — violates Confidentiality

Prompts:
- Can data in transit be read? (no TLS, weak TLS, sniffable wireless, side channels)
- Can data at rest be read by the wrong principal? (broken file perms, public S3 bucket, unencrypted backup)
- Does the process leak data through error messages, debug output, log files, telemetry?
- Are secrets stored in code, config, environment, command line?
- Are caches, swap, core dumps, or memory-mapped files protected?
- Side channels: timing, power, EM, cache, microarchitectural?

Common in medical / IoT / embedded:
- Is PHI exposed in DICOM tags, log files, or debug output?
- Can over-the-air firmware leak via radio sniffing?
- Are unique device identifiers leaked in ways that enable tracking?

### Denial of Service — violates Availability

Prompts:
- Can the process be crashed by malformed input?
- Can the process be exhausted? (memory, CPU, file handles, connection pool, thread pool, disk)
- Can the data store be filled, slow-queried, or locked?
- Can the data flow be flooded, blocked, or jammed?
- Are dependencies (DNS, NTP, identity provider, key vault, license server) single points of failure?
- For safety-critical systems: does the failure mode put a person / asset at risk?

Common in medical / IoT / embedded:
- Can the device be wedged remotely? Bricked? Battery-drained?
- Can a DoS on a clinical system delay treatment? (this is a *safety* event, not just availability)
- Are watchdogs and graceful degradation paths in place?
- For wireless flows / processes: can a rogue device on a *shared physical medium* (e.g. the 2.4 GHz ISM band shared across Wi-Fi / BLE / Zigbee, the same RF channel, the same power rail) jam or DoS the flow without ever associating with the system? (P3 between-device surface — see `methodologies.md` § "Risk-prioritized cyber-physical attack paths".)

### Elevation of Privilege — violates Authorization

Prompts:
- Can a low-privileged user do high-privileged things? (broken access control, IDOR, missing checks at boundaries)
- Can untrusted input become code? (injection, deserialization, template injection, prompt injection for LLM components)
- Can the process gain more privilege than needed? (sudo misconfig, capabilities, setuid binaries, container privilege)
- Can horizontal privilege escalation happen between tenants / users at the same trust level?
- Is there a path from "I have a shell" to "I have the keys to the kingdom"?

Common in medical / IoT / embedded:
- Can a network-side compromise reach the safety-critical control loop?
- Can a low-privilege technician account access patient data?
- Does any input path eventually feed into firmware update logic?

## Per-element example threats

These are illustrative, not exhaustive. Use them as seeds, not a checklist. Each is written in the threat-table cell format from `SKILL.md` § "Threat enumeration" — a ≤6-word title, a colon, then a 1–2 sentence concrete description.

### External entity (e.g. Clinician using PACS workstation)

- **S** — **Clinician credential phishing**: An attacker phishes a clinician's credentials and authenticates to the workstation as them.
- **R** — **Denied PHI export**: A clinician performs an unauthorized PHI export and later credibly denies it because no attributable log exists.

### Process (e.g. PACS server)

- **S** — **DICOM AE Title spoofing**: An attacker spoofs a trusted AE Title to deliver malicious imagery to the PACS server.
- **T** — **DICOM parser buffer overflow**: An attacker sends a crafted DICOM object that overflows a parser buffer and modifies process execution.
- **R** — **Audit log erasure**: An attacker with operational access erases server-local audit logs after tampering with stored images.
- **I** — **PHI leak via temp files**: An attacker reads PHI from temporary files the server writes during DICOM processing.
- **D** — **Connection-pool exhaustion**: An attacker opens crafted associations until the connection pool is exhausted and legitimate modalities can't connect.
- **E** — **Parser RCE to domain admin**: An attacker exploits a parser RCE to run as the PACS service account, then pivots to domain admin.

### Data flow (e.g. DICOM C-STORE between modality and PACS)

- **T** — **Pixel data tampering in transit**: An attacker on the segment modifies pixel data in transit because the flow has no TLS and no integrity check.
- **I** — **PHI capture in transit**: An attacker on the segment captures PHI from the unencrypted DICOM flow.
- **D** — **Segment flooding during a procedure**: An attacker floods the network segment to delay imaging while a procedure is underway.

### Data store (e.g. PACS image database)

- **T** — **Historical study modification**: An attacker with DB-write access modifies historical studies.
- **I** — **Study catalog exfiltration**: An attacker with DB-read access exfiltrates the study catalog.
- **D** — **Volume-fill halt**: An attacker fills the underlying storage volume so the database can't accept new acquisitions.
- **R** (audit log only) — **Audit row deletion**: An attacker deletes audit rows covering their access window.

## STRIDE → security property → typical mitigations

| STRIDE | Property | Typical mitigations |
|--------|----------|---------------------|
| Spoofing | Authentication | mTLS, MFA, signed tokens, certificate pinning, hardware-backed identity (TPM, SE) |
| Tampering | Integrity | hashes, MACs, digital signatures, write-once / append-only logs, secure boot, code signing |
| Repudiation | Non-repudiation | signed audit logs, NTP-synced timestamps, attribution to individual identities, log forwarding to an isolated collector |
| Information Disclosure | Confidentiality | encryption (at-rest + in-transit), key management, RBAC/ABAC, secret hygiene, side-channel hardening |
| Denial of Service | Availability | rate limiting, quotas, backpressure, redundancy, autoscaling, graceful degradation, watchdogs |
| Elevation of Privilege | Authorization | least privilege, sandboxing, input validation, output encoding, RBAC/ABAC, separation of duties |

## Choosing the right primitive

Two rules layered on top of the typical-mitigations table:

**Prefer built-ins over custom code.** Platform IAM, KMS / HSM, managed mTLS, K8s NetworkPolicy, browser CSP / SOP / SameSite, Subresource Integrity, framework CSRF tokens, ORM parameterization, OS sandboxing (SELinux / AppArmor / seccomp / capabilities), secure boot / TPM / TEE / secure element, memory-safe languages — these already carry the vendor's threat model and ship with the vendor's patches. A hand-rolled equivalent inherits the team's bandwidth and tends to drift. The decision to roll a custom control over an available built-in is itself worth recording as an `ASM#` with a residual-risk note.

**When the control is cryptographic, name the suite — not the class.** *"Use encryption"* is the "Generic mitigations" anti-pattern. Default suites for new designs:

- **Transport**: TLS 1.3 (or TLS 1.2 with AEAD ciphers only); mTLS for service-to-service.
- **Symmetric / AEAD**: AES-256-GCM or ChaCha20-Poly1305 — never CBC + HMAC for new designs.
- **Signatures**: Ed25519, ECDSA P-256 / P-384; RSA ≥ 3072 if asymmetric must be RSA.
- **Key agreement**: X25519, ECDH P-256+.
- **KDFs**: HKDF for context derivation; Argon2id (or scrypt) for password hashing — never bare SHA-256 over passwords.
- **Hashes**: SHA-256 / SHA-384 / SHA-3 family.
- **Random**: OS CSPRNG (`getrandom`, `/dev/urandom`, `BCryptGenRandom`) — never `rand()` / `Math.random()` for security.
- **Module**: FIPS 140-3 validated when regulated (federal, healthcare, payment).

Avoid: MD5, SHA-1, RC4, DES / 3DES, CBC-without-MAC, RSA < 3072 for new keys, ECB mode. Pointers: NIST SP 800-131A (algorithm transitions), NIST SP 800-175B (key management), IETF BCP-195 (TLS configuration), BSI TR-02102 (German baseline).

## NIST SP 800-53 r5 control-family linkage (optional)

Every Mitigate row **may** carry an SP 800-53 r5 control-family tag alongside its CSF function. The CSF function answers *what role the control plays*; the 800-53 family answers *which compliance baseline it threads through* — useful when FedRAMP / FISMA / CMMC / HITRUST mappers consume the model. Skip when no compliance baseline applies.

Common families (full catalog: NIST SP 800-53 r5):

| Family | Name | Typical mitigations |
|---|---|---|
| AC | Access Control | RBAC, ABAC, session management, separation of duties |
| AU | Audit & Accountability | Signed audit logs, log forwarding, retention |
| CM | Configuration Management | Immutable images, drift detection, baseline configs |
| CP | Contingency Planning | Backup, restore, failover, DR runbook |
| IA | Identification & Authentication | mTLS, MFA, machine identity, password policy |
| IR | Incident Response | IR plan, on-call rotation, isolation playbook |
| SC | System & Communications Protection | TLS, encryption at rest, segmentation, key management |
| SI | System & Information Integrity | Input validation, code signing, secure boot, integrity monitoring |
| SR | Supply Chain Risk Management | SBOM, dependency pinning, vendor risk reviews |
| PE | Physical & Environmental Protection | Tamper-evident enclosure, facility access |

Annotation format on a §3 row or SR: *"SR-001 (Protect, SC-8 / SC-12): TLS 1.3 + mTLS using X.509 certs from internal CA"*.

## NIST CSF function — Protect / Detect / Respond / Recover

The STRIDE → property → control axis above answers *what kind of control*. There's a second axis FDA reviewers and the MITRE Threat Modeling Playbook §2.5.2 expect to see populated alongside it: **which NIST Cybersecurity Framework function** the control implements. **Lead with Protect**: preventive controls are the primary class — most of the Mitigate stack on a typical threat should be Protect. Detect / Respond / Recover are compensating layers that catch the Protect failure when (not if) one fires silently, not four equal partners. A mitigation stack made entirely of Protect controls on a high-impact threat is still a finding — name at least one Detect path and one Respond / Recover path so the design has a fallback.

| NIST CSF function | What the control does | Examples |
|---|---|---|
| **Protect** | Prevent the threat from succeeding in the first place | mTLS, signed firmware, RBAC, input validation, rate limiting (most of the controls in the table above) |
| **Detect** | Notice the attack happening or that it has happened | Anomaly detection, audit-log forwarding to SIEM, integrity-check alerts on signed binaries, watchdogs on safety-critical loops, intrusion detection at trust boundaries |
| **Respond** | Contain and act on a detected event | Incident-response runbooks, automated key rotation on suspected compromise, network isolation playbooks, alarm-acknowledgement procedures, the device's "fail-safe mode" trigger |
| **Recover** | Restore service and integrity after a successful attack | Backup-restore procedures, firmware rollback, key re-issuance, post-incident state reconciliation, communications plan to clinicians/operators |

(`Identify` and `Govern` round out the CSF v2 model but live in the strategic stratum / governance program, not in per-threat mitigations — record them in §1 prose, not in §3 rows.)

How to use this in §3:

- **Tag every Mitigate row with the CSF function it implements.** A `controls[]` row in the TM-BOM extension namespace can carry this as `threat-modeler.tmskill/csf-function-by-control` keyed by the control's symbolic name; in the markdown, add a column or inline annotation: *"SR-001 (Protect): mTLS for service-to-service calls"*.
- **Audit the table for balance.** When all controls cluster under `Protect`, prompt the team for at least one `Detect` control per high-risk threat and at least one `Respond` / `Recover` path per safety-critical threat. The question to ask: *"if this Protect control fails silently, what tells us, and what do we do?"*
- **Detect/Respond/Recover gaps are findings.** A model where a high-risk threat has only `Protect` controls and no `Detect` is incomplete — log it as an open question in §4 or as a derived requirement (`SR-###`) for the missing function.

The CSF function is independent of STRIDE — Spoofing controls can be Protect (mTLS), Detect (impossible-travel detection on the IdP), Respond (force-logout on suspected token compromise), or Recover (re-issue all session tokens after a confirmed key compromise). The two axes (STRIDE category, CSF function) compose; populate both.

NIST CSF v2.0: https://www.nist.gov/cyberframework

## When STRIDE-Per-Element misses things

STRIDE is element-centric and design-time. It tends to under-cover certain things, but the reasons differ:

**Inventory problems (fix by expanding the model, not by adding methodology):**

- **Supply chain threats** — compromise of an upstream library, build pipeline, signing key, or container base image. These aren't a STRIDE gap; they're an inventory gap. An Artifactory server is *an asset*; a CI/CD pipeline is *a process*; a signed-firmware distribution flow is *a flow*. If they're not in your DFD, add them and STRIDE will work fine. See `centric-methods.md`.
- **Operational threats** — credential leakage from a developer laptop, on-call account abuse, log access. Same story — these are processes and assets that just weren't enumerated. Process-centric entry point catches them naturally.

**Genuine methodology gaps (fix by supplementing the lens):**

- **Privacy-specific threats** — see LINDDUN in `methodologies.md`. STRIDE's "Information Disclosure" is a coarse bucket compared to LINDDUN's seven privacy categories.
- **Business-logic flaws** — STRIDE won't tell you that "users can refund themselves twice"; abuse cases (user-needs-centric entry point) will.
- **Adversarial-ML threats** — for systems with ML components, supplement with model-specific threats (prompt injection, training-data poisoning, model extraction, etc.).

If any of these feel under-covered after the STRIDE pass, do a focused supplementary pass and add a short "Beyond STRIDE" subsection to the threat enumeration.

## Enumeration tactics

Practical guidance for working through STRIDE-Per-Element efficiently. Drawn from Shostack's chapter on threat enumeration; the tactics are method-agnostic but most relevant when STRIDE-Per-Element is the working lens.

- **Top-down, breadth-first.** Start from the highest-level view of the whole system and iterate breadth-first across elements rather than depth-first into one component. Bottom-up — building per-feature models and trying to aggregate them — does not produce a coherent system view, and is a known failure mode.
- **Pick an iteration axis.** Three valid options: walk **across trust boundaries** (highest-value threats live there), walk **across diagram elements** (good when multiple sub-teams collaborate), or walk **across the threats themselves** ("where are all the spoofing threats?" — good for finding related threats). Pick one; don't let the choice become a straitjacket. If unsure, start at trust boundaries.
- **Start with external entities** (or whatever crosses the outermost trust boundary). It's a natural anchor for "where could an attacker reach in?".
- **Never skip a threat because it's not what you're looking for right now.** If you're walking Spoofing and notice a Tampering issue, write it down. Redundancy is fine.
- **Focus on feasible threats.** "Someone might insert a backdoor at the chip factory" is a real possibility, but for most systems it's not a productive threat to model. Stay where the team can act.
- **Threats follow data flow, not control flow** — and they cluster around trust boundaries *and* complex parsing. Trust-boundary clustering is a starting point, not a limit; complex parsers (deserializers, image decoders, structured-document parsers) are threat-rich even inside a trust zone.
- **Five threats per diagram element is a reasonable rule of thumb** when using DFD + STRIDE. Significantly fewer probably means an element was under-analyzed; significantly more probably means redundant threats that can be combined.
- **Document assumptions as you go**, not upfront. When someone says "I assume that…", write it down right then, in falsifiable form. The assumption list becomes test cases, not pedantry. (More in `validation.md` § "Document assumptions as you go".)
- **Threats become bugs.** Every recorded threat ends up as a tracked work item in whatever bug/issue system the team uses. This is the exit point from threat modeling — once threats are bugs, normal triage and engineering machinery takes over.
- **Mitigations are themselves attack surface.** Once mitigations are drafted, do a second pass on them — a new auth service introduces new spoofing paths; a new logging path introduces new disclosure paths.
