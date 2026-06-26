# SHT4x (SHT45) — Application Summary

**Manufacturer:** Sensirion AG
**Type:** 4th-generation digital relative-humidity & temperature sensor with I²C interface and integrated heater
**Package:** 4-pin open-cavity DFN (1.5 mm × 1.5 mm), optional PTFE membrane (IP67) or removable protective cover
**Datasheet:** Datasheet SHT4x, Rev. 7.1 (March 2025) (`docs/components/SHT4x_5/HT_DS_Datasheet_SHT4x_5.md`)
**Design guide:** Design Guide for Humidity and Temperature Sensors, Rev. 2 (March 2024) (`docs/components/SHT4x_5/Sensirion_Humidity_Temperature_Design_Guide/…`)

> This summary is a reference for validating a board design that uses an **SHT4x-family** sensor. The KiCad symbol `SHT4x_5` corresponds to the **SHT45** (best-accuracy, ±1.0 %RH / ±0.1 °C, I²C address 0x44) variant; family-wide values are noted where they apply and SHT45-specific values are flagged. Packaging, tape-and-reel, soldering, mechanical-dimension, and protocol/firmware detail are omitted except where they impose an electrical or layout requirement on the board.

---

## 1. Short Description

The SHT4x is an ultra-low-power **digital humidity and temperature sensor** built on a capacitive polymer RH element and a band-gap temperature sensor, with on-chip CMOSens® signal conditioning, A/D conversion, and an **I²C Fast-Mode-Plus** interface. It outputs calibrated, fully linearized, temperature-compensated RH and T values (16-bit each, CRC-protected) — no analog front-end or ADC is required on the host.

Defining features for board design:

- **Wide single-rail supply:** VDD = 1.08 V … 3.6 V (3.3 V typ), so it runs directly from most logic rails or a coin cell.
- **Extremely low power:** ~0.4 µA average at 1 measurement/s (low repeatability), 80 nA idle — suited to battery / energy-harvesting designs.
- **Integrated power-trimmed heater** (20 / 110 / 200 mW, 0.1 s or 1 s pulses) for condensation removal and creep mitigation in high-humidity environments. The heater draws **up to ~75–100 mA**, which is the dominant supply-design constraint if used.
- **Multiple fixed I²C addresses** (0x44 / 0x45 / 0x46, set by ordering option) allow several sensors on one bus.
- **No clock stretching**; returns NACK while busy.

**Typical applications:** HVAC and building automation, indoor air-quality monitors, smart-home/IoT devices, RH/T data loggers and trackers, white goods, wearables, medical/lab instruments, and any application needing accurate ambient humidity/temperature with a small, low-power digital sensor.

---

## 2. Electrical Requirements

### 2.1 Absolute Maximum Ratings (stress limits — do not operate here)

| Parameter | Rating |
|---|---|
| Voltage on any pin | VSS − 0.3 V … VDD + 0.3 V |
| Operating temperature range | −40 °C … +125 °C |
| Storage temperature range | −40 °C … +150 °C |
| ESD HBM | 2 kV |
| ESD CDM | 500 V |
| Latch-up (JESD78 Class II, 125 °C) | ±100 mA |

> Stress beyond these limits may cause permanent damage; ratings are tested one at a time. The part is **ESD-sensitive** — apply standard ESD precautions in handling and assembly.

### 2.2 Recommended Operating Conditions

| Parameter | Symbol | Min | Typ | Max | Unit | Notes |
|---|---|---|---|---|---|---|
| Supply voltage | V_DD | 1.08 | 3.3 | 3.6 | V | — |
| Power-on / brown-out threshold | V_POR | 0.6 | — | 1.08 | V | Static supply; sensor below this is reset |
| Supply-voltage slew rate | V_DD,slew | — | — | 20 | V/ms | Faster ramps may cause a reset |
| Specified RH range | — | 0 | — | 100 | %RH | Functional incl. condensing |
| Specified T range | — | −40 | — | +125 | °C | — |
| **Recommended normal range** (best performance) | — | 5 … 60 °C, 20 … 80 %RH | | | | Prolonged operation outside (esp. high RH) can offset RH (e.g. +3 %RH after 60 h > 80 %RH); recovers on return |
| I²C clock | f_SCL | 0 | — | 1000 | kHz | Standard / Fast / Fast-Mode-Plus |

### 2.3 DC Electrical Characteristics

| Parameter | Symbol | Conditions | Min | Typ | Max | Unit |
|---|---|---|---|---|---|---|
| Low-level input voltage | V_IL | — | 0 | — | 0.3·V_DD | V |
| High-level input voltage | V_IH | — | 0.7·V_DD | — | V_DD | V |
| Low-level output voltage | V_OL | V_DD < 1.62 V, R_p > 820 Ω | — | — | 0.2·V_DD | V |
| | | V_DD = 1.62–2.0 V, R_p > 390 Ω | — | — | 0.2·V_DD | V |
| | | V_DD > 2.0 V, R_p > 390 Ω | — | — | 0.4 | V |

### 2.4 Supply Current

| Condition | Symbol | Typ | Max | Unit |
|---|---|---|---|---|
| Idle, 25 °C | I_DD | 0.08 | 1.0 | µA |
| Idle, 125 °C | I_DD | — | 3.4 | µA |
| Power-up | I_DD | 50 | — | µA |
| Measuring | I_DD | 320 | 500 | µA |
| Average, 1 meas/s — high repeatability | I_DD | 2.2 | — | µA |
| Average, 1 meas/s — medium repeatability | I_DD | 1.2 | — | µA |
| Average, 1 meas/s — low repeatability | I_DD | 0.4 | — | µA |
| **Heater on — "200 mW"** | I_DD | 60 | **100** | mA |
| **Heater on — "110 mW"** | I_DD | 33 | 55 | mA |
| **Heater on — "20 mW"** | I_DD | 6 | 10 | mA |

> Heater power values are typical at V_DD = 3.3 V. The heater is the **only high-current load** on this part — a supply that sources the measurement current (≤ 0.5 mA) is otherwise sufficient.

### 2.5 I²C Pull-up & Bus-Load Requirements (mandatory)

These are **not optional application notes** — the datasheet specifies minimum pull-up resistance vs. supply, and maximum bus capacitance vs. pull-up:

| Parameter | Symbol | Condition | Limit | Unit |
|---|---|---|---|---|
| Pull-up resistor (min) | R_p | V_DD < 1.62 V | ≥ 820 | Ω |
| Pull-up resistor (min) | R_p | V_DD ≥ 1.62 V | ≥ 390 | Ω |
| Capacitive bus load (max) | C_b | R_p ≤ 820 Ω, Fast Mode (t_rise = 300 ns) | 400 | pF |
| Capacitive bus load (max) | C_b | R_p = 390 Ω, V_DD > 1.62 V, Fast Mode Plus (t_rise = 120 ns) | 340 | pF |

> Bus load and pull-up are linked by C_b < t_rise / (0.8473·R_p). The typical application circuit (Fig. 1) uses **10 kΩ** pull-ups on SDA and SCL at 3.3 V — acceptable for short, low-capacitance single-board buses at Standard/Fast Mode; lower R_p is required to meet Fast-Mode-Plus rise times. The sensor does **not** support clock stretching.

### 2.6 Key Timing

| Parameter | Symbol | Typ | Max | Unit |
|---|---|---|---|---|
| Power-up time (V_DD ≥ V_POR → idle) | t_PU | 0.3 | 1 | ms |
| Soft-reset time | t_SR | — | 1 | ms |
| Measurement duration — low repeatability | t_MEAS,l | 1.3 | 1.6 | ms |
| Measurement duration — medium repeatability | t_MEAS,m | 3.7 | 4.5 | ms |
| Measurement duration — high repeatability | t_MEAS,h | 6.9 | 8.3 | ms |
| Heater-on duration — long pulse | t_Heater | 1.0 | 1.1 | s |
| Heater-on duration — short pulse | t_Heater | 0.1 | 0.11 | s |

> The host must allow **t_PU = 1 ms max** after the supply crosses V_POR before issuing I²C traffic, and must wait the measurement duration (or poll for ACK) before reading.

### 2.7 Sensor Performance Highlights (SHT45 unless noted)

| Parameter | SHT45 typ | Family notes | Unit |
|---|---|---|---|
| RH accuracy | ±1.0 | SHT40/41/43: ±1.8 typ | %RH |
| T accuracy | ±0.1 | SHT40/41: ±0.2 typ | °C |
| RH resolution / T resolution | 0.01 / 0.01 | — | %RH / °C |
| RH hysteresis (25 °C) | ±0.8 | — | %RH |
| RH response time τ₆₃ | 4 | depends on design-in/airflow | s |
| T response time τ₆₃ | 2 | depends on thermal mass | s |
| RH long-term drift | < 0.2 | — | %RH/yr |
| T long-term drift | < 0.03 | SHT43: < 0.01 | °C/yr |

---

## 3. Sample Applications & Reference Circuits

### 3.1 Pin Assignment

| Pin | Name | Function |
|---|---|---|
| 1 | SDA | Serial data, bidirectional (open-drain) |
| 2 | SCL | Serial clock, input |
| 3 | VDD | Supply voltage |
| 4 | VSS | Ground |
| — | Die pad | Center pad — **not connected to any pin; do not solder / no copper** (see §4) |

### 3.2 Typical Application Circuit (Datasheet Fig. 1)

The reference circuit is minimal — three external passives:

| Component | Connection | Value | Purpose |
|---|---|---|---|
| C_DEC | VDD → VSS | **100 nF** | Supply decoupling, placed close to the sensor |
| R_SDA | SDA → VDD | 10 kΩ (typ; ≥ 390 Ω at V_DD ≥ 1.62 V) | I²C data pull-up |
| R_SCL | SCL → VDD | 10 kΩ (typ; ≥ 390 Ω at V_DD ≥ 1.62 V) | I²C clock pull-up |

Connections:
- **VDD** to the logic/supply rail (1.08–3.6 V) with the 100 nF decoupling cap adjacent.
- **VSS** to ground.
- **SDA / SCL** to the host I²C master, each pulled up to VDD.
- After V_DD reaches V_POR and t_PU (≤ 1 ms) elapses, the sensor is ready: write a measurement command (e.g. `0xFD` high precision), wait the measurement duration, then read 6 bytes (T, CRC, RH, CRC).

### 3.3 Supply Design When the Heater Is Used

If the integrated heater is used, the supply must tolerate the heater current without browning the sensor out:

- Heater current is **up to ~75 mA (200 mW setting)**, transient by design but enough to cause a reset if the rail sags below V_POR.
- The **power supply must be stiff** at the sensor — size bulk/decoupling capacitance and trace width so V_DD stays > V_POR (and within slew limits) during heater pulses.
- The heater is limited to a **≤ 10 % duty cycle** and a max 1 s on-time per command (auto-off); extended heating needs periodic re-triggering.
- Heater must only be operated at **ambient < 65 °C**, and total sensor temperature must never exceed 125 °C.
- **Battery-powered / low-power designs should avoid heater use** to preserve the µA-class power budget — budget for it explicitly only if condensation/creep mitigation is required.

### 3.4 Connection / Configuration Requirements Driven by Function

- **Fixed I²C address per ordering option** (0x44 / 0x45 / 0x46). One address per bus segment — use the address-variant ordering codes (suffix A/B/C) to place multiple SHT4x on one bus, or a mux otherwise.
- **No clock stretching** — the host I²C master must be configured for a non-clock-stretching slave; the sensor NACKs a read header while busy rather than stretching.
- **CRC-8** (poly 0x31, init 0xFF) on every 16-bit word — host should verify, but no board impact.
- **Reset options:** soft reset (`0x94`), I²C general-call reset (`0x06` to addr `0x00`), or power-down (pull SCL/SDA low). If sharing the bus, be aware a general-call reset affects all devices.

---

## 4. Component Placement, Routing & Layout Best Practices

### 4.1 Land Pattern / Die Pad (datasheet — mandatory)

- **Do not solder the central die pad, and place no copper (no exposed pad) underneath the sensor** other than the four pin pads. A soldered/coppered center pad acts as a **heat sink that prevents the heater from working to spec** and increases thermal coupling to the board.
- Use the Sensirion-recommended land pattern (Fig. 17); pin pads only.

### 4.2 Thermal Decoupling (design guide — critical for accuracy)

RH is strongly temperature-dependent: at 90 %RH a **1 °C** error produces ~**5 %RH** error. Heat conduction through the PCB from nearby hot components is the dominant error source.

- **Keep the sensor away from heat sources** — MCUs, regulators, power devices, displays, and self-heating loads (e.g. charging circuitry).
- **Thermally isolate via the copper:** keep copper connections to the sensor **thin/narrow**, remove unnecessary copper around it, and add **milled slots / etched slits** in the PCB around the sensor to break heat-conduction paths.
- For demanding cases, mount the sensor on a **flex PCB** or a thin PCB neck to minimize conducted heat.
- **Shield from heated airflow** — place the sensor so heated air from other components does not flow over it; use a wall/compartment with an opening to the environment.

### 4.3 Exposure & Response Time (housing/design-in)

- Place the sensor **as exposed to the measured environment and as isolated from the device as possible**, ideally with ambient airflow over it.
- **Minimize dead volume** between aperture and sensor and **maximize aperture size** for fast humidity response; compartmentalize to keep the sensing volume small.
- The on-package **PTFE membrane** (IP67) protects against dust/water with negligible response-time penalty; the **protective cover** (polyimide) is removed after assembly (e.g. after conformal coating).

### 4.4 I²C / Signal Integrity (design guide)

- Keep **SDA/SCL traces short**, avoid sharp bends, and keep them away from switching nodes; respect the **C_b limit** (≤ 340–400 pF) when choosing pull-up value.
- Ensure a **common, solid ground** between the sensor and the I²C master to minimize noise.
- For high-EMI environments or long runs, use shielding / I²C buffers/isolators; I²C is intended for short on-board distances.

### 4.5 Power & Protection

- Place the **100 nF decoupling cap directly at VDD/VSS** with a short return loop.
- Observe the **20 V/ms supply slew-rate** limit — avoid abrupt rail steps that could reset the sensor.
- Maintain standard **ESD precautions**; for an exposed sensor consider a wide-mesh metallic Faraday-cage enclosure where the housing offers no ESD protection.

---

## 5. Design-Validation Checklist (SHT4x / SHT45)

- [ ] V_DD stays within **1.08–3.6 V** across all operating conditions; supply slew ≤ 20 V/ms; rail stays > V_POR (1.08 V) during any heater pulse.
- [ ] **100 nF** decoupling cap placed directly at VDD/VSS, short return.
- [ ] SDA and SCL pull-ups present to VDD; **R_p ≥ 390 Ω** (V_DD ≥ 1.62 V) / **≥ 820 Ω** (V_DD < 1.62 V); bus capacitance within C_b limit; bus idles high.
- [ ] Host I²C master configured for **no clock stretching**; correct fixed slave address (0x44/0x45/0x46) — one per bus segment.
- [ ] Central **die pad NOT soldered**; **no copper under the sensor** except the four pin pads; Sensirion land pattern used.
- [ ] Sensor **thermally decoupled** from heat sources: thin copper, slots/slits or flex neck, kept clear of MCU/regulator/charger heat.
- [ ] Sensor **exposed to the measured environment** with adequate aperture / minimal dead volume / airflow; membrane or protective cover selected per environment.
- [ ] If the **heater is used:** supply sized for ≤ ~75–100 mA pulses, ≤ 10 % duty cycle, ambient < 65 °C, sensor T ≤ 125 °C; battery designs avoid heater use unless required.
- [ ] Host firmware allows **t_PU ≤ 1 ms** after power-up and the per-command measurement time before reading; verifies CRC-8.
- [ ] Standard **ESD handling** in assembly; exposed-sensor designs consider a Faraday-cage mesh.

---

*Sources: SHT4x Datasheet (Sensirion, Rev. 7.1, March 2025) — Highlights, Quick Start Guide / Typical Application Circuit (Fig. 1), Sensor Specifications (Tables 1–2), Recommended Operating Conditions (§2.3), Electrical Specifications (Table 4), Timings (Table 5), Absolute Maximum Ratings (Table 6), I²C Communication (§4.1–4.9), Heater Operation (§4.9, Table 9), Land Pattern (§5.3), Pin Assignment (§5.4), Thermal Information (§5.5), Protection Options (§6); Sensirion Humidity & Temperature Design Guide (Rev. 2, March 2024) — Thermal Considerations (§3), Electrical & Signal / I²C Considerations (§4), Housing & PCB-Design Integration (§5), Examples (§7).*
