---
name: ford-ranger-raptor-tuning
description: Ford Ranger Raptor 3.0L EcoBoost V6 twin-turbo tuning assistant using HP Tuners VCM Suite. Use when the user asks about Ford Ranger Raptor tuning, 3.0L EcoBoost calibration, HP Tuners VCM Editor, datalog analysis, boost/timing/fuel targets, tuning stages, ethanol/flex fuel setup, mod recommendations, or build planning for the 2024-2026 Ford Ranger Raptor 3.0L EcoBoost. Triggers on mentions of Ford Ranger Raptor, 3.0 EcoBoost, 3.0L V6, twin-turbo EcoBoost, HP Tuners, VCM Editor, VCM Scanner, MPVI4, MG1CS036, KOM, TIP, Driver Demanded Torque, Wastegate Position, Speed Density, 10R80 transmission, Garrett PowerMax, Nostrum HPFP, or any Ford Ranger Raptor performance tuning topic.
---

# Ford Ranger Raptor 3.0L EcoBoost V6 Tuning Assistant

## Vehicle Identification

Identify the vehicle and model year before answering:

| Model Year | Engine | PCM | Key Traits |
|------------|--------|-----|------------|
| 2024–2026 (6th Gen NA) | 3.0L EcoBoost V6 Twin-Turbo | Bosch MG1CS036 | 405 HP, CGI block, forged internals, parallel twin 39mm BorgWarner turbos, electronic WG, factory anti-lag in Baja mode |
| 2022–2023 (Global) | 3.0L EcoBoost V6 Twin-Turbo | Bosch MG1CS036 | Same engine, market-dependent power (292–397 HP) |

This engine is shared with the Bronco Raptor, Explorer ST, and Lincoln Aviator. Tuning principles are the same, but transmission calibrations are NOT cross-compatible.

If unclear, ask the user for their model year, market, and current mods.

## Core Tuning Principle

The 3.0L EcoBoost operates under **closed-loop torque control full time**. The tuning order is:

1. **LSPI limits** (raise after 3,500 RPM — fixes popcorn/combustion stability)
2. **Torque limiters** (Table 861 → 650–700 range)
3. **Load limits** (adjust enough to prevent throttle closures)
4. **TIP Desired Max** (lower first to test at reduced boost)
5. **Driver Demand** (conservative: +50 Nm mid, +25 Nm top)
6. **Torque model** (forward/inverse must be mathematically aligned)
7. **Borderline timing** (primary WOT timing tables for pump gas)
8. **Fuel targets** (Desired Lambda / Power Demand table)
9. **Wastegate Position Base** (feed-forward table for smooth boost)
10. **Transmission** (shift schedule, pressures, TCC lockup)

Never advise raising boost without addressing torque/load limits first — the PCM will close the throttle.

## Reference Files

### Engine Specs (load for specs, model year differences, known issues, vehicle identification)
- Read [references/ford-30-ecoboost-engine-specs.md](references/ford-30-ecoboost-engine-specs.md)

### Tuning Guide (load for tuning, calibration, tables, HP Tuners parameters, datalogs, boost control)
- Read [references/ford-30-ecoboost-tuning-guide.md](references/ford-30-ecoboost-tuning-guide.md)

### Build Path (load for stages, mods, upgrade order, power targets, ethanol builds, turbo upgrades)
- Read [references/ford-ranger-raptor-30-build-path.md](references/ford-ranger-raptor-30-build-path.md)

### HP Tuners Platform (load for VCM Editor, VCM Scanner, MPVI hardware, credits, licensing)
- Read [references/hptuners-platform-overview.md](references/hptuners-platform-overview.md)

## Datalog Analysis

### Source PIDs (verify correct operating mode first)

| PID | Expected at WOT | Problem Value | Meaning |
|-----|-----------------|---------------|---------|
| Spark Source | 2 (Borderline) | 5 (Cyl Pressure/LSPI) | Hitting protection tables |
| Fuel Source | 5 (Power Demand) | 0 (Stoichiometric) | WOT enrichment not active |
| Airflow Limit Source | 0 (No Clip) | 2 (WG/Turbo), 5 (LSPI) | Something limiting airflow |
| KOM | +1 | Dropping toward -1 | Tune aggressive or bad fuel |

### Red Flag Thresholds

| Parameter | Concern Threshold | Action |
|-----------|------------------|--------|
| Knock Count | Sustained across pulls | Reduce timing, check fuel, check plugs |
| TIP Actual > TIP Desired Max | Throttle closures | Lower TIP ceiling or reduce boost target |
| Turbo PID I-Term | Beyond +/-5% | Poor wastegate tuning, oscillation |
| Throttle Closures | >20% of max at redline | TIP/boost target too high |
| Fuel Rail Pressure | Drops under load | HPFP limit — upgrade Nostrum |
| LTFT | Beyond +/-10% | Systemic fueling problem |
| KOM | Trending toward -1 | Tune too aggressive or poor fuel quality |

## Key Reminders

- **ECU swaps boost for spark** — this is normal, not a bug
- **Reset adaptives after every flash** — idle, drive normally 10–15 min, let KOM settle
- **Leave spark mostly stock, refine boost** — experienced tuners report best results
- **LSPI tables are critical safety** — never blank out or disable
- **CGI block + forged internals** — robust bottom end, 700+ whp demonstrated on stock internals
- **Stock fuel system bottleneck** — adequate for ~450–500 whp on 93, requires Nostrum HPFP for ethanol

## Platform & Hardware

- **Tuning Platform:** HP Tuners (MPVI4 + VCM Suite BETA)
- **PCM:** Bosch MG1CS036 — direct OBDII flash, no PCM swap required
- **Credits Required:** 4 Universal Credits (PCM); TCM separate
- **Repo scope:** MPVI4 workflows only
- **Software:** VCM Suite (free download, Windows only)
