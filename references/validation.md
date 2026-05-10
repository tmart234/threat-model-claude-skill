# Validating the threat model (Q4)

> **Related**: ← `SKILL.md` (entry point) • `methodologies.md` (hybrid framing) • `dfd-mermaid.md` (diagramming → checklist below) • `stride-prompts.md` (threats checklist below) • `manifesto.md` ("Admiration for the problem" anti-pattern)

The fourth question — "Did we do a good enough job?" — is the most-skipped step. The Manifesto's "Admiration for the problem" anti-pattern lives here: lots of teams enumerate threats and never close the loop on whether the model is right, whether the responses landed, or whether the implementation actually matches the model. This file is dedicated to making Q4 a real activity, not a closing pleasantry.

This file is the **canonical home** of the three closing checklists this skill emits — Diagramming, Threats (hybrid: contextual / operational / strategic / cross-stratum), Validating-threats. SKILL.md and `dfd-mermaid.md` reference them; don't duplicate.

The frame and most of the practical wisdom here is paraphrased from Adam Shostack's *Threat Modeling: Designing for Security* (Wiley, 2014), Chapter 1 ("Checking Your Work") and Chapter 10 ("Validating That Threats Are Addressed"), with attribution.

## Three things to check

Shostack frames Q4 as three checks. The skill output should explicitly address each:

1. **Model/reality conformance** — Does the diagram match what was actually built?
2. **Task and process completion** — Has every threat been resolved (mitigated, eliminated, transferred, or explicitly accepted), and have those resolutions become tracked work items?
3. **Bug checking** — As the work proceeds, are the bugs/issues for each threat actually being closed, with mitigations tested?

## Model/reality conformance

The model goes stale fast. If the team did three architecture revamps after the diagram was last drawn, the threats found against the old diagram aren't necessarily relevant to what was built. Worse, threats against the *current* system that didn't exist in the old diagram have never been considered.

Concrete checks:

- Walk the diagram with someone who built the system and ask: *is this complete? is it accurate? does it cover all the security decisions made? could we start the next version with this diagram unchanged?* If everyone says yes, the model is good. If not, update it.
- Anytime threats led to substantial redesign or architectural change, re-check the diagram against the code at check-in time, or against the deployed system at deployment time. Do not assume the planned mitigation was implemented as drawn.
- For deployed/operational systems, "complexities of deployment often lead to on-the-fly changes" (firewall rules turned off in a pinch, IAM roles widened during incident response). If those happened, the threat model probably needs a revamp.

A useful framing for the conformance check: bring the people who made architecture or code changes into a room and have *them* explain what changed and which security features apply. (Heads up: this can devolve into security finding new threats against the new system, which can feel like moving the goalposts. Set expectations upfront.)

## Task and process completion

Every threat in the enumeration must have a recorded response. Skipping this is the signature of "admiration for the problem." For each threat:

- Was a response chosen? (Mitigate / Eliminate / Transfer / Accept — pick one.)
- For "Mitigate" — is there a concrete control, and is that control assigned an owner?
- For "Accept" — is the rationale recorded and the acceptance attributed to a named decision-maker?
- For "Transfer" — does the receiving party know they're now on the hook?
- Is each of these tracked in the team's normal bug/issue tracker?

The bug tracker integration matters. Threats that live only in the threat model document tend to get lost; threats filed as bugs are subject to the team's normal triage, prioritization, and review. **Bugs act as an exit point from threat modeling — they allow the normal machinery to take over and let you say "we're done threat modeling that."**

## Bug checking

As the team gets close to shipping or deploying, run a query for every threat-derived bug and check it. For each:

- Is it closed?
- If closed: is there a test that demonstrates the mitigation works?
- If open: has it been triaged with attention to the gravity of the bug, and either fixed or explicitly moved to the next revision?

A `tmtest` or `threat-model` tag/label on each bug makes this query trivial.

## The closing checklists

Drop these into the threat-model document itself with check-marks. The Diagramming and Validating-threats checklists are adapted from Shostack Chapter 1's end-of-chapter checklists; the Threats checklist is extended to cover this skill's hybrid output (three strata + cross-stratum).

### Diagramming checklist

- [ ] We can tell a story about how the system works without changing the diagram.
- [ ] We can tell that story without using the words "sometimes" or "also" — anywhere those came up, we broke the case into separate flows or added detail.
- [ ] We can look at the diagram and see exactly where the software will make a security decision.
- [ ] Every element is inside exactly one zone subgraph — no floating elements (SKILL.md DFD rule #1).
- [ ] Every trust boundary is rendered as a dashed-border subgraph (`dfd-mermaid.md` § "Rendering trust boundaries as dashed subgraphs").
- [ ] Sub-trust-boundaries are nested subgraphs where they exist — controller dual-OS, account/VPC/subnet, host/VM/container, device/secure-element (`dfd-mermaid.md` § "Nested subgraphs for sub-trust boundaries").
- [ ] Every `AS#` from §1 appears in the DFD as a tagged element label, and every §1 trust boundary is drawn (SKILL.md DFD rule #3).
- [ ] Methodology framing (Purdue, AD Tier, cloud account/VPC, mobile sandbox) is named in §1 prose, not repeated on every subgraph label.
- [ ] The diagram shows all trust boundaries: every UID/account boundary, every application role, every network interface, every place different principals interact.
- [ ] Every subgraph is labeled `<owner> | <env-type> | <trust>` per `dfd-mermaid.md` § "Subgraph labeling convention" — env-type without redundant prefixes (`Purdue L2`, not `OT/ICS (Purdue L2)`); compression-rule key applied where owner/env-type repeats.
- [ ] The per-environment trust-boundary patterns from `references/environments.md` have been checked for every environment type in scope (cloud / on-prem enterprise / embedded / OT/ICS / mobile) — boundaries that apply but aren't drawn are explicitly justified.
- [ ] Every data flow is labeled with the actual protocol and authentication (e.g. "DICOM C-STORE over TCP/11112", "HTTPS + mTLS", "Modbus/TCP", "BLE GATT") — generic labels like "data" mean threats will be missed.
- [ ] The diagram reflects the current or planned reality — not an aspirational version.
- [ ] We can see where all the data goes and who uses it (no data sinks — every written piece of data has a reader).
- [ ] We see the processes that move data from one data store to another (data can't move itself).

### Threats checklists (hybrid)

**Contextual stratum (§2.1)**
- [ ] Have we looked for each STRIDE category, at each element, *and* at each data flow specifically? (Flows get overlooked — belt-and-suspenders.)
- [ ] Has at least one supplementary entry-point pass been run (data-centric / asset-centric / user-needs-centric / process-centric — driven by the system-type matrix in `methodologies.md`)?
- [ ] LINDDUN pass run if PII/PHI is in scope?
- [ ] AI/ML-specific threats considered if ML components are present?

**Operational stratum (§2.2)**
- [ ] At least the top 3 threats carry the full **STRIDE → CAPEC → CWE → mitigation** chain (CAPEC pattern with explicit abstraction level, the CWE(s) it exploits, and the mitigation class)?
- [ ] At least the top 3 threats mapped to ATT&CK technique IDs?
- [ ] Where no Detailed CAPEC pattern exists for a domain-specific protocol (DICOM, HL7, ICS), the closest Standard or Meta pattern is cited *and the row says so explicitly* (per `capec.md` § "Honest about CAPEC coverage")?
- [ ] Kill-chain / CVSS references added where they aid handoff to SOC / IR / engineering?

**Strategic stratum (§2.3)**
- [ ] Sector ISAC / threat-intel context noted (or explicitly marked "not applicable: <reason>")?
- [ ] Regulatory framing captured (FDA / IEC / HIPAA / GDPR / PCI / EU AI Act — whichever apply)?
- [ ] Named-adversary context included if the sector has one?

**Cross-stratum**
- [ ] Threats are *cross-referenced* across strata, not duplicated?
- [ ] All threats share one ID space and one risk-rating scale, so the §3 prioritized list is single-sorted?

### Validating-threats checklist

- [ ] We have written down or filed a bug for each threat.
- [ ] There is a proposed/planned/implemented way to address each threat.
- [ ] We have a test case per threat (or per mitigation).
- [ ] The software has passed the test.

## Penetration testing — complement, not substitute

Pen testing is sometimes pitched as the validation activity for security. It isn't, by itself, the validation activity for *threat modeling*.

**You can't pen-test your way to a secure design.** Pen testing tells you what's broken now; threat modeling tells you what should and shouldn't exist. The two complement each other: pen testers can use the threat model as a roadmap, and pen test findings can drive threat-model updates.

Two flavors of pen testing matter for Q4:

- **Black-box pen testing** — testers are given only the running software. They have to do reconnaissance to figure out the attack surface. Expensive and slow, and largely tests assumptions about how hard it is to discover the system.
- **Glass-box pen testing** — testers are given the threat model, the design, and the code. They can target the threats the model identified and report back on which ones are actually exploitable in the real implementation. Much higher value per dollar.

If the team is using pen testers as part of validation, give them the threat model. Then, **be explicit about scope**. If the test report comes back with "found nothing but XSS and SQL injection" and the team is unhappy because they wanted architecture-level findings, the failure was scope alignment, not testing.

## Validating threats addressed in third-party / acquired code

Most modern systems include code the team didn't write. Threat-modeling acquired code is harder because internal documentation, architecture diagrams, and source may be missing. Shostack's Chapter 10 sketches an inside-out approach. For each acquired component:

- Enumerate **accounts and processes** the component creates or runs as.
- Enumerate **listening ports** and inter-process communication endpoints.
- Enumerate **administrative interfaces** — including documented ones, account-recovery paths, and service personnel accounts (backdoors for "if you forget your password" are unfortunately common).
- Enumerate **technical dependencies and platform changes** — OS changes (for appliances/VMs), firewall rule changes, permission changes, auto-updater behavior, unpatched vulnerabilities at install time.

Tools that help on Linux: `ps`, `netstat`, `find -newer`. On Windows: Sysinternals' Autoruns walks the many ways to start a program at boot. The result is enough information to construct a software model and run threat enumeration against it. Components that come with their own threat model and operational security guide are worth significantly more than those that don't — flag this when it comes up in a buy/build decision.

## Document assumptions as you go (not all upfront)

A subtle but important practice. The common advice is "document all assumptions" — but this leads to teams trying to enumerate every possible assumption upfront, which is unbounded and quickly turns into pedantry ("we assume the laws of physics will continue to hold"). Better practice:

- During threat modeling, when someone says "I assume that…" — write it down right then.
- Phrase the assumption falsifiably: "the load balancer terminates TLS before traffic reaches the app server" is testable; "the network is secure" is not.
- For each assumption, ask: *could it ever not be true? what can break if it's false, incomplete, or an overgeneralization?*
- Hand the assumption list to whoever does testing — assumptions become test cases.

The output of this is a focused, useful list of assumptions tied to specific threat-modeling moments, instead of an exhaustive document nobody reads.

## When to re-validate

Q4 isn't done at ship time. Re-validate when any of:

- A new feature or major component lands.
- An architectural change affects trust boundaries, identity, or data flow.
- A security incident reveals a gap the model didn't anticipate.
- The deployment environment changes (cloud migration, new tenancy model, regulatory boundary change).
- Periodically on a calendar trigger (annually for stable systems, more often for fast-moving ones).

Record the next-review trigger in the document so the team has an answer to "when do we re-look at this?"
