# STPA — loss-driven safety + security analysis

> **Last verified**: 2026-05. The methodology and the IEC 61508 / 62304 / 81001-5-1 / 62443 / ISO 26262 framings are stable; re-confirm regulator edition years on any submission deliverable.
> **Sources paraphrased**: Leveson (2011), *Engineering a Safer World* (MIT Press) — original STPA; Young & Leveson (2014), *An Integrated Approach to Safety and Security Based on Systems Theory*, CACM 57(2) — STPA-Sec extension; Friedberg, McLaughlin, Smith, Laverty & Sezer (2017), *STPA-SafeSec*, Journal of Information Security and Applications 34 (CC-BY) — integrated workflow + component-layer mapping; Thomas (2013, MIT thesis) — UCA state-space approach; Khalil, Bahsi & Korõtko (2023). Substantive direct quotes need upstream attribution; paraphrasing here is permitted under each license.

> **Related**: ← `SKILL.md` • `methodologies.md` (where this methodology is registered alongside LINDDUN / PASTA / attack trees) • `centric-methods.md` (entry-point taxonomy; STPA is not an entry point) • `risk-rating.md` (the generic `AV / PR / AC` scheme; STPA hazard severity composes with the row's exposure — the medical-device ISO 14971 / AAMI TIR57 mapping is in `industries/medical/risk-rating.md`) • `validation.md` (Q4 framing).

> **Naming.** "STPA" in this skill is the umbrella term for the loss-driven safety + security workflow this file describes. The lineage is **STPA** (Leveson 2011, original safety-only hazard analysis) → **STPA-Sec** (Young & Leveson 2014, security extension) → **STPA-SafeSec** (Friedberg et al. 2017, integrated workflow + component-layer mapping). The workflow below is closest to STPA-SafeSec because the component layer is what closes the abstraction gap and makes mitigations enforceable on real hardware — but we use "STPA" as the working name throughout. Cite the specific paper when the distinction matters (e.g. citing STPA-SafeSec's integrity/availability causal-factor tables).

A safety + security joint analysis. Originally for control-loop systems but applies to any system where the worst case is physical harm to people, equipment, or the environment.

What STPA generates is **hazard scenarios**, not threats. A hazard scenario is one specific way a system flaw leads to a UCA leads to a hazard leads to a loss. Adversarial-cause hazard scenarios (driven by `CSTR-I-*` / `CSTR-A-*`) are threat-shaped; non-adversarial-cause hazard scenarios (sensor failure, controller logic bug, missing feedback) are safety-failure-shaped. The output is a superset of what flow-centric + STRIDE produces.

For where this fits among the skill's other methodology choices, see `methodologies.md`. For entry-point taxonomy (STPA is not an entry point), see `centric-methods.md`.

## When to load STPA

**Load this file whenever the system has a safety impact** — i.e. any plausible worst-case outcome involves physical harm to a person, damage to equipment or property, or environmental release. Don't gate it behind regulator presence or "is it strictly a control loop". The trigger is the *consequence*, not the system shape.

Examples that should load STPA on the basis of safety impact alone:
- A medical device of any class (infusion pump, ventilator, monitor, surgical robot, dialysis, ECMO, smart anesthesia).
- An industrial control system (PLC, RTU, HMI, historian, SIS).
- Automotive (ECUs, ADAS, infotainment-to-CAN bridges).
- Aerospace, robotics, drones, autonomous vehicles.
- Building control with safety implications (fire suppression, elevator control, access control on safety doors).
- Consumer IoT that controls something that can hurt someone (smart locks, garage doors, large appliances, e-bike batteries).

**Don't load** when the worst case is purely a data breach, financial fraud, or unauthorized access with no physical-harm path. STPA under-weights confidentiality and audit threats — use STRIDE alone.

## Two modes: swap or supplement

STPA can be used two ways inside this skill's hybrid output. Pick by what artifact the team needs.

**Swap mode — STPA replaces flow-centric + STRIDE for the contextual core.** The §2.1 contextual stratum becomes the STPA workflow end-to-end (Losses → Hazards → Constraints → Control layer → Component layer → UCAs → System flaws → Hazard-scenario trees). One artifact covers safety and security jointly. Use this when:

- Regulators require the joint safety+security artifact anyway (FDA premarket cybersecurity for control-loop devices; IEC 62304 + IEC 81001-5-1 for medical software; IEC 61508 for industrial functional safety; ISO 26262 for automotive). Producing one document instead of two saves work and forces the team to confront safety/security interactions.
- The team is greenfield on both safety analysis and threat modeling — pick STPA once rather than maintaining two parallel practices.
- The system is purely a control loop with negligible non-control attack surface (no telemetry uploads, no admin web UI, no audit log channel). When the entire system *is* the control loop, splitting STRIDE off makes no sense.

**Supplement mode — STPA runs alongside flow-centric + STRIDE.** §2.1.a (flow-centric STRIDE-Per-Element) and §2.1.b (supplementary entry-point pass) run normally; STPA adds a new pass (§2.1.e in the template) that produces hazard scenarios. Hazard scenarios cross-reference threats they propagate from (`HS-3 ← T7`) and add non-adversarial safety failures the threat model would miss (`HS-4: sensor degradation; not a threat`). Use this when:

- The team already has a STRIDE/DFD threat-modeling practice and an existing threat-model artifact — bolt safety hazard analysis onto it rather than restart.
- The system has substantial non-control attack surface alongside its control loops (telemetry, OTA, audit, admin UI, mobile companion app). STRIDE+DFD covers the non-control surface; STPA covers the control loops.
- The security review is being done independently of the safety analysis (e.g., third-party security audit on a system that already has its own safety case). The safety case stays separate; STPA borrows from it to inform the security work.
- Safety and security functions are owned by separate teams producing separate artifacts.

Both modes use the same workflow below — what differs is whether the output replaces §2.1.a/b or adds a new §2.1.e. **When in doubt, supplement** — bolting STPA onto an existing STRIDE practice is lower-risk than swapping the contextual core.

## Mapping to the Four Question Framework

|Four Question|Default (flow-centric + STRIDE)    |STPA                                                                  |
|-------------|-----------------------------------|------------------------------------------------------------------------------|
|Q1           |DFD with trust boundaries          |Losses → Hazards → Constraints → Control layer → Component layer              |
|Q2           |STRIDE-Per-Element on DFD elements |UCAs → System flaws → Hazard scenarios (tree)                                 |
|Q3           |Mitigations + security requirements|Mitigations at control or component layer + requirements traced to constraints|
|Q4           |Standard checklists (see `validation.md` § "The closing checklists") |Standard checklists + tree-coverage + traceability checks                     |

## Q1 — Losses, hazards, constraints, two-layer system model

**1. Identify losses.** Stakeholder-language outcomes that must never happen. Format: `L-1: <unacceptable outcome>`. Aim for 3–7. Don’t pad with abstractions like “company reputation” — same anti-pattern as asset-rabbit-holing.

**2. Identify hazards.** System states that, in worst-case conditions, lead to a loss. Format: `H-1 [L-1, L-3]: <system state>`. Each hazard maps to one or more losses; each loss should have at least one hazard.

**3. Derive constraints.** Format: `CSTR-S-1 [H-1]: System SHALL <prevent / limit / detect> <hazard condition>`. Simplest derivation is negation of each hazard. At this stage all constraints typically read as “safety” — security constraints get added during Q2 refinement.

**4. Build the control layer.** Mermaid `flowchart TD` (top-down rather than the skill’s default LR — control hierarchy reads vertically). Shows the **logical** control loops:

- Controllers (CTRL-N-x) → boxes
- Controlled processes → boxes lower in the diagram
- Control actions (CTRL-C-x) → solid arrows downward, labeled with the action
- Feedback → dashed arrows upward, labeled with the measurement
- Human operators → boxes at the top with their own control actions and feedback

**5. Build the component layer.** Shows the **physical implementation** of the control layer:

- Concrete nodes (CPT-N-x) — actual hardware: the Raspberry Pi running the algorithm, the microcontroller doing D2A conversion, the GPS antenna, the network switch, the firewall
- Concrete connections (CPT-C-x) — actual links: USB serial, hardwired, IP-over-LAN, IP-over-WAN, GPS RF link
- Distinguish wired vs wireless and end-to-end vs IP-routed connections visually

A single control-layer node maps to one or more component-layer nodes. A single control-layer connection often maps to a chain of hops (switch → firewall → WAN → remote endpoint). The component layer is what lets abstract constraints become enforceable on real hardware.

## Q2 — UCAs, system flaws, hazard scenarios

**6. Identify Unsafe/Unsecure Control Actions (UCAs).** For each control action, walk the four classic STPA guide phrases:

1. Action **not provided** when needed → hazard
1. Action **provided** when not needed → hazard
1. Action provided **too early, too late, or in wrong order** → hazard
1. Action **stopped too soon** or **applied too long** → hazard

Format: `HC-N: <controller> <provides/withholds/...> <action> when <context>, leading to <hazards>.`

Record only guide-phrase combinations that lead to a hazard. UCA enumeration can be partially automated using Thomas’s (2013) state-space approach.

**7. Identify system flaws.** Each UCA traces back to one or more system flaws — design conditions that allow the UCA. Format: `F-N: <Controller incorrectly believes X>` / `<wrong setpoint received by Y>`.

**8. Map constraints to layers and add security causal factors.** Classic STPA causal factors (Leveson 2011) cover non-adversarial causes: inadequate process model, controller logic flaw, sensor failure, actuator failure, missing or wrong feedback. STPA adds adversarial causal factors as integrity threats and availability threats:

- **Integrity (CSTR-I)**: command injection, command drop, command manipulation, command delay, measurement injection, measurement drop, measurement manipulation, measurement delay
- **Availability (CSTR-A)**: communication delay, communication dropped, node overloaded (delay), node overloaded (drop)

For each system flaw, walk both the classic safety causal factors and the integrity/availability causal factors. Map each potential constraint violation to the specific component(s) at which it could occur. Friedberg’s tables also indicate which entity’s compromise (sender, receiver, network node, physical link, protocol) is sufficient vs. necessary-but-insufficient to realize each threat — that informs where to place mitigations.

**9. Build hazard-scenario trees.** A **hazard scenario** is a textual description of one specific way a system flaw leads to a UCA leads to a hazard. Format:

```
Scenario N: <description>
  Hazards: H-X, H-Y
  System Flaw: F-Z
  Hazardous Control Action: HC-W
  Control Level Components: CTRL-N-..., CTRL-C-...
  Component Level Components: CPT-N-..., CPT-C-...
  Safety Constraints: <list>
  Security Constraints: CSTR-I-..., CSTR-A-...
```

Organize scenarios as a **tree**: one root scenario per system flaw, refined into sub-scenarios down to the level where mitigation can be designed. Stop refining when sub-scenarios are concrete enough to drive a mitigation decision.

## Q3 — Mitigations and requirements

Same response taxonomy as the rest of the skill: **Mitigate / Eliminate / Transfer / Accept**. Mitigations can be applied at **either** layer:

- **Control-layer mitigation** — change the control logic, add interlocks or redundant feedback, redesign the loop. Example: a hardwired safety device at a circuit breaker that locally measures voltage/frequency/phase and refuses to close if differences are too high. Mitigates the high-level hazard regardless of cyber compromise — but can mask compromise from operators.
- **Component-layer mitigation** — conventional security controls on specific concrete components: authenticate the command channel, sign firmware, integrity-check feedback at the application layer, harden network boundaries, place SLAs on links not under your control.

A mitigation strategy is **effective** if it cuts every leaf-to-root path in the hazard-scenario tree.

When **no mitigation is feasible** (legacy equipment that can’t be replaced, WAN links not under your control), use **loss prioritization** — explicitly accept a less-critical loss to prevent a more-critical one (e.g., accept temporary power supply interruption to prevent injury to humans).

**Hazard severity lives here, not on the cyber threat row.** Each STPA hazard (`H-#`) carries severity at the hazard level — `S1` (Negligible) through `S5` (Catastrophic) using the ISO 14971 scale, or the team's domain-specific equivalent. Harm to people is almost always `S4` (Critical) or `S5` (Catastrophic). Cyber threats that *trigger* a hazard cross-reference the `H-#` from their `Element` cell in §2.1; the §3 risk register composes the threat's `AV / PR / AC` exposure with the hazard's severity. For medical-device submissions, the regulator-facing composition into ISO 14971 / AAMI TIR57 risk language is in `industries/medical/risk-rating.md`.

Derived requirements trace upward: requirement → mitigation → hazard scenario → UCA → constraint → hazard → loss.

## Q4 — Validation additions

Standard Q4 checklists (`validation.md` § "The closing checklists") still apply. Add:

- [ ] Every loss has at least one hazard
- [ ] Every hazard has at least one constraint
- [ ] Every UCA traces to at least one system flaw
- [ ] Every system flaw has a root hazard scenario refined to where mitigation can be designed
- [ ] Every leaf-to-root path in each hazard-scenario tree is mitigated, OR the residual loss has been explicitly accepted via prioritization
- [ ] Adversarial causal factors (CSTR-I-1..8, CSTR-A-1..4) considered for every relevant constraint
- [ ] Component layer reflects current/planned reality, not a sanitized version

The analysis is iterative — re-run as the system evolves (new components, new connections, new threats). Within one analysis, work each control loop independently before composing.

## Relationship to traditional security analysis

STPA does not replace pen testing, vulnerability assessment, or fuzzing — it **prioritizes** them. The component layer plus hazard-scenario trees identify the components where deep security analysis matters most. Pen-test results, CVEs, and fuzzing findings then attach back to the relevant hazard scenarios, refining the tree.

STRIDE is not the default companion to STPA — the integrity/availability causal-factor tables are more specific to control-loop systems and cover the relevant adversarial cases. Layer STRIDE in only for non-control flows that appear at the component layer (telemetry uploads, OTA updates, audit log paths) where confidentiality and repudiation matter; the integrity/availability tables don’t cover those.

For multi-device cyber-physical environments where compromise of a low-criticality peripheral can reach a high-criticality target across the room or plant floor (IoMT patient rooms, ICS plant floors with shared RF, robotics cells), the safety-critical components STPA produces are natural roots for the **risk-prioritized cyber-physical attack-path** construction in `methodologies.md` § "Risk-prioritized cyber-physical attack paths (Stellios et al., 2021)" — STPA answers "which target," and the Stellios construction answers "which paths to it are worth analyzing." Pair them when the system has both control-loop safety implications and a multi-device shared-physical-space attack surface.

## Section template (copy when in scope)

When STPA is in scope, paste this scaffold into §1 of the threat-model document (under "Data of interest" or in its own subsection); it carries the Q1 outputs the rest of the analysis depends on. The threat-model template at `assets/threat-model-template.md` deliberately omits this scaffold so non-safety-critical work isn't burdened with it.

```
### Losses, hazards, constraints (STPA)

- **Losses** (`L-1` …): unacceptable stakeholder outcomes (patient harm, equipment destruction, environmental damage, mission failure)
- **Hazards** (`H-1 [L-x]` …): system states that, in worst case, lead to a loss
- **Constraints** (`CSTR-S-1 [H-x]` …): system SHALL prevent / limit / detect each hazard condition
- **Control layer** (Mermaid `flowchart TD` — controllers `CTRL-N-x`, controlled processes, control actions `CTRL-C-x` as solid arrows down, feedback as dashed arrows up)
- **Component layer** (concrete nodes `CPT-N-x`, concrete connections `CPT-C-x`; wired vs wireless, end-to-end vs IP-routed)
```

In §2.1 of the threat-model document, two modes apply:

- **Swap mode**: §2.1.a (flow-centric STRIDE-Per-Element) is replaced by the STPA workflow — UCAs (`HC-N`), system flaws (`F-N`), hazard-scenario trees, integrity/availability constraints (`CSTR-I-N`, `CSTR-A-N`). STRIDE may still layer in for non-control flows (telemetry, OTA, audit) at the component layer.
- **Supplement mode**: §2.1.a runs normally; add an §2.1.e "Hazard scenarios (STPA)" sub-section. Hazard scenarios cross-reference the threats they propagate from (`HS-3 ← T7`) and add non-adversarial safety failures STRIDE misses (`HS-4: sensor degradation; not a threat`).

Q4 additions for either mode are in § "Q4 — Validation additions" above.

## References

- Young, W. & Leveson, N. (2014). *An Integrated Approach to Safety and Security Based on Systems Theory*. CACM 57(2).
- Friedberg, McLaughlin, Smith, Laverty & Sezer (2017). *STPA: Safety and security analysis for cyber-physical systems*. Journal of Information Security and Applications 34. CC-BY.
- Leveson, N. (2011). *Engineering a Safer World*. MIT Press.
- Thomas, J. (2013). *Extending and automating a systems-theoretic hazard analysis for requirements generation and analysis*. PhD thesis, MIT.
- Khalil, Bahsi & Korõtko (2023). *Threat modeling of industrial control systems: A systematic literature review*. Computers & Security.
