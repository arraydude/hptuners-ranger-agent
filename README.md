# HP Tuners Ford Ranger Raptor 3.0L EcoBoost V6 — Tuner Agent

Knowledge base and future AI agent for Ford Ranger Raptor ECU tuning using HP Tuners. Covers the 2024–2026 Ford Ranger Raptor 3.0L EcoBoost V6 twin-turbo with the 10R80 10-speed automatic transmission.

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
| 2024–2026 | 3.0L EcoBoost V6 Twin-Turbo | Bosch MG1CS036 | 405 HP, CGI block, forged internals, parallel twin 39mm BorgWarner turbos |

Shared engine with Bronco Raptor (418 HP), Explorer ST (400 HP), Lincoln Aviator (400 HP). Same tuning principles, different transmission calibrations.

## Key Domain Concepts

- **Torque-based PCM model**: Closed-loop torque control full time. Tuning path: LSPI limits -> torque limiters -> load limits -> TIP ceiling -> Driver Demand -> torque model alignment -> Borderline timing -> fuel targets -> wastegate -> transmission.
- **Speed Density airflow**: MAP-based, no MAF primary. VVE (Virtual Volumetric Efficiency) abstraction.
- **KOM (Knock Octane Modifier)**: Dynamic fuel quality adaptation. KOM = +1 is optimal, -1 = bad fuel or aggressive tune.
- **Boost/Spark Swap**: ECU dynamically trades between boost and spark — normal behavior.
- **HP Tuners VCM Suite (BETA)**: MG1CS036 direct OBDII flash. MPVI3/MPVI4/RTD4 only. 4 credits per PCM.
- **LSPI Protection**: 3 load limit tables blended by KOM. Never disable.
- **CGI Block + Forged Internals**: Robust bottom end, 700+ whp demonstrated on stock internals.

## Directory Structure

- `sources/` — Synthesized reference material (markdown)
- `ford-ranger-tuning/` — Claude Code skill (SKILL.md + references/)

## Sources

| File | Content |
|------|---------|
| `ford-30-ecoboost-engine-specs.md` | Engine specs, CGI block, twin-turbo system, variants, factory anti-lag, known issues, 10R80 |
| `ford-30-ecoboost-tuning-guide.md` | Torque model, HDFX modes, KOM, spark sources, specific table numbers (861, 1775, 3634, 7719, etc.), datalog parameters, tuning methodology |
| `ford-ranger-raptor-30-build-path.md` | Stage 1–4 progression with verified dyno results, Nostrum fuel system, Garrett PowerMax turbos, tuning providers |
| `hptuners-platform-overview.md` | HP Tuners platform: VCM Editor, VCM Scanner, MPVI hardware, credits/licensing |
| `ranger-raptor-3.0-ecoboost-tuning-research.md` | Raw community research compilation |
