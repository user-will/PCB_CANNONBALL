# USBLC6-4 — Application Summary

**Manufacturer:** STMicroelectronics · **Order code:** USBLC6-4SC6 · **Marking:** UL46
**Package:** SOT23-6L · **Type:** Very-low-capacitance, 4-line + VBUS ESD protection array
*Source: ST datasheet `usblc6-4.md` (DocID11068 Rev 7) and embedded figures.*

---

## 1. Short Description

The USBLC6-4 is a monolithic, application-specific ESD-protection device dedicated to **high-speed serial interfaces** (USB 2.0, Ethernet, video lines). It clamps transient surges (ESD) on **four data I/O lines** and on the **VBUS supply line** using a *rail-to-rail* steering-diode topology with an internal Zener clamp referenced to GND.

Its key design property is **very low line capacitance (3 pF typ.)** with tightly matched I/O channels, which preserves signal integrity on data lines up to **480 Mb/s (USB 2.0 High Speed)** while diverting ESD energy to ground. It guarantees **IEC 61000-4-2 level 4** immunity at the device level (15 kV air / 8 kV contact).

**Role on a board:** placed at the connector edge to absorb ESD strikes before they reach sensitive transceiver / SoC pins, without degrading the protected high-speed signals.

---

## 2. Pinout & Internal Topology

SOT23-6L, top view (per functional diagram):

| Pin | Name  | Function |
|-----|-------|----------|
| 1   | I/O1  | Protected data line (e.g. D+ of channel 1) |
| 2   | GND   | Ground reference / ESD return |
| 3   | I/O2  | Protected data line (e.g. D− of channel 1) |
| 4   | I/O3  | Protected data line (e.g. D− of channel 2) |
| 5   | VBUS  | Bus-supply rail protection (connect to +VCC / VBUS) |
| 6   | I/O4  | Protected data line (e.g. D+ of channel 2) |

**Topology:** Each I/O pin connects to VBUS and to GND through a pair of low-capacitance steering diodes. A central **Zener (TRANSIL) clamp** sits between VBUS and GND. Positive transients on an I/O are steered up to VBUS and clamped by the Zener; negative transients are steered up from GND through the lower diode. ESD energy injected on a data line is therefore routed to ground via the VBUS rail.

---

## 3. Electrical Requirements

### 3.1 Absolute Maximum Ratings

| Symbol | Parameter | Condition | Value | Unit |
|--------|-----------|-----------|-------|------|
| V_PP | Peak pulse voltage | IEC 61000-4-2 air discharge | 15 | kV |
| V_PP | Peak pulse voltage | IEC 61000-4-2 contact discharge | 15 | kV |
| V_PP | Peak pulse voltage | MIL-STD-883C Method 3015-6 | 25 | kV |
| T_stg | Storage temperature | — | −55 to +150 | °C |
| T_j | Operating junction temperature | — | −40 to +125 | °C |
| T_L | Lead solder temperature | 10 s | 260 | °C |

### 3.2 Electrical Characteristics (T_amb = 25 °C)

| Symbol | Parameter | Test condition | Min | Typ | Max | Unit |
|--------|-----------|----------------|-----|-----|-----|------|
| I_RM | Leakage current | V_RM = 5.25 V | — | 10 | 150 | nA |
| V_BR | Breakdown voltage (VBUS–GND) | I_R = 1 mA | 6 | — | 10 | V |
| V_F | Forward voltage | I_F = 10 mA | — | — | 0.86 | V |
| V_CL | Clamping voltage | I_PP = 1 A, 8/20 µs, any I/O to GND | — | — | 12 | V |
| V_CL | Clamping voltage | I_PP = 5 A, 8/20 µs, any I/O to GND | — | — | 17 | V |
| C_io-GND | Capacitance I/O to GND | V_R = 1.65 V | — | 3 | 4 | pF |
| ΔC_io-GND | I/O-to-GND capacitance match | — | — | 0.015 | — | pF |
| C_io-io | Capacitance I/O to I/O | V_R = 1.65 V | — | 1.85 | 2.7 | pF |
| ΔC_io-io | I/O-to-I/O capacitance match | — | — | 0.04 | — | pF |
| — | Peak pulse power | 8/20 µs | — | 130 | — | W |

### 3.3 Application-Relevant Notes

- **Working voltage:** rated against the USB 5 V rail; leakage spec given at V_RM = 5.25 V. The VBUS/I/O lines must operate **below the 6 V (min) breakdown voltage** — suitable for 3.3 V and 5 V signaling. It is **not** intended for rails above ~5.25 V continuous.
- **Capacitance matching** (ΔC_io-io = 0.04 pF typ) keeps D+/D− balanced, meeting USB 2.0 differential requirements (< 1 pF mismatch budget).
- **Frequency response (S21):** essentially flat (≈ 0 dB) from DC to >100 MHz, suitable for 480 Mb/s data; a notch of ≈ −15 dB appears near ~2 GHz, so the device additionally attenuates out-of-band noise (e.g. GSM ~900 MHz, RF interference) while passing the wanted signal.
- **Analog crosstalk** between channels is < −55 dB up to 240 MHz.

---

## 4. Surge / Clamping Behavior

The clamping voltage for the rail-to-rail topology:

- Positive surge: `V_CL+ = V_TRANSIL + V_F`
- Negative surge: `V_CL− = −V_F`
- with `V_F = V_T + R_d·I_p` (V_T = forward threshold, R_d = dynamic resistance)

**Worked example (datasheet):** R_d ≈ 0.5 Ω, V_T ≈ 1.1 V, IEC 61000-4-2 level 4 contact discharge (V_g = 8 kV, R_g = 330 Ω → I_p ≈ 24 A), VBUS = +5 V:
- V_CL+ ≈ **+31.2 V**, V_CL− ≈ **−13.1 V** (ideal, ignoring parasitics).

**Parasitic-inductance penalty:** with poorly routed 10 mm × 0.5 mm tracks (~6 nH each) and a 1 ns ESD rise time (dI/dt ≈ 24 A/ns), each track adds ~144 V of overshoot (L·dI/dt). This **dominates the residual clamp voltage**, so layout — not the die — sets real-world protection quality.

---

## 5. Sample Applications & Reference Circuits

### 5.1 USB 2.0 Port (primary application — Figure 14)
- Device placed directly at the **USB connector**, between connector and transceiver/hub.
- I/O pins protect the **D+ / D−** pairs (two pairs available → can serve two USB data pairs, or one pair plus other lines).
- **VBUS pin tied to the USB +5 V (VBUS) rail**, decoupled to GND with a **100 nF capacitor** (C_BUS). This capacitor provides the ESD return path to ground for energy steered up from the data lines.
- Series resistors R_S and pull-up/pull-down resistors (R_PU/R_PD) belong to the USB transceiver design, not the protection device.
- Supports Low/Full/High-speed modes (USB switch sequencing per transceiver).

### 5.2 Other Supported Interfaces
- **Ethernet 10/100 Mb/s**, **T1/E1** (Figure 15): I/O lines protect the transformer-coupled Tx/Rx pairs toward the data transceiver; complemented on the line side by dedicated TVS (e.g. SMP75-8). VCC pin decoupled with 100 nF.
- **Video lines**, **SIM-card protection**, general portable-electronics high-speed I/O.

### 5.3 Reference Decoupling
- A **100 nF capacitor from VBUS (pin 5) to GND (pin 2)** is the canonical companion component shown in every application/functional figure. Place it close to the device.

---

## 6. Placement, Routing & Layout Best Practices

Layout is the dominant factor in achieving the datasheet-grade ESD performance (see §4 — parasitic L causes hundreds of volts of overshoot).

- **Place the device as close as possible to the disturbance source** — i.e. right at the **connector**, ahead of any branch to the protected IC. ESD must hit the protector first.
- **Minimize all three critical track lengths and inductances:**
  - Data line → I/O pin,
  - VCC/VBUS → VBUS pin,
  - GND pin → ground plane.
  Short, wide tracks reduce L·dI/dt overshoot.
- **"Optimized" vs "unsuitable" routing (Figures 6–8):** route the protected signal **into the I/O pin and back out toward the protected IC in-line** (the trace should pass by / tap the pin), rather than stubbing off to a pin on a long branch. Avoid long stubs and detours to the protection pins.
- **Solid, low-impedance ground:** connect the GND pin to the ground plane with the shortest possible path (ideally a via directly at the pad) to keep the ESD return inductance low.
- **VBUS decoupling:** place the **100 nF VBUS-to-GND capacitor immediately adjacent** to pins 5/2 with short tracks, so the steered ESD current has a low-inductance path to ground.
- **Keep the differential pair symmetric** through the device (matched trace lengths to I/O1/I/O2, I/O3/I/O4) to preserve the device's inherent low capacitance mismatch and USB 2.0 signal integrity.
- **Compact footprint:** SOT23-6L occupies ≤ 9 mm², allowing close-to-connector placement even in dense layouts.

---

## 7. Design-Validation Checklist (for board review)

- [ ] VBUS (pin 5) tied to the protected supply rail (≤ 5.25 V continuous; below 6 V V_BR min).
- [ ] 100 nF decoupling cap from VBUS to GND, placed close to the part.
- [ ] Device located at the connector, before the branch to the protected IC.
- [ ] GND pin tied to plane via a short, direct path (via-in-pad or adjacent via).
- [ ] Protected data lines routed in-line through the I/O pins (no long stubs).
- [ ] D+/D− (and other differential pairs) length-matched and symmetric through the device.
- [ ] Operating signal levels are 3.3 V / 5 V class; no continuous rail above ~5.25 V on any pin.
- [ ] All protector tracks (data, VBUS, GND) kept short to minimize ESD clamp overshoot.
