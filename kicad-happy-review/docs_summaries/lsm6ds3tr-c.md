# LSM6DS3TR-C — Application Summary

**Manufacturer:** STMicroelectronics
**Type:** iNEMO 6-axis inertial module — always-on 3D digital accelerometer + 3D digital gyroscope (system-in-package)
**Package:** LGA-14L, 2.5 mm × 3.0 mm × 0.83 mm
**Interfaces:** SPI (3- & 4-wire) and I²C slave; optional I²C master ("sensor hub") for up to 4 external sensors
**Datasheet:** DocID030071 Rev 3 (`docs/components/lsm6ds3tr-c/lsm6ds3tr-c.md`)

> This summary is a reference for validating a board design that uses the **LSM6DS3TR-C**. It covers electrical, application-circuit, and layout considerations only. Packaging, mechanical drawings, soldering, and the (very large) register map / software programming model are intentionally omitted, except where a programming choice forces a specific electrical configuration (noted inline). Register-level configuration of ODR, full-scale, filters, FIFO, and embedded functions is done entirely over the serial bus and has no external-component impact.

---

## 1. Short Description

The LSM6DS3TR-C is a **6-axis MEMS inertial measurement unit (IMU)** combining a 3-axis accelerometer and a 3-axis gyroscope in one LGA-14 package. It is designed for "always-on" motion sensing at very low power — **0.90 mA** for the combined accel+gyro in high-performance mode — and is Android-compliant.

Key application-relevant traits:

- **Full-scale ranges:** accelerometer ±2 / ±4 / ±8 / ±16 g; gyroscope ±125 / ±250 / ±500 / ±1000 / ±2000 dps (all software-selectable).
- **Output data rates** up to 6.66 kHz for both sensors; independent ODR and power mode per sensor.
- **Independent supplies:** an analog/core supply `Vdd` (1.71–3.6 V) and a **separate I/O supply `Vdd_IO`** (down to 1.62 V) so the digital bus can run at a different (lower) rail than the core — useful for level-matching a 1.8 V host.
- **4 kbyte smart FIFO** with dynamic accel/gyro/external/timestamp batching, letting the host stay asleep and burst-read.
- **Embedded hardware features** (no external parts): pedometer / step detector / step counter, significant-motion, tilt, absolute wrist tilt, free-fall, wake-up, 6D/4D orientation, single/double-tap, activity/inactivity — each routable to **INT1** or **INT2**.
- **Two pin-connection modes:** *Mode 1* (host bus only) and *Mode 2* (host bus **plus** an on-chip I²C master to read external sensors such as a magnetometer; supports hard/soft-iron correction).
- **Two device-level interrupt pins** (INT1, INT2) plus optional DEN data-enable and sensor-sync features.

**Typical applications:** pedometer/step counting, indoor navigation, tap/double-tap UI, IoT/connected devices, intelligent power saving in handhelds, vibration monitoring, free-fall and 6D-orientation detection.

---

## 2. Electrical Requirements

### 2.1 Absolute Maximum Ratings (stress limits — do not operate here)

| Symbol | Rating | Max value | Unit |
|---|---|---|---|
| `Vdd` | Supply voltage | **−0.3 to 4.8** | V |
| `Vin` | Input voltage on any control pin (`CS`, `SCL/SPC`, `SDA/SDI/SDO`, `SDO/SA0`) | **−0.3 to Vdd_IO + 0.3** | V |
| `T_STG` | Storage temperature | −40 to +125 | °C |
| `Sg` | Mechanical shock (acceleration, 0.2 ms) | 10 000 | g |
| `ESD` | ESD protection (HBM) | 2 | kV |

> **Supply voltage on any pin must never exceed 4.8 V.** Note the control-pin inputs are referenced to **`Vdd_IO`**, not `Vdd` — never drive a logic pin above `Vdd_IO + 0.3 V`. The part is sensitive to mechanical shock and ESD; handle accordingly.

### 2.2 Recommended / Operating Conditions

| Symbol | Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|---|
| `Vdd` | Core/analog supply voltage | **1.71** | 1.8 | **3.6** | V |
| `Vdd_IO` | I/O supply voltage | **1.62** | — | **Vdd + 0.1** | V |
| `Top` | Operating temperature range | −40 | — | +85 | °C |

> `Vdd_IO` may be equal to or **lower** than `Vdd` (up to `Vdd + 0.1 V` max). It can also be tied to `Vdd` if a single rail is acceptable. The 1.62 V floor lets the bus interface a 1.8 V (or 3.3 V) host directly.

### 2.3 Key Electrical Characteristics (Vdd = 1.8 V, T = 25 °C unless noted)

| Symbol | Parameter | Test condition | Typ | Unit |
|---|---|---|---|---|
| `IddHP` | Accel + gyro current, high-performance mode | ODR = 1.6 kHz | **0.90** | mA |
| `IddNM` | Accel + gyro current, normal mode | ODR = 208 Hz | 0.45 | mA |
| `IddLP` | Accel + gyro current, low-power mode | ODR = 52 Hz | 0.29 | mA |
| `LA_IddHP` | Accel-only, high-performance | ODR < 1.6 kHz / ≥ 1.6 kHz | 150 / 160 | µA |
| `LA_IddNM` | Accel-only, normal mode | ODR = 208 Hz | 85 | µA |
| `LA_IddLM` | Accel-only, low-power mode | ODR = 12.5 Hz | 9 | µA |
| `IddPD` | Power-down current (both sensors off) | — | 3 | µA |
| `Ton` | Turn-on time | — | 35 | ms |

### 2.4 Digital I/O Levels (referenced to `Vdd_IO`)

| Symbol | Parameter | Min | Max | Unit |
|---|---|---|---|---|
| `V_IH` | High-level input voltage | 0.7 × Vdd_IO | — | V |
| `V_IL` | Low-level input voltage | — | 0.3 × Vdd_IO | V |
| `V_OH` | High-level output voltage (I_OH = 4 mA) | Vdd_IO − 0.2 | — | V |
| `V_OL` | Low-level output voltage (I_OL = 4 mA) | — | 0.2 | V |

> Max DC drive/sink on a digital pad is **4 mA**. Logic thresholds scale with `Vdd_IO`, so the chosen I/O rail must be compatible with the host's logic levels.

### 2.5 Sensor Performance Highlights (typical, from Mechanical Characteristics)

| Quantity | Value | Notes |
|---|---|---|
| Accel sensitivity | 0.061 / 0.122 / 0.244 / 0.488 mg/LSB | for ±2 / ±4 / ±8 / ±16 g |
| Gyro sensitivity | 4.375 / 8.75 / 17.5 / 35 / 70 mdps/LSB | for ±125…±2000 dps |
| Accel zero-g offset accuracy | ±40 mg | after factory trim |
| Gyro zero-rate level | ±3 dps | after factory trim |
| Accel noise density (HP mode) | 90 µg/√Hz (130 at ±16 g) | independent of ODR |
| Gyro rate noise density (HP mode) | 5 mdps/√Hz | independent of ODR/FS |
| Accel/gyro ODR range | 1.6 / 12.5 Hz … 6.66 kHz | software-selectable |

### 2.6 Temperature Sensor

| Symbol | Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|---|
| `TSen` | Temperature sensitivity | — | 256 | — | LSB/°C |
| `Toff` | Temperature offset (0 LSB ≈ 25 °C) | −15 | — | +15 | °C |
| `TODR` | Refresh rate | — | 52 | — | Hz |
| `T_ADC_res` | ADC resolution | — | 16 | — | bit |

### 2.7 Serial-Interface Timing (key limits)

| Interface | Parameter | Limit |
|---|---|---|
| **SPI (3- & 4-wire)** | Max clock `f_c(SPC)` | **10 MHz** |
| **I²C slave** | Clock `f_(SCL)` | 0–100 kHz (standard) / 0–400 kHz (fast) |
| **I²C master** (Mode 2) | Clock `f_(SCL)` | **Fast mode only**, ~116 kHz typ (400 kHz max) |

> SPI measurement points are at 0.2·Vdd_IO and 0.8·Vdd_IO. SPI supports both 4-wire (separate SDI/SDO) and 3-wire (shared SDA = SDIO) operation, selectable via the `SIM` bit.

---

## 3. Sample Applications & Reference Circuits

### 3.1 Pinout (LGA-14, top view)

| Pin | Name | Mode 1 function | Mode 2 function |
|---|---|---|---|
| 1 | `SDO/SA0` | SPI 4-wire data out (SDO) / I²C address LSB (SA0) | same |
| 2 | `SDx` | **Tie to Vdd_IO or GND** | I²C master serial data (MSDA) |
| 3 | `SCx` | **Tie to Vdd_IO or GND** | I²C master serial clock (MSCL) |
| 4 | `INT1` | Programmable interrupt 1 | same |
| 5 | `Vdd_IO` | I/O supply | same — **100 nF decoupling** |
| 6 | `GND` | 0 V supply | same |
| 7 | `GND` | 0 V supply | same |
| 8 | `Vdd` | Core/analog supply | same — **100 nF decoupling** |
| 9 | `INT2` | Programmable interrupt 2 / Data-enable (DEN) | INT2 / DEN / I²C-master sync (MDRDY) |
| 10 | `NC` | **Leave unconnected, soldered to PCB** | same |
| 11 | `NC` | **Leave unconnected, soldered to PCB** | same |
| 12 | `CS` | I²C/SPI select (1 = SPI idle / I²C enabled; 0 = SPI mode / I²C disabled) | same |
| 13 | `SCL/SPC` | I²C clock (SCL) / SPI clock (SPC) | same |
| 14 | `SDA/SDI/SDO` | I²C data / SPI data-in / 3-wire data-out | same |

**Internal pin states worth knowing for design:**
- Pins **10 & 11 (NC)** have an **internal pull-up enabled by default** (30–50 kΩ, depends on Vdd_IO). Leave them electrically open but soldered. (Pull-up can be disabled via a register sequence if leakage matters.)
- **`CS` (pin 12)** is **input with an internal pull-up by default** → the device powers up in I²C mode unless `CS` is actively driven low for SPI.
- `SDO/SA0` (pin 1) is input without pull-up by default (a pull-up enables only in SPI 3-wire mode via `SIM`).
- `SDx`/`SCx` (pins 2/3) are inputs without pull-up by default; an internal pull-up can be enabled (`PULL_UP_EN`, reg 1Ah) for the Mode-2 master bus.
- `INT1`/`INT2` default to **output forced to ground** until configured.
- `SCL`/`SDA` (pins 13/14) are inputs without internal pull-up — **external pull-ups required for I²C**.

### 3.2 Interface Selection / Strapping

| Goal | Required connection |
|---|---|
| **I²C mode** | Tie `CS` (pin 12) **high to Vdd_IO**. Add external pull-ups (≈10 kΩ) on `SCL` and `SDA` to Vdd_IO. |
| **SPI mode (4-wire)** | Drive `CS` from host; data out on `SDO` (pin 1), data in on `SDA/SDI` (pin 14). |
| **SPI mode (3-wire)** | `SDA/SDIO` (pin 14) is bidirectional; set `SIM` bit. |
| **I²C 7-bit address** | `110101xb`. `SDO/SA0` = GND → `1101010b` (0x6A); `SDO/SA0` = Vdd_IO → `1101011b` (0x6B). Allows two IMUs on one bus. |
| **Mode 1 (no sensor hub)** | Pins `SDx`/`SCx` (2/3) **must be tied to Vdd_IO or GND** — do not leave floating. |
| **Mode 2 (sensor hub)** | `SDx`/`SCx` become the master I²C bus (MSDA/MSCL) to external sensors; `INT2` can carry MDRDY sync. |

### 3.3 Reference Connection — Mode 1, I²C (datasheet Fig. 15)

| Component | Connection | Value | Purpose |
|---|---|---|---|
| C1 | `Vdd` (pin 8) → GND | 100 nF ceramic | Core-supply decoupling |
| C2 | `Vdd_IO` (pin 5) → GND | 100 nF ceramic | I/O-supply decoupling |
| R_pu | `SCL` (pin 13) → Vdd_IO | ~10 kΩ | I²C clock pull-up |
| R_pu | `SDA` (pin 14) → Vdd_IO | ~10 kΩ | I²C data pull-up |
| — | `CS` (pin 12) → Vdd_IO | — | Select I²C |
| — | `SDO/SA0` (pin 1) → Vdd_IO or GND | — | Set I²C address LSB |
| — | `SDx`, `SCx` (pins 2,3) → Vdd_IO or GND | — | Unused master bus, must not float |
| — | `NC` (pins 10,11) | open | Soldered, not connected |
| — | `INT1`/`INT2` → host GPIO | — | Optional interrupt/DRDY/event lines |

### 3.4 Reference Connection — Mode 2, sensor hub (datasheet Fig. 16)

Same supply decoupling (2 × 100 nF) and host-bus pull-ups as Mode 1. Additionally:
- `SDx` = **MSDA**, `SCx` = **MSCL** route to the external sensor(s) (e.g. a magnetometer); add pull-ups on that master bus to Vdd_IO.
- `INT2` may be used as **MDRDY** (external-sensor synchronization).
- The on-chip I²C master runs **fast mode only**; external sensors are read into the SENSORHUB registers / FIFO with optional hard- and soft-iron correction for a connected magnetometer.

### 3.5 Notes Relevant to the Application

- Both supplies (`Vdd`, `Vdd_IO`) each want a **100 nF** decoupling cap placed as close as possible to their pins.
- Turn-on time is ~35 ms; allow this before expecting valid data after power-up.
- If `Vdd_IO` and `Vdd` come from different rails, observe no specific sequencing requirement in the datasheet, but `Vdd_IO ≤ Vdd + 0.1 V` must hold at all times — including during ramp.

---

## 4. Component Placement, Routing & Layout Best Practices

The datasheet's application section is brief on layout (it gives decoupling and pin-handling rules); the points below combine those explicit rules with standard MEMS-IMU practice that the datasheet's mechanical/terminology notes justify.

- **Decoupling caps adjacent to pins:** Place the **`Vdd` (pin 8)** and **`Vdd_IO` (pin 5)** 100 nF ceramics **as near as possible** to their respective pins with short, low-impedance returns to the GND pins (6, 7) — explicitly called out in the datasheet.
- **Solid ground under the part:** Tie both GND pins (6, 7) to a continuous ground pour. Keep the return path short and avoid splitting ground under the device.
- **No floating control pins:** In Mode 1, **`SDx`/`SCx` (pins 2, 3) must be tied to Vdd_IO or GND** (the datasheet is explicit). `CS` must be at a defined level (high for I²C, host-driven for SPI). Don't rely on internal pull-ups for noise-critical lines.
- **NC pins:** Leave **pins 10 & 11 electrically unconnected but soldered** to the PCB for mechanical strength (do not route signals to them; they carry an internal pull-up).
- **I²C pull-ups:** Provide external pull-ups (≈10 kΩ typical) on `SCL`/`SDA` to `Vdd_IO`; size for bus capacitance and the 100/400 kHz mode in use. Mode-2 master bus needs its own pull-ups.
- **SPI routing:** Keep `SPC` (clock) ≤ 10 MHz routed cleanly; the host should reference 0.2/0.8·Vdd_IO thresholds. Keep `CS` clean to avoid spurious mode switches.
- **Mechanical-stress / mounting (MEMS-specific):** Zero-g offset and zero-rate level **can shift after the sensor is soldered to the PCB or exposed to board stress** (datasheet §4.6.2). To preserve accuracy:
  - Mount the device on a **mechanically stable, low-flex region** of the board; keep it away from board-edge flex, mounting screws, connectors, and high-stress zones.
  - Avoid placing the IMU near components that warp the board with heat or near large thermal gradients (offset drifts with temperature: accel ±0.5 mg/°C, gyro ±0.05 dps/°C).
  - Follow the recommended pad/stencil/soldering profile so post-reflow stress is minimized (mechanical/soldering detail is in the datasheet's soldering & package sections — out of scope here).
- **Axis orientation:** Note the package's defined sense axes (top-view acceleration/angular-rate directions, datasheet §3) and orient the footprint so the board's X/Y/Z match the system's intended frame — easier to fix in layout than in firmware.
- **Keep the IMU away from vibration/noise sources** (motors, switching inductors, speakers) unless vibration sensing is the intent; mechanical coupling shows up directly in the data.

---

## 5. Design-Validation Checklist (LSM6DS3TR-C)

- [ ] `Vdd` within **1.71–3.6 V**; `Vdd_IO` within **1.62 V to Vdd + 0.1 V** at all times (incl. ramp); neither pin ever exceeds 4.8 V absolute max.
- [ ] Logic-pin drive levels referenced to `Vdd_IO` (V_IH ≥ 0.7·Vdd_IO, V_IL ≤ 0.3·Vdd_IO); host I/O rail compatible.
- [ ] **100 nF** decoupling on **both** `Vdd` (pin 8) and `Vdd_IO` (pin 5), placed close to the pins.
- [ ] Both `GND` pins (6, 7) tied to a solid ground plane.
- [ ] Interface strapping correct: I²C → `CS` tied to Vdd_IO + 10 kΩ pull-ups on SCL/SDA; SPI → `CS` host-driven, clock ≤ 10 MHz.
- [ ] `SDO/SA0` set to choose the I²C address LSB (0x6A vs 0x6B); no address clash on the bus.
- [ ] **Mode 1:** `SDx`/`SCx` (pins 2, 3) tied to Vdd_IO or GND — **not floating**. **Mode 2:** master-bus pull-ups present and external sensor addressed.
- [ ] `NC` pins 10 & 11 left open but soldered; no traces routed to them.
- [ ] `INT1`/`INT2` routed to host GPIOs as needed (default = output low until configured).
- [ ] IMU placed on a low-stress, low-flex board region away from mounting hardware, board edges, heat sources, and vibration; footprint axes oriented to the system frame.
- [ ] Allow ~35 ms turn-on time before reading valid data.

---

*Sources: LSM6DS3TR-C datasheet (STMicroelectronics, DocID030071 Rev 3) — Features/Applications/Description, §3 Pin description (Table 2), §4.1 Mechanical characteristics (Table 3), §4.2 Electrical characteristics (Table 4), §4.3 Temperature sensor (Table 5), §4.4 Communication interface timing (Tables 6–8), §4.5 Absolute maximum ratings (Table 9), §4.6 Terminology (offset/stress notes), §6 Digital interfaces (I²C address, SPI/I²C selection), §7 Application hints (Fig. 15 Mode 1, Fig. 16 Mode 2, Table 18 Internal pin status).*
