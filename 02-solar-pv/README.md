# Solar PV Boost Converter with MPPT (P&O) — LTspice

Months 7-8 catch-up work. Two parts: a working PV-fed boost converter (fixed duty cycle), and an in-progress attempt at closed-loop Perturb & Observe MPPT.

## Part 1: PV-fed boost converter (WORKING)

### PV panel model

Modeled as the standard simplified single-diode equivalent circuit: a fixed current source (I1 = 3A, representing photocurrent) in parallel with a diode (D2, 1N4148, representing the p-n junction). This is a genuine simplification of the full single-diode model — series resistance (Rs) and shunt resistance (Rsh) were omitted, since including them meaningfully requires real datasheet values from a specific panel, which weren't available here. The core behavior this captures — panel voltage sags as current draw increases, and there's a distinct power peak — is preserved even without Rs/Rsh.

### Boost topology

```
PV+ → L1 ──┬── D1(anode) ──cathode──→ output node → C1, R1
           │
          M1 Drain
           │
          M1 Source ──→ GND
```

Key structural difference from a buck converter: the inductor sits first (off the source), and the switch (M1, IRFH3707) sits after it, pulling that node to ground when on. Gate drive is simpler than the buck converter's — since Source is grounded here (not floating on a switching node), the gate signal can reference ground directly.

L1 = 100µH, C1 = 100µF, R1 = 10Ω, D1 = 1N4148.

### Result (fixed duty cycle, D=0.4)

PV operating voltage settled around 2.4V (with switching ripple), load voltage settled around 2.85V. Note: the ideal boost formula (V_out = V_in/(1-D)) doesn't directly predict this, because the PV model isn't a fixed voltage source — its voltage depends on how much current the converter draws from it, which is the entire reason MPPT exists as a problem in the first place.

## Part 2: Perturb & Observe MPPT (IN PROGRESS — not yet converging correctly)

### The goal

Rather than a fixed duty cycle, dynamically adjust duty cycle in real time to track the PV panel's maximum power point, using the standard P&O algorithm: perturb duty cycle slightly, observe whether power improved, and use that to decide the next perturbation direction.

### Why this is hard in LTspice specifically

LTspice is an analog circuit simulator, not a digital logic environment — there's no native "if power increased, keep going" block. The full algorithm was built using six chained behavioral sources (B-sources), each computing part of the decision:

| Block | Expression | Purpose |
|---|---|---|
| B1 | `V=V(n001)*I(L1)` | Instantaneous power (V×I at PV terminal) |
| B2 | `V=delay(V(pwr),200u)` | Remembers power from 200µs ago |
| B3 | `V=if(V(pwr)>V(pwr_prev), 1, -1)` | Did power improve? |
| B4 | `V=delay(V(duty_direction),200u)` | Remembers previous perturbation direction |
| B5 | `V=if(time<200u, 1, if(V(pwr_direction)>0, V(prev_direction), -V(prev_direction)))` | Core decision: keep direction if power improved, reverse if not |
| B6 | Capacitor (1µF, ic=0.5) + current source (`I=V(duty_direction)*100u`) in parallel | Accumulates the direction decisions into an actual, slowly-changing duty cycle command (`duty_cmd`) |

`duty_cmd` was then meant to be compared against a fast triangle wave (100kHz) via a comparator (`V=if(V(triangle)<V(duty_cmd), 5, 0)`) to generate the actual PWM gate signal — replacing the fixed PULSE source.

### Debugging lessons from this attempt

- **Algebraic loops (B4↔B5) cause solver failures.** B4 depends on B5's output, B5 depends on B4's output, with no genuine physical delay breaking the circularity from the solver's perspective at t=0. `delay()` alone wasn't sufficient. Adding an RC filter (1kΩ resistor + 1nF capacitor) in the path between B4's output and the node B5 reads gave the solver real continuous dynamics to converge around, rather than a pure instantaneous algebraic equality — though even this did not fully resolve stable tracking behavior in the full circuit.
- **`delay()` has no initial-value parameter in native LTspice** (unlike some other tools). Before real delayed history exists, it defaults to 0 — which, combined with the B4/B5 loop, created a permanent zero deadlock. Fixed with a time-gated bootstrap in B5's expression (`if(time<200u, 1, ...)`), forcing a real starting value for the first perturbation period.
- **`.tran ... uic`** (use initial conditions) forces ALL reactive components to start from their specified initial conditions, not just the one you're trying to fix — this caused a violent, unrealistic power spike when used, because the inductor's initial current defaulted to 0A with no smooth startup transition. Reverted; this is not the right tool for seeding a single capacitor's initial condition inside a larger circuit with other reactive elements.
- **Floating ground connections on behavioral sources** — every B-source's negative terminal needs an explicit, verified ground connection. This caused several "floating node" and "singular matrix" errors throughout the build, traced one at a time.
- **Net labels can silently detach** when editing/deleting nearby components. Worth re-verifying labels are still correctly attached after any edit near a labeled wire.

### Current status

The full circuit runs without fatal errors, but `duty_cmd` is not behaving correctly — it sits flat rather than drifting based on the P&O decisions, despite the underlying power signal (`pwr`) showing real, switching behavior. This suggests the issue is isolated to the B1-B6 decision chain rather than the power circuit itself, but it hasn't been confirmed working in isolation yet.

### Next steps

- Build B1-B6 in an isolated test schematic, replacing B1 with a simple fake power signal (e.g. a SINE source) instead of the real PV/boost circuit, to confirm the decision logic works correctly on its own before reconnecting to the real converter.
- Once confirmed working in isolation, reconnect to the real PV boost converter and re-verify.
- Consider Incremental Conductance as a follow-up MPPT algorithm once P&O is working — theoretically settles more cleanly at the peak rather than continuously oscillating around it, which is P&O's known characteristic limitation.

## Files

- `pv_boost_fixed_duty.asc` — Part 1, working fixed-duty-cycle version
- `pv_boost_mppt_attempt.asc` — Part 2, full P&O build (not yet converging correctly)
- Screenshots of key results and error states, for reference
