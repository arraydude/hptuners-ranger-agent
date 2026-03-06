# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a knowledge base and future AI agent for Ford Ranger Raptor ECU tuning. It contains reference documentation for the HP Tuners tuning platform, covering the 2024–2026 Ford Ranger Raptor 3.0L EcoBoost V6 twin-turbo engine with the 10R80 10-speed automatic transmission. The goal is to build an agent that can assist with tuning decisions, datalog analysis, and calibration guidance.

## Engine Coverage

**3.0L EcoBoost V6 Twin-Turbo** — 60-degree V6, Bosch MG1CS036 PCM, Compacted Graphite Iron (CGI) block with forged internals (crank, rods, pistons, cams). Parallel twin BorgWarner 39mm turbochargers with electronic wastegate position control. 405 HP / 430 lb-ft in Ranger Raptor (US). Direct injection only. Factory anti-lag in Baja mode.

Shared engine with Bronco Raptor (418 HP), Explorer ST (400 HP), Lincoln Aviator (400 HP). Same tuning principles apply, but transmission calibrations are NOT cross-compatible.

## Key Domain Concepts

- **Torque-based PCM model**: Closed-loop torque control full time. Signal flow: Driver Demand → Torque Limits → Torque-to-Load (inverse tables) → Load Limits (32+ tables, 32+ inverse sanity checks = 64+ tables) → Load-to-TIP → Wastegate Position. ECU does NOT directly target boost — it targets torque.
- **Boost/Spark Swap**: ECU dynamically trades between boost and spark based on conditions (air density, AFR, etc.) — this is normal behavior.
- **KOM (Knock Octane Modifier)**: Ford's dynamic fuel quality adaptation. KOM = +1 is optimal (good fuel, timing advanced). KOM = -1 = bad fuel or aggressive tune (timing retarded). KAM stored, persists through restarts, takes days to settle after reflash.
- **LSPI Protection**: 3 load limit tables blended by KOM value. KOM +1 = least restrictive, KOM -1 = most restrictive. NEVER disable.
- **Electronic Wastegate**: Commanded position control, NOT traditional vacuum-operated WGDC. Allows very precise control and fast response.
- **HDFX Modes**: 16 cam phasing modes determining which spark/SD/load tables are active. Log "Mapped Points" to identify active mode.
- **Speed Density**: MAP-based airflow (no MAF primary). VVE (Virtual Volumetric Efficiency) abstraction for SD coefficients.
- **HP Tuners VCM Suite (BETA)**: MG1CS036 direct OBDII flash — first-to-market, no PCM swap needed. MPVI3/MPVI4/RTD4 only (MPVI2 NOT supported). 4 Universal Credits per PCM, TCM separate.
- **Four Spark Control Methods**: MBT, Borderline (primary WOT), Cylinder Pressure, Pre-Ignition/LSPI. ECU picks the lowest timing.

## Critical Table Numbers (3.0L EcoBoost)

| Table | Function |
|-------|----------|
| 861 | Maximum Torque — set to 650–700 |
| 1775, 9704 | Wastegate control — adjust incrementally for upper-RPM WG limits |
| 3634 | Desired Canister Pressure Feedforward — modify LAST |
| 7304 | EOI vs Pressure — formula: SOI Minimum - Max Injection Angle |
| 7309 | Max Duty Cycle vs ECT — set to 1.0 |
| 7310 | Max Duty Cycle — set to 1.0 |
| 7719 | Max Injection Angle — stock ~253°, increase to 270° (max 283°) |
| 44553 | Exhaust temp protection — max out |
| 44616 | Cylinder pressure limits |
| 45559 | Turbo Pressure Ratio — be VERY careful, overspeed risk |
| 28941 | High Pressure System Loss vs Turbo Airflow — careful, overspeed risk |

## Directory Structure

- `sources/` — Synthesized reference material (markdown)
- `ford-ranger-tuning/` — Claude Code skill (SKILL.md + references/)

## Sources

| File | Content |
|------|---------|
| `ford-30-ecoboost-engine-specs.md` | Full engine specs, CGI block, twin-turbo system, all application variants, factory anti-lag, known issues (valve fracture recall, cam phaser, wastegate rattle TSBs), 10R80 transmission |
| `ford-30-ecoboost-tuning-guide.md` | Complete tuning guide: torque model with forward/inverse tables, HDFX modes, KOM system, 4 spark control methods, electronic wastegate/TIP control, Power Demand fueling, specific table numbers, datalog parameters with expected values, limiter encounter order, tuning methodology |
| `ford-ranger-raptor-30-build-path.md` | Stage 1–4 with verified dyno results per tuner, Nostrum fuel system stages, Garrett PowerMax turbos, turbo upgrade path, mod priority order, popular tuning providers ranked |
| `hptuners-platform-overview.md` | HP Tuners platform: VCM Editor UI/features, VCM Scanner datalogging, MPVI3/4 hardware, credits/licensing, read/write workflow, Ford-specific tools (Torque Inverse Calculator) |
| `ranger-raptor-3.0-ecoboost-tuning-research.md` | Raw community research with source URLs |

## Working with This Repository

- When answering tuning questions, read the relevant engine spec + tuning guide first.
- For datalog analysis, the tuning guide has specific PID values and thresholds.
- For build planning, the build path has verified dyno results per tuner/stage.
- Always warn about LSPI when discussing low-RPM power increases.
- Always check KOM in datalogs — it's the #1 indicator of tune health.
- The ECU swaps boost for spark — explain this to users who notice varying boost levels.
- Critical table numbers are in the tuning guide and listed above — use them when discussing specific calibration changes.
- Turbo Pressure Ratio (45559) and High Pressure System Loss (28941) tables must be modified with extreme care — aggressive changes cause turbine overspeed (200k+ RPM).
