---
name: ford-ranger-tuning
description: Ford Ranger 2.3L EcoBoost tuning assistant using HP Tuners VCM Suite. Use when the user asks about Ford Ranger tuning, EcoBoost calibration, HP Tuners VCM Editor, datalog analysis, boost/timing/fuel targets, tuning stages, ethanol/flex fuel setup, mod recommendations, or build planning for the 2019-2026+ Ford Ranger 2.3L EcoBoost. Triggers on mentions of Ford Ranger, 2.3 EcoBoost, HP Tuners, VCM Editor, VCM Scanner, MPVI3, WGDC, Driver Demanded Torque, Speed Density, 10R80 transmission tuning, or any Ford Ranger performance tuning topic.
---

# Ford Ranger 2.3L EcoBoost Tuning Assistant

## Vehicle Identification

Identify the vehicle and model year before answering. Use these cues:

| Model Year | Engine | PCM | Block | Key Traits |
|------------|--------|-----|-------|------------|
| 2019–2023 (5th Gen) | 2.3L EcoBoost | Bosch | Open-deck aluminum | 10.0:1 CR, GDI only, 150 bar fuel rail, pre-2020 block prone to head gasket issues |
| 2024 (6th Gen) | 2.3L EcoBoost | Bosch | Open-deck aluminum | Same engine, redesigned chassis, 2.7L V6 also available |
| 2025+ (6th Gen) | 2.3L EcoBoost MPC | Bosch | MPC architecture | NEW engine: 84x102mm, 10.634:1 CR, port+DI (350 bar), different tuning approach |

If the user has a 2025+ Ranger, note that it uses a **completely different engine** (MPC architecture) with dual fuel injection. Tuning guidance for 2019–2024 does NOT apply directly.

If unclear, ask the user for their model year and current mods.

## Core Tuning Principle

The Ford 2.3L EcoBoost uses a **torque-based PCM strategy** with **Speed Density** airflow. The tuning order is:

1. **Torque limiters** (raise the ceiling)
2. **Driver Demanded Torque** (tell the PCM to request more power)
3. **Torque model** (recalculate with Torque Inverse Calculator)
4. **Load / boost targets** (increase airflow demand)
5. **Spark advance** (optimize combustion)
6. **Fueling / AFR targets** (ensure safe mixtures)
7. **Transmission** (shift points, firmness, TC lockup)

Never advise raising boost without addressing torque limiters and Driver Demand first — the PCM will close the throttle to maintain the torque ceiling.

## Reference Files

Load the relevant reference based on the user's question:

### Engine Specs (load when asked about specs, mods compatibility, model year differences, or vehicle identification)
- Read [references/ford-23-ecoboost-engine-specs.md](references/ford-23-ecoboost-engine-specs.md)

### Tuning Guide (load when asked about tuning, calibration, tables, HP Tuners parameters, datalogs, or boost control)
- Read [references/ford-23-ecoboost-tuning-guide.md](references/ford-23-ecoboost-tuning-guide.md)

### Build Path (load when asked about stages, mods, upgrade order, power targets, or ethanol builds)
- Read [references/ford-ranger-23-ecoboost-build-path.md](references/ford-ranger-23-ecoboost-build-path.md)

### HP Tuners Platform (load when asked about HP Tuners features, VCM Editor, VCM Scanner, MPVI hardware, credits, licensing, or read/write workflow)
- Read [references/hptuners-platform-overview.md](references/hptuners-platform-overview.md)

## Datalog Analysis

When reviewing datalogs, check these critical indicators first:

### Source PIDs (verify correct operating mode)

| PID | Expected at WOT | Problem Value | Meaning |
|-----|-----------------|---------------|---------|
| Spark Source | 2 (Borderline) | 5 (Cyl Pressure/LSPI) | Hitting protection tables |
| Fuel Source | 5 (Power Demand) | 0 (Stoichiometric) | WOT enrichment not active |
| Airflow Limit Source | 0 (No Clip) | 2 (WG/Turbo), 5 (LSPI) | Something is limiting airflow |
| OAR | Near -1.0 | Trending toward +1.0 | Tune too aggressive or bad fuel |

### Red Flag Thresholds

| Parameter | Concern Threshold | Action |
|-----------|------------------|--------|
| Knock Count | Sustained across pulls | Reduce timing, check fuel quality, check plugs |
| WGDC Actual | Sustained >90% | Turbo near max — lower targets or upgrade hardware |
| Boost Pressure | >27 PSI on stock turbo | Beyond safe stock turbo limit |
| AFR / Lambda | Leaner than 0.85λ at WOT | Dangerously lean — reduce boost or enrich fueling |
| IAT | >38°C (100°F) under boost | Power limiting begins — intercooler upgrade needed |
| ECT | >115°C | Cooling system issue |
| Fuel Rail Pressure | Drops under load | HPFP can't keep up — upgrade HPFP |
| Fuel Trims | Beyond ±5% | Investigate SD calibration, injectors, or vacuum leaks |

## Stage Recommendations

When asked "what stage should I get" or "what mods do I need":

1. Ask about current mods, fuel availability (87/91/93/E30/E85), and goals
2. Load the build path reference
3. Match to the appropriate stage
4. List required supporting mods (don't let users skip hardware for higher stages)
5. Emphasize that a tune without supporting hardware can cause damage
6. Highlight LSPI prevention for all stages

## LSPI Warning

Low Speed Pre-Ignition is the #1 safety concern on the 2.3L EcoBoost. Always remind users:
- Use API SP / ILSAC GF-6 rated oil
- Avoid heavy throttle below 3,000 RPM
- Downshift before accelerating hard
- Install colder spark plugs when tuned
- Do NOT disable combustion stability limits at low RPM

## Platform & Hardware

- **Tuning Platform:** HP Tuners (MPVI3/MPVI4 + VCM Suite)
- **Credits Required:** 4 Universal Credits per PCM (TCM is separate)
- **Software:** VCM Suite (free download, Windows only)
- **Read/Edit/Save is free** — credits only consumed when flashing
- **Standalone datalogging works without a license**

When recommending HP Tuners setup:
- MPVI3 or MPVI4 hardware + 4 credits minimum
- PROLINK+ cable recommended for wideband AFR logging
- Remote tuning via RTD device if working with a professional tuner
