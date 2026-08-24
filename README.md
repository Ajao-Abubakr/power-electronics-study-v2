# Power Electronics Study

Documented progress through a self-directed 24-month roadmap in Power Electronics, built alongside a Year 4 Electrical and Computer Engineering degree at UNILAG (graduating 2027).

## Background

I'm working toward roles in power electronics at companies like Texas Instruments, Infineon, ABB, Siemens, and Analog Devices, with postgraduate research (MSc/PhD) as a longer-term goal. Alongside this roadmap, I'm completing an internship at CCETC Power Plant in Lagos, working with diesel-driven synchronous alternators, AVR and excitation systems, three-phase distribution, and protection relays.

## Roadmap Structure

**Months 1-6: Foundations**
Circuit fundamentals, converter design (buck, boost, flyback), and PCB design for power electronics (layout, grounding, thermal management, EMI reduction in KiCad).

**Months 7-19: Domain Applications**
Solar/renewable energy, EV and motor drives, SMPS and industrial power. Solar was chosen first because it builds directly on the boost and isolated converter topologies from the foundation phase, introduces source-side physics, and offers one-directional power flow as an accessible entry point to systems-level thinking.

**Months 20-24: Advanced/Research Topics**
Deeper theoretical and research-oriented work building on the domain applications phase.

## Repo Structure

- `01-foundations/` — circuits, converter design, PCB design fundamentals
- `02-solar-pv/` — MPPT algorithms, boost converter design for PV, grid-tied inverter design, LTspice simulations
- `03-advanced-solar/` — three-phase SVPWM systems, battery storage integration, MATLAB/Simulink models
- `04-flyback-build/` — real 30W offline flyback converter (UC3842), schematic through KiCad PCB layout and bring-up
- `resources/` — roadmap notes and reference material

## Tools

LTspice, MATLAB/Simulink, PVWatts, KiCad (two-layer PCB design)

## Status

Currently on Month 9 (EV/Motor Drives), following completion of PCB design fundamentals and a two-month deep dive into solar PV systems (PV physics through MW-scale utility design, economics, and grid integration).
