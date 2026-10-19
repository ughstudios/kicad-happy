# Select Checks for the Actual PCB

Use this before a full design review or readiness assessment. For a targeted
request, apply it to the requested circuit and its relevant dependencies;
do not expand a pinout question into a full-board qualification campaign.

## Establish the board and review boundary

Make a separate assessment for each PCB and revision, even when boards share
a project, panel, firmware, or power supply. Record the information needed
for the checks being considered:

- Function, assembly variant, design stage, and requested decision: schematic
  correctness, fabrication readiness, prototype bring-up, or product acceptance.
- Supply range and current through each relevant board path, including common
  returns, connectors, vias, and protection. Distinguish the external load's
  power from power dissipated on this PCB. A positive feed bypassing the board
  does not remove heating in a return path that still carries the load current.
- Actual interfaces and routed signals, edge rates, PWM modes, cable lengths,
  load behavior, and relevant startup/shutdown or fault states.
- Operating ambient, enclosure/cooling, duty cycle, life/performance targets,
  and manufacturing or compliance requirements where relevant.
- Evidence: raw circuit/layout, manufacturer datasheet revision and section,
  interface specification, fabrication capability, and user requirements.

Do not invent missing operating limits. Obtain them from project evidence or
ask only when they prevent the requested decision; useful independent work
can continue. Assess shared supplies, cables, grounding, and cooling at system
boundaries when they affect a board. A module's internal implementation is
outside the carrier review unless in scope; its carrier-facing supply,
interface, antenna/placement, and environmental requirements still apply.

## Decide applicability before choosing a tool

For relevant domains, record a compact row:

| PCB / revision | Circuit evidence | Failure mode / requirement | Check and method | Acceptance criterion / source | Disposition and result |
|---|---|---|---|---|---|
| [board] | [refs/nets and conditions] | [specific effect] | [inspection/calculation/simulation/measurement] | [limit or qualitative requirement with source] | [applicable / not applicable / unresolved / outside requested scope; evidence/result] |

**Applicable** means the mechanism or requirement exists; record whether it is
resolved by inspection/calculation, simulated, measured, failed, or still open.
**Not applicable** requires evidence that the mechanism or requirement is
absent. A low-risk thermal screen can close a thermal question without making
heat physically inapplicable. **Unresolved** means applicability or acceptance
depends on missing data. **Outside requested scope** identifies a relevant
question this review does not answer. An unrun tool is not a failed check if
another adequate method closes the requirement. Missing coverage for an
applicable requirement is a review limit; it affects readiness only to the
extent that the requested decision depends on that requirement.

For a full review, start with basic circuit/layout correctness: fitted-part
pinouts and footprints, intended connectivity/polarity, applicable ratings,
schematic/PCB agreement, and relevant ERC/DRC and mechanical constraints.
Then screen the domains below. Report decisions compactly; no empty chapters
or exhaustive lists of absent peripherals are needed.

| Domain | Circuit-based reason to select it | Proportionate verification |
|---|---|---|
| Power and thermal | Losses or local current density could exceed a part, copper, joint, connector, or temperature limit; restrictive ambient/cooling; explicit thermal requirement | Bound worst relevant operating losses and margins first. Escalate to board-specific modeling or load/temperature measurements when the bound cannot close the requirement. Include common returns and simultaneous loads. No universal wattage or ampere threshold selects every board. |
| Digital timing and signal integrity | An actual interface has pulse-width, setup/hold, skew, jitter, edge-rate, termination, or channel-loss requirements | Identify the routed interface and operating mode; compare its budget with propagation delay, edges, loading, and receiver requirements. Use the actual device/specification. Net names, component count, and nominal clock rate alone do not establish a timing requirement or exemption. |
| PWM and power switching | A switching power stage has minimum pulses, gate-charge/drive constraints, loss/overshoot, startup, or shutdown requirements | Check the supported PWM frequency/duty range and relevant transitions. Dead time applies only to topologies that require nonoverlap. Independent color outputs do not acquire a channel-alignment requirement merely because they share PWM firmware. |
| Analog behavior / SPICE | Accuracy, loading, dynamics, stability, resonance, or transients remain uncertain after inspection and calculation; or simulation is explicitly requested | State the question, representative model, operating corners, and acceptance limit. Use simulation when it adds decision-relevant evidence. A simple divider can be closed by arithmetic and tolerance bounds. Repeating an equation in an ideal simulation is not independent real-part verification. |
| EMC / ESD | Switching edges, noise-sensitive circuitry, radio, exposed connectors, cables, or a specified EMC requirement create a coupling or discharge path | Review the actual source, coupling/return path, and victim. Select relevant layout/protection checks; escalate to targeted modeling or measurement when needed. Full pre-compliance work and a laboratory test plan need a product/request basis. |
| Startup, inrush, protection, faults | Stored energy, hot-plug, brownout, inductive paths, reset behavior, or credible faults could violate requirements | Trace the actual energy and control paths; check relevant ratings and transitions. Apply scenarios supported by topology and intended use. Do not invent a fault-test campaign for absent mechanisms. |
| Sleep current / leakage | Battery life, standby power, off-state glow, backfeed, or another off-state requirement matters | Analyze the actual enabled/disabled states and paths; bound leakage under relevant conditions. A mains-powered board does not automatically need a battery-life study. |
| Isolation / spacing | Working/transient voltages, insulation boundaries, environment, or product requirements impose spacing or isolation limits | Establish the applicable working conditions and source of limits first. Check fabrication spacing for ordinary nets; add insulation analysis only where justified. Never infer a product safety class from BOM labels alone. |
| Manufacturing / fabrication outputs | Layout or fabrication readiness is in scope | Use the selected process and assembly requirements. Verify source-to-export consistency when release outputs exist or are requested. Missing Gerbers are expected during a schematic review; they are a gap in a requested final fabrication package. |
| Lifecycle / sourcing | Procurement, availability, substitution, product life, or production release is in scope | Audit the relevant fitted parts against current manufacturer/distributor evidence. Network access and MPN coverage alone do not require a lifecycle exercise. |

## Choose depth and preserve evidence

Tool availability, the presence of both JSON files, and an analyzer's ability
to emit a result do not justify an analysis. Use the least elaborate method
that adequately resolves the actual requirement. Running a broad analyzer
for efficient data collection is acceptable; triage its findings against
the board's applicability assessment before reporting them as issues.

Package-average thermal lookups, generic via-count thresholds, protocol
guesses, and default EMC scores are screening heuristics. Check their inputs
and conditions before assigning severity. Use part-specific thermal data and
actual board/cooling conditions; package theta-JA alone does not establish
the assembled board's temperature or current rating. See the sources below.

Distinguish inspected, calculated, simulated, and measured results. A build,
unit test, or simulation does not establish physical leakage, temperature,
or compliance. Identify the remaining measurements when the requested
acceptance decision needs them; do not turn every pre-fabrication design
review into a requirement to possess hardware already.

On revisions, rerun affected checks and dependencies. Reuse evidence only
when the circuit, part identity, limits, environment, and assumptions it
depends on remain valid. A fabrication change still needs the relevant
native checks and export consistency verification.

## Examples that constrain both over-testing and under-testing

- A passive low-current indicator board may close power/thermal concerns with
  resistor/LED ratings and bounded losses. With no timed interface or switching
  stage, a digital timing campaign is not applicable.
- A high-current analog strip controller needs checks of FET losses, common
  returns, connectors, gate drive, PWM transitions, and required off states.
  Independent colors do not imply tight interchannel skew. Any addressable
  data output is assessed separately against that part's protocol.
- A short board with a small part count can still carry a timing-sensitive
  interface; examine edge rates and receiver limits before dismissing it.
- A carrier and its power daughterboard get separate assessments. Evidence
  about low carrier current does not clear the daughterboard's load-current
  paths, and the reverse does not mandate power-stage tests on the carrier.

## Authoritative technical basis

These references explain selection criteria; the actual fitted part and
interface documents establish the design's acceptance limits.

- [TI, Semiconductor and IC Package Thermal Metrics, SPRA953D, sections 1–1.2](https://www.ti.com/lit/an/spra953d/spra953d.pdf): thermal metrics depend on the test board and environment; package values alone are inadequate for system temperature claims.
- [TI, High Speed Layout Guidelines, SCAA082A](https://www.ti.com/lit/an/scaa082a/scaa082a.pdf): digital edge content and transmission-line behavior inform which signals need detailed layout analysis.
- [TI, Fundamentals of MOSFET and IGBT Gate Driver Circuits, SLUA618A, sections 2.7–2.8](https://www.ti.com/lit/an/slua618a/slua618a.pdf): switching times, gate drive, and parasitics affect switching loss and overshoot.
