# Per-environment trust-boundary patterns

> **Last verified**: 2026-05. Cloud-provider service names, Purdue Reference Architecture levels, Microsoft AD Tier model conventions, mobile-OS sandbox/keystore primitives, OWASP LLM/ML Top 10 entries, EU AI Act / NIST AI RMF citations, and the OS-vendor and carrier ownership boundaries listed here all change on independent cadences. Re-confirm cloud-managed-service names and AI-Act risk-classification language before citing them in a deliverable.
> **Sources paraphrased**: AWS / Azure / GCP / OCI shared-responsibility documentation (public); Purdue Enterprise Reference Architecture (ISA-95); Microsoft Enterprise Access Model (public Microsoft documentation, paraphrase); Apple iOS Security Guide (public, paraphrase); Android Open Source Project security documentation (Apache 2.0); Stellios et al. 2021 (paraphrase, see methodologies.md); OWASP LLM Top 10 / ML Security Top 10 (CC-BY 4.0); NIST AI RMF (US public domain); EU AI Act (EU public).

> **Related**: ← `SKILL.md` (Round 1.5 references this) • `dfd-mermaid.md` (subgraph labeling convention; generic boundary list this file extends) • `methodologies.md` (system-type matrix — picks which contextual supplements apply) • `centric-methods.md` (entry-point taxonomy)

This file is the **canonical home** for environment-specific trust-boundary patterns and for the ownership taxonomy that generalizes cloud's shared-responsibility model to embedded, OT, mobile, and enterprise. The generic boundary list in `dfd-mermaid.md` § "Trust boundaries — what to include" is the floor (UID, NIC, VLAN, container, org, tenant); this file is the ceiling — the patterns that catch boundaries the generic list misses.

Use it after Round 1.5 of the interview, when you've named the environment type per zone and the owner of each environment, and you're ready to draw the DFD.

## Why environment matters for the DFD

A DFD without environmental context tends to draw one or two trust boundaries (e.g. "internet vs. internal") and miss the ones that actually carry threats. Each environment type has *characteristic* boundaries — patterns that apply to most systems of that kind regardless of vendor or domain — and missing them is a consistent source of incomplete models. The list below isn't exhaustive; it's the boundaries that should make you suspicious if they're *absent* from the DFD.

A second function of this file: making **ownership** explicit. Cloud's shared-responsibility model is well known, but the same idea applies to every environment. The mobile-OS vendor owns the kernel; the carrier owns the baseband; the user owns the device; the app developer owns the app sandbox contents. The plant operator owns the PLC; the integrator owns the engineering workstation; the OEM owns the firmware. When mitigations are assigned in §3 of the threat model, ownership tells you who can actually implement them.

## Ownership taxonomy

For every subgraph in the DFD, name the owner. Use this taxonomy as a starting point — extend it where the system needs more granularity:

| Owner | Typical responsibilities (controls they can change) |
|---|---|
| **Vendor / maker** (you, if you're building the product) | Source code, signing keys, default config, OTA update server, support backdoors |
| **Customer organization** (org deploying the product) | Network segmentation, firewall rules, IAM, host hardening, patch cadence |
| **End-user** (individual using the product) | Device passcode, app permissions, OS updates (mobile), physical custody |
| **Cloud provider** (AWS / Azure / GCP / OCI / etc.) | Hypervisor, control plane, managed-service internals, data-center physical security |
| **Mobile-OS vendor** (Apple / Google) | Kernel, sandbox enforcement, keychain/keystore, app store gating, OS patches |
| **Carrier / MNO** | Baseband firmware, SIM, MNO core network, SMS path |
| **Hospital IT / plant operator / utility** (sector-specific deployer) | Network, AD, jump hosts, segmentation, monitoring; sector-specific |
| **Third-party SaaS** | Anything inside the SaaS boundary — you only get configuration knobs and audit logs |
| **Integrator / VAR** | Engineering workstation, deployment scripts, custom integration code |
| **Open-source / upstream** | Library / firmware components you depend on but don't control |
| **No single owner (public internet, public space)** | Public-internet transit, kiosk-public-area surroundings — no controls available; treat as untrusted. Use the literal label `Unowned` or `Public` in the subgraph |

Mark every subgraph with its owner using the labeling convention from `dfd-mermaid.md` § "Subgraph labeling convention" (`subgraph ID["<owner> | <env-type> | <trust>"]`). If an owner can't be named, record an explicit assumption (e.g. "ASM2: assumed customer IT owns the on-prem network; correct if otherwise") and proceed — don't block the model on it.

Ownership ≠ data custody (who holds the data) and ≠ legal accountability (who's on the hook for a breach). It's narrower: who can change configuration, who runs the patch cycle, whose IAM controls access, whose physical security applies. Custody and accountability often follow ownership but not always — call out the divergence when it matters (e.g. cloud provider owns the host; customer holds the data; HIPAA accountability sits with the customer).

## Environment patterns

### Cloud (AWS / Azure / GCP / OCI / etc.)

**Boundaries to expect:**

- **Account / subscription / project / tenancy boundary** — the outermost cloud-side boundary. Every cross-account access is a boundary crossing.
- **Region and availability-zone boundaries** — when data egresses cross-region, especially across regulator boundaries (EU ↔ US, US ↔ CN).
- **VPC / VNet / subnet boundaries** — public subnet vs. private subnet, ingress/egress controls.
- **IAM principal boundaries** — every IAM role, instance profile, or service principal is a trust boundary. Cross-account assume-role is a boundary crossing.
- **Managed-service control plane vs. data plane** — control-plane API access (e.g. `kms:*`) vs. data-plane operations (e.g. `kms:Decrypt`). Many cloud privilege-escalation findings cross this boundary.
- **Tenant boundary** in multi-tenant SaaS — row-level / object-level isolation; KMS-key-per-tenant boundaries.
- **KMS key / HSM boundary** — the boundary between code that holds plaintext and code that only has handles.
- **CI/CD ↔ runtime boundary** — your build-time IAM vs. your runtime IAM. Often under-modeled.
- **Cloud provider control plane ↔ your workload** — the line of shared responsibility. Note explicitly.

**Ownership default:** cloud provider owns hypervisor, managed-service internals, physical security; you own everything inside your account boundary; customer owns their account in B2B SaaS scenarios.

**Subgraph examples:**

```
subgraph ProdAcct["Customer Org | cloud (AWS prod account) | high trust"]
subgraph BuildAcct["Vendor | cloud (AWS build account) | medium trust"]
subgraph CSP["AWS | cloud provider control plane | out-of-scope owner"]
```

**Common misses:** the build/CI account as a separate trust zone; the KMS key boundary; cross-region replication; the IAM boundary between control plane and data plane.

### On-prem enterprise (Windows AD / Linux server / mixed)

**Boundaries to expect:**

- **AD forest / domain / OU boundaries** — forest is the security boundary in AD; trust between domains is a boundary crossing. Bidirectional/transitive trusts are flow-direction-specific.
- **Identity provider / SSO federation boundaries** — Okta / Azure AD / ADFS / Ping, including SAML/OIDC trust to relying parties.
- **Tier 0 / Tier 1 / Tier 2 boundary** (Microsoft's enterprise access model) — domain controllers and identity systems (Tier 0), servers (Tier 1), workstations (Tier 2). Cross-tier authentication is the most consequential boundary in AD environments.
- **Jump host / bastion boundary** — the only path to admin a privileged zone. If the diagram doesn't show one, ask whether it should.
- **Service account vs. interactive user boundary** — service accounts often have broad privileges and weak rotation; treat them as a separate trust level.
- **GPO scope boundaries** — different GPOs apply different policies; a workstation moving between OUs changes its security posture.
- **Sudo / UAC / privilege-escalation boundaries** — within a single host, the boundary between a normal user shell and root/SYSTEM.
- **DMZ ↔ internal boundary** — public-facing servers vs. internal corporate network.
- **VLAN / network segment boundaries** — corp net, server net, OOB management net, guest net.
- **VPN concentrator boundary** — remote workers and vendor support sessions cross here.

**Ownership default:** customer IT owns AD, network, hosts, GPOs, IAM. Vendor owns application code and configuration *defaults*. Watch for "the vendor service account has Domain Admin" — common and a serious finding.

**Subgraph examples:**

```
subgraph Tier0["Customer IT | on-prem enterprise (AD Tier 0) | very high trust"]
subgraph CorpNet["Customer IT | on-prem enterprise (corp VLAN) | medium trust"]
subgraph DMZ["Customer IT | on-prem enterprise (DMZ) | low trust"]
```

**Common misses:** the AD tier model; service account privilege levels; the VPN/jump-host as a distinct zone; SSO-federated relying parties as separate trust boundaries.

### Embedded / IoT (microcontroller, SoC, appliance, medical device)

**Boundaries to expect:**

- **Secure boot / verified boot chain boundaries** — bootloader → kernel → application; each handoff is a trust transition.
- **Secure element / TEE / TPM boundary** — the boundary between code that holds key material and code that only requests signing. ARM TrustZone (TrustZone-A on Cortex-A, TrustZone-M on Cortex-M); dedicated secure elements (NXP A71CH, Microchip ATECC, Infineon OPTIGA); TPMs. (Server-class TEEs like Intel SGX or AMD SEV occasionally appear on industrial-PC-class "embedded" gateways — note explicitly when they do, since their threat model differs.)
- **Hardware debug interface boundary** — JTAG, SWD, UART console, ICE. Often the most consequential physical-attack boundary on a device. If the device ships with debug ports unfused, draw them as a boundary crossing into the device's privileged state.
- **Bootloader / recovery / DFU mode boundary** — alternative boot paths that bypass normal application controls.
- **OTA update server ↔ device boundary** — signed firmware delivery; rollback protection.
- **Sensor/actuator I/O boundary** — physical-world inputs (sensors, GPIO, ADC) and outputs (motors, relays). Attackers in physical proximity can spoof or tamper.
- **Removable media / USB boundary** — SD card, USB host/device modes.
- **Wireless boundary** — BLE, Wi-Fi, Zigbee, LoRa, Z-Wave, NFC, sub-GHz proprietary; each wireless interface is a boundary crossing reachable without physical access.
- **Cellular / baseband boundary** — separate processor with its own firmware; treat as distinct trust zone.
- **Device ↔ companion app boundary** (mobile or desktop pairing app).
- **Device ↔ network gateway / hub boundary** (home hub, industrial gateway).
- **Physical-tamper boundary** — case open, tamper switch, anti-tamper mesh.
- **Side-channel boundary** — power, EM, timing. Usually only matters for crypto operations and high-value targets; document if in scope.

**Ownership default:** vendor owns firmware, signing keys, OTA infrastructure; end-user (or hospital / plant) owns the physical device; carrier may own the SIM and baseband.

**Subgraph examples:**

```
subgraph DeviceApp["Vendor | embedded (application core) | medium trust"]
subgraph DeviceSE["Vendor | embedded (secure element) | very high trust"]
subgraph Modem["Carrier | embedded (cellular baseband) | unknown trust"]
subgraph Physical["End-user / Hospital | physical device custody | low trust"]
```

**Common misses:** the secure-element boundary (people draw the device as one box); JTAG/UART as a boundary; the baseband as a separate trust zone; bootloader-mode as an alternative path; wireless interfaces drawn as flows but not as boundary crossings.

**Multi-device shared-space note.** When several embedded devices coexist in one physical space (smart building, IoMT patient room, vehicle cabin, robotics cell), the most consequential boundary often isn't around any single device — it's the shared physical medium *between* them: shared 2.4 GHz ISM band (Wi-Fi / BLE / Zigbee co-channel interference), shared power rail, shared HVAC duct for acoustic side channels. Per-device boundary modeling will miss the cross-device hop. Classify each between-device surface as P1 (direct contact), P2 (proximity), or P3 (shared medium without proximity), and use the **risk-prioritized cyber-physical attack-path** construction in `methodologies.md` § "Risk-prioritized cyber-physical attack paths (Stellios et al., 2021)" rather than independent per-device passes.

### OT / ICS (PLC / RTU / HMI / historian / safety system)

Use the **Purdue Enterprise Reference Architecture** as the boundary skeleton — it's the lingua franca for ICS network segmentation and what auditors expect to see.

**Boundaries to expect:**

- **Level 0 / Level 1 boundary** — physical process (sensors, actuators) vs. basic control (PLCs, RTUs). Field-bus protocols (4-20 mA, fieldbus, HART).
- **Level 1 / Level 2 boundary** — basic control vs. supervisory control (HMIs, SCADA servers, engineering workstations).
- **Level 2 / Level 3 boundary** — supervisory control vs. site operations / manufacturing execution (historians, MES, batch servers).
- **Level 3.5 — IT/OT DMZ** — the most-modeled and most-attacked boundary in ICS. Jump hosts, historians replicating to enterprise, vendor remote access. Always draw this explicitly.
- **Level 4 / Level 5 boundary** — site IT / corporate IT. Cross here for business reporting and remote vendor access.
- **Safety Instrumented System (SIS) boundary** — the SIS is a separate, lockstep-isolated system that initiates safe-state actions. Compromise here has direct safety implications. Always a high-trust, low-attack-surface zone — and any path *into* it is a finding.
- **Engineering workstation boundary** — the workstation that programs PLCs is the highest-leverage host on the OT network. Treat as Tier 0 equivalent.
- **Vendor remote-access boundary** — the most common ICS attack vector. Often a vendor jump host on the IT/OT DMZ.
- **Removable-media boundary** — USB drives are a primary malware ingress for air-gapped or semi-air-gapped systems; draw as a flow across the IT/OT boundary.
- **Protocol-translation gateway boundary** — Modbus → MQTT, OPC-UA bridge, etc. The gateway is its own trust zone.
- **Wireless field-network boundary** — WirelessHART, ISA100, proprietary 900 MHz/2.4 GHz; treat each as its own boundary.

**Ownership default:** plant operator / utility owns levels 0–3 and the IT/OT DMZ (3.5); enterprise IT owns levels 4–5; vendors/integrators own engineering workstations and remote-access tooling. Physical custody and safety accountability sit with the plant operator.

**Subgraph examples:**

```
subgraph L1["Plant Operator | OT/ICS (Purdue L1 — PLCs/RTUs) | very high trust"]
subgraph L2["Plant Operator | OT/ICS (Purdue L2 — HMIs/SCADA) | high trust"]
subgraph L35["Plant Operator | OT/ICS (Purdue L3.5 — IT/OT DMZ) | medium trust"]
subgraph SIS["Plant Operator | OT/ICS (Safety Instrumented System) | safety-critical, isolated"]
subgraph VendorRA["Integrator | OT/ICS (vendor remote access) | low trust"]
```

**Common misses:** the SIS as a separate zone; the engineering workstation as Tier 0–equivalent; vendor remote access; removable-media flows across air gaps; protocol-translation gateways collapsed into the destination.

**Multi-device shared-space note.** Plant floors with several PLCs / IIoT gateways / sensors sharing wireless field networks (WirelessHART, ISA100, proprietary 2.4 GHz / 900 MHz) or co-located in one cabinet have the same cross-device composition risk as IoMT rooms. When per-Purdue-level analysis doesn't surface the path from a low-criticality field device to a safety-critical actuator, reach for the **risk-prioritized cyber-physical attack-path** construction in `methodologies.md` § "Risk-prioritized cyber-physical attack paths (Stellios et al., 2021)".

### Mobile (iOS / Android, app + OS + carrier)

**Boundaries to expect:**

- **App sandbox boundary** — the OS-enforced boundary around your app's process and storage. Other apps cannot read inside it (modulo bugs).
- **OS keychain / keystore boundary** — Keychain (iOS), Keystore (Android), often hardware-backed (Secure Enclave on iOS, StrongBox / TEE on Android). Only handles cross out, key material does not.
- **Hardware-backed enclave boundary** — Secure Enclave / StrongBox / Titan M. Even the OS kernel cannot extract keys.
- **Inter-app IPC boundary** — Intents (Android), URL schemes / Universal Links (iOS), App Groups, content providers, exported services. Each is a flow across an app boundary.
- **OS vendor ↔ app boundary** — the OS kernel, sandbox enforcement, and update mechanism are owned by the OS vendor. You depend on their security model.
- **Carrier ↔ device boundary** — baseband, SIM, eSIM, carrier SMS path (used for 2FA — a known boundary worth modeling).
- **MDM / managed-profile boundary** — MDM-enrolled vs. personal profile; work profile (Android), Managed App Configuration (iOS).
- **Jailbreak / root / debug boundary** — a jailbroken/rooted device collapses many of the above boundaries; jailbreak detection is itself a boundary signal, not a security control.
- **App-to-backend boundary** — the network call to your servers; certificate pinning, mTLS.
- **Sensor / camera / location boundary** — permission-gated access; user owns the consent decision.
- **App store gating boundary** — code-signing, store review, update distribution. Owned by OS vendor.

**Ownership default:** OS vendor owns kernel/sandbox/keystore/store; carrier owns baseband/SIM; end-user owns physical device, passcode, app installs; you (the app developer) own only what's inside your sandbox and your backend. MDM shifts some end-user ownership to enterprise IT.

**Subgraph examples:**

```
subgraph App["Vendor | mobile (app sandbox) | medium trust"]
subgraph Keystore["OS Vendor (Apple/Google) | mobile (hardware-backed keystore) | very high trust"]
subgraph OSKernel["OS Vendor (Apple/Google) | mobile (OS kernel) | high trust, owner-controlled"]
subgraph Device["End-user | mobile (device hardware) | low trust"]
```

**Common misses:** the keystore as a separate zone (people draw it inside the app); inter-app IPC channels; the carrier baseband; MDM profile vs. personal profile; the user as an explicit owner whose passcode strength gates everything.

### Hybrid / cross-environment

Most real systems span multiple environment types: a cloud backend + mobile app + embedded device, or a cloud SaaS + on-prem connector + AD-integrated SSO. When this happens:

- Pick the *dominant* environment for the system-type matrix in `methodologies.md` (the one where most threats live), and add the others as additional zones.
- Draw each environment's characteristic boundaries within its zone — don't flatten "the cloud" into one box if the cloud side has its own meaningful internal boundaries.
- The boundaries *between* environments (mobile app ↔ cloud backend, embedded device ↔ vendor cloud, on-prem connector ↔ SaaS) are usually the highest-value boundaries in the model. Label flows across them with the actual protocol and authentication ("mTLS over HTTPS", "device-cert OAuth2 client credentials", "WebSocket with bearer token after device-attestation").
- Ownership often *changes* across the boundary (vendor-owned cloud ↔ customer-owned device, customer-owned backend ↔ user-owned phone). Make these transitions explicit in the trust-boundary prose section.

## Multi-device shared-space modeling

When several devices share one physical space (smart building zone, plant-floor cabinet, vehicle cabin, robotics cell, clinical room), one device's RF / acoustic / power surface can reach another. Element-by-element STRIDE often misses multi-hop cyber-physical attack paths where each individual device looks fine but their composition over inter-device hops reaches a high-value target.

Classify the **between-device** surface (Stellios et al. 2021):

- **P1 — direct contact.** USB, JTAG/UART/SWD, attached cable, attached storage, hardwired serial console.
- **P2 — proximity.** BLE/NFC reach, line-of-sight IR, audible-range acoustic, near-field EM, Wi-Fi-Direct / Wi-Fi associated network, Zigbee mesh.
- **P3 — shared physical medium without proximity.** The 2.4 GHz ISM band shared across Wi-Fi / BLE / Zigbee in a building, where a rogue BLE peripheral can DoS a Wi-Fi-attached device without ever associating with it; shared power lines; shared HVAC for acoustic; shared EM environment.

For systems where this composition is plausible, run a target-rooted walk from each safety-critical or high-value asset outward through P1/P2/P3 hops — see `methodologies.md` § "Risk-prioritized cyber-physical attack paths" for the construction. Score paths with the CVV scoring (Stellios) so the top paths get mitigations rather than every theoretically-possible path.

Domains where this is regularly worth running: medical (IoMT — clinical rooms with infusion pumps + monitors + Wi-Fi APs + nurse-call peripherals + clinician phones; OR/cath-lab; NICU bays); industrial (plant-floor cabinets with shared backplane; robotics cells with shared safety controllers); automotive (vehicle cabin with infotainment + telematics + body-control modules); smart buildings (HVAC + access-control + lighting on shared OT segments).

## Domain notes

The skill's defaults are domain-neutral. These subsections collect domain-specific defaults, regulators, and gotchas that don't fit into the per-environment boundary catalog above. Load the relevant subsection when the system fits the domain. Medical-device specifics (PACS / DICOM / IoMT / FDA / IEC 81001-5-1 / HL7 / patient-as-asset / clinical workflow misuse) live in `medical.md` rather than here.

### Embedded / IoT — domain notes

- **Physical attack surface belongs in the DFD.** Debug ports (JTAG/UART/SWD), USB, removable media, exposed serial consoles, programming headers, test pads — each is a trust boundary or an opening into one. Tamper protections (secure boot chain, secure element / TEE / TrustZone, anti-glitching, encrypted firmware-at-rest) are mitigations against specific named threats; if the device claims them, name the specific protections in the DFD prose and threat rows.
- **The trust boundary between device and network is often the most consequential.** A device that "trusts the LAN it's on" is treating LAN compromise as out-of-scope; that assumption needs to be explicit and falsifiable.
- **Side channels** (timing, power, EM, acoustic) are legitimate threats for devices handling cryptographic secrets without an HSM-class component. CAPEC-189 (Black Box Reverse Engineering) and CAPEC-188 (Reverse Engineering) are the operational-stratum mappings.
- **Firmware attack surface** — code-signing chain, OTA-update server, update manifest format, anti-rollback. Each is a control whose absence promotes Tampering threats (CAPEC-441, CWE-494).
- **Supply chain** is a real and rarely-fixable surface — provenance (SBOM, signed releases, reproducible builds), manufacturing-stage compromise (CAPEC-438), distribution-stage tampering (CAPEC-439). When a supply-chain finding has no in-product mitigation, name it as Accept with explicit residual-risk language.
- **Multi-device shared space** — see § "Multi-device shared-space modeling" above. Routine in IoMT, smart buildings, vehicle cabins, robotics cells, plant cabinets.
- **Safety-critical control loops** — for control-loop devices (automotive ECUs, robotics, drones, industrial actuators), add STPA-SafeSec — swap or supplement, depending on whether ISO 26262 / IEC 61508 require the joint artifact. See `stpa-safesec.md` § "Two modes".

### Cloud-native — domain notes

- **Shared-responsibility model.** The cloud provider owns the hypervisor, control plane, managed-service internals, and data-center physical security. You own everything inside your account — IAM, network, host configuration, workload code, data, secrets. Make this split explicit in §1's ownership table; threats below the provider's line are out of scope but should be named, not silently dropped.
- **IAM is usually under-modeled.** "Authentication is via Cognito" doesn't enumerate the IAM threats — privilege escalation through misconfigured roles, cross-account confused-deputy attacks, instance-profile credential theft, OIDC trust-policy misconfigurations. Walk IAM separately as an asset-centric pass for any non-trivial cloud system.
- **Managed services** are usually under-modeled. The default assumption is "the managed service is secure"; what's missing is the user's *configuration* of that service (S3 bucket policy, KMS key policy, RDS parameter groups, EventBridge rules). Treat each managed-service interaction as a flow whose boundary is the configuration the user controls.
- **Account / VPC / subnet nesting.** Cloud trust boundaries nest three deep at minimum (account → VPC → subnet); often more (organization → account → VPC → subnet → security group → workload). Use nested subgraphs in the DFD.
- **Strategic stratum**: SOC 2 / ISO 27001 framing is conventional; CIS Benchmarks for the specific cloud(s) supply concrete strategic-stratum references; sector regulator on top if the workload is regulated.

### AI / ML — domain notes

- **STRIDE still works**, but extend with model-specific threats: prompt injection (direct and indirect), training-data poisoning, model extraction / stealing, membership inference, model inversion, adversarial examples, supply chain on pre-trained weights, jailbreak / system-prompt exfiltration, tool-use abuse for agentic systems, hallucination-driven downstream failure.
- **Data-centric supplement is usually warranted** — training data, validation/test data, model weights, fine-tuning data, RAG corpus, embedding stores, system prompts, and user-data-in-prompts each have distinct security objectives and authorized locations.
- **The model itself is an asset.** Model weights are exfiltratable via the API surface (model extraction); training data is reconstructable via membership inference. Both belong in §1's asset table.
- **Operational stratum**: ATT&CK has limited AI/ML coverage; supplement with **OWASP LLM Top 10** and **OWASP ML Security Top 10** as the primary technique catalogs.
- **Strategic stratum**: EU AI Act, NIST AI RMF, sector-specific AI regulators where they exist (e.g. healthcare AI). The risk classification under the AI Act drives the regulatory framing.
- **Agentic systems** add tool-use as an attack surface: any tool a model can call is a control-flow path the prompt can hijack. Treat each tool integration as a trust boundary.

#### DFD pattern for an LLM application

LLM apps look unfamiliar in a classic DFD because untrusted *content* mixes with trusted *instructions* in the prompt, and the model can issue control-plane actions (tool calls) from data it just read. The canonical mapping:

| LLM-app concept | DFD element | Why |
|---|---|---|
| End-user prompt | Data flow (untrusted input → app process) | Same as any user input — but never separated from system instructions on the model side |
| System prompt / developer prompt | Data store (read by app process at request time) — usually a config asset (`AS#`) | Treat as a high-value secret-adjacent asset; exfiltration is a known threat (prompt-leak / system-prompt extraction) |
| RAG / retrieved documents | Data flow + data store (the index / vector DB) | Each retrieved chunk is a data flow whose origin trust must be tracked. Indirect prompt injection rides this flow |
| Long-term memory / chat history | Data store | Same store treatment as any user-persistent data — includes the persistence trust boundary an attacker can poison across sessions |
| The LLM itself (provider API or self-hosted) | Process | Not a data store; it transforms input to output. If self-hosted, the weights are an `AS#` |
| Model weights (self-hosted) | Data store + asset (`AS#`) | Exfiltratable via model extraction; supply-chain compromise of pre-trained weights is a tampering threat |
| Tools the model can call (HTTP, DB, shell, code interpreter, MCP server) | One process per tool, each a *separate trust zone* | Prompt-driven tool invocation is a control-flow hijack surface — each tool is a boundary the prompt can cross |
| Context window | Implicit data store with a write-only-during-request lifecycle | Lifecycle-only location; data-centric pass picks this up where flow-centric won't |
| Output passed to a downstream renderer (browser, terminal, IDE, another agent) | Data flow → process | Hallucinated or attacker-controlled output becomes the next stage's input — XSS-style sinks apply |

**Trust crossings to draw explicitly** (these are where the threats live; mark each with the canonical dashed-border `classDef tb` pattern):

- **End-user → app**: classic untrusted-input boundary. Direct prompt injection.
- **App → LLM provider**: trust crosses if the provider is third-party. Confidentiality of system prompt and customer data; vendor lock-in for model behavior.
- **Retrieved document → app → LLM**: this is the **indirect prompt injection** path. The retrieved-document store sits in a trust zone that may include attacker-controlled content (web crawler, multi-tenant uploads, public-internet sources). Walk it as a separate subgraph.
- **LLM → tool**: the model's output becomes the tool's input. **Each tool is its own trust zone**, and the boundary between the LLM process and each tool is where authorization checks must enforce that the prompt didn't smuggle a privileged action through. For agentic systems, this is the highest-value boundary in the diagram.
- **Tool output → LLM → user**: completes the loop. Tool output flows back into the model's context window and out to the user; if the tool reads attacker-controlled data, that data is now reaching the user via a trusted-looking channel.

**Worked sketch — minimal LLM chatbot with tools**:

```mermaid
flowchart LR
    subgraph User["End-user | public internet | untrusted"]
        U[User]
    end

    subgraph App["Vendor | cloud (app account) | medium trust"]
        Web(Web App / API)
        SysPrompt[(System Prompt + tool descriptions (AS1))]
        ChatStore[(Chat History (AS2))]
        VectorDB[(RAG vector store (AS3))]
    end

    subgraph LLM["LLM Provider | cloud (provider account) | out-of-scope owner"]
        Model("Frontier LLM API")
    end

    subgraph Tools["Vendor | cloud (tool sandboxes) | low trust"]
        ToolHTTP("HTTP fetch tool")
        ToolDB("Internal DB tool")
    end

    U -- "user prompt (HTTPS)" --> Web
    Web -- "read system prompt" --> SysPrompt
    Web -- "read recent turns" --> ChatStore
    Web -- "RAG retrieval (top-k)" --> VectorDB
    Web -- "prompt = sys + history + retrieved + user (HTTPS)" --> Model
    Model -- "tool call (function calling)" --> Web
    Web -- "dispatch (signed)" --> ToolHTTP
    Web -- "dispatch (parameterized query)" --> ToolDB
    ToolHTTP -. "untrusted web fetch" .-> Web
    ToolDB -- "row results" --> Web
    Web -- "tool result back as model input (HTTPS)" --> Model
    Model -- "assistant message" --> Web
    Web -- "render (HTTPS)" --> U

    classDef tb fill:none,stroke:#888,stroke-dasharray: 5 5
    class User,App,LLM,Tools tb
```

Threats this diagram makes legible (not exhaustive):

- **Direct prompt injection** at the `U → Web → Model` flow (Spoofing of intent, Elevation via tool call). CWE-1427 maps here.
- **Indirect prompt injection** at the `VectorDB → Web → Model` flow when retrieved chunks come from attacker-influenced sources.
- **Tool-call hijack** at the `Model → Web → ToolHTTP / ToolDB` boundary — the highest-leverage authorization check in the system.
- **System-prompt exfiltration** via `Model → Web → U` (Information Disclosure on `AS1`).
- **Model extraction / membership inference** at the `User ↔ Model` API surface if the user is allowed many requests.
- **Tool-result smuggling** at the `ToolHTTP → Web → Model` flow (note the dashed arrow) when tool output is attacker-controlled and is fed back into the model's context window.
- **Cross-session memory poisoning** at the `Web → ChatStore` writes if the system reuses memory across sessions or users.

For agentic / multi-tool systems, replicate the `Tools` subgraph with one zone per tool when the tools have meaningfully different trust levels (a code interpreter, a billing API, and a public web fetcher each warrant their own zone). Don't collapse them into one box — that hides exactly the boundaries the prompt is trying to cross.

### Third-party / acquired code — domain notes

For components the team didn't write (acquired products, OEM firmware, third-party SaaS embedded as a dependency, open-source libraries with significant attack surface), internal documentation, architecture diagrams, and source may all be missing. The inside-out workflow in `validation.md` § "Validating threats addressed in third-party / acquired code" is the entry point: enumerate accounts and processes the component runs as, listening ports and IPC endpoints, administrative interfaces (including documented backdoors and account-recovery paths), and platform changes the component makes (OS changes, firewall rules, permissions, auto-updater behavior). The result is enough to construct a software model and run threat enumeration against it.

Components that ship with their own threat model and operational security guide are worth significantly more than those that don't — flag this in any buy/build decision.

## How to use this file

After Round 1.5 you know each environment type and owner. Pick the matching section(s), walk the boundary list as a checklist (in DFD, should be, or explicitly out — "ASM4: no MDM in scope, personal-device deployment only"), label subgraphs with the convention, and fill in the trust-boundary table (`dfd-mermaid.md` § "Trust boundary prose template") with owners on each side. Cross-check the system-type matrix in `methodologies.md` for matching contextual supplements (data-centric / asset-centric / etc.).

If the system is medical / PACS / DICOM / IoMT, also load `medical.md` — domain-specific defaults (patient-as-asset, clinical workflow misuse, DICOM/HL7 STRIDE specifics, FDA / IEC 81001-5-1 framing) live there rather than in this file.

This is a checklist, not a straitjacket. Drop boundaries that genuinely don't apply, but say so explicitly — silent omission is the anti-pattern that this file is here to prevent.
