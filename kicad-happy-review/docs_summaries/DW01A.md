# DW01A — Application Summary

**Manufacturer:** PingJing Semiconductor (PJ) — also widely second-sourced (FORTUNE/others)
**Type:** Single-cell Li-ion / Li-polymer battery protection IC (overcharge / overdischarge / overcurrent / short-circuit)
**Package:** SOT-23-6
**Datasheet:** DW01A (`docs/components/DW01A/DW01A.md`)

> This summary is a reference for validating a board design that uses the **DW01A**. Packaging, soldering, and mechanical-dimension detail are omitted except where they impose an electrical or layout requirement. The DW01A is a *controller only* — it switches an **external back-to-back dual N-channel MOSFET** (e.g. FS8205A / 8205) that carries the actual battery current; the protection performance depends on both parts together.

---

## 1. Short Description

The DW01A is a protection IC for a **single 1S Li-ion / Li-polymer cell**. It continuously monitors the cell voltage and the voltage across an external dual MOSFET (used as the current-sense element) and drives two gate-control outputs (OC for charge, OD for discharge) to disconnect the cell on any fault:

- **Overcharge protection** (cell rises above 4.28 V)
- **Overdischarge protection** (cell drops below 2.40 V)
- **Discharge overcurrent** and **load short-circuit** protection (sensed as a voltage across the MOSFET R_DS(on))
- **Charge overcurrent** protection
- **Load-detection** and **charger-detection** for releasing fault states
- **"Charge a 0 V battery"** support and very low quiescent current (1.5 µA operating, 0.7 µA in overdischarge)

It is intended for cells of **≤ ~1000 mAh** capacity (per datasheet recommendation). The IC contains no power FETs — it is always used with an external dual MOSFET that forms the charge/discharge switch and the current-sense resistor.

> **Pin-name note:** the datasheet calls the sense pin **CS** in the pin table but uses **VM** in the electrical-characteristics conditions (e.g. "VM−VSS"). CS and VM are the same node — the connection to the P− side of the MOSFET pair.

---

## 2. Electrical Requirements

### 2.1 Absolute Maximum Ratings (T_A = 25 °C)

| Parameter | Symbol | Rating | Unit |
|---|---|---|---|
| Power supply voltage | VCC | −0.3 … 6 | V |
| CS (VM) input pin voltage | V_CS | VCC−15 … VCC+0.3 | V |
| Operating temperature range | T_OPR | −40 … 85 | °C |
| Storage temperature range | T_STG | −55 … 125 | °C |

> Exceeding any absolute-maximum rating may permanently damage the chip. Note the wide negative range allowed on CS relative to VCC (down to VCC−15 V) — this accommodates the large transient seen across the MOSFETs during a short-circuit event.

### 2.2 Operating Conditions & Supply Current

| Parameter | Symbol | Conditions | Min | Typ | Max | Unit |
|---|---|---|---|---|---|---|
| Operating supply voltage | VCC | — | 1.0 | — | 5.5 | V |
| Operating supply current | I_VCC | VCC = 3.5 V | — | 1.5 | 5.0 | µA |
| Power-down (overdischarge) current | I_PD | VCC = 1.5 V | — | 0.7 | 1.5 | µA |
| 0 V-battery charge start voltage | V_0VCH | allow charging to 0 V cell | 1.2 | — | — | V |

### 2.3 Protection Thresholds & Delays (the defining parameters)

| Function | Symbol | Min | Typ | Max | Unit | Test condition |
|---|---|---|---|---|---|---|
| Overcharge protection voltage | V_OC | 4.230 | **4.280** | 4.330 | V | VCC 3.5→4.5 V |
| Overcharge release voltage | V_OCR | 4.030 | **4.080** | 4.130 | V | VCC 4.5→3.5 V |
| Overcharge delay time | T_OC | — | 80 | 160 | ms | — |
| Overdischarge protection voltage | V_OD | 2.300 | **2.400** | 2.500 | V | VCC 3.5→2.0 V |
| Overdischarge release voltage | V_ODR | 2.900 | **3.000** | 3.100 | V | VCC 2.0→3.5 V |
| Overdischarge delay time | T_OD | — | 40 | 80 | ms | — |
| Discharge overcurrent threshold | V_EC | 0.140 | **0.160** | 0.180 | V | sensed VM−VSS |
| Discharge overcurrent delay | T_EC | — | 10 | 20 | ms | — |
| Charge overcurrent threshold | V_CHA | −0.180 | **−0.150** | −0.120 | V | sensed VSS−VM |
| Charge overcurrent delay | T_CHA | — | 10 | 20 | ms | — |
| Short-circuit threshold | V_SHORT | 0.700 | **1.000** | 1.300 | V | sensed VM−VSS |
| Short-circuit delay | T_SHORT | — | 300 | 600 | µs | — |

> **Overcurrent / short-circuit trip current depends on the MOSFET R_DS(on)**, not the DW01A alone. Trip current ≈ V_threshold / R_DS(on)(2 FETs in series). Example: V_EC = 0.16 V across two FETs of ~25 mΩ each (≈50 mΩ total) → discharge overcurrent trips at ≈ 3.2 A. Choose the MOSFET R_DS(on) to set the desired trip current.

---

## 3. Sample Applications & Reference Circuits

### 3.1 Pin Description (SOT-23-6)

| Pin | Symbol | Description |
|---|---|---|
| 1 | OD | Gate drive for the **discharge-control** MOSFET |
| 2 | CS (VM) | Current-sense / charger-detect input — connects to the P− side of the MOSFET pair through R2 |
| 3 | OC | Gate drive for the **charge-control** MOSFET |
| 4 | NC | Not connected |
| 5 | VCC | Supply — connects to cell **B+** through R1 |
| 6 | GND | Ground — connects to cell **B−** |

### 3.2 Typical Application Circuit (Datasheet)

A 1S protection circuit: cell (B+/B−), the DW01A, an external **dual N-MOSFET** (charge + discharge FETs in series, sources common at B−), and three passives. Pack terminals are P+ / P−.

| Component | Connection | Value | Rating range | Purpose |
|---|---|---|---|---|
| **R1** | B+/P+ → VCC (pin 5) | **470 Ω** | 470 Ω … 1.5 kΩ | Supply current-limit / isolation resistor — **mandatory, must be ≥ 470 Ω** |
| **C1** | VCC (pin 5) → GND (pin 6) | **0.1 µF** | ≥ 0.1 µF | VCC decoupling / noise filter — forms RC with R1 |
| **R2** | CS/VM (pin 2) → P− | **2 kΩ** | 1 kΩ … 3 kΩ | CS-pin current limit / sense connection — **mandatory** |

Topology:
- **B+** → P+ (and to VCC via R1).
- **B−** → common source of the dual MOSFET; the FET pair sits in the **negative line** between B− and P−.
- **OD (pin 1)** drives the discharge FET gate, **OC (pin 3)** drives the charge FET gate.
- **CS/VM (pin 2)** connects through R2 to **P−** (the drain node of the MOSFET pair), sensing the voltage developed across the two FETs' R_DS(on) and detecting an attached charger.
- **GND (pin 6)** to B−.

> **Datasheet note: R1 and R2 cannot be omitted, and R1 must be ≥ 470 Ω.** These resistors limit current into the VCC and CS pins during transients (e.g. hot-plug, short-circuit) and are part of the protection — do not remove or reduce below the stated minimums.

### 3.3 Functional Behavior Relevant to the Board

- **Overcharge:** when cell > V_OC for T_OC, OC turns the charge FET off (charging through its body diode still blocked correctly). Released when cell < V_OCR (charger removed) — or via load detection if a load is present.
- **Overdischarge:** when cell < V_OD for T_OD, OD turns the discharge FET off; the IC enters the low-current power-down state. Released when a charger is applied or the cell recovers above V_ODR.
- **Overcurrent / short-circuit:** sensed as V_CS rising; OD turns off. Released by removing the load so V_CS < V_EC.
- **0 V-battery charging:** with a charger above V_0VCH (≥1.2 V), the charge FET turns on so a deeply-discharged cell can be revived; current initially flows through the discharge FET body diode until the cell exceeds V_OD.

---

## 4. Component Placement, Routing & Layout Best Practices

The DW01A datasheet gives no dedicated layout section, but the topology imposes clear layout requirements:

### 4.1 Sense Accuracy (most critical)

- The **CS/VM pin senses millivolt-level drops** across the MOSFET pair (160 mV overcurrent, 1 V short-circuit). The overcurrent trip point depends directly on the MOSFET R_DS(on) and on **Kelvin-style sensing**: route the CS/R2 connection and the IC GND so the sense path picks up the FET drain–source voltage and *not* extra IR drop in the high-current copper.
- Keep R2 (and R1, C1) **close to the IC pins**; the sense node (P−/VM) tap should connect at the MOSFET drain, not somewhere along a high-current trace.

### 4.2 High-Current Path

- The **battery current flows through the dual MOSFET in the B−/P− line**, not through the DW01A. Make the B−–FET–P− copper **wide and short** to carry full charge/discharge and short-circuit current with low resistance and minimal heating. Excess trace resistance here adds to the sensed voltage and shifts the overcurrent trip.
- Place the **dual MOSFET close to the cell B− terminal and the pack P− terminal**.

### 4.3 Decoupling & Pin Protection

- Place **C1 (0.1 µF) directly across VCC–GND** at the IC for noise immunity; R1 + C1 form the supply filter that protects the VCC pin from cell-side transients.
- Maintain **R1 ≥ 470 Ω and R2 = 1–3 kΩ** — these are protection elements, not optional decoupling.

### 4.4 General

- Keep the IC and its three passives as a **compact cluster** near the MOSFET to minimize loop area on the gate-drive (OC/OD) and sense (CS) nets.
- Observe the **−40 … +85 °C** operating range; keep the protection IC away from significant heat sources (e.g. the charger IC or the MOSFET hot-spot under high current).

---

## 5. Design-Validation Checklist (DW01A)

- [ ] Used with a suitable **external dual N-MOSFET** in the B−/P− negative line; sources common at B−.
- [ ] **R1 ≥ 470 Ω** (470 Ω–1.5 kΩ) from B+/P+ to VCC (pin 5) — present and not reduced.
- [ ] **C1 ≥ 0.1 µF** directly across VCC–GND at the IC.
- [ ] **R2 = 1–3 kΩ** from CS/VM (pin 2) to P−; present and not omitted.
- [ ] GND (pin 6) tied to cell **B−**; VCC reaches the cell through R1 only.
- [ ] OC (pin 3) → charge-FET gate, OD (pin 1) → discharge-FET gate (correct FET assignment).
- [ ] **Overcurrent trip current** verified from MOSFET R_DS(on): I_trip ≈ V_EC / (2·R_DS(on)) within the desired range.
- [ ] High-current B−–FET–P− copper is **wide/short**; CS sense node taps at the FET drain (Kelvin) to avoid trace-IR error.
- [ ] Cell capacity within the recommended range (≤ ~1000 mAh) or trip currents re-checked accordingly.
- [ ] VCC never exceeds **6 V** abs-max (and CS within VCC−15 … VCC+0.3 V); operating temperature within −40 … +85 °C.
- [ ] Pin 4 (NC) left unconnected.

---

*Sources: DW01A datasheet (PingJing Semiconductor) — Description & Features, Typical Application Circuit (R1/R2/C1 table and figure), Functional Pin Description, Block Diagram, Absolute Maximum Ratings, Electrical Characteristics, and Description of Operation (overcharge / overdischarge / overcurrent / short-circuit / 0 V-charge). MOSFET-dependent trip-current behavior is inherent to the protection topology and must be evaluated with the specific external dual MOSFET used on the board.*
