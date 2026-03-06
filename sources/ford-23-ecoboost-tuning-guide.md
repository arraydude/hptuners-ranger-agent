# Ford 2.3L EcoBoost Tuning Guide — HP Tuners (Ranger Focus)

This guide covers tuning the 2019–2024 Ford Ranger 2.3L EcoBoost using HP Tuners VCM Suite. The engine uses a torque-based PCM strategy with Speed Density airflow calculation.

## PCM Architecture

| Parameter | Value |
|-----------|-------|
| ECU | Bosch PCM |
| Airflow Model | Speed Density (MAP-based, no MAF sensor) |
| Control Strategy | Torque-based (Driver Demanded Torque) |
| Throttle | Drive-by-Wire (DBW) |
| Boost Control | PID-controlled WGDC via electronic wastegate |
| Cam Control | HDFX (High Degree of Freedom Executive) — 16 modes |
| Fuel Quality Adaptation | OAR (Octane Adjust Ratio) — KAM stored |

## Torque-Based ECU Model

The Ford EcoBoost PCM uses a **torque-demand model**. The ECU does NOT directly target boost — it targets torque, converts to a required air load, determines the manifold pressure needed, then controls the wastegate to achieve that pressure.

### Signal Flow

```
Driver Requested Torque (pedal → DDT table)
  → Torque Limit / Reduction (all limiters checked)
    → Torque-to-Load conversion (torque model)
      → Load Limit / Reduction (up to 32 load limit tables)
        → Load-to-Desired-MAP/TIP
          → Desired TIP to WGDC (wastegate PID)
```

### 1. Driver Demanded Torque (DDT)

When the accelerator pedal moves, the PCM references a **Driver Demand** table (RPM vs. pedal position) to determine desired torque in Newton Meters. This is the central table controlling power output.

- **ETC Driver Demand tables:** Engine RPM vs. Pedal Position → Torque Demand
- Two variants: Driver Demand Wheel (output shaft speed) and Driver Demand Engine (engine RPM)
- **Pedal Map Ratio** table is a multiplier to DDT — values over 100% at higher throttle positions yield more demanded torque
- Stock DDT is ~450 Nm — increase to desired target for more power
- Raising DDT alone is NOT enough — load and boost limit tables must also be raised

### 2. Torque Limiters Check

The PCM cross-references ALL limiters before allowing demanded torque. If ANY limiter shows a ceiling lower than driver demand, it takes precedence immediately:

| Limiter | Purpose |
|---------|---------|
| Gear ratio limiters | Different torque limits per gear for traction |
| Gearset limiters | Protect the transmission |
| Steering angle limits | Full power only within a few degrees of center |
| Combustion stability / LSPI limits | Control LSPI by limiting load at low RPM |
| Exhaust Gas Temperature | Protect turbo and exhaust components |
| Throttle Inlet Pressure | TIP pressure ceiling |
| Manifold Temperature | Charge air temperature limits |
| Knock limits | Pull timing on detonation detection |
| Fuel Enrichment limits | Fueling safety boundaries |
| Ambient Temp & Barometric Pressure | Environmental derating |
| RPM limiter | Redline protection |
| MPH limiter | Speed limiter (typically raised to 200 MPH) |
| Transmission limiters | Gearbox torque ceiling |

The two main "offenders" causing throttle closures on tuned vehicles: **Load limits** and **Boost limits**. Adjust these higher to unlock full throttle control.

### 3. Torque-to-Air Load Model

The approved torque is converted through a model determining how much air load is required to produce that crankshaft torque, then converted to air mass requirements.

**Torque Inverse Calculator:** HP Tuners built-in tool at `Tools > Torque Inverse Calculator`. Enter torque data → "Calculate Inverse" → updates the two Torque Calculation tables at `Engine > Torque Model > General`. Running on stock values "smooths out" to an ideal model.

### 4. Fast Path vs. Slow Path Torque Control

- **Slow path:** Throttle position (takes time for air to travel to cylinder)
- **Fast path:** Spark retard and fuel cut (nearly instant, per-cylinder)
- Factory calibration may close throttle to **less than 20%** (of the 82% max opening) for consistent torque delivery — this is why tune-only gains are significant.

**Critical implication:** Raising boost without adjusting the torque model hits torque limiters — the PCM will close the throttle to maintain the torque ceiling. You MUST raise torque limits first.

## HDFX Modes (Cam Phasing)

HDFX (High Degree of Freedom Executive) is the cam phasing mode system. There are **16 different HDFX modes** (Table 01–15 plus Optimum Power).

### How HDFX Works

- HDFX mode is determined by the relative positions of intake and exhaust cams (VCT position)
- Each HDFX mode selects which tables are active for: **ignition timing, speed density, load limiting, and load translation**
- **Mapped Points (Exhaust)/(Intake) tables** define the translation between exhaust cam and intake cam position
- **HDFX Weight Tables 01–15** represent proportional weights of all modes
- **Optimum Power (OP) mode** is NOT used in Ranger or most other supported EcoBoost vehicles despite its name

### Tuning Implications

- When modifying spark or SD tables, you must account for which HDFX mode is active at your target RPM/load point
- Changes to cam phasing tables cascade to ignition timing, speed density, load limiting, and load translation
- VCT Manual Mode can be enabled (e.g., -18° intake, 22° exhaust) but can cause excessive heat or stress at high boost
- Only slight gains available from fine-tuning cam phasing per OEM calibration development experience — focus on spark/boost/fuel first

## Speed Density Airflow Model

The 2019+ Ranger uses **MAP-based airflow (Speed Density)** with no MAF sensor. While some EcoBoost vehicles have a MAF sensor, it is not used for primary airflow calculations.

### Aircharge Calculation

```
Aircharge = (Slope × MAP) + Intercept
```

- **Slope** = MAP vs Air Charge tables (how aircharge scales with manifold pressure)
- **Intercept** = Zero Charge tables (aircharge at zero MAP)
- **MAP** = Manifold Absolute Pressure (measured)

### Virtual Volumetric Efficiency (VVE)

- VVE is an abstraction for SD coefficient tables — different from traditional VE tables
- HP Tuners VCM Suite has a **Virtual VE Editor** for calibrating SD coefficient tables
- Peak VE occurs at RPM of peak torque, progressively drops on either side
- VE increases as MAP increases
- **VE Compensation (MCT Pivot) table:** Critical for SD aircharge calculation, uses MAP, RPM, VE, displacement, and cylinder charge temperature
- Speed density uses inlet air temperature and manifold pressure via MAP sensor to calculate air density

## Spark / Ignition Control

The Ford EcoBoost has an extremely complex timing strategy with **four primary spark control methods**. The ECU compares the output of each and selects the **LOWEST timing** at any given HDFX/Load/RPM point.

### Four Spark Control Methods

#### 1. MBT (Minimum timing for Best Torque)
- Tables determined through lab testing for maximum brake torque at a given load/RPM
- Achieved on high-octane fuel
- Ford designs EcoBoost engines to run near MBT rather than detonation-limited under heavy load

#### 2. Borderline Spark
- Maximum timing to remain on the border of the knock threshold
- Result of extensive testing on engine dyno using standard pump 91 octane (95 RON)
- **Most commonly in use under medium to high load**
- **Pay most attention to these when tuning WOT ignition** — this is the primary WOT timing table for pump gas

#### 3. Cylinder Pressure
- Maximum timing to limit cylinder pressure
- You may encounter Spark Source 5 (Cylinder Pressure) when running an **upgraded intercooler** or **high ethanol content fuels** due to denser charge air

#### 4. Pre-Ignition / LSPI
- Protection tables that limit timing at low-speed, high-load conditions
- Should NEVER be disabled or blanked out

### Spark Source PID Values

Monitor the **Spark Source** PID in datalogs to identify which ignition table family is active:

| Spark Source Value | Meaning | Notes |
|---|---|---|
| 0 | No Clip Applied | Base timing, no limiting active |
| 1 | MBT | Maximum brake torque limiting |
| 2 | Borderline | Most common at WOT on pump gas |
| 3 | EFT Clip | Ethanol-related limiting |
| 5 | Cylinder Pressure / LSPI | Hitting protection tables — investigate |

### Ethanol and Timing

- With full E85, timing can reach **+20° advance at WOT** vs 0 to -5° on 91 octane gasoline
- Ethanol tunes should add **more timing than boost** — this is where ethanol's value lies

## OAR (Octane Adjust Ratio)

OAR is Ford's **dynamic fuel quality adaptation multiplier**. It functions like a "fuel trim" but for ignition and load/boost targets.

### How OAR Works

| OAR Value | Meaning | Effect |
|-----------|---------|--------|
| Trending toward **-1.0** | Good fuel quality detected | ECU adds timing and boost — more performance |
| Near **0** | Neutral / learning | ECU at baseline calibration |
| Trending toward **+1.0** | Poor fuel quality / knock detected | ECU removes timing and reduces boost — less performance |

- OAR range: -1.0 to +1.0
- OAR is a **KAM (Keep Alive Memory)** stored value — persists through restarts
- After ECU reflash, OAR resets and takes a **few days of driving** to fully settle
- A timing table is multiplied by OAR and added to overall ignition timing:
  - OAR of -1 adds approximately **+4°** of timing near redline
  - OAR of +1 removes approximately **4°** of timing near redline

### OAR and LSPI Protection

LSPI protection uses OAR to blend between 3 LSPI load limit tables:

| OAR Value | LSPI Table Used |
|-----------|-----------------|
| +1 | LSPI Load Limit (High) — most restrictive |
| 0 | LSPI Load Limit (Nominal/Mid) |
| -1 | LSPI Load Limit (Low) — least restrictive |

Intermediate OAR values interpolate between tables.

### HP Tuners PID Names for OAR

May appear as: `Inferred Octane`, `Octane_Ratio`, or `OAR`.

**Target:** OAR should settle close to **-1.0** on good fuel with a properly calibrated tune.

## Boost Control

### TIP (Throttle Inlet Pressure) vs. MAP

The ECU controls **TIP (pre-throttle pressure)** and **MAP (post-throttle pressure)** independently:

- The wastegate targets a "desired TIP" that is usually **higher** than desired MAP
- The throttle acts as a "tap" feeding pressure from the inlet to the manifold
- **If TIP Actual falls below TIP Minimum:** Wastegate PID adds WGDC to bring pressure up
- **If TIP Actual goes above TIP Maximum:** ECU closes the throttle to control boost — this is what you feel as **throttle closures**

### Wastegate Duty Cycle (WGDC) System

**WGDC Base** is the primary wastegate table:
- X-Axis: Airflow Mass
- Y-Axis: WGDC Y-Factor
- Z-Data: Direct wastegate duty cycle percentage (%)

These are the base WGDC % before the wastegate PID control becomes active. Adjusting the main WGDC table is mostly used for large boost targeting corrections.

### Wastegate PID Control

```
WGDC Actual = WGDC Base + WGDC P-Term + WGDC I-Term
```

- **PID I-Term (Max):** Primary table for positive WGDC corrections. Referenced by Airflow Mass. Z-Data = maximum WGDC % that can be added when boost target is not achieved.
- The WGDC Base table provides the feed-forward value — it is NOT a direct boost target. Do not make large/incorrect adjustments.

### Canister Pressure Control (Newer Strategies)

Newer EcoBoost strategies use a canister pressure feed-forward PID system:

- **WG Canister Pressure Desired** table as a base:
  - X-Axis: Exhaust Mass Flow Estimated
  - Y-Axis: Exhaust Flow Fraction
  - Z-Data: Wastegate Canister Pressure Desired (Relative)

### Safe Boost Pressure Ranges (Stock Turbo)

| Boost | Assessment |
|-------|------------|
| 18 PSI | Safe (stock range) |
| 20–21 PSI | Safe |
| 22–23 PSI | Stage 1 tune territory, safe with proper calibration |
| 24–25 PSI | Approaching limits, requires careful calibration |
| 25+ PSI | Turbo operating in inefficient region — no additional power, potential performance loss |

Running more than 2–3 PSI above stock in later models can result in failed turbocharger assemblies.

## WOT Fueling — Power Demand Strategy

Ford EcoBoost uses a feature called **Power Demand** to regulate WOT fueling, cam control, and load limiting.

### How Power Demand Works

- Power Demand is a **binary threshold** determined by pedal position
- Set by **Power Demand Threshold (APP)** — once accelerator pedal position exceeds this value, power enrichment begins
- Factory fueling is slightly rich with moderate delay timers before allowing additional fuel at WOT

### Fuel Source PID Values

| Value | Meaning | Notes |
|-------|---------|-------|
| 0 | Stoichiometric | Closed-loop, cruise/part-throttle |
| 5 | Power Demand | WOT enrichment — this is what you want at WOT |

**Always verify Fuel Source = 5** during WOT pulls in datalogs.

### Desired Fuel Target (Power Demand) Table

The primary means to adjust WOT fueling:

- Referenced by ECT (Engine Coolant Temperature) and Engine Speed (RPM)
- Values are **Lambda targets**
- Once engine is warmed up, it effectively becomes a 2D table (RPM-only lookup)
- **Changing Desired Lambda has a significant effect on ignition timing** — the two are interlinked

### Lambda / AFR Targets

| Lambda | AFR (Gasoline) | Use Case |
|--------|---------------|----------|
| 1.00 | 14.7:1 | Stoichiometric (cruise) |
| 0.85 | 12.5:1 | Lean max-power range for forced induction |
| 0.80–0.85 | 11.76–12.5:1 | Typical WOT target range |
| 0.78–0.80 | 11.5–11.76:1 | Rich protection during high-demand conditions |

### Fuel System Hardware Limits

| Configuration | Max Rail Pressure | Injector Flow | Ethanol Limit |
|---|---|---|---|
| Stock | 200 bar (2,900 PSI) | ~1,100–1,300 cc/min | ~E30 |
| Nostrum Std Bore+ HPFP | 250 bar (3,626 PSI) | Stock injectors | ~E50 |
| Nostrum Big Bore HPFP + HF Injectors | 250 bar | 19.7 g/s at 100 bar (+40%) | E85 |
| Xtreme-DI XDI-HPFP35 | 200 bar | Stock or upgraded | E50–E85 |

## Key Tuning Tables (HP Tuners VCM Editor)

### Torque Tables

| Table Path | Description |
|------------|-------------|
| Engine > Torque Management > Torque Reduction | Enable/disable torque reduction strategies |
| Engine > Torque Management > Torque Intervention | Checks between throttle and airflow load calculations |
| Engine > Torque Management > Fuel Cut Torque Ratio vs Requestor | Torque ratio threshold for fuel cut by requestor |
| Engine > Torque Management > Spark Torque Ratio Limit vs Requestor | Spark-based torque limiting by requestor |
| Engine > Torque Model > General > Torque Calculation | Core torque model (use Torque Inverse Calculator) |
| Engine > Torque Model > Indicated Engine Torque | RPM vs Load → indicated torque |
| Engine > Torque Model > Engine Friction Torque | RPM vs ECT → friction losses |

### Airflow / Throttle Tables

| Table Path | Description |
|------------|-------------|
| Engine > Airflow > Electronic Throttle > ETC Driver Demand | RPM vs Pedal Position → torque demand (main power table) |
| Engine > Airflow > Electronic Throttle > ETC Throttle Angle Gain | PID gains (Proportional, Integral, Derivative) |
| Engine > Airflow > Electronic Throttle > ETC Throttle Angle Max/Min | Throttle opening limits |
| Engine > Airflow > General > Speed Density | RPM vs MAP → airmass per cylinder (organized by HDFX mode) |
| Engine > Airflow > General > Charge Temp Compensation | Temperature-based aircharge correction |

### Boost / Turbo Tables

| Table Path | Description |
|------------|-------------|
| Engine > Turbo/Boost > Desired Boost Pressure | Target boost by RPM/load |
| Engine > Turbo/Boost > WGDC Base | Airflow Mass (X) vs WGDC Y-Factor (Y) → duty cycle % (feed-forward) |
| Engine > Turbo/Boost > WGDC PID I-Term (Max) | Maximum positive WGDC correction by airflow mass |
| Engine > Turbo/Boost > WG Actuator Canister Pressure | Canister pressure (X) vs Compressor Outlet Pressure Relative (Y) → WGDC % |
| Engine > Turbo/Boost > WG Canister Pressure Desired | Newer strategies: exhaust mass flow vs exhaust flow fraction → desired canister pressure |
| Engine > Turbo/Boost > TIP Desired Min / Max | Throttle Inlet Pressure bounds — controls when throttle closures occur |
| Engine > Turbo/Boost > Turbo Boost Mult vs ECT | Boost multiplier by engine coolant temperature |
| Engine > Turbo/Boost > Turbo Boost Mult vs Baro | Boost multiplier by barometric pressure (altitude) |
| Engine > Turbo/Boost > Turbo Boost Mult vs TPS | Boost multiplier by throttle position |

### Spark / Ignition Tables

| Table Path | Description |
|------------|-------------|
| Engine > Spark > MBT | Maximum Brake Torque — fully populated spark reference, upper limit and torque reference |
| Engine > Spark > Borderline | Knock boundary tables — **primary WOT timing tables for pump gas** |
| Engine > Spark > Cylinder Pressure | Max timing to limit cylinder pressure — encountered with intercooler upgrades or ethanol |
| Engine > Spark > LSPI / Pre-Ignition | Protection tables at low-speed high-load — **never disable** |
| Engine > Spark > Spark Advance | RPM vs Load → degrees of advance (organized by HDFX mode) |

**Virtual Torque Window:** Series of tables corresponding to amounts of spark advance/retard. X-axis: RPM, Y-axis: cylinder airmass (mg) or MAP (kPa). ECM uses the table closest to current spark advance.

### Fuel Tables

| Table Path | Description |
|------------|-------------|
| Engine > Fuel > Desired Fuel Target (Power Demand) | ECT vs RPM → Lambda target at WOT (primary WOT fueling table) |
| Engine > Fuel > Lambda/AFR targets | Commanded lambda at various RPM/load points |
| Fuel System > Desired Injector Pressure Drop vs Mass | Fuel rail pressure target by injection mass |
| Fuel System > Fuel Pump Voltage vs Flow vs Pressure | LPFP voltage calibration |
| Fuel System > Injector Pressure Drop Adder vs Ambient Temp | Temperature compensation |

### Load Limit Tables

| Table Path | Description |
|------------|-------------|
| Engine > Load > Load Limits | Up to 32 load limit tables in newer Ford ECUs |
| Engine > Load > LSPI Load Limit (High) | Most restrictive — used when OAR = +1 |
| Engine > Load > LSPI Load Limit (Nominal/Mid) | Baseline — used when OAR = 0 |
| Engine > Load > LSPI Load Limit (Low) | Least restrictive — used when OAR = -1 |

### VCT / Cam Tables

| Table Path | Description |
|------------|-------------|
| Engine > VCT > Mapped Points (Intake) | Intake cam position mapping for HDFX modes |
| Engine > VCT > Mapped Points (Exhaust) | Exhaust cam position mapping for HDFX modes |
| Engine > VCT > VCT Targets | Target cam positions by RPM/load |
| Engine > VCT > Manual Mode | Override cam positions (use with caution at high boost) |

### Transmission Tables (10R80)

| Table Path | Description |
|------------|-------------|
| Transmission > Torque Management | Transmission-side torque limits |
| Transmission > Shift Points | Upshift/downshift RPM by gear |
| Transmission > Shift Pressure / Firmness | Clutch apply pressure and shift speed |
| Transmission > Shift Time | Duration from shift initiation to completion |
| Transmission > Line Pressure | Clutch apply pressure schedules (stock idle: ~65 PSI, WOT: ~350 PSI) |
| Transmission > Torque Converter Lockup | TCC engagement strategy — tune to lock up sooner |
| Transmission > Skip Shift Logic | Stock 10-speed skips gears at part-throttle — can be modified |

**TCM tuning requires separate HP Tuners licensing (additional credits).**

## Datalog Parameters (VCM Scanner)

### Critical Channels to Monitor

| Parameter | Normal Range | Concern Threshold | Action |
|-----------|-------------|-------------------|--------|
| Knock Count (per cylinder) | 0 during a pull | 1–3 across all cylinders not uncommon; sustained = problem | Reduce timing, check fuel quality, check plugs |
| Spark Source | 2 (Borderline) at WOT | 5 (Cylinder Pressure/LSPI) | Hitting protection tables — investigate |
| Fuel Source | 5 (Power Demand) at WOT | 0 (Stoichiometric) at WOT | WOT enrichment not active — check Power Demand threshold |
| OAR (Octane Adjust Ratio) | Near -1.0 | Trending toward +1.0 | Tune too aggressive or poor fuel quality |
| Boost Pressure Actual | 17–22 PSI (stock) | >27 PSI on stock turbo | Beyond safe stock turbo limit |
| TIP Desired vs TIP Actual | Should track closely | Large divergence | Wastegate control issue |
| Load Actual vs Load Desired | Should match | Load clipping (actual < desired) | Hit a load limit — identify which one |
| Airflow Limit Source | 0 (No Clip) | 2 (WG/Turbo Clip), 5 (LSPI), 6 (Partial Throttle) | Tells you what is currently limiting airflow |
| WGDC Actual | 20–30% at WOT | Sustained >90% | Turbo near max — lower targets or upgrade hardware |
| AFR / Lambda | 0.78–0.85λ at WOT | Leaner than 0.85λ at WOT | Dangerously lean — reduce boost or enrich fueling |
| IAT (Intake Air Temp) | <38°C (100°F) under boost | >38°C: power limiting begins; >82°C: significant | Intercooler inadequate — heat soak |
| ECT (Engine Coolant Temp) | 90–105°C | >115°C | Cooling system issue or excessive load |
| Fuel Rail Pressure | Matches commanded | Drops under load | HPFP can't keep up — upgrade HPFP |
| Fuel Trims (LTFT/STFT) | ±3% | Beyond ±5% | Investigate SD calibration, injectors, or vacuum leaks |
| APP (Accelerator Pedal Position) | 100% during WOT pulls | <100% | Confirm full pedal input |

### Boost Control Monitoring Channels

| Channel | Purpose |
|---------|---------|
| Boost Pressure Actual | Measured boost (use 2-bar PID for higher boost levels) |
| Boost Pressure Desired | Target boost from calibration |
| TIP Desired / TIP Actual | Pre-throttle pressure target vs actual |
| WGDC Actual | Wastegate duty cycle output |
| WGDC Base | Feed-forward WGDC value |
| WGDC P-Term | Proportional correction |
| WGDC I-Term | Integral correction |
| WG Canister Pressure | Wastegate actuator pressure |
| WGDC Y-COP Relative | Compressor outlet pressure relative |
| WGDC Y-Factor | Y-axis factor for WGDC base table |
| Exhaust Mass Flow Est. | Estimated exhaust gas flow |

### Custom Math Parameters (VCM Scanner)

Useful derived channels to set up:
- **Boost Error** = Desired Boost - Actual Boost (positive = under-boosting)
- **TIP Error** = TIP Desired - TIP Actual
- **Knock Sum** = Total knock events per log session
- **AFR Error** = Commanded Lambda - Actual Lambda

### Red Flags in Datalogs

- **Excessive knock retard** — more than occasional small corrections is concerning
- **OAR trending toward +1.0** — tune too aggressive or poor fuel quality
- **Spark Source jumping to 5** — hitting Cylinder Pressure/LSPI protection tables
- **Airflow Limit Source showing 2 or 5** — hitting turbo/wastegate clip or LSPI limits
- **Boost instability** — large oscillations between desired and actual
- **IAT above 100°F (38°C)** — heat soak, need better intercooler
- **COT above 350°F (177°C)** — ECU starts protecting turbocharger
- **AFR leaner than ~12.5:1 at WOT** — dangerous for forced induction
- **Misfires during WOT** — check spark plug gap first, then coils (23% misfire rate at low RPM or 5% at high RPM triggers fault)
- **Fuel pressure dropping** at high RPM/boost — HPFP or injector limit reached
- **Fuel Source = 0 at WOT** — Power Demand enrichment not active

## Thermal Limits and Safety Thresholds

| Parameter | Threshold | ECU Response |
|-----------|-----------|-------------|
| IAT > 100°F (38°C) | Power limiting begins | Gradual timing/boost reduction |
| IAT > 180°F (82°C) | Significant power limit | Major derating |
| COT > 350°F (177°C) | Turbo protection | ECU limits power to protect turbocharger |
| Misfires > 23% at low RPM | DTC triggered | 46 misfires per 200 revolutions |
| Misfires > 5% at high RPM | DTC triggered | Threshold drops at high RPM |

## LSPI (Low Speed Pre-Ignition) Protection

### What is LSPI?

Premature ignition of the main fuel charge in turbocharged DI engines. Most common at low speed, high load. Events are random but can be catastrophic — in dyno testing, while other cylinders at 75 bar peak pressure, a pre-igniting cylinder can exceed **160 bar** peak pressure. Super/Mega Knock from LSPI: at best blows head gaskets, more commonly destroys piston ring lands, cracks pistons, bends connecting rods.

### Factory LSPI Protection Strategy

Upon pre-ignition detection, ECU lowers load limit by blending 3 LSPI tables using OAR:
- **OAR = +1** → LSPI Load Limit (High) — most restrictive
- **OAR = 0** → LSPI Load Limit (Nominal/Mid)
- **OAR = -1** → LSPI Load Limit (Low) — least restrictive

Modern management systems also strictly limit the **rate at which torque can rise** at low RPMs.

### Prevention

- Use API SP or ILSAC GF-6 rated oil (reduced calcium sulfonate, increased magnesium sulfonate)
- Install colder spark plugs for tuned applications
- Avoid heavy throttle below 3,000 RPM
- Downshift before accelerating hard
- **NEVER blank out or disable LSPI tables** — some tuners do this for easy AFR optimization, which is extremely dangerous, especially for towing
- The #1 culprit for misfires on boosted EcoBoost: **spark plugs and plug gap** — close the gap first when troubleshooting

## Tuning Strategy Summary

### Order of Operations

1. **Raise torque limiters** — lift the ceiling so the PCM doesn't clip power
2. **Raise load limits** — adjust just enough in needed areas to prevent throttle closures
3. **Adjust Driver Demand tables** — tell the PCM to request more torque at given pedal positions
4. **Update torque model** — use Torque Inverse Calculator to recalculate torque tables
5. **Adjust Borderline spark tables** — primary WOT timing tables for pump gas
6. **Set WOT fueling targets** — adjust Desired Fuel Target (Power Demand) for proper Lambda
7. **Fine-tune WGDC** — adjust wastegate base table for desired boost profile
8. **Datalog and iterate** — log OAR, knock counts, spark source, boost actual vs desired, AFR
9. **Tune transmission** — shift points, firmness, TC lockup strategy

### Key Tuning Tips

- "Because these are torque-based controllers, you demand a torque value and as long as you don't exceed torque/boost/spark limiters, it does what you want."
- Factory fueling strategies are overly conservative with long delay timers before delivering additional fuel at WOT.
- EcoBoost Rangers make **15–20% less HP on 87 octane vs 93** — always use premium fuel when tuned.
- When switching to ethanol blends, add **more timing than boost** — this is where ethanol's value lies.
- OAR should settle close to **-1.0** on good fuel with a properly calibrated tune.
- A "bad tune" ignores knock feedback and fuel trims, or demonstrates unstable boost.

### Fuel Type Considerations

| Fuel | Timing Headroom | Boost Headroom | Power Potential | Fuel System Mods? |
|------|----------------|----------------|-----------------|-------------------|
| 87 octane | Lowest | Conservative | Stock or mild | No |
| 91 octane | Moderate | Moderate | +30–50 whp | No |
| 93 octane | Good | Good | +35–60 whp | No |
| E30 | Very good | Very good | +53–80 whp | No (stock system OK) |
| E50 | Excellent | Excellent | Best perf/fuel system balance | HPFP recommended |
| E85 | Maximum | Maximum | +15%+ over pump gas | **HPFP + injectors required** |
