# HP Tuners Ford Ranger 2.3L EcoBoost — Tuner Agent

Knowledge base and future AI agent for Ford Ranger ECU tuning using HP Tuners. Covers the 2019–2026+ Ford Ranger 2.3L EcoBoost engine with the 10R80 10-speed automatic transmission.

## Purpose

Build an agent that can assist with:
- Tuning decisions and stage recommendations
- Datalog analysis (VCM Scanner logs)
- Calibration guidance (VCM Editor tables)
- Build planning and modification order
- LSPI prevention and safety considerations

## Engine Coverage

| Model Year | Engine | PCM | Notes |
|------------|--------|-----|-------|
| 2019–2024 | 2.3L EcoBoost (open-deck) | Bosch | Primary focus. GDI only, 150 bar, 10.0:1 CR |
| 2025+ | 2.3L EcoBoost MPC | Bosch | New architecture: port+DI, 350 bar, 10.634:1 CR |

## Key Domain Concepts

- **Torque-based PCM model**: Ford EcoBoost uses Driver Demanded Torque strategy. Raising boost without adjusting torque limits hits the torque ceiling — the PCM closes the throttle.
- **Speed Density airflow**: No MAF sensor on the Ranger. Aircharge = (Slope x MAP) + Intercept.
- **HP Tuners VCM Suite**: VCM Editor (calibration editing) + VCM Scanner (datalogging). MPVI3/MPVI4 hardware interface. Credit-based licensing (4 credits per Ford PCM).
- **LSPI (Low Speed Pre-Ignition)**: Critical risk on all DI turbo engines. Prevention requires proper oil (API SP), colder plugs, and avoiding heavy throttle below 3,000 RPM.

## Directory Structure

- `sources/` — Synthesized reference material (markdown)
- `ford-ranger-tuning/` — Claude Code skill (SKILL.md + references/)

## Sources

| File | Content |
|------|---------|
| `ford-23-ecoboost-engine-specs.md` | Engine specs, variants, model year differences, known failure points |
| `ford-23-ecoboost-tuning-guide.md` | Torque-based ECU model, key HP Tuners tables, datalog parameters, tuning strategy |
| `ford-ranger-23-ecoboost-build-path.md` | Stage 1–4 build progression, mod priority, fuel system strategy, popular tuners |
| `hptuners-platform-overview.md` | HP Tuners platform: VCM Editor, VCM Scanner, MPVI hardware, credits/licensing |
