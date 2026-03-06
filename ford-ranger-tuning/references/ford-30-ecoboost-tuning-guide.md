# Ford 3.0L EcoBoost V6 Tuning Guide — HP Tuners (Ranger Raptor Focus)

This guide covers tuning the 2024–2026 Ford Ranger Raptor 3.0L EcoBoost V6 twin-turbo using HP Tuners VCM Suite. The engine uses a torque-based PCM strategy with Speed Density airflow calculation and electronic wastegate position control.

## PCM Architecture

| Parameter | Value |
|-----------|-------|
| ECU | Bosch MG1CS036 |
| Airflow Model | Speed Density (MAP-based) |
| Control Strategy | Torque-based (closed-loop torque control full time) |
| Throttle | Drive-by-Wire (DBW) |
| Boost Control | Electronic wastegate position control (commanded position, NOT traditional vacuum WGDC) |
| Cam Control | HDFX (16 modes based on VCT cam positions) |
| Fuel Quality Adaptation | KOM (Knock Octane Modifier) — KAM stored |
| Spark Control | 4 methods: MBT, Borderline, Cylinder Pressure, Pre-Ignition |

## Torque-Based ECU Model

The 3.0L EcoBoost operates under **closed-loop torque control full time**. The ECU does NOT directly target boost — it targets engine torque, converts to air load, then controls wastegate position to achieve the required TIP (Throttle Inlet Pressure).

### Signal Flow

```
Driver Requested Torque (pedal → Driver Demand table)
  → Torque Limit / Reduction (all limiters checked)
    → Torque-to-Load conversion (Torque Inverse tables)
      → Load Limit / Reduction (32+ load limit tables, plus 32+ inverse sanity checks = 64+ tables)
        → Load-to-Desired-MAP/TIP
          → Desired TIP to Wastegate Position (electronic WG + PID)
```

### 1. Driver Demand / "Driver's Wish"

When you press the accelerator, the PCM references a **Driver Demand** table (RPM vs. Pedal Position) to determine desired torque in Nm.

- **Pedal Map Ratio** table: Multiplier applied to Driver Demand based on OSS (Output Shaft Speed, effectively per-gear). Creates a 4D lookup — adds the OSS axis that DDT doesn't have. Values over 100% at higher throttle positions yield more demanded torque.
- **Pedal Characteristic** tables: Govern torque delivery sensation in dedicated drive modes.
- Start conservatively: **+50 Nm mid-range, +25 Nm at top-end**
- "Don't get greedy, demanding more than the engine can handle too early in the RPMs" — excessive low-RPM torque requests risk connecting rod failure

### 2. Torque Limiters

The PCM cross-references ALL limiters. If ANY shows a ceiling lower than driver demand, it takes precedence immediately. The two main "offenders" causing throttle closures: **Load limits** and **Boost/TIP limits**.

Three types of limiters commonly adjusted: **RPM, MPH, and Transmission**. The MPH limiter is typically raised to 200 MPH.

### 3. Torque Model — Forward and Inverse

The torque model requires mathematical balance between two sets of tables:

- **Torque-to-Load (Torque Inverse):** X-axis labeled "ETC Torque" — converts torque request to required air load
- **Load-to-Torque (Indicated Torque):** Converts air load back to estimated torque output

These MUST be mathematically inverse of each other. Misalignment causes throttle closures.

**HP Tuners Torque Inverse Calculator:** `Tools > Torque Inverse Calculator`. Calculates the inverse relationship and updates tables at `Engine > Torque Model > General`.

**Extending torque tables for higher power:**
1. Shift the final column upward one position
2. Use the HP Tuners calculator or Excel spreadsheet to extend table values
3. Maintain mathematical inverse relationship

### 4. Boost/Spark Swap

The ECU can and will **swap boost for spark** when it determines it is more suitable:
- Run more spark to combust a slower-burning AFR
- Run less boost when air is cold and dense
- Run more boost when air is hot and less dense

This is normal behavior, not a bug. Understand that boost and timing interact dynamically.

### 5. Fast Path vs. Slow Path

- **Slow path:** Throttle position (takes time for air to travel to cylinder)
- **Fast path:** Spark retard and fuel cut (nearly instant, per-cylinder)
- Factory calibration may close throttle to **less than 20%** for consistent torque delivery

## HDFX Modes (Cam Phasing)

16 HDFX modes (Table 01–15 plus Optimum Power) determined by VCT cam positions.

- Each mode selects which tables are active for: **ignition timing, speed density, load limiting, load translation**
- **Mapped Points** — log this parameter to identify which ECU calculation point is active for ignition, fuel, and VE
- **Optimum Power (OP) mode** is NOT used in Ranger Raptor despite its name
- Changes to cam phasing cascade to all dependent table sets

## Speed Density Airflow Model

The Ranger Raptor uses **MAP-based airflow (Speed Density)**.

### Aircharge Calculation
```
Aircharge = (Slope × MAP) + Intercept
```

### Virtual Volumetric Efficiency (VVE)
- VVE is an abstraction for SD coefficient tables
- HP Tuners VCM Suite has a **Virtual VE Editor** / Speed Density Calculator
- Peak VE at RPM of peak torque, progressively drops on either side

## Spark / Ignition Control

Four primary methods — ECU compares all and selects the **LOWEST timing** at any given HDFX/Load/RPM point:

### 1. MBT (Maximum Brake Torque)
- Maximum torque timing determined on engine dyno with high-octane fuel
- Should NOT be used as reference for pump gas tuning
- Ford designs EcoBoost to run near MBT under heavy load

### 2. Borderline Spark
- Maximum timing on the border of knock threshold
- Tested on 91 octane (95 RON)
- **Most commonly in use at medium to high load — the tables you pay most attention to at WOT**
- Primary WOT timing tables for pump gas

### 3. Cylinder Pressure
- Limits timing to cap cylinder pressure
- Encountered when running **upgraded intercooler** or **high ethanol content** (denser charge air)
- **Table 861:** Maximum torque, set to 650–700 range
- Cylinder pressure limit tables control max ignition advance by RPM vs airload — avoid excessive values

### 4. Pre-Ignition / LSPI
- Protection at low-speed, high-load conditions
- **NEVER disable or blank out these tables**

### Spark Source PID Values

| Value | Meaning | Notes |
|---|---|---|
| 0 | No Clip Applied | Base timing |
| 1 | MBT | Max brake torque limiting |
| 2 | Borderline | Most common at WOT on pump gas |
| 3 | EFT Clip | Ethanol-related limiting |
| 5 | Cylinder Pressure / LSPI | Hitting protection — investigate |

## KOM (Knock Octane Modifier)

Ford's dynamic fuel quality adaptation multiplier (equivalent to OAR on earlier platforms).

| KOM Value | Meaning | Effect |
|-----------|---------|--------|
| **+1** | Good fuel quality | ECU advances timing — **optimal** |
| **0** | Neutral / learning | Baseline |
| **-1** | Poor fuel quality / knock | ECU retards timing — tune too aggressive or bad fuel |

- KOM is **KAM stored** — persists through restarts
- After reflash, KOM resets and takes a **few days of driving** to settle
- KOM compensation table values are multiplied by current KOM and added to Borderline Timing
- KOM = +1: timing advanced; KOM = -1: timing retarded by equal opposite amount
- **LSPI protection uses KOM** to blend between 3 LSPI load limit tables:
  - KOM = +1 → LSPI Load Limit (High) — least restrictive
  - KOM = 0 → LSPI Load Limit (Nominal/Mid)
  - KOM = -1 → LSPI Load Limit (Low) — most restrictive

**Target:** KOM should settle at **+1** on good fuel with a properly calibrated tune.

## Boost Control

### TIP (Throttle Inlet Pressure) Control

The ECU controls TIP (pre-throttle) and MAP (post-throttle) independently:
- The wastegate targets a "desired TIP" usually **higher** than desired MAP
- The throttle acts as a "tap" feeding pressure from inlet to manifold
- **If TIP Actual < TIP Minimum:** Wastegate closes to build pressure
- **If TIP Actual > TIP Maximum:** ECU closes the throttle — this causes **throttle closures**
- **TIP Desired Max (Ceiling):** Maximum allowed TIP. Lowering this is the recommended starting point for testing at lower boost levels

### Wastegate Position Control

The 3.0L uses **electronic wastegate position control** (commanded position, not traditional vacuum WGDC):

- **Wastegate Position Base table (feed-forward):**
  - X-Axis: Turbo Turbine Flow Estimated
  - Y-Axis: Turbo MFRACT Desired / Mass Fraction
  - Z-Data: Wastegate position (%) or Canister Pressure Desired (Relative)
  - Best way to reduce throttle closures
  - Monitor: "(C) WG Canister Pressure Base"

- **Table 3634 (Desired Canister Pressure Feedforward):** Fine-tuning table — modify ONLY after other limiters are resolved and tables are aligned

- **Expected TIP table:** PCM sanity check ensuring boost targets align with turbo capability. Unrealistic values degrade boost response and throttle control.

### Wastegate PID Control

- Well-tuned table should have **I-Term staying within +/-5%**
- Throttle closures should stay within **10% of max at lower RPMs, 20% at redline**

### Tables 1775 and 9704 (Wastegate Control)

- Adjust incrementally upward when hitting wastegate limits at upper RPMs
- Turbo airflow limit is ~55 lbs/min for 2020–2024 models

### Min/Max vs Turbo Airflow

- Raising the Min table achieves more boost up to a point — modest 1–2 PSI increases
- **Underboost table:** Raise to 50–80 seconds during calibration to prevent false underboost faults

### High Pressure Drop & Pressure Differential

- Leave stock unless intercooler piping or throttle body changed
- Modifications cause undesired throttle position effects

## WOT Fueling — Power Demand Strategy

### Power Demand

- Binary threshold determined by pedal position
- **Power Demand Threshold (APP):** Once exceeded, power enrichment begins
- **Fuel Source PID:** 0 = Stoichiometric (cruise), **5 = Power Demand** (WOT enrichment)
- Always verify **Fuel Source = 5** during WOT pulls

### Desired Lambda (Power Demand) Table

Primary WOT fueling table:
- Referenced by ECT and Engine Speed (RPM)
- Values are Lambda targets
- Once warmed up, effectively a 2D RPM-only lookup
- Changing Lambda significantly affects ignition timing — the two are interlinked
- Lambda range 0.77–0.79 achievable on load
- Target 0.80–0.85 for typical WOT power tuning

### Injector Limits

| Table | Stock Value | Recommended | Notes |
|-------|------------|-------------|-------|
| **7719 (Max Injection Angle)** | ~253° | 270° initially, max 283° | Beyond 283° risks knock and post-combustion fuel spray |
| **7310 (Max Duty Cycle)** | <1.0 | 1.0 | Higher only with alcohol fuels |
| **7309 (Max DC vs ECT)** | <1.0 | 1.0 | Match to 7310 |
| **7304 (EOI vs Pressure)** | Stock | SOI Minimum - Max Injection Angle | Formula: e.g., 383 - 270 = 113 |

## Key Tuning Tables (HP Tuners VCM Editor)

### Torque Tables

| Table / Path | Description |
|------------|-------------|
| Engine > Torque Management > Driver Demand | RPM vs Pedal Position → Torque (Nm) — main power table |
| Engine > Torque Management > Pedal Map Ratio | OSS-based multiplier to Driver Demand (per-gear refinement) |
| Engine > Torque Management > Pedal Characteristic | Drive-mode-specific torque delivery feel |
| Engine > Torque Management > Torque Limiters | Maximum torque at various conditions |
| Engine > Torque Model > Torque Inverse (Torque-to-Load) | X-axis = ETC Torque. Must be mathematical inverse of Indicated Torque |
| Engine > Torque Model > Indicated Torque (Load-to-Torque) | Air load → estimated torque. Must match inverse tables |
| **Table 861** (Max Torque) | Set to 650–700 range across all operating modes |
| Max Torque 1, 2, 3 | Additional max torque tables (may have misaligned hysteresis) |

### Load Limit Tables

| Table / Path | Description |
|------------|-------------|
| Engine > Load > Load Limits | 32+ load limit tables in 3.0L EcoBoost |
| Engine > Load > Inverse Torque tables | 32+ sanity-check tables (load → torque, reverse direction) |
| LSPI Load Limit (High) | Most restrictive — used when KOM = -1 |
| LSPI Load Limit (Nominal/Mid) | Baseline — used when KOM = 0 |
| LSPI Load Limit (Low) | Least restrictive — used when KOM = +1 |

**For popcorn/combustion stability limiters:** Raise LSPI limits Nominal & High after 3,500 RPM. Adjusting combustion stability tables alone is ineffective — LSPI modifications are most reliable.

### Boost / Wastegate Tables

| Table / Path | Description |
|------------|-------------|
| TIP Desired Max (Ceiling) | Maximum allowed throttle inlet pressure — start here for testing |
| Wastegate Position Base | Feed-forward: Turbine Flow Est (X) vs Mass Fraction (Y) → WG position % |
| **Table 3634** (Canister Pressure Feedforward) | Fine-tuning desired vs actual TIP — modify last |
| Expected TIP | PCM sanity check — unrealistic values degrade boost response |
| Min/Max vs Turbo Airflow | Marginal boost increases (1–2 PSI via Min table) |
| **Tables 1775, 9704** | Wastegate control parameters — adjust incrementally for upper-RPM WG limits |
| Underboost Timer | Raise to 50–80 seconds during calibration |
| High Pressure Drop | Leave stock unless intercooler piping changed |
| Pressure Differential | Leave stock unless throttle body changed |

### Exhaust Temperature Control

| Table / Path | Description |
|------------|-------------|
| EGT Control Thresholds | Slightly raise with inverse flange adjustments |
| **Table 44553** | Max out to address exhaust temp protection limits |
| Exhaust Inverse Flange | Raise alongside LSPI adjustments to prevent premature richening |

### Spark / Ignition Tables

| Table / Path | Description |
|------------|-------------|
| Engine > Spark > MBT | Max Brake Torque — upper limit, NOT for pump gas reference |
| Engine > Spark > Borderline | Primary WOT timing tables — tune these for pump gas |
| Engine > Spark > Cylinder Pressure | **Table 861** — max torque 650–700 range |
| Engine > Spark > LSPI / Pre-Ignition | Protection — never disable |
| KOM Compensation | Multiplied by current KOM, added to Borderline |

### Fuel Tables

| Table / Path | Description |
|------------|-------------|
| Desired Lambda (Power Demand) | ECT vs RPM → Lambda target at WOT |
| **Table 7719** (Max Injection Angle) | Stock ~253°, increase to 270° (max 283°) |
| **Table 7310** (Max Duty Cycle) | Adjust to 1.0 |
| **Table 7309** (Max DC vs ECT) | Adjust to 1.0 |
| **Table 7304** (EOI vs Pressure) | = SOI Minimum - Max Injection Angle |

### Speed Density Tables

| Table / Path | Description |
|------------|-------------|
| VVE tables | Virtual Volumetric Efficiency (organized by HDFX mode) |
| SD Coefficient tables | Speed Density coefficients |

### Transmission Tables (10R80)

| Table / Path | Description |
|------------|-------------|
| Shift Schedule | When upshifts/downshifts occur — per drive mode |
| Shift Timing | How fast shifts execute |
| Shift Pressures | Clutch apply pressure — increase when adding engine power |
| Torque Converter Lockup | TCC lock schedule — per drive mode |
| RPM Limiters | Engine and output shaft limits |
| Desired Overall Slip | Reduce by ~20% (caution on torque source 7 timers) |

**TCM requires separate HP Tuners licensing (additional credits).**
Up to **3 separate transmission profiles** in the same tune.

## Datalog Parameters (VCM Scanner)

### Critical Channels

| Parameter | Expected | Concern | Action |
|-----------|----------|---------|--------|
| Spark Source | 2 (Borderline) at WOT | 5 (Cyl Pressure/LSPI) | Hitting protection tables |
| Fuel Source | 5 (Power Demand) at WOT | 0 (Stoichiometric) | WOT enrichment not active |
| Airflow Limit Source | 0 (No Clip) | 2 (WG/Turbo), 5 (LSPI) | Something limiting airflow |
| KOM | +1 | Dropping toward -1 | Bad fuel or aggressive tune |
| Knock Count (per cylinder) | 0 during a pull | Sustained events | Reduce timing, check fuel/plugs |
| Boost / TIP Actual | Matches TIP Desired | Large divergence | Wastegate control issue |
| Throttle Position | Near 100% at WOT | Closures during WOT | TIP Actual > TIP Desired Max |
| Turbo PID I-Term | Within +/-5% | Beyond +/-5% | Poor WG tuning, oscillation |
| Fuel Rail Pressure | Matches commanded | Drops under load | HPFP limit reached |
| STFT | +/-5% at idle | Beyond +/-5% | Fueling accuracy issue |
| LTFT | Within +/-10% | Beyond +/-10% | Systemic fueling problem |
| IAT | Cool under boost | Heat soak | Intercooler upgrade needed |
| ECT | Stable | Overheating | Thermal management issue |
| Lambda / AFR | Matches commanded | Lean at WOT | Dangerous — reduce boost or enrich |

### Essential Logging Channels

| Channel | Purpose |
|---------|---------|
| Fuel Rail Pressure Actual vs Desired | Detect HPFP limitation |
| Start/End of DI Injection Angle | Verify injector timing |
| ETC Torque | Actual torque at electronic throttle |
| Desired Torque | PCM's torque target |
| Scheduled Torque | Torque after limiter checks |
| Engine Brake Torque | Actual engine output |
| Turbo Airflow Desired vs Actual | Verify turbo is achieving targets |
| TIP Desired vs Actual | Boost target vs achieved |
| Mapped Points | Identifies which HDFX calculation point is active |
| WG Canister Pressure Base | Feed-forward wastegate command |
| Throttle Position | Detect closures |

**Recommended datalog window: 2,000–6,500 RPM** to capture full sweep.

### Custom Math Parameters

- **TIP Error** = TIP Desired - TIP Actual
- **Torque Error** = Desired Torque - Engine Brake Torque
- **Fuel Pressure Error** = Fuel Rail Desired - Fuel Rail Actual

## Primary Limiters (In Order of Encounter)

When tuning the 3.0L EcoBoost, you will hit these limiters in roughly this order:

### 1. Popcorn / Combustion Stability
- **Solution:** Raise LSPI limits Nominal & High after 3,500 RPM
- Raising combustion stability tables alone is ineffective
- LSPI table modifications are most reliable

### 2. Wastegate Limit
- At elevation or with increased targets, turbo airflow reaches limits (~55 lbs/min)
- **Solution:** Adjust tables 1775 and 9704 incrementally upward
- Table 3634 (canister pressure feedforward) should be modified LAST

### 3. Exhaust Temperature Protection
- **Solution:** Slightly raise EGT control thresholds with inverse flange adjustments
- Table 44553: max out to address exhaust temp protection
- Maintain protection on stock catalytic converters

### 4. Injector Limits
- **Solution:** Increase Max Injection Angle (table 7719) from ~253° toward 270°
- Adjust Max Duty Cycle (tables 7310, 7309) to 1.0
- Recalculate EOI vs Pressure (table 7304): SOI Minimum - Max Injection Angle

## Torque Model Adjustment Methodology

### For Elevation Tuning
Higher elevations produce greater airload per boost PSI:
1. Lower Indicated Torque values 2–4% from 1.6+ airload, 3,500 RPM and up
2. Run HP Tuners calculator to adjust inverse table X-axis
3. For full-throttle operation, lower inverse X-axis (ETC Torque values 369–590 range)

### Two Approaches
1. **Purist method:** Keep inverse X-axis stock during calculator use — allows throttle modulation
2. **Open throttle method:** Adjust inverse X-axis down after calculator — forces full throttle

### Key Principle
> "The whole goal is to get every parameter the PCM uses to calculate torque, and all inferred parameters vs actual, to track close together."

## Tuning Strategy Summary

### Order of Operations

1. **Raise LSPI limits** (after 3,500 RPM) — address popcorn/combustion stability
2. **Raise torque limiters** — lift the ceiling (Table 861 → 650–700 range)
3. **Raise load limits** — adjust just enough in needed areas
4. **Lower TIP Desired Max** to test at reduced boost first
5. **Adjust Driver Demand** — conservative increases (+50 Nm mid, +25 Nm top)
6. **Calibrate torque model** — ensure forward/inverse tables are mathematically aligned
7. **Adjust Borderline timing** — primary WOT tables for pump gas
8. **Set fuel targets** — Desired Lambda (Power Demand) table
9. **Tune wastegate** — Wastegate Position Base table for smooth boost control
10. **Verify PID behavior** — I-Term within +/-5%, throttle closures within 10–20%
11. **Address injector limits** if hitting fuel delivery ceiling
12. **Datalog and iterate** — KOM, knock, spark source, TIP, fuel pressure, throttle position
13. **Tune transmission** — shift schedule, pressures, TCC lockup

### Critical Rules

- **Never just raise boost without raising torque model** — ECU will fight with throttle closures
- **Retain protection strategies** — disabling them risks engine damage and poor driveability
- **Datalog before and after every change** — verify KOM, knock, fuel trims, wastegate behavior
- **Test at lower boost first** — assess calibration without dangerous conditions
- **ECU will swap boost for spark** — this is normal, not a bug
- **E85 needs 30% more fuel** — don't run E85 without HPFP + injector upgrades
- **Reset adaptives after every flash** — idle, drive normally 10–15 minutes, let KOM settle
- **Leave spark mostly stock and refine boost** — experienced tuners report best results this way

### Fuel Type Considerations

| Fuel | Power Potential | Fuel System Mods? | Notes |
|------|----------------|-------------------|-------|
| 87 octane | Stock or mild | No | Factory calibration |
| 91 octane | Conservative tune | No | Mild power gains |
| 93 octane | +60–100 whp | No | Standard for aftermarket tunes |
| E30–E50 | +125–150 whp | HPFP required at minimum | Best performance/fuel system balance |
| E85 | Maximum gains | **HPFP + injectors required** | Stock fuel system cannot support E85 |
