# HP Tuners Platform Overview

## What is HP Tuners?

HP Tuners is a professional automotive ECU tuning platform founded in 2003, headquartered in Buffalo Grove, Illinois. It provides hardware (MPVI3/MPVI4 interface) and software (VCM Suite) for reading, editing, and flashing vehicle ECU/PCM calibrations, plus real-time datalogging and diagnostics. It supports Ford, GM, Stellantis, Volkswagen/Audi, and powersports vehicles.

## VCM Suite (Free Software)

VCM Suite is the core software package — **free download**, installable on unlimited computers. Current version: VCM Suite 5.2. Requires Windows 10+, .NET Desktop Runtime 8, 2 GHz x86 CPU, 4 GB RAM.

VCM Suite consists of two integrated applications:

### VCM Editor (Calibration Editing)

The calibration editing tool for reading, modifying, and writing ECU/PCM tune files.

**Parameter Navigator:**
- Explorer-style tree on the left side, parameters listed hierarchically by category/subcategory
- Dynamic filtering as you type in search field
- Toolbar icons filter by data type, parameter group, and difficulty level (basic/advanced)
- Undockable navigator window, customizable favorites list
- Tab-style navigation alternative to tree view

**Parameter Types:**
- **Switches:** Drop-down menus with specific values
- **Scalars:** Simple numerical input fields
- **Tables:** Open in Table Editor for graphical viewing and editing

**Ford Parameter Tree (Top-Level Categories):**
- **Engine > Airflow** — Electronic Throttle (ETC), General Airflow, Speed Density
- **Engine > Torque Management** — Torque Reduction, Torque Intervention, Indicated Engine Torque, Engine Friction Torque
- **Engine > Torque Model > General** — Torque Calculation tables (updated by Torque Inverse Calculator)
- **Engine > Idle** — Idle Airflow, Idle RPM targets
- **Engine > Spark** — MBT (Maximum Brake Torque) tables, spark advance maps
- **Engine > Turbo/Boost** — Desired Boost Pressure, WGDC Base table, Boost Multipliers
- **Fuel System** — Fuel Pump, Injector Pressure Drop, Fuel Pump Voltage/Flow/Pressure
- **Transmission** — Torque Management, Shift Points, Pressures

**Table Editor Features:**
- 2D and 3D graphical editing (click and drag)
- Cell selection (individual or ranges) for bulk operations
- Keyboard +/- key increment/decrement
- 1-click copy/paste of entire tables
- Right-click menu: Copy, Paste, Undo, Multiply by %, Multiply by % - Half, Paste Special
- Table smoothing (general, vertical, horizontal)
- Table interpolation
- Scaling (multiply values) and shifting (add/subtract offset)
- Renumbering table axes (modify breakpoints)
- Multiple tables open simultaneously
- Save/print/load individual tables

**Compare Tools:**
- Open two tune files simultaneously
- Three compare views with color-coded differences
- Comparison log showing all differences in tree format
- Cross-OS compare (different OS IDs, similar parameters only)
- Copy over selected differences from compare file to tune file

**Ford-Specific Tools:**
- **Torque Inverse Calculator** (`Tools` menu): Enter torque data → "Calculate Inverse" → updates Torque Calculation tables at `Engine > Torque Model > General`. Uses an engine torque model to smooth values.
- **ETC Effective Area Calculator** (`Tools` menu): For throttle body swaps. Updates Throttle Body Model tables at `Engine > Airflow > Electronic Throttle`.
- **Change VIN Wizard:** Change VIN of file being edited to any valid supported VIN.
- **TunerLock:** Lock PCM to prevent access by other HP Tuners users or third-party software.

**Template Editor:**
- `Tools > Tune Template Editor`
- Create parameter value templates, apply across multiple tunes
- Available in VCM Editor 4.3.431+

### VCM Scanner (Datalogging & Diagnostics)

Real-time datalogging, diagnostics, and vehicle controls.

**Display Panel Types:**
- **Gauge Panels:** Sensor data displayed as gauge clusters
- **Graph Panels:** Tabular format for gathering sample data
- **Chart vs. Time Panels:** Sensor outputs plotted on timeline
- **Histogram Panels:** Customizable histograms with filters, printable output

**Key Features:**
- Customizable layouts (save/load named layout files)
- Live reconfiguration while actively scanning
- Variable logging/playback speeds
- Real-time monitoring: RPM, spark advance, IAT, ECT, AFR, gear, timing, open/closed loop
- Up to 10 custom math parameters (e.g., Desired Boost - Actual Boost = Boost Error)
- DTC reading and clearing
- Freeze frame data viewing
- Export to CSV for Excel analysis
- Standalone data logging (no laptop required with MPVI3/4)

**Bidirectional Controls (VCM Controls):**
- Command fans, open/closed loop, gear select, timing, AFR without flashing
- Cylinder balance test (cuts injectors 1 at a time, compares RPM drop)
- Throttle relearn
- View/reset engine and transmission adaptives

**Wideband AFR Integration:**
- PROLINK+ cable: wire in 2x analog signals (wideband, MAP) + 1x CAN bus signal
- CAN bus wideband setup supported natively
- Wideband data overlaid with all other parameters in real-time

## Hardware

### MPVI3

- **Connectivity:** USB-C (4 MB/s), Bluetooth 5.0
- **Connector:** M8 motorsports-grade screw-on OBD-II connector
- **Storage:** 8 GB internal (2x MPVI2)
- **Features:** RGB LEDs, high-resolution accelerometer, standalone data logging
- **Pro Feature Set:** Included standard (standalone logging + PROLINK+ compatibility)
- **Package:** MPVI3 device, USB A to C cable, lanyard, decals

### MPVI4 (Current Generation — October 2025)

- **Processor:** Dual-core 1.7 GHz
- **Memory:** 1 GB RAM (30x previous generation)
- **Connector:** New tapered OBD-II design for stronger connection
- **Wi-Fi:** Coming via future firmware update
- **Price:** ~$399.99 USD (device only, no credits)
- **Pro Feature Set:** Standard
- **Note:** You can read, edit, and save files without purchasing credits — credits only needed when writing

### PROLINK+ (Accessory Cable)

- Records 2x analog signals (wideband sensor, MAP, etc.) + 1x CAN bus signal simultaneously
- Sold separately, compatible with MPVI3/MPVI4 out of the box

### RTD (Remote Tuning Device)

- Physical device for tuning shops to distribute tunes to customers remotely
- RTD4 (Gen 4) is current version
- Plugs directly into vehicle OBD-II port
- Loads tunes without laptop via TDN network + smartphone app
- Customers cannot view or modify tune files (tuner-controlled)

## Credits & Licensing System

### How Credits Work

Credits are the currency to license vehicle modules to your MPVI device. Think of them like prepaid phone minutes.

- **Universal Credits:** Work on any supported vehicle regardless of make/model/year
- Most vehicles require **2 to 6 credits** depending on controller type
- **Modern Ford vehicles: 4 Universal Credits per PCM** (TCM is separate)
- Credits are universal across MPVI2/3/4 devices

### License Types

| Type | Description | Best For |
|------|-------------|----------|
| Single Vehicle (VIN) | Tied to VIN + PCM serial + PCM OS. Unlimited reflashes forever. | Individual owners |
| Year/Model Type | Unlimited vehicles of that exact year/model. | Shops specializing in specific platforms |
| Tuner Shop Packages | Bulk licensing for multiple year/model combinations. | Professional shops |

### Important Rules

- **Read/Edit/Save is FREE** — credits only consumed when **writing/flashing** for the first time
- **Standalone datalogging works WITHOUT a license**
- Once licensed, tune that vehicle (VIN+OS+Serial) **indefinitely** with unlimited reflashes
- If PCM is replaced, new serial number needs re-licensing
- Licenses are **permanent** — cannot delete or swap
- Credits cannot be transferred between devices (exception: device exchange program)
- **No refunds** on digital products including credits
- Unused credits carry forward to different vehicles

## Read/Write Workflow

### Reading the Stock ECU

1. Connect MPVI device to laptop via USB-C
2. Connect device to vehicle OBD-II port
3. Turn ignition ON (do not start), all accessories off, doors closed
4. VCM Editor: `Flash > Read Vehicle`
5. Wait 10–15 seconds for communication
6. File opens automatically in .HPT format
7. **Save stock file as backup before any changes**

### Editing Calibration

1. Open .HPT file in VCM Editor
2. Navigate parameter tree (Engine > Spark, Fuel, Airflow, Torque Management, etc.)
3. Modify tables using Table Editor tools
4. Use Ford Torque Inverse Calculator when modifying torque model
5. Compare stock vs. modified with Compare tools
6. Save modified .HPT file

### Writing/Flashing

1. VCM Editor: `Flash > Write Vehicle`
2. Ensure battery fully charged, charger connected
3. Most PCMs write in ~30 seconds
4. **Never power down during a write failure**
5. Always use same file to retry writing
6. Built-in automatic PCM recovery for protection against reflashing problems

### Real-Time Tuning (RTT)

- Tables can be modified in real time via integrated RTT interface
- Controlled by VCM Scanner
- Changes applied to running engine without full reflash

## Tune Delivery Network (TDN)

- Cloud-based, browser-based platform + iOS/Android app
- Built for professional tuners, shops, and aftermarket companies
- Manage customers, vehicles, receive read files, push tunes
- Compatible with all HP Tuners devices

## HP Tuners vs. Competitors (Ford Ranger Context)

| Platform | Strengths | Weaknesses |
|----------|-----------|------------|
| **HP Tuners** | Full parameter access, DIY-friendly, extensive datalogging, most popular for custom tuning | More expensive for single vehicle (~4 credits), Windows only |
| **SCT** | Simpler for novices, BDX programmer | Less parameter access, limited drivability calibration control, considered dated |
| **Ford Performance ProCal 4** | Official Ford tool, CARB-legal calibrations | VIN-locked, single vehicle, limited to Ford Performance tunes |
| **Livernois MyCalibrator** | Pre-loaded calibrations, easy to use | Less custom tunability than HP Tuners |
| **nGauge RTD** | Remote tuning without laptop | Depends on tuner for calibration files |

HP Tuners is the recommended platform for custom tuning the Ford Ranger Raptor 3.0L EcoBoost V6 due to full MG1CS036 PCM + 10R80 TCM access and the largest tuner community. HP Tuners achieved first-to-market direct OBDII flashing of the MG1CS036 (no PCM swap/unlock required). VCM Suite support is currently in BETA for MG1CS036 vehicles. Only MPVI3, MPVI4, and RTD4 are supported — older MPVI2 devices are NOT compatible.
