# HP Tuners Platform Overview

## What is HP Tuners?

HP Tuners is a professional automotive ECU tuning platform founded in 2003, headquartered in Buffalo Grove, Illinois. It provides hardware (MPVI3/MPVI4 interface) and software (VCM Suite) for reading, editing, and flashing vehicle ECU/PCM calibrations, plus real-time datalogging and diagnostics. It supports Ford, GM, Stellantis, Volkswagen/Audi, and powersports vehicles.

## VCM Suite (Free Software)

VCM Suite is the core software package — **free download**, installable on unlimited computers. Current version: VCM Suite 5.2. Requires Windows 10+, .NET Desktop Runtime 8, 2 GHz x86 CPU, 4 GB RAM.

VCM Suite consists of two integrated applications:

### VCM Editor (Calibration Editing)

The calibration editing tool for reading, modifying, and writing ECU/PCM tune files. Document revision: 28 October 2025 (154 pages).

**Recommended Workflow:**
1. Check vehicle for DTCs before anything else (reading clears DTCs)
2. Datalog with VCM Scanner to understand current vehicle behavior
3. Read the current tune via VCM Editor
4. **Immediately save the stock read as backup** (File > Save)
5. Locate and modify parameters
6. Save changes, write to vehicle
7. Datalog again with VCM Scanner to verify changes
8. Iterate steps 5–7 until tuning goals are met

**Main Window Layout:**

| # | Component | Description |
|---|-----------|-------------|
| 1 | Menu Bar | File, Edit, Compare, Flash, Tools, Window, Help |
| 2 | Main Toolbar | Open/save/close, tune details, remote tuning, history, compare, unit conversion, read/write vehicle |
| 3 | Favorites Icon | Quick-access parameters window |
| 4 | Tab Navigation | Icons to browse parameter groups in the open tune file |
| 5 | Workspace | Area where table editors and daughter windows appear |

**Menu Bar:**
- **File**: Open, Close, Save tune files (.HPT format)
- **Edit**: Calibration Details, Remote Tuning, View/Change History Logs, Virtual VE, Virtual Torque, Gear/Tire Wizard, Change VIN Wizard, ETC Effective Area Calculator, Torque Inverse Calculator, VE Neural Network Trainer, User Defined Parameters, Template Applicator, Navigator, View (Basic/Advanced)
- **Compare**: Open Compare File, Show Main/Compare/Differences, View Comparison Log, Segment Swapper
- **Flash**: Read Vehicle, Write Vehicle
- **Tools**: Unit Conversion, Calculator, Options, Template Editor
- **Window**: Arrange open daughter windows
- **Help**: Help system, License Information, Resync Interface, VCM Suite Information, MPVI Application Keys, MPVI2 Verification Code, About

**Parameter Types:**
- **Switches:** Drop-down menus with specific values. In Template Editor, must enter exact string value (no dropdown).
- **Scalars:** Simple numerical input fields. Hover to see permitted range (lower-right corner). Click green unit label to cycle units.
- **Tables:** Open in Table Editor for graphical viewing and editing. Can be 1D, 2D, or 3D.
- **Code Modifications:** OS group buttons that apply HP Tuners patches. Must use WRITE ENTIRE after applying.

**Parameter Color Coding:**
- Single file: Blue = unchanged, other color = changed since last save
- With compare file: 6 states tracking match/mismatch + changed/unchanged

**Parameter Right-Click Options:** Copy, Paste, Undo Changes, Units, Decimals, Favorites (add/remove)

**Ford Parameter Tree (Top-Level Categories):**
- **Engine > Airflow** — Electronic Throttle (ETC), General Airflow, Speed Density
- **Engine > Torque Management** — Torque Reduction, Torque Intervention, Indicated Engine Torque, Engine Friction Torque
- **Engine > Torque Model > General** — Torque Calculation tables (updated by Torque Inverse Calculator)
- **Engine > Idle** — Idle Airflow, Idle RPM targets
- **Engine > Spark** — MBT (Maximum Brake Torque) tables, spark advance maps
- **Engine > Turbo/Boost** — Desired Boost Pressure, WGDC Base table, Boost Multipliers
- **Fuel System** — Fuel Pump, Injector Pressure Drop, Fuel Pump Voltage/Flow/Pressure
- **Transmission** — Torque Management, Shift Points, Pressures

#### Table Editor

**Views:** Table View (default, most editing), 2D Graph View (click-drag points), 3D Graph View (rotate/zoom/drag), Split View (horizontal or vertical split).

**Cell Selection:**

| Target | Procedure |
|--------|-----------|
| Single cell | Click the cell |
| Row or column | Click the row/column header |
| All cells | Click upper-left corner box |
| Block of cells | Click-drag from corner to opposite corner |

**Toolbar Math Operations:**
- **Replace (=):** Enter value in toolbar box, click = — replaces selected cells
- **Add (+):** Enter value, click + (use negative to subtract)
- **Multiply (x):** Enter value, click x (use decimal to divide, e.g., 0.5 = ÷2)

**Keyboard Shortcuts (Table View only):**

| Key | Action |
|-----|--------|
| `+` / `-` | Increment / decrement selected cell values |
| `Ctrl+C` | Copy values |
| `Ctrl+V` | Paste values |
| `Ctrl+Z` | Undo last modification |

**Smoothing & Interpolation:**
- Smooth entire selection / horizontal only / vertical only
- Interpolate entire selection (flat plane) / horizontal bounds / vertical bounds
- Smoothing level determined by current precision setting

**Right-Click Menu:**
- **Copy** — copy selected cells
- **Copy With Axis** — copy cells including axis labels (enables Paste Special with auto-alignment)
- **Paste** — paste into selected cell
- **Paste Special** (requires Copy With Axis): Add, Subtract, Multiply %, Multiply % Half, Average. Can paste VCM Scanner histogram data (e.g., knock retard graph into spark tables).
- **Undo Last / Undo All**
- **Units** — change displayed units
- **Column Axis / Row Axis** — copy labels, edit axis, change units
- **View Inverted** — swap X/Y axes

**Renumbering Table Axes:**
- Click underlined axis name to open axis editor
- Change endpoint values, click evenly-space icon to redistribute intermediate values
- Axis changes do NOT auto-update cell data — must update cells manually
- Some tables have PCM-enforced min/max endpoint limits

**2D/3D Graph Editing:**
- 2D: Click data point, drag to new position
- 3D: Click-drag data points. Zoom: hold both mouse buttons + move up/down. Rotate: right-click + drag.

#### Compare Tools

- Open two tune files simultaneously (any make/model/OS)
- Three views: Show Main File, Show Compare File, Show Differences (numerical delta)
- Per-table compare via Table Editor toolbar (changes view for that table only)
- **Comparison Log** (Compare > View Comparison Log): Quick overview of all differing parameters
- **Copy Over Differences**: Single parameter/group (right-click > Copy Over Selected) or all (Copy Over All Differences). v5.2+ includes DTCs.
- Cross-OS compare: different OS IDs → only similar parameter types compared

#### Ford-Specific Tools

**Torque Inverse Calculator** (Edit > Torque Inverse Calculator):
- Enter torque data in calculator table → click "Calculate Inverse"
- Updates TWO tables at `Engine > Torque Model > General`:
  - **Engine Torque** — torque (before friction/accessory losses) based on RPM and engine air load. Values copied directly from calculator.
  - **Inverse** — converts scheduled torque to engine air load needed. Mathematical inverse of Engine Torque table.
- Axis labels come from Engine Torque table — change them there, not in calculator.

**ETC Effective Area Calculator** (Edit > ETC Effective Area Calculator):
- For throttle body swaps. X-axis = effective area, Y-axis = throttle vacuum.
- Enter throttle angle data → click "Calculate Effective Area"
- Updates TWO tables at `Engine > Airflow > Electronic Throttle`:
  - **Effective Area** — PCM calculates airflow through throttle body (roughly inverse of calculator table)
  - **Predicted Throttle Angle** — PCM determines throttle angle for desired airflow. Angles copied directly from calculator.

**Change VIN Wizard** (Edit > Change VIN Wizard):
- Changes VIN without writing a new tune file. Does NOT change VIN in the open file.
- Procedure: Read vehicle → Edit > Change VIN Wizard → enter new VIN → Commit → wait 15 sec → ignition off 30 sec → on 30 sec → off 30 sec → on 30 sec.
- May require purchasing additional license for new VIN.

#### Template Editor

`Tools > Tune Template Editor` — Create parameter value templates, apply across multiple tunes.

**Creating a Template:**
1. Click New in Template Editor toolbar
2. Add parameters via: import from existing template, select from loaded tune, import unsaved changes, or import compare file differences
3. Edit imported values as needed
4. Save

**Applying a Template** (Edit > Template Applicator):
1. Open target tune → Edit > Template Applicator
2. Open template file → parameters appear (all checked by default)
3. Preview tab: edit values before applying (changes NOT saved to template)
4. Apply Changes tab: click Apply → results show Skipped/Success/Failed/Not Found

#### Tune History

Edit > View Change/History Logs:
- **Current Unsaved Changes tab**: Tree listing all changes since last save. Double-click to view/edit.
- **Complete Saved History Logs tab**: Timestamped records per save. Compare current to any previous version. Roll back to previous version.

### VCM Scanner (Datalogging & Diagnostics)

Real-time datalogging, diagnostics, and vehicle controls. Document revision: 07 January 2026 (6,600+ lines).

**Main Window Layout:**

| # | Component | Description |
|---|-----------|-------------|
| 1 | Menu Bar | Log File, Vehicle, Layout, Tools, Help |
| 2 | Toolbar | Quick-access buttons for common operations |
| 3 | Data Display Panels | Gauge, Graph, Chart vs. Time, Drive Cycle panels |
| 4 | Timeline | Slider for log file playback position |
| 5 | Details Tab | Log file and vehicle info |
| 6 | Channels Tab | View and configure channels |

**Menu Bar:**
- **Log File**: Open, Close, Save As, Export (.csv), Recent Logs, Exit
- **Vehicle**: Connect, Disconnect, Vehicle Profiles, Repoll for Supported Parameters, Start/Stop Scanning, Diagnostics & Info, Controls & Special Functions, MPVI Pro Data Logging, MPVI2 Data Logging
- **Layout**: Open Layout, Save Layout As, Recent Layouts, Default Layouts, Add To Layout, Lock Layout
- **Tools**: Unit Conversion, Math Parameters, Quantities & Units, Calculator, Options
- **Help**: Help, Resync Interface, VCM Suite Info, MPVI Application Keys, MPVI2 Verification Code, About

**Scanning Procedure:**
1. Connect interface device to laptop (USB-C) and vehicle (OBD-II)
2. Close VCM Editor, open VCM Scanner
3. Turn ignition on (engine can be running or stopped)
4. Click Connect, then Start Scanning
5. While scanning: press **M** to insert a marker, press **C** to insert a marker with comment (longer key press = wider marker bar)
6. Click Stop Scanning when done
7. Click Save to generate log file

**Language Support:** Tools > Options > Language dropdown (partial multi-language, requires restart).

#### Display Panel Types

**Gauge Panel:**
- Round Analog Gauge or Vertical Bar Gauge
- Configurable: label, parameter, unit, decimals, filter (update interval)
- Ticks: min/max, major/minor tick counts, arc start/sweep (degrees)
- Ghost pointers: draw max/min peaks (right-click > Reset Peaks)
- Color ranges: start/end values, 3 levels (green/yellow/red by default)
- Overlapping gauges: lower in list renders in front, adjustable stacking

**Graph Panel — Tables (3D data plots):**
- Plot observed values of a parameter as function of two other parameters (column axis + row axis)
- Cell data views: Highest, Lowest, Average, Last, Hit Count (toggle via icons)
- Time range: entire recording or Chart vs. Time panel window
- Color shading: High/Mid/Low value gradient
- **Copy to VCM Editor**: Select cells > right-click > Copy with Axis > in VCM Editor, Table Editor > right-click > Paste Special

**Graph Panel — Histograms:**
- Plot distribution of observed values for single parameter
- Bars show percentage of time at each value
- Same Copy to VCM Editor workflow via Paste Special

**Expression Filtering:** Define conditional expressions (TRUE/FALSE per frame) to filter which data populates graphs. Variable modifiers: `avg(x)` for rolling average, `slope(x)` for rate of change, `shift(x)` for time-offset values.

**Chart vs. Time Panel:**
- Left column: current parameter values (color-coded)
- Main area: line charts over time with 3 vertical axis labels per parameter
- Vertical bar = current time index (click to jump during playback)
- Parameters organized into signal groups with configurable color, min/max limits, reference lines

**Drive Cycle Panel (v4.13+):**
- Guide driver through standardized drive cycles for emissions/fuel economy testing
- White line = target speed, green line = current speed
- Useful after clearing DTCs or battery replacement for emissions monitor self-tests

**Hot Keys:**
- Keypad 1–0: View graph by number
- Page Up / Page Down: Cycle through graphs
- Ctrl+C: Copy selected cells

#### Channels

All data input into VCM Scanner is defined by channels. Channels supply data to layouts and math parameters.

**Channel Types:**

| Icon | Type | Description |
|------|------|-------------|
| White bg | Polled | Scanner must request data from controller. More polled channels = slower scan rate. |
| Green bg | Broadcast | Controller constantly broadcasts. No performance impact. Preferred. |
| — | External | From external device via serial port, EIO, ProLink, or ProLink+ |

**Channel Data Types:** Scalar (numerical), Switch (selection list), Flag (ON/OFF)

**Adding OBD Port Channels:**
1. Connect to vehicle (or select stored profile)
2. Channels tab > Add Channel icon
3. Browse/search Channel Selector (blue = already added, green icon = broadcast)
4. Double-click to add
5. Optionally adjust polling interval (right-click > Polling Interval)

**Performance Tip:** Lower polling interval on slow-changing parameters (e.g., ECT to 10 sec) to free bandwidth for fast-changing ones (e.g., Spark Advance).

**ProLink+ Analog Inputs:** RED (Analog 1) / BLUE (Analog 2) wires, 0–5V, 100 Hz. Add via External Inputs > MPVI2 > Pro Link. Apply Transform for voltage conversion.

**ProLink+ CAN Bus Inputs:** ORANGE (CAN High) / YELLOW (CAN Low), 500 kbps. Sensors listed by manufacturer in Channel Selector.

**Serial Port Devices:** RS-232 via USB-to-serial adapter. External Inputs > Serial Port, organized by manufacturer.

**Transforming a Channel:** Right-click > Transform. Select preset transform or create Custom Transform (linear two-point calibration). User transforms stored with channel config.

**Loading/Saving Channel Config:** Auto-saved to `VCM Scanner.cfg` on close. Manual save as .xml. Load SAE Defaults or Load Vehicle Defaults.

**Advanced Channel Properties:** OS Fallback (when vehicle doesn't report OS ID), OS Override (always use specified OS definitions).

#### Math Parameters

Tools > Math Parameters — up to 10 user-defined parameters plus built-in predefined ones.

**Defining:**
1. Select slot in Maths - User folder
2. Enter Name, Abbreviation, Expression, Unit, Decimals

**Expression Types:**
- **Calculated**: Math on sensor/PID outputs. Example: `100 * ([50119.238] - [50118.238]) / [50118.238]`
- **Conditional**: Returns 1 (TRUE) or 0 (FALSE). Example: `([50010] > 25 OR [25] = 50) AND [3131] > 10`

**Variable Syntax:** `[ParameterID]`, `[ParameterID.UnitID]`, `[ParameterID.UnitSymbol]`

**Variable Modifiers:**

| Modifier | Function |
|----------|----------|
| `avg(x)` | Average over x ms before current time |
| `slope(x)` | Rate of change between x ms ago and current |
| `shift(x)` | Value at x ms before current time |

Negative x values look forward (playback only, not live).

**Supported Operations:**

| Category | Operations |
|----------|-----------|
| Arithmetic | `+`, `-`, `*`, `/`, `^` |
| Functions | `sin(x)`, `cos(x)`, `tan(x)`, `abs(x)`, `round(x)` |
| Relational | `>`, `<`, `=` (returns 1 or 0) |
| Logical | `AND` / `&`, `OR` / `\|` |

**Variable Wizard:** GUI for creating variables — select Parameter → Unit → Special Function (Average/Slope/Shift) with Period (ms).

**Sharing:** User math parameters stored in `VCM Scanner.cfg`. Not distributed with Channel or Layout configs.

**Quantities & Units** (Tools > Quantities & Units): Look up numerical IDs and symbols for variable specifications.

#### Vehicle Profiles

Each vehicle creates a profile saving channel/parameter setup. Auto-selected on reconnection.

- Only OBD channels saved in profiles (external channels are vehicle-independent)
- **Create**: Automatic on first data connection. Vehicle > Repoll for Supported Parameters to update.
- **Manual OS ID/VIN Entry**: Vehicle > Vehicle Profiles > Vehicle Profile Editor — for vehicles with missing/incorrect OS IDs
- **Save/Load**: Save profile to file (shareable with remote tuners). Import from data log file.

#### Standalone Data Logging

**Supported:** MPVI2/2+ (with Pro Feature Set), MPVI3, MPVI4. Works for all vehicles programmable by VCM Suite.

**Setup:** Connect device → Help > Resync Interface → Vehicle > MPVI2 Data Logging (or MPVI4 Features) → Resync Interface Resources → optionally Write Channels Config. MPVI4 skips Resync Interface Resources step.

**Start Triggers (optional):**
- Short Button Press (BT button, enabled by default)
- Instantaneous Acceleration (threshold in G)
- Sustained Acceleration (G threshold + duration in seconds)
- Presets: Street Car, Race Car (fill suggested values)
- MPVI4 additional: Disabled (no logging), Always (auto-start on boot/connect)
- Note: OBD extension cables make accelerometer triggers unreliable

**Stop Triggers (optional):**
- OR Trigger Group: any condition stops logging (Short Button Press, Sustained Acceleration, RPM threshold + duration)
- AND Trigger Group: all conditions must be true to stop
- Auto-stop: no movement + zero RPM for 1 minute

**Recording:** Disconnect from laptop → plug into vehicle OBD-II → start engine → logging starts 5–10 sec after trigger conditions met. Amber OBD light flashes when active.

**Retrieval:** Vehicle > MPVI2 Data Logging (or MPVI4 Features) > Log Files tab. Newest files at top, named by sequence number + VIN.

#### Real Time Tuning (RTT)

Tables can be modified in real time via integrated RTT interface controlled by VCM Scanner. Changes applied to running engine without full reflash.

**RTT Window:** Appears when connected to RTT-capable vehicle with custom OS applied.
- Table selection via numbered buttons
- **Use RAM Version**: Copies flash table to RAM, makes it active and editable in real time
- **Use Flash Version**: Reverts single table to flash version
- Copy finalized table: right-click > Copy in RTT window → paste into VCM Editor
- Cycling ignition erases all RAM changes

#### Controls and Special Functions

Vehicle > Controls & Special Functions (while connected). Available functions vary by manufacturer, controller, and OS version.

**Ford-relevant controls:**
- **Crank Relearn**: Engine at operating temp, park, accessories off. Gradually rev to fuel cutoff (~4000–5000 RPM) over ~4 sec, release immediately. Ignition off 15+ sec to store.
- **Trans Clean**: Engine at idle, not moving, Park, trans temp 70–110°C. Engine speed varies automatically.
- **Trans Fast Learn**: Engine at idle, trans temp 70–95°C, not moving, brake applied. 2-second compliance window for instructions or test aborts. Engine speed varies automatically.

**Diagnostics & Information:**
- DTC reading and clearing
- Freeze frame data viewing
- Emissions monitor status checking

#### Wideband AFR Integration

- ProLink+ cable: wire in 2x analog signals (wideband, MAP) + 1x CAN bus signal
- CAN bus wideband setup supported natively (sensors listed by manufacturer)
- Wideband data overlaid with all other parameters in real-time
- Channel setup: External Inputs > MPVI2 > Pro Link in Channel Selector → apply Transform for voltage-to-AFR conversion

## Hardware

### MPVI3

- **Connectivity:** USB-C (4 MB/s), Bluetooth 5.0
- **Connector:** M8 motorsports-grade screw-on OBD-II connector
- **Storage:** 8 GB internal (2x MPVI2)
- **Features:** RGB LEDs, high-resolution accelerometer, standalone data logging
- **Pro Feature Set:** Included standard (standalone logging + ProLink+ compatibility)
- **Package:** MPVI3 device, USB A to C cable, lanyard, decals

### MPVI4 (Current Generation — October 2025)

- **Processor:** Dual-core 1.7 GHz
- **Memory:** 1 GB RAM (30x previous generation)
- **Connectivity:** USB-C (to PC), Bluetooth Low Energy (BLE) for TDN app, Wi-Fi (coming via firmware update)
- **Connector:** New tapered OBD-II design for stronger connection
- **Ports:** OBD-II connector (port 1), HPTNET screw-on connector for ProLink+ (port 2), USB-C (port 3)
- **Drivers:** Plug and Play on Windows 10+ (no manual driver install)
- **Price:** ~$399.99 USD (device only, no credits)
- **Pro Feature Set:** Included standard (standalone logging + ProLink+ compatibility)
- **Note:** You can read, edit, and save files without purchasing credits — credits only needed when writing

**Face Plate LEDs:**
- **Status LED (RGB):** Solid blue = powering on, medium green = main app running, fast blinking green = USB activity, fast blinking green+magenta = USB+OBD communication, blinking yellow = task running, solid red = exit failed or over-voltage (>15.5V)
- **BLE LED:** Slow blinking blue = pairing mode, fast blinking blue = paired, cyan = connected, solid red = BT failure
- **WiFi LED:** Reserved for future firmware update
- **Standalone Button:** Press to start/stop standalone data logging or initiate Bluetooth pairing. Fast blinking blue = logging started, slow blinking blue = logging in progress, blinking purple = failed to start

**Environmental Specs:** Operating temp -20 to 50°C, altitude 0–2000m, humidity 10–90%. Not intended for exposed use.

**Registration:** Connect via USB → open VCM Scanner → Help > MPVI2 Verification Code → note serial + verification ID → register at hptuners.com/myaccount under My Devices.

**Over-voltage Protection:** Status LED turns solid red when voltage >15.5V. Operating above 15.5V can damage the device. LED returns to green once voltage drops below threshold.

**Overcurrent Protection:** Firmware cycles OBD bus OFF/ON up to 3 times. If current remains high after all attempts, LED turns solid red — requires power cycle or OBD reset to recover.

### PROLINK+ (Accessory Cable)

- Records 2x analog signals + 1x CAN bus signal simultaneously
- Sold separately, compatible with MPVI3/MPVI4 out of the box
- Connects via HPTNET screw-on connector on MPVI4
- **Analog inputs:** Red wire = Analog 1, Blue wire = Analog 2, Black wire = GND. 0–5V range, 100 Hz sampling rate
- **CAN bus input:** Orange wire = CAN High, Yellow wire = CAN Low. 500 kbps CAN bus
- Channel setup configured in VCM Scanner or TDN app after connecting

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
3. Close VCM Scanner if open, open VCM Editor
4. Close any open tune file (File > Close)
5. Turn ignition ON (do not start), all accessories off, doors closed
6. VCM Editor: `Flash > Read Vehicle` → wait 10–15 seconds → click "Read"
7. Wait for completion (instrument panel may flash warnings — normal)
8. Turn off ignition, unplug interface
9. File opens automatically in .HPT format
10. **Save stock file as backup before any changes**

### Editing Calibration

1. Open .HPT file in VCM Editor
2. Navigate parameter tree (Engine > Spark, Fuel, Airflow, Torque Management, etc.)
3. Modify tables using Table Editor tools
4. Use Ford Torque Inverse Calculator when modifying torque model
5. Compare stock vs. modified with Compare tools
6. Save modified .HPT file

### Writing/Flashing

1. VCM Editor: `Flash > Write Vehicle`
2. Ensure battery fully charged, charger connected. Laptop on power.
3. Ignition ON/RUN, don't start engine. Close doors, accessories off.
4. Wait 10–15 seconds, click "Write"
5. **Never power down during a write failure**
6. Do NOT open doors or use accessories during write
7. Do NOT program with battery below 11.5V
8. Always use same file to retry writing
9. Built-in automatic PCM recovery for protection against reflashing problems
10. Wait at least 15 seconds after completion before restarting vehicle

**VCM Recovery (if write fails):**
1. Do NOT pull VCM fuse or disconnect battery
2. Ensure battery + laptop have power, check cables
3. Restart VCM Editor, open the SAME file from the failed write
4. Try Write VCM again
5. If problem persists, contact HP Tuners support

### Real-Time Tuning (RTT)

- Tables can be modified in real time via integrated RTT interface
- Controlled by VCM Scanner
- Changes applied to running engine without full reflash

## Tune Delivery Network (TDN)

- Cloud-based, browser-based platform + iOS/Android app
- Built for professional tuners, shops, and aftermarket companies
- Manage customers, vehicles, receive read files, push tunes
- Compatible with all HP Tuners devices
- MPVI4 connects to TDN app via Bluetooth (no laptop required)

## File Architecture

**Tune Files:** .HPT format — HP Tuners proprietary.

**Scanner Files:**
- Log files: saved scan data (v4.13+ format not backward compatible)
- Channel config: .xml (also auto-saved to `VCM Scanner.cfg`)
- Layouts: save/load named layout files
- Math parameters: stored in `VCM Scanner.cfg`
- Vehicle profiles: auto-created per vehicle, shareable as files

**Scanner stores actively:** Active Channel Configuration, Channel Display Properties, Layout Configuration, Layout Display Properties, User Math Parameters, User Transforms, User Preferences.

## HP Tuners vs. Competitors (Ford Ranger Context)

| Platform | Strengths | Weaknesses |
|----------|-----------|------------|
| **HP Tuners** | Full parameter access, DIY-friendly, extensive datalogging, most popular for custom tuning | More expensive for single vehicle (~4 credits), Windows only |
| **SCT** | Simpler for novices, BDX programmer | Less parameter access, limited drivability calibration control, considered dated |
| **Ford Performance ProCal 4** | Official Ford tool, CARB-legal calibrations | VIN-locked, single vehicle, limited to Ford Performance tunes |
| **Livernois MyCalibrator** | Pre-loaded calibrations, easy to use | Less custom tunability than HP Tuners |
| **nGauge RTD** | Remote tuning without laptop | Depends on tuner for calibration files |

HP Tuners is the recommended platform for custom tuning the Ford Ranger Raptor 3.0L EcoBoost V6 due to full MG1CS036 PCM + 10R80 TCM access and the largest tuner community. HP Tuners achieved first-to-market direct OBDII flashing of the MG1CS036 (no PCM swap/unlock required). VCM Suite support is currently in BETA for MG1CS036 vehicles. HP Tuners' official device support is broader, but this repo standardizes on MPVI4-only workflows.
