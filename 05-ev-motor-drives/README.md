# DC Motor Speed Control (Closed-Loop, PID)

Month 9 project — first hands-on build in the EV/Motor Drives phase of the roadmap. A closed-loop PID speed controller for a DC motor, built in Simulink.

## Why this project

The buck converter work (Months 1-6, `04-flyback-build/`) proved out closed-loop control for voltage regulation. This project applies the same control structure to a different physical target: motor speed instead of converter voltage. Same skeleton (reference → controller → power stage → plant → feedback), different plant and different power stage.

## Tooling note

This was built without Simscape Electrical, since Simscape isn't supported on the free/basic tier of MATLAB Online, and computer access at the time was limited to short library sessions. Rather than wait for full desktop access, the motor and H-bridge were modeled at the signal/behavioral level in plain Simulink. This is a legitimate, historically standard way to model motor control (used long before physical component libraries like Simscape existed), and the tradeoff is documented here rather than hidden.

## Block diagram

```
[Speed Reference] → (compare) → [PID] → [H-Bridge subsystem] → (+ disturbance) → [Motor Transfer Fcn] → [Speed Output]
                        ↑                                                                                      |
                        └──────────────────────────── Feedback (measured speed) ──────────────────────────────┘
```

## Motor model — derivation

The DC motor is modeled as a single transfer function, combining its electrical and mechanical dynamics.

**Electrical (armature circuit):**

V_a(s) = (R_a + L_a s) I_a(s) + K_b Ω(s)

**Mechanical (torque balance):**

K_t I_a(s) = (J s + B) Ω(s) + T_load(s)

**Combined transfer function (voltage → speed):**

Ω(s) / V_a(s) = K_t / [ (Js + B)(R_a + L_a s) + K_t K_b ]

**Parameters used** (typical small-motor values, no specific datasheet):

| Parameter | Symbol | Value |
|---|---|---|
| Armature resistance | R_a | 1 Ω |
| Armature inductance | L_a | 0.5 H |
| Torque constant | K_t | 0.01 N·m/A |
| Back-EMF constant | K_b | 0.01 V/(rad/s) |
| Inertia | J | 0.01 kg·m² |
| Damping | B | 0.001 N·m·s |

Resulting Transfer Fcn block: numerator `[0.01]`, denominator `[0.005 0.0105 0.0011]`.

(Note: an initial damping value of B = 0.1 was tried first and produced an unrealistically low steady-state speed for a given voltage — traced back to the damping term dominating the DC gain. Lowering B to 0.001 brought the model in line with expected behavior for a small motor.)

## H-Bridge — behavioral model

Modeled as a subsystem: Gain (12, representing supply voltage) → Saturation (±12). This captures the real H-bridge's core function — producing a bounded, bidirectional average voltage from a control signal — without modeling the underlying switch-level/PWM behavior. A true switch-level model would require Simscape Electrical.

## Disturbance rejection test

A step disturbance (representing a sudden load increase) is injected at the motor's input, between the H-bridge output and the motor transfer function — approximating an added load torque, given the single-input transfer function structure used here. A more rigorous model would inject this directly into the torque balance via a second transfer function input.

**Test setup:** reference speed = 12 (rad/s), disturbance step of +2 introduced at t = 5s, subtracted from the H-bridge's output before entering the motor.

**Result:** the motor settles at the reference (12) from rest, dips briefly when the disturbance hits at t = 5s, and recovers back to 12 as the PID compensates.

## Controller

PID Controller block, Simulink default gains (not manually tuned for this version). Future work: run through the built-in PID Tuner for a faster, less oscillatory response, and compare before/after.

## Files

- `dc_motor_speed_control.slx` — the Simulink model
- `scope_results.png` — scope output showing reference tracking and disturbance rejection

## Next steps

- Tune the PID (auto-tune via Simulink's PID Tuner) and compare response
- Rebuild motor as a two-input transfer function (voltage + load torque) for a more rigorous disturbance injection point
- Revisit with Simscape Electrical once available, to compare behavioral H-bridge vs. switch-level H-bridge results
