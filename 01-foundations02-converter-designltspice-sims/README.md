# Buck Converter — LTspice Simulation

First LTspice build, part of catching up on deferred practical simulation work (Months 1-6 foundations). Open-loop buck converter, switch-level model.

## Why this project

Foundations work (Months 1-6) covered buck converter theory and design on paper, but simulation was deferred due to limited computer access. This is that catch-up — verifying the theory actually holds using a real switch-level circuit simulator, not just Simulink transfer functions like the motor drive project.

## Circuit

```
       S1               L1
V1 ────/  ──┬──────/\/\/\──┬── output node
            │              │
           D1             C1        R1
            │              │         │
           GND            GND       GND
```

- **V1** — 12V DC input source
- **S1** — voltage-controlled switch (LTspice `SW` primitive), driven by a separate PULSE source across its control pins
- **D1** — 1N4148, freewheeling diode
- **L1** — 100µH
- **C1** — 100µF output filter capacitor
- **R1** — 10Ω load

Switch control: `.model SWMOD SW(Ron=1 Roff=1Meg Vt=2.5 Vh=0.1)`, driven by a PULSE source at 100kHz (10µs period).

## What's being tested

Ideal buck converter theory: V_out = D × V_in, where D is duty cycle (Ton/Period).

This is open-loop — fixed duty cycle, no feedback, no PID. The point is to confirm the switching behavior and steady-state output match theory, using a real (non-ideal) switch and diode model rather than pure math blocks.

## Results

| Duty Cycle (D) | Ton | Ideal V_out (D × 12V) | Simulated V_out |
|---|---|---|---|
| 0.5 | 5µs | 6.0V | ~6.0V |
| 0.4 | 4µs | 4.8V | 3.99V |

## Observations

- At D=0.4, simulated output (3.99V) came in below the ideal prediction (4.8V) — a real, expected gap, not an error. Accounted for by the diode's forward voltage drop (~0.6-0.7V when conducting) plus switch Ron losses. This is the kind of gap between ideal-formula theory and real simulated behavior that shows up in actual hardware too.
- Output shows a brief overshoot before settling — expected underdamped LC filter response to the initial turn-on transient, consistent with second-order system behavior.
- A debugging note worth keeping: LTspice failed with "Undefined model" errors twice during this build — once because a `.model` line was placed as a Comment instead of a SPICE Directive (so it was never actually read into the simulation), and once because time values were written with an invalid unit suffix (`5uS` instead of `5u`). Both are easy, silent mistakes worth double-checking on any future LTspice build.

## Files

- `buck_converter.asc` — LTspice schematic
- `buck_converter_d0.5.png` — scope output at D=0.5
- `buck_converter_d0.4.png` — scope output at D=0.4

## Next steps

- Test additional duty cycles to build out the table further
- Zoom into a single switching period to observe and measure actual ripple
- Extend into the deferred solar/MPPT LTspice simulations (Months 7-8)
