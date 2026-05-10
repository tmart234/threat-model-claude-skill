# STPA-SafeSec — loss-driven entry mode

> **Related**: ← `SKILL.md` • `methodologies.md` (full-methodology swap context) • `centric-methods.md` (entry-point alternatives that keep STRIDE) • `risk-rating.md` (safety-bump rule) • `validation.md` (Q4 framing).

A full-methodology alternative to flow-centric + STRIDE for safety-critical control-loop systems. **STPA-Sec** (Young & Leveson, 2014) extends Leveson’s STPA hazard analysis to security. **STPA-SafeSec** (Friedberg et al., 2017) integrates safety and security into one workflow and adds a component-layer mapping that makes the analysis actionable. Use STPA-SafeSec, not plain STPA-Sec — the component layer is what closes the abstraction gap.

This is a methodology swap, not an entry-point swap. Replaces both the DFD and STRIDE-Per-Element. For methodology-swap context, see `methodologies.md`. For entry-point alternatives that keep STRIDE, see `centric-methods.md`.

## When to pick STPA-SafeSec

Use it when:

- The worst case is **physical, not informational** — patient harm, operator injury, environmental damage, equipment destruction. Medical devices, ICS, automotive, aerospace, robotics.
- A safety analysis is required by regulation (FDA premarket cybersecurity, IEC 62304, IEC 81001-5-1, IEC 61508, ISO 26262) and producing one artifact is better than two.
- The system is **control-loop-shaped**: controllers issuing commands, sensors providing feedback, actuators acting on the physical world.
- You need to prioritize where to spend in-depth security analysis effort (pen testing, fuzzing, deep code review).

Don’t use it when:

- The worst case is a data breach or unauthorized access. STPA-SafeSec under-weights confidentiality and audit threats. Use STRIDE.
- There’s no controlled physical process.

## Mapping to the Four Question Framework

|Four Question|Default (flow-centric + STRIDE)    |STPA-SafeSec                                                                  |
|-------------|-----------------------------------|------------------------------------------------------------------------------|
|Q1           |DFD with trust boundaries          |Losses → Hazards → Constraints → Control layer → Component layer              |
|Q2           |STRIDE-Per-Element on DFD elements |UCAs → System flaws → Hazard scenarios (tree)                                 |
|Q3           |Mitigations + security requirements|Mitigations at control or component layer + requirements traced to constraints|
|Q4           |Standard checklists (see SKILL.md) |Standard checklists + tree-coverage + traceability checks                     |

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

**8. Map constraints to layers and add security causal factors.** Classic STPA causal factors (Leveson 2011) cover non-adversarial causes: inadequate process model, controller logic flaw, sensor failure, actuator failure, missing or wrong feedback. STPA-SafeSec adds adversarial causal factors as integrity threats and availability threats:

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

Risk rating inherits the skill default: qualitative L/M/H. Patient/operator harm is almost always High impact (see `risk-rating.md`).

Derived requirements trace upward: requirement → mitigation → hazard scenario → UCA → constraint → hazard → loss.

## Q4 — Validation additions

Standard Q4 checklists (in SKILL.md) still apply. Add:

- [ ] Every loss has at least one hazard
- [ ] Every hazard has at least one constraint
- [ ] Every UCA traces to at least one system flaw
- [ ] Every system flaw has a root hazard scenario refined to where mitigation can be designed
- [ ] Every leaf-to-root path in each hazard-scenario tree is mitigated, OR the residual loss has been explicitly accepted via prioritization
- [ ] Adversarial causal factors (CSTR-I-1..8, CSTR-A-1..4) considered for every relevant constraint
- [ ] Component layer reflects current/planned reality, not a sanitized version

The analysis is iterative — re-run as the system evolves (new components, new connections, new threats). Within one analysis, work each control loop independently before composing.

## Relationship to traditional security analysis

STPA-SafeSec does not replace pen testing, vulnerability assessment, or fuzzing — it **prioritizes** them. The component layer plus hazard-scenario trees identify the components where deep security analysis matters most. Pen-test results, CVEs, and fuzzing findings then attach back to the relevant hazard scenarios, refining the tree.

STRIDE is not the default companion to STPA-SafeSec — the integrity/availability causal-factor tables are more specific to control-loop systems and cover the relevant adversarial cases. Layer STRIDE in only for non-control flows that appear at the component layer (telemetry uploads, OTA updates, audit log paths) where confidentiality and repudiation matter; the integrity/availability tables don’t cover those.

## References

- Young, W. & Leveson, N. (2014). *An Integrated Approach to Safety and Security Based on Systems Theory*. CACM 57(2).
- Friedberg, McLaughlin, Smith, Laverty & Sezer (2017). *STPA-SafeSec: Safety and security analysis for cyber-physical systems*. Journal of Information Security and Applications 34. CC-BY.
- Leveson, N. (2011). *Engineering a Safer World*. MIT Press.
- Thomas, J. (2013). *Extending and automating a systems-theoretic hazard analysis for requirements generation and analysis*. PhD thesis, MIT.
- Khalil, Bahsi & Korõtko (2023). *Threat modeling of industrial control systems: A systematic literature review*. Computers & Security.
