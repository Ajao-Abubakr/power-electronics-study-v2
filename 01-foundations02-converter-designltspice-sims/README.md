# Buck Converter — LTspice Simulation

Part of catching up on deferred practical simulation work (Months 1-6 foundations). Two phases: an ideal-switch version to prove the topology, and a real MOSFET version to see how a real component changes the picture.

## Why this project

Foundations work (Months 1-6) covered buck converter theory and design on paper, but simulation was deferred due to limited computer access. This is that catch-up — verifying the theory actually holds using a real switch-level circuit simulator, not just math blocks.

## Circuit topology (both phases)

```
       Switch           L1
V1 ────/  ──┬──────/\/\/\──┬── output node
            │              │
           D1             C1        R1
            │              │         │
           GND            GND       GND
```

- **V1** — 12V DC input source
- **D1** — 1N4148, freewheeling diode
- **L1** — 100µH
- **C1** — 100µF output filter capacitor
- **R1** — 10Ω load

---

## Phase 1: Ideal switch

Used LTspice's voltage-controlled switch primitive (`SW`), model: `.model SWMOD SW(Ron=1 Roff=1Meg Vt=2.5 Vh=0.1)`, driven by a separate PULSE source across its control pins (1+/1-), 100kHz switching (10µs period).

### Results

| Duty Cycle (D) | Ton | Ideal V_out (D × 12V) | Simulated V_out |
|---|---|---|---|
| 0.5 | 5µs | 6.0V | ~6.0V |
| 0.4 | 4µs | 4.8V | 3.99V |

Gap at D=0.4 explained by diode forward drop (~0.6-0.7V) plus switch Ron losses — expected, not an error.

### Debugging lessons (Phase 1)

- A `.model` line placed as a Comment instead of a SPICE Directive is silently ignored by the simulator — must explicitly select SPICE Directive when adding it via the Edit → Text tool.
- Time value unit suffixes must be exact — `5uS` is invalid and breaks silently; must be `5u`.

---

## Phase 2: Real MOSFET (IRFH3707)

Upgraded from the ideal switch to a real N-channel MOSFET, IRFH3707 — chosen because it's explicitly rated for high-frequency buck converter applications (30V, 62A, low Rds-on, low gate impedance).

### Why this matters

The ideal switch abstracts away real device behavior. A real MOSFET introduces genuine physical effects: gate charge/capacitance (switching isn't instantaneous), real Rds-on, and — critically — the gate must be driven relative to the MOSFET's own **Source** terminal, not circuit ground. This is a real practical consideration in actual gate-drive circuit design (high-side gate drive), not just a simulation quirk.

### Wiring

| Terminal | Connects to |
|---|---|
| Drain | V1(+) only |
| Source | L1(left) + D1(cathode) — the switching node |
| Gate | Pulse source (+) |
| Pulse source (−) | Source / switching node (NOT ground — gate voltage is referenced to Source) |
| V1(−) | Ground |
| D1(anode) | Ground |

### Debugging lessons (Phase 2)

- **Missing ground symbol.** The circuit appeared fully wired in a closed loop, but without an explicit ground (`G`) symbol placed on the reference rail, LTspice has no node-0 reference to solve against. This produced wildly invalid results — voltage spikes into the kilovolt range (17-24KV), which looked like a genuine circuit fault (classic inductive-kickback signature) but was actually just a missing ground reference. Placing the ground symbol on the bottom rail resolved it completely.
- Wires that visually cross on the schematic are NOT automatically connected in LTspice — only shared endpoints/junctions (shown as a solid dot) count as a real connection. Worth double-checking visually crossing wires aren't accidentally assumed to be joined.
- A MOSFET's gate must be referenced to its own Source terminal, not ground — different from the ideal switch's control pins, which were ground-referenced.

### Result (D=0.5, 100kHz)

Output rises, overshoots briefly to ~8.8V, settles at ~6.0V — consistent with the D=0.5 duty cycle, and comparable in shape to the Phase 1 ideal-switch result at the same duty cycle. Confirms the MOSFET-based version behaves consistently with the idealized version, with the expected transient/overshoot shape from the underdamped LC filter response.

## Files

- `buck_converter_ideal_switch.asc` — Phase 1 schematic
- `buck_converter_mosfet.asc` — Phase 2 schematic
- `buck_converter_d0.5.png`, `buck_converter_d0.4.png` — Phase 1 results
- `buck_converter_mosfet_result.png` — Phase 2 result

## Next steps

- Compare Phase 1 vs Phase 2 output more rigorously (steady-state value, ripple, rise time) side by side
- Test additional duty cycles on the MOSFET version
- Zoom into a single switching period to observe real MOSFET switching losses/transition time vs the ideal switch
- Extend into the deferred solar/MPPT LTspice simulations (Months 7-8)
