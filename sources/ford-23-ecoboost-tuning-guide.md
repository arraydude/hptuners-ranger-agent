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

## Torque-Based ECU Model

The Ford EcoBoost PCM uses a **torque-demand model** — conceptually similar to BMW's torque-based DME. The tuning order is critical:

### 1. Driver Demanded Torque (DDT)

When the accelerator pedal moves, the PCM references a **Driver Demand** table (RPM vs. pedal position) to determine desired torque in Newton Meters. This is the central table controlling power output.

- **ETC Driver Demand tables:** Engine RPM vs. Pedal Position → Torque Demand
- Two variants: Driver Demand Wheel (output shaft speed) and Driver Demand Engine (engine RPM)
- Raising DDT alone is NOT enough — load and boost limit tables must also be raised

### 2. Torque Limiters Check

The PCM cross-references ALL limiters before allowing demanded torque. If ANY limiter shows a ceiling lower than driver demand, it takes precedence immediately:

| Limiter | Purpose |
|---------|---------|
| Gear ratio limiters | Different torque limits per gear for traction |
| Gearset limiters | Protect the transmission |
| Steering angle limits | Full power only within a few degrees of center |
| Combustion stability limits | Control LSPI by limiting load at low RPM |
| Exhaust Gas Temperature | Protect turbo and exhaust components |
| Throttle Inlet Pressure | MAP pressure ceiling |
| Manifold Temperature | Charge air temperature limits |
| Knock limits | Pull timing on detonation detection |
| Fuel Enrichment limits | Fueling safety boundaries |
| Ambient Temp & Barometric Pressure | Environmental derating |

### 3. Torque-to-Air Load Model

The approved torque is converted through a model determining how much air load is required to produce that crankshaft torque. This is then converted to air mass requirements.

**Torque Inverse Calculator:** HP Tuners provides a built-in tool at `Tools > Torque Inverse Calculator`. Enter torque data → "Calculate Inverse" → updates the two Torque Calculation tables at `Engine > Torque Model > General`. Running on stock values "smooths out" to an ideal model.

### 4. Fast Path vs. Slow Path Torque Control

- **Slow path:** Throttle position (takes time for air to travel to cylinder)
- **Fast path:** Spark retard and fuel cut (nearly instant, per-cylinder)
- The throttle is the primary torque control mechanism via DBW. Factory calibration may close throttle to less than 20% for consistent torque delivery — this is why tune-only gains are significant.

**Critical implication:** Raising boost without adjusting the torque model hits torque limiters — the PCM will close the throttle to maintain the torque ceiling. You MUST raise torque limits first.

## Speed Density Airflow Model

The 2019+ Ranger uses **MAP-based airflow (Speed Density)** with no MAF sensor.

### Aircharge Calculation

```
Aircharge = (Slope × MAP) + Intercept
```

- **Slope** = MAP vs Air Charge tables (how aircharge scales with manifold pressure)
- **Intercept** = Zero Charge tables (aircharge at zero MAP)
- **MAP** = Manifold Absolute Pressure (measured)

### Virtual Volumetric Efficiency (VVE)

- VVE is an abstraction for SD coefficient tables — different from traditional VE tables
- Peak VE occurs at RPM of peak torque, progressively drops on either side
- VE increases as MAP increases
- **VE Compensation (MCT Pivot) table:** Critical for SD aircharge calculation, uses MAP, RPM, VE, displacement, and cylinder charge temperature

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
| Engine > Airflow > General > Speed Density | RPM vs MAP → airmass per cylinder |
| Engine > Airflow > General > Charge Temp Compensation | Temperature-based aircharge correction |

### Boost / Turbo Tables

| Table Path | Description |
|------------|-------------|
| Engine > Turbo/Boost > Desired Boost Pressure | Target boost by RPM/load |
| Engine > Turbo/Boost > WGDC Base | Airflow Mass (X) vs WGDC Y-Factor (Y) → duty cycle % (feed-forward) |
| Engine > Turbo/Boost > WG Actuator Canister Pressure | Canister pressure (X) vs Compressor Outlet Pressure Relative (Y) → WGDC % |
| Engine > Turbo/Boost > Turbo Boost Mult vs ECT | Boost multiplier by engine coolant temperature |
| Engine > Turbo/Boost > Turbo Boost Mult vs Baro | Boost multiplier by barometric pressure (altitude) |
| Engine > Turbo/Boost > Turbo Boost Mult vs TPS | Boost multiplier by throttle position |

**Boost Control PID:**
```
WGDC Actual = WGDC Base + WGDC P-Term + WGDC I-Term
```

The WGDC Base table provides the feed-forward value — it is NOT a direct boost target. The PID controller adjusts around it. Do not make large/incorrect adjustments to the WGDC table.

### Spark / Ignition Tables

| Table Path | Description |
|------------|-------------|
| Engine > Spark > MBT (Maximum Brake Torque) | Fully populated spark reference: max torque at all RPM/load points. Upper spark limit and torque reference. |
| Engine > Spark > Borderline Spark | Knock boundary tables |
| Engine > Spark > Spark Advance tables | RPM vs Load → degrees of advance |

**Virtual Torque Window:** Series of tables corresponding to amounts of spark advance/retard. X-axis: RPM, Y-axis: cylinder airmass (mg) or MAP (kPa). ECM uses the table closest to current spark advance.

### Fuel Tables

| Table Path | Description |
|------------|-------------|
| Fuel System > Desired Injector Pressure Drop vs Mass | Fuel rail pressure target by injection mass |
| Fuel System > Fuel Pump Voltage vs Flow vs Pressure | LPFP voltage calibration |
| Fuel System > Injector Pressure Drop Adder vs Ambient Temp | Temperature compensation |
| Engine > Fuel > Lambda/AFR targets | Commanded lambda at various RPM/load points |

**Fueling Notes:**
- Factory fueling is slightly rich with moderate delay timers before allowing additional fuel at WOT
- Lambda target of 0.80 (11.76:1 AFR) is safe for DI under boost
- Increasing fuel rail pressure reduces relative injector duty cycle and allows more fuel mass delivery in the short DI injection window

### Transmission Tables (10R80)

| Table Path | Description |
|------------|-------------|
| Transmission > Torque Management | Transmission-side torque limits |
| Transmission > Shift Points | Upshift/downshift RPM by gear |
| Transmission > Line Pressure | Clutch apply pressure schedules |
| Transmission > Torque Converter Lockup | Lockup RPM and conditions |

**TCM tuning requires separate HP Tuners licensing (additional credits).**

## Datalog Parameters (VCM Scanner)

### Critical Channels to Monitor

| Parameter | Normal Range | Concern Threshold | Action |
|-----------|-------------|-------------------|--------|
| Knock Retard | 0° | Any positive value | Reduce timing, check fuel quality, check for LSPI |
| WGDC Actual | 20–30% at WOT | Sustained >90% | Turbo is near max — needs hardware upgrade or lower targets |
| Boost Pressure Actual | 17–22 PSI (stock) | >27 PSI on stock turbo | Beyond safe stock turbo limit |
| AFR / Lambda | 0.78–0.85λ at WOT | Leaner than 0.85λ at WOT | Dangerously lean — reduce boost or enrich fueling |
| IAT (Intake Air Temp) | <50°C under boost | >55°C sustained | Intercooler inadequate — heat soak |
| ECT (Engine Coolant Temp) | 90–105°C | >115°C | Cooling system issue or excessive load |
| EGT (Exhaust Gas Temp) | <900°C | >870°C (1,600°F) | Risk of exhaust valve damage, pre-ignition |
| Fuel Rail Pressure | 150 bar (stock target) | Drops under load | HPFP can't keep up — upgrade HPFP |
| Fuel Trims (LTFT/STFT) | ±3% | Beyond ±5% | Investigate injectors, vacuum leaks, SD calibration |
| Injector Duty Cycle | <80% | >85% | Injector capacity limit — upgrade injectors |

### Boost Control Monitoring Channels

| Channel | Purpose |
|---------|---------|
| Boost Pressure Actual | Measured boost |
| Desired Boost Pressure | Target boost from calibration |
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
- **Knock Sum** = Total knock events per log session
- **AFR Error** = Commanded Lambda - Actual Lambda

## ECU Limiting Factors

The PCM will cut boost before target is reached if ANY of these are exceeded:

1. Exhaust Gas Temperature
2. Throttle Inlet Pressure
3. MAP Pressure
4. Manifold Temperature
5. Knock detection
6. Fuel Enrichment limits
7. Ambient Temperature
8. Barometric Pressure
9. LOAD limits
10. Combustion stability (LSPI prevention)

## Tuning Strategy Summary

### Order of Operations

1. **Raise torque limiters** — lift the ceiling so the PCM doesn't clip power
2. **Adjust Driver Demand tables** — tell the PCM to request more torque at given pedal positions
3. **Update torque model** — use Torque Inverse Calculator to recalculate torque tables
4. **Raise load/boost targets** — increase desired boost and adjust WGDC feed-forward
5. **Optimize spark advance** — increase timing where safe (higher octane = more timing headroom)
6. **Adjust fueling** — ensure safe AFR under boost, adjust fuel rail pressure if needed
7. **Tune transmission** — adjust shift points, firmness, TC lockup strategy

### Fuel Type Considerations

| Fuel | Timing Headroom | Boost Headroom | Power Potential |
|------|----------------|----------------|-----------------|
| 87 octane | Lowest | Conservative | Stock or mild |
| 91 octane | Moderate | Moderate | +30–50 whp |
| 93 octane | Good | Good | +35–60 whp |
| E30 | Very good | Very good | +53–80 whp |
| E50 | Excellent | Excellent | Best performance/fuel system balance |
| E85 | Maximum | Maximum | +15%+ over pump gas, **requires HPFP/injector upgrade** |

### LSPI Prevention

Low Speed Pre-Ignition is the primary safety concern on the 2.3L EcoBoost:
- Use API SP or ILSAC GF-6 rated oil (reduced calcium sulfonate, increased magnesium sulfonate)
- Install colder spark plugs for tuned applications
- Avoid heavy throttle below 3,000 RPM
- Downshift before accelerating hard
- PCM has built-in LSPI prevention strategies — do not disable combustion stability limits at low RPM
