# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a knowledge base and future AI agent for Ford Ranger ECU tuning. It contains reference documentation for the HP Tuners tuning platform, covering the 2019–2026+ Ford Ranger 2.3L EcoBoost engine with the 10R80 10-speed automatic transmission. The goal is to build an agent that can assist with tuning decisions, datalog analysis, and calibration guidance.

## Engine Coverage

1. **2.3L EcoBoost (2019–2024)** — Open-deck aluminum I4, Bosch PCM, 10.0:1 CR, GDI only (150 bar), twin-scroll turbo, electronic wastegate. 270 HP / 310 lb-ft. Paired with 10R80 10-speed automatic.
2. **2.3L EcoBoost MPC (2025+)** — Modular Power Cylinder architecture. Completely different engine: 84x102mm bore/stroke, 10.634:1 CR, dual injection (port + 350 bar DI), new turbo. Same power rating but different tuning approach.

## Key Domain Concepts

- **Torque-based PCM model**: The Ford EcoBoost PCM uses a Driver Demanded Torque strategy. The tuning path is: torque limiters → Driver Demand tables → torque model (Torque Inverse Calculator) → load/boost targets → spark → fuel. Raising boost without adjusting the torque model hits torque limiters — the PCM closes the throttle.
- **Speed Density airflow**: No MAF sensor on the Ranger. Aircharge = (Slope x MAP) + Intercept. VVE (Virtual Volumetric Efficiency) is the abstraction for SD coefficient tables.
- **HP Tuners VCM Suite**: VCM Editor (calibration editing, .HPT files) + VCM Scanner (datalogging, .HPL files). MPVI3/MPVI4 hardware interface via OBD-II. Credit-based licensing (4 Universal Credits per Ford PCM, TCM separate).
- **LSPI (Low Speed Pre-Ignition)**: Critical risk on DI turbo engines. API SP oil, colder plugs, avoid heavy throttle below 3,000 RPM.
- **10R80 Transmission**: 10-speed automatic, 590–700 lb-ft capacity. Tunable via HP Tuners (separate TCM license). Shift points, firmness, TC lockup strategy.
- **Boost Control**: PID-controlled WGDC. `WGDC Actual = WGDC Base + P-Term + I-Term`. The WGDC Base table is feed-forward, NOT a direct boost target.

## Directory Structure

- `sources/` — Synthesized reference material (markdown)
- `ford-ranger-tuning/` — Claude Code skill (SKILL.md + references/)

## Sources Directory

| File | Content |
|------|---------|
| `ford-23-ecoboost-engine-specs.md` | Engine specs, all variants, model year differences, cross-platform comparison, known failure points, 10R80 info |
| `ford-23-ecoboost-tuning-guide.md` | Torque-based ECU model, Speed Density airflow, key HP Tuners VCM Editor tables, datalog parameters, boost control PID, tuning strategy |
| `ford-ranger-23-ecoboost-build-path.md` | Stage 1–4 build progression, power targets, mod priority order, fuel system strategy, ethanol considerations, popular tuning providers |
| `hptuners-platform-overview.md` | HP Tuners platform overview: VCM Editor UI/workflow, VCM Scanner datalogging, MPVI3/MPVI4 hardware, credits/licensing system, read/write workflow, compare tools, Ford-specific tools |

## Working with This Repository

- When answering tuning questions, read the relevant engine spec + tuning guide markdown files first for quick context.
- The markdown files are synthesized from HP Tuners documentation, community forums, and manufacturer specs.
- When analyzing datalogs, refer to `ford-23-ecoboost-tuning-guide.md` for the datalog parameter tables and concern thresholds.
- For build planning, `ford-ranger-23-ecoboost-build-path.md` has the stage progression and fuel system strategy.
- For HP Tuners platform questions, refer to `hptuners-platform-overview.md`.
- Always warn about LSPI when discussing low-RPM power increases.
- Always check model year — 2025+ uses a completely different engine (MPC architecture).
