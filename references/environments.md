# Per-environment trust-boundary patterns

The generic boundary list in `dfd.md` is the floor — UID, network segment, container, org, tenant, privilege. This file is the ceiling: the boundaries characteristic of a *specific* environment type that the generic list tends to miss. A DFD without environmental context usually draws one or two boundaries ("internet vs. internal") and overlooks the ones that actually carry threats.

Once you know the environment type of each zone, walk the matching section below as a checklist. Boundaries that genuinely don't apply can be dropped — but say so, don't omit silently.

## Ownership

For every zone, name the **owner** — who can change its configuration, run its patch cycle, and control its access. Ownership is what makes Q3 mitigations assignable: "implement mTLS" needs an addressee.

| Owner | Controls they can change |
|---|---|
| **Vendor / maker** (you, if building the product) | Source code, signing keys, default config, update server |
| **Customer organization** (deployer) | Network segmentation, firewall rules, IAM, host hardening, patch cadence |
| **End-user** | Device passcode, app permissions, OS updates, physical custody |
| **Cloud provider** | Hypervisor, control plane, managed-service internals, datacenter physical security |
| **Mobile-OS vendor** | Kernel, sandbox enforcement, keystore, app-store gating, OS patches |
| **Carrier** | Baseband firmware, SIM, mobile core network |
| **Third-party SaaS** | Everything inside the SaaS boundary — you get configuration knobs and audit logs only |
| **Integrator / VAR** | Engineering workstation, deployment scripts, integration code |
| **No single owner** | Public-internet transit, public physical space — no controls available; treat as untrusted |

Ownership is the generalization of cloud's shared-responsibility model to every environment. It is *not* the same as data custody or legal accountability — call out the divergence where it matters (a cloud provider owns the host; the customer holds the data; regulatory accountability sits with the customer). If an owner can't be named, record an assumption and proceed.

## Cloud (AWS / Azure / GCP / …)

Boundaries to expect:

- **Account / subscription / project boundary** — the outermost cloud-side boundary; every cross-account access is a crossing.
- **Region / availability-zone boundaries** — especially where data crosses a regulatory boundary.
- **VPC / VNet / subnet boundaries** — public vs. private subnets, ingress/egress controls.
- **IAM principal boundaries** — every role, instance profile, service principal; cross-account assume-role is a crossing.
- **Managed-service control plane vs. data plane** — many cloud privilege-escalation findings cross this line.
- **Tenant boundary** in multi-tenant SaaS — row/object-level isolation, key-per-tenant.
- **KMS / HSM boundary** — between code holding plaintext and code holding only handles.
- **CI/CD ↔ runtime boundary** — build-time IAM vs. runtime IAM; often under-modeled.

Common misses: the build/CI account as a separate zone; the KMS key boundary; the control-plane/data-plane IAM split.

## On-prem enterprise (Windows AD / Linux / mixed)

Boundaries to expect:

- **AD forest / domain boundaries** — the forest is the AD security boundary; inter-domain trust is a crossing.
- **Identity-provider / SSO federation boundaries** — SAML/OIDC trust to relying parties.
- **Tier 0 / 1 / 2 boundary** — domain controllers and identity (Tier 0), servers (Tier 1), workstations (Tier 2). Cross-tier authentication is the most consequential boundary in AD environments.
- **Jump host / bastion boundary** — the only path into a privileged zone.
- **Service account vs. interactive user** — service accounts often have broad privilege and weak rotation; treat as a separate trust level.
- **DMZ ↔ internal boundary**; **VLAN / segment boundaries** (corp, server, management, guest).
- **Privilege-escalation boundary within a host** — normal shell vs. root/SYSTEM.
- **VPN concentrator boundary** — remote workers and vendor support cross here.

Common misses: the AD tier model; service-account privilege levels; the VPN/jump-host as a distinct zone; a vendor service account holding Domain Admin (common, and a serious finding).

## Embedded / IoT

Boundaries to expect:

- **Secure-boot chain boundaries** — bootloader → kernel → application; each handoff is a trust transition.
- **Secure element / TEE / TPM boundary** — between code holding key material and code that only requests signing.
- **Hardware debug interface boundary** — JTAG, SWD, UART console. Often the most consequential physical-attack boundary; if debug ports ship unfused, draw them as a crossing into privileged state.
- **Bootloader / recovery / DFU boundary** — alternative boot paths that bypass normal controls.
- **Update server ↔ device boundary** — signed firmware delivery, rollback protection.
- **Sensor / actuator I/O boundary** — physical-world inputs and outputs; an attacker in proximity can spoof or tamper.
- **Wireless boundaries** — BLE, Wi-Fi, Zigbee, LoRa, NFC, sub-GHz; each interface is a crossing reachable without physical access.
- **Cellular / baseband boundary** — a separate processor with its own firmware; a distinct trust zone.
- **Physical-tamper boundary** — case open, tamper switch, anti-tamper mesh.

Common misses: the secure element (people draw the device as one box); JTAG/UART as a boundary; the baseband as a separate zone; wireless interfaces drawn as flows but not as boundary crossings.

## OT / ICS

Use the **Purdue model** as the boundary skeleton — it's what ICS auditors expect.

- **Level 0/1** — physical process (sensors, actuators) vs. basic control (PLCs, RTUs).
- **Level 1/2** — basic control vs. supervisory (HMIs, SCADA, engineering workstations).
- **Level 2/3** — supervisory vs. site operations (historians, MES).
- **Level 3.5 — the IT/OT DMZ** — the most-attacked boundary in ICS; jump hosts, historian replication, vendor remote access. Always draw it.
- **Level 4/5** — site IT / corporate IT.
- **Safety Instrumented System (SIS) boundary** — a separate, isolated system that triggers safe states; any path *into* it is a finding.
- **Engineering workstation** — the host that programs PLCs; the highest-leverage host on the OT network, Tier 0–equivalent.
- **Vendor remote-access boundary** — the most common ICS attack vector.
- **Removable-media boundary** — USB drives are a primary ingress for air-gapped systems.

Common misses: the SIS as a separate zone; the engineering workstation's leverage; vendor remote access; removable-media flows across air gaps.

## Mobile (iOS / Android)

Boundaries to expect:

- **App sandbox boundary** — the OS-enforced boundary around your app's process and storage.
- **OS keychain / keystore boundary** — often hardware-backed (Secure Enclave, StrongBox); handles cross out, key material doesn't.
- **Inter-app IPC boundary** — Intents, URL schemes, content providers, exported services.
- **OS vendor ↔ app boundary** — kernel, sandbox enforcement, update mechanism, all owned by the OS vendor.
- **Carrier ↔ device boundary** — baseband, SIM, the SMS path (still used for 2FA).
- **MDM / managed-profile boundary** — managed vs. personal profile.
- **Jailbreak / root boundary** — a rooted device collapses many of the above; jailbreak detection is a signal, not a control.
- **App ↔ backend boundary** — the network call to your servers; certificate pinning, mTLS.

Common misses: the keystore as a separate zone; inter-app IPC; the carrier baseband; MDM vs. personal profile; the user as an explicit owner whose passcode gates everything.

## Hybrid / cross-environment

Most real systems span environments — a cloud backend plus a mobile app plus an embedded device. When they do:

- Pick the *dominant* environment for choosing supplements, and add the others as additional zones.
- Draw each environment's characteristic boundaries within its own zone — don't flatten "the cloud" into one box.
- The boundaries *between* environments (mobile app ↔ backend, device ↔ vendor cloud, on-prem connector ↔ SaaS) are usually the highest-value boundaries in the model. Label those flows with the real protocol and authentication.
- Ownership often changes across the boundary — make those transitions explicit in the trust-boundary table.
