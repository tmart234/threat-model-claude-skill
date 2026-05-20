# Safety-critical and cyber-physical systems

Reach for this file only when the worst case is **physical harm** — patient injury, operator harm, equipment destruction, environmental damage — and the system is a **control loop** (controllers issuing commands, sensors providing feedback, actuators acting on the world). Medical devices, industrial control, automotive, aerospace, robotics.

If the worst case is a data breach or unauthorized access, STRIDE on a DFD is the right tool — skip this file. STPA-SafeSec under-weights confidentiality and audit threats.

Two techniques live here: **STPA-SafeSec** (a safety+security hazard analysis for control loops) and **cyber-physical attack paths** (for environments where many devices share a physical space).

## STPA-SafeSec

STPA-Sec (Young & Leveson, 2014) extends Leveson's STPA hazard analysis to security; STPA-SafeSec (Friedberg et al., 2017) integrates safety and security into one workflow and adds a component-layer mapping that makes the analysis actionable. Use STPA-SafeSec — the component layer is what closes the gap from abstract constraint to enforceable control.

It produces **hazard scenarios**, not threats: one specific way a system flaw leads to an unsafe control action leads to a hazard leads to a loss. Adversarial-cause scenarios are threat-shaped; non-adversarial ones (sensor failure, controller logic bug) are safety-failure-shaped. The output is a superset of what STRIDE produces.

### Swap or supplement

- **Swap** — STPA-SafeSec replaces the DFD + STRIDE pass. Use when regulators require a joint safety+security artifact anyway (FDA premarket cybersecurity, IEC 62304 / 81001-5-1, IEC 61508, ISO 26262), or when the system is purely a control loop with negligible non-control surface.
- **Supplement** — STPA-SafeSec runs alongside the DFD + STRIDE pass and adds hazard scenarios that cross-reference threats. Use when there's an existing STRIDE practice, or substantial non-control surface (telemetry, OTA, admin UI) that STRIDE still needs to cover.

Same workflow either way.

### Workflow

**Q1 — losses, hazards, constraints, system model**

1. **Losses** (`L-1` …) — unacceptable stakeholder outcomes. 3–7 of them; don't pad with abstractions.
2. **Hazards** (`H-1 [L-1]` …) — system states that, in worst-case conditions, lead to a loss.
3. **Constraints** (`CSTR-S-1 [H-1]` …) — "the system SHALL prevent / limit / detect <hazard condition>". Often just the negation of each hazard.
4. **Control layer** — a Mermaid `flowchart TD` of the logical control loops: controllers, controlled processes, control actions (solid arrows down), feedback (dashed arrows up), human operators at the top.
5. **Component layer** — the physical implementation: concrete hardware nodes and concrete connections (wired vs. wireless, end-to-end vs. routed). One control-layer node maps to one or more component-layer nodes; one control-layer connection often maps to a chain of hops.

**Q2 — unsafe control actions, system flaws, hazard scenarios**

6. **Unsafe/Unsecure Control Actions (UCAs)** — for each control action, walk the four STPA guide phrases: action *not provided* when needed; *provided* when not needed; provided *too early / too late / wrong order*; *stopped too soon* or *applied too long*. Record only combinations that lead to a hazard.
7. **System flaws** — each UCA traces to design conditions that allow it ("controller incorrectly believes X").
8. **Causal factors** — for each flaw, walk both the classic safety factors (inadequate process model, controller logic flaw, sensor/actuator failure, missing feedback) and the adversarial ones STPA-SafeSec adds: integrity factors (command/measurement injection, drop, manipulation, delay) and availability factors (communication delay/drop, node overload).
9. **Hazard-scenario trees** — one root scenario per system flaw, refined into sub-scenarios down to the level where a mitigation can be designed.

**Q3 — mitigations**

Same response taxonomy as the rest of the skill (Mitigate / Eliminate / Transfer / Accept). Mitigations apply at either layer:

- **Control-layer** — change the control logic, add interlocks or redundant feedback. Mitigates the hazard regardless of cyber compromise, but can mask compromise from operators.
- **Component-layer** — conventional security controls on concrete components: authenticate the command channel, sign firmware, integrity-check feedback.

A mitigation strategy is effective if it cuts every leaf-to-root path in a hazard-scenario tree. When no mitigation is feasible (legacy equipment, links you don't control), use **loss prioritization** — explicitly accept a less-critical loss to prevent a more-critical one. Patient/operator harm is almost always High impact (`risk-rating.md`).

**Q4 — additions to the standard checklist**

- Every loss has a hazard; every hazard has a constraint.
- Every UCA traces to a system flaw; every flaw has a root hazard scenario refined to where mitigation can be designed.
- Every leaf-to-root path is mitigated, or the residual loss is explicitly accepted.
- The component layer reflects reality, not a sanitized version.

STPA-SafeSec doesn't replace pen testing or fuzzing — it **prioritizes** them, by identifying the components where deep security analysis matters most.

## Cyber-physical attack paths (multi-device environments)

When many devices share one physical space — a smart building, an industrial cabinet, a vehicle cabin, a robotics cell, a clinical room — the most dangerous threat is often *composition*: no single device looks dangerous under per-element STRIDE, but a chain of hops between commodity devices reaches a safety-critical target. Per-element analysis misses this.

The construction (Stellios et al., *Assessing IoT-enabled cyber-physical attack paths against critical systems*, Computers & Security 107, 2021):

- **Root the tree at the high-value target** (the safety-critical actuator, the patient-attached device) and walk *outwards* across each device's physical interaction surface to enumerate which other devices can influence it. Then walk inwards from each attacker entry point. The candidate paths are the intersection — not the full reachability graph.
- **Classify each between-device hop** by physical surface:
  - **P1 — direct contact** (USB, JTAG, an attached cable).
  - **P2 — proximity** (BLE/NFC reach, line-of-sight IR, audible-range acoustic, near-field EM).
  - **P3 — shared medium without proximity** (the 2.4 GHz ISM band shared across Wi-Fi/BLE/Zigbee, a shared power rail, a shared HVAC duct). A device on a P3 surface can disrupt another without ever associating with it.
- **Prune by risk before expanding.** Score each hop with a CVSS-derived metric and drop low-scoring paths before expanding the next layer. Without this, the graph explodes combinatorially and becomes unreviewable — the pruned subset is the auditable artifact.

Pair this with STPA-SafeSec (which identifies the safety-critical targets to root the tree at) and with flow-centric STRIDE (which surfaces the per-device surfaces the hops connect).

## Sources

- Young, W. & Leveson, N. (2014). *An Integrated Approach to Safety and Security Based on Systems Theory*. CACM 57(2).
- Friedberg, McLaughlin, Smith, Laverty & Sezer (2017). *STPA-SafeSec: Safety and security analysis for cyber-physical systems*. Journal of Information Security and Applications 34. CC-BY.
- Leveson, N. (2011). *Engineering a Safer World*. MIT Press.
- Stellios, Kotzanikolaou & Grigoriadis (2021). *Assessing IoT-enabled cyber-physical attack paths against critical systems*. Computers & Security 107.
