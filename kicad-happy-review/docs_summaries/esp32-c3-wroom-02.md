# ESP32-C3-WROOM-02 / ESP32-C3-WROOM-02U — Application Summary

**Manufacturer:** Espressif Systems
**Type:** 2.4 GHz Wi-Fi (802.11 b/g/n) + Bluetooth LE 5 module, RISC-V single-core SoC with integrated SPI flash
**Datasheet:** Version 1.7, 2026-03-05 (`docs/components/esp32-c3-wroom-02/esp32-c3-wroom-02_datasheet_en.md`)

> This summary is a reference for validating a board design that integrates the ESP32-C3-WROOM-02(U). It covers electrical, application-circuit, and layout considerations only. Packaging, mechanical/physical dimensions, soldering/reflow, and software/programming details are omitted except where a configuration directly affects the electrical design.
>
> **Variant note:** `-02` has an **on-board PCB antenna**; `-02U` has a **U.FL/MHF-I external-antenna connector**. They are otherwise electrically identical. Where this document says "the module" it applies to both; antenna-specific items are called out.

---

## 1. Short Description

The ESP32-C3-WROOM-02(U) is a fully-certified, surface-mount wireless MCU module built around the ESP32-C3 SoC (32-bit RISC-V single core, up to 160 MHz). It bundles the radio, an on-module **40 MHz crystal**, and an external **SPI flash** (4 MB / 8 MB, up to 16 MB on request) into a single shielded package, exposing **19 pins** (15 usable GPIOs).

Functional traits relevant to the application:

- **Connectivity:** Wi-Fi 802.11 b/g/n (2412–2484 MHz, 20/40 MHz BW, up to 150 Mbps) and Bluetooth LE 5 (125 kbps–2 Mbps, mesh, long range).
- **Peripherals (via flexible GPIO matrix):** 2× UART, SPI (SPI2 general-purpose), I²C, I²S, TWAI® (CAN 2.0), USB Serial/JTAG, LED PWM, RMT (IR), 2× 12-bit SAR ADC, temperature sensor, timers/watchdogs.
- **Native USB Serial/JTAG** (full-speed, 12 Mbps) on GPIO18/GPIO19 — supports flashing and debug without an external USB-UART bridge.
- **Boot configuration via strapping pins** GPIO2, GPIO8, GPIO9 (latched at reset, then free as regular IO).
- **EN (CHIP_EN)** active-high enable; **must not be left floating**.
- Low-power modes down to ~5 µA deep-sleep / ~1 µA power-off.

Typical applications: smart home, industrial automation, healthcare, consumer electronics, smart agriculture, POS, low-power IoT sensor hubs and data loggers.

---

## 2. Electrical Requirements

### 2.1 Absolute Maximum Ratings (stress limits — do not operate here)

| Symbol | Parameter | Min | Max | Unit |
|---|---|---|---|---|
| VDD33 | Power supply voltage | −0.3 | 3.6 | V |
| T_STORE | Storage temperature | −40 | 105 | °C |

> ESD rating (module): HBM ±2000 V, CDM ±500 V. Operation at or beyond absolute-maximum conditions can cause permanent damage.

### 2.2 Recommended Operating Conditions

| Symbol | Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|---|
| VDD33 | Power supply voltage | 3.0 | 3.3 | 3.6 | V |
| I_VDD | Current the external supply must be able to deliver | **0.5** | — | — | A |
| T_A | Operating ambient temp. (85 °C version) | −40 | — | 85 | °C |
| T_A | Operating ambient temp. (105 °C version) | −40 | — | 105 | °C |

> **The 3V3 rail must be sized for ≥ 0.5 A.** Although average draw is far lower, Wi-Fi TX produces large current bursts (see §2.4); the regulator and bulk capacitance must absorb them without the rail sagging below 3.0 V.

### 2.3 DC Characteristics (3.3 V, 25 °C)

| Symbol | Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|---|
| C_IN | Pin capacitance | — | 2 | — | pF |
| V_IH | High-level input voltage | 0.75 × VDD | — | VDD + 0.3 | V |
| V_IL | Low-level input voltage | −0.3 | — | 0.25 × VDD | V |
| I_IH / I_IL | Input leakage current | — | — | 50 | nA |
| V_OH | High-level output voltage | 0.8 × VDD | — | — | V |
| V_OL | Low-level output voltage | — | — | 0.1 × VDD | V |
| I_OH | High-level source current (VDD=3.3 V, PAD_DRIVER=3) | — | 40 | — | mA |
| I_OL | Low-level sink current (VDD=3.3 V, PAD_DRIVER=3) | — | 28 | — | mA |
| R_PU / R_PD | Internal weak pull-up / pull-down | — | 45 | — | kΩ |
| V_IH_nRST | CHIP_EN reset-release (high) voltage | 0.75 × VDD | — | VDD + 0.3 | V |
| V_IL_nRST | CHIP_EN reset (low) voltage | −0.3 | — | 0.25 × VDD | V |

> At 3.3 V: logic-high input ≥ ~2.48 V, logic-low ≤ ~0.825 V. GPIOs are **3.3 V — not 5 V tolerant.** Per-pin drive defaults are weaker; the 40/28 mA figures require PAD_DRIVER=3 (max drive strength) set in firmware.

### 2.4 Current Consumption

**Active mode (Wi-Fi, peak):**

| Mode | Description | Peak (mA) |
|---|---|---|
| TX | 802.11b, 1 Mbps @20.5 dBm | **345** |
| TX | 802.11g, 54 Mbps @18 dBm | 285 |
| TX | 802.11n HT20 MCS7 @17.5 dBm | 280 |
| TX | 802.11n HT40 MCS7 @17 dBm | 280 |
| RX | 802.11b/g/n HT20 | 82 |
| RX | 802.11n HT40 | 84 |

**Modem-sleep (Wi-Fi clock-gated):** 13–28 mA depending on CPU freq (80/160 MHz) and idle/running.

**Low-power modes (typical):**

| Mode | Description | Typ |
|---|---|---|
| Light-sleep | VDD_SPI & Wi-Fi powered down, GPIOs Hi-Z | 130 µA |
| Deep-sleep | RTC timer + RTC memory retained | 5 µA |
| Power off | CHIP_EN low, chip off | 1 µA |

> Design the PDN for the **345 mA TX peak** (worst case 802.11b), not the average. The 0.5 A supply requirement in §2.2 derives from these bursts plus margin.

### 2.5 EN (CHIP_EN) Pin — Enable / Reset

- Active high: **High = chip on, Low = chip off / reset.** Thresholds follow V_IH_nRST / V_IL_nRST above.
- **Must not float.** Tie high (via pull-up) for always-on; drive from a host MCU GPIO if power sequencing is needed.
- An **RC delay on EN is recommended** to guarantee the supply is stable before the chip activates (see §3.2).

### 2.6 Power-up / Reset Timing

| Parameter | Description | Value | Unit |
|---|---|---|---|
| t_STBL | Power-rail stabilization time before CHIP_EN is pulled high | 50 (min) | ms |
| t_RST | CHIP_EN must stay below V_IL_nRST to reset the chip | 50 (min) | ms |
| t_SU | Strapping-pin setup time before CHIP_EN high | 0 (min) | ms |
| t_H | Strapping-pin hold time after CHIP_EN high | 3 (min) | ms |

### 2.7 Integrated Flash (memory-vendor data)

3.3 V flash: 2.7–3.6 V; max clock 80 MHz (default; 120 MHz available on request); 100,000 program/erase cycles; 20-year data retention. Relevant only in that the module's 3V3 rail also powers the flash — clean power is required for reliable flash access.

---

## 3. Sample Applications & Reference Circuits

### 3.1 Module Internal Reference (Figure 8-1) — informational

Inside the module, the ESP32-C3 die is supported by: the 40 MHz crystal (with load caps, R1 series), the SPI flash on a dedicated VDD_SPI rail with 0.1 µF + 1 µF decoupling, an LC/π RF matching network (L1, C5, C6 …) to the PCB antenna or U.FL port, an **ESD diode (D1) on the VDD_SPI/flash supply**, and 100 Ω series resistors on the flash SPI lines. **These are built into the module — the integrator does not replicate them.**

### 3.2 Peripheral / Baseboard Application Circuit (Figure 9-1) — what you implement

This is the reference circuit for the **baseboard around the module**:

```
                3V3 rail
                  │
        ┌─────┬───┴────┐
      C1 10µF  C2 0.1µF │
        │     │        │            ┌──[ R1 10k ]──┐
       GND   GND   ┌────┴── 3V3(1)  │   (EN pull-up / RC)
                   │                │
   EN RC/RESET ────┤ EN(2) ◄──┬─────┘
                   │          │
              [ SW1 ]──[R2 0Ω]┤   C3 (=1µF) ──┐
                   │          │               │
                  C4 0.1µF   GND             GND
                   │
                  GND

  Strapping:  IO9 ──[R8 10k]── 3V3   (default boot = SPI flash boot)
              IO2 ──[R9 10k]── 3V3
              IO8 ── (JP2 boot-option header to GND for download)

  USB:  IO18/IO19 ──[R4,R6 0Ω series]── USB D-/D+  (C5,C6 footprints NC)
  UART: TXD/RXD (GPIO21/GPIO20) ── debug/flash header JP4
  JTAG: IO4–IO9 ── optional JTAG header JP1
```

Key reference values from the datasheet:

| Item | Component | Value / Note |
|---|---|---|
| Bulk + HF decoupling on 3V3 | C1 / C2 | **10 µF + 0.1 µF**, at the module 3V3 pin |
| EN RC delay | R1 / C (EN→GND) | **R = 10 kΩ, C = 1 µF** (recommended; tune to supply ramp) |
| Reset switch | SW1 + R2 + C4 | momentary button to GND, 0 Ω series, 0.1 µF debounce |
| Boot strap (IO9) | R8 | 10 kΩ pull-up to 3V3 (default state = high → SPI boot) |
| USB D+/D− series | R4 / R6 | 0 Ω (placeholders for series matching if needed) |

### 3.3 Boot / Strapping Configuration (electrical implications)

Strapping pins are sampled at reset, then released for normal IO use. Their reset-time levels set boot mode, so **any external circuit on these pins must respect the required boot state**:

| Strapping pin | Internal default | Bit | Role |
|---|---|---|---|
| GPIO2 | Floating | — | Must be **high** at boot (pull-up recommended to avoid glitches) |
| GPIO8 | Floating | — | Must be **high** for normal SPI boot |
| GPIO9 | Weak pull-up | 1 | High = SPI boot; pull **low** + GPIO8 high → download boot |

| Boot Mode | GPIO2 | GPIO8 | GPIO9 |
|---|---|---|---|
| SPI boot (normal run) | 1 | any | 1 |
| Joint download boot (flash via UART0/USB) | 1 | 1 | 0 |

**Design implication:** Do not hang strong pull-downs or low-driving loads on GPIO2/GPIO8/GPIO9 that would corrupt the boot decision. GPIO9 is the usual "BOOT" button (pull low while resetting to enter download). Honor t_SU/t_H (§2.6).

### 3.4 Programming / Debug Interface (electrical)

- **USB Serial/JTAG:** native full-speed USB on **GPIO18 (D−) / GPIO19 (D+)** — 0 Ω series resistors per reference; connect a USB connector directly for flashing/debug/CDC.
- **UART0:** TXD = GPIO21, RXD = GPIO20 — default ROM-download / log UART; expose on a header for bootloader access and console.
- These dictate that GPIO18/19/20/21 be reserved (or carefully shared) if firmware download/debug is required in-system.

### 3.5 Antenna (electrical/RF interface)

- **-02 (PCB antenna):** no external RF parts; only the keep-out (§4) must be honored.
- **-02U (external antenna):** U.FL / MHF-I / AMC-compatible connector, **50 Ω**. Choose a 2.4 GHz, 50 Ω antenna whose **gain does not exceed the certified value (2.33 dBi non-FCC / 1.57 dBi FCC)** or re-certification (incl. EMC) may be required.

---

## 4. Component Placement, Routing & Layout Best Practices

### 4.1 Antenna keep-out (the dominant constraint for the -02)

- The module footprint includes an **antenna keep-out zone** (Figure 11-1): the **"Antenna Area" must be free of copper on all layers** (no ground pour, traces, planes, or components) — roughly the **18 mm × ~6–7.1 mm** region at the antenna end of the module.
- **Place the module at the edge of the baseboard**, antenna end overhanging or flush with the board edge, projecting outward. Do not place metal, batteries, displays, or large components near the antenna.
- The `-02U` (connector variant) has **no keep-out zone** — route the U.FL trace as a controlled 50 Ω line and keep it short.

### 4.2 Power distribution

- Place **C2 (0.1 µF)** immediately at the module 3V3 pin (pin 1) with the shortest possible loop to the GND pin; place **C1 (10 µF)** bulk nearby.
- Use wide, low-impedance 3V3 traces / a power pour to handle the **345 mA TX bursts** without droop; keep the regulator's output cap close and provide a solid return.
- Tie the module **GND pins (9, 19)** and the **center EPAD** to a continuous ground plane with multiple vias.

### 4.3 Thermal / EPAD

- Soldering the **exposed center pad (EPAD) to baseboard ground is optional but recommended** for thermal performance. Use the recommended **thermal-via array** (Figure 11-1: a grid of vias in the ~2.9 × 2.9 mm pad region, ~0.4 mm pitch / ~0.4–0.7 mm features) down to the ground plane.
- Apply the correct solder-paste amount on the EPAD — excess paste lifts the module and can starve the perimeter pin joints.

### 4.4 EN / reset / strapping routing

- Route the **EN trace** short and away from switching/noisy nodes; the high-impedance RC node (10 kΩ/1 µF) is noise-sensitive. Keep the RC components close to the EN pin. Never leave EN floating.
- Route boot-strap pins (GPIO2/8/9) so reset-time levels are deterministic; keep any boot button + debounce cap close to the pin.

### 4.5 USB / high-speed signals

- Route **USB D+ (GPIO19) / D− (GPIO18)** as a **90 Ω differential pair**, length-matched, with the 0 Ω series resistors placed near the module and a continuous reference ground beneath.

### 4.6 Land pattern (Figure 11-1, key dimensions)

- Overall module 18 × 20 mm; pin pitch **0.9 mm** along the two side rows (pins 1–9 and 10–18), pad reach **1.5 mm**, end-pad spacing 0.5 mm; central thermal pad ~**2.9 × 2.9 mm** with thermal-via grid. Use Espressif's released land-pattern / 3D STEP source files for exact geometry.
- Follow *ESP32-C3 Hardware Design Guidelines › General Principles of PCB Layout for Modules* for full placement rules.

---

## 5. Pinout Reference (Module, 19 pins)

| Pin | Name | Type | Primary / notable functions |
|---|---|---|---|
| 1 | 3V3 | P | Power supply (3.0–3.6 V) |
| 2 | EN | I | Chip enable (high = on); do not float; RC-delay recommended |
| 3 | IO4 | I/O/T | GPIO4, ADC1_CH4, FSPIHD, **MTMS** (JTAG) |
| 4 | IO5 | I/O/T | GPIO5, ADC2_CH0, FSPIWP, **MTDI** |
| 5 | IO6 | I/O/T | GPIO6, FSPICLK, **MTCK** |
| 6 | IO7 | I/O/T | GPIO7, FSPID, **MTDO** |
| 7 | IO8 | I/O/T | GPIO8 — **strapping** (boot/ROM-print) |
| 8 | IO9 | I/O/T | GPIO9 — **strapping** (BOOT button, weak pull-up) |
| 9 | GND | P | Ground |
| 10 | IO10 | I/O/T | GPIO10, FSPICS0 |
| 11 | RXD | I/O/T | GPIO20, **U0RXD** |
| 12 | TXD | I/O/T | GPIO21, **U0TXD** |
| 13 | IO18 | I/O/T | GPIO18, **USB_D−** |
| 14 | IO19 | I/O/T | GPIO19, **USB_D+** |
| 15 | IO3 | I/O/T | GPIO3, ADC1_CH3 |
| 16 | IO2 | I/O/T | GPIO2, ADC1_CH2, FSPIQ — **strapping** |
| 17 | IO1 | I/O/T | GPIO1, ADC1_CH1, XTAL_32K_N |
| 18 | IO0 | I/O/T | GPIO0, ADC1_CH0, XTAL_32K_P |
| 19 | GND | P | Ground |
| — | EPAD | P | Center thermal/ground pad |

> Type: P = power, I = input, O = output, T = high-impedance. ADC1 (GPIO0–4) is factory-calibrated; ADC2 (1 channel) is not and is inoperable on some chip revisions. Most peripherals route to any GPIO via the GPIO matrix; UART0 and USB are fixed as above.

---

## 6. RF Summary (for reference / certification context)

- **Wi-Fi:** TX power ~20.5 dBm (802.11b) down to ~17 dBm (802.11n HT40 MCS7); RX sensitivity to −98 dBm (11b 1 Mbps).
- **Bluetooth LE:** TX range −24 to +20 dBm; RX sensitivity to −105 dBm (125 kbps coded/long-range).
- Module is pre-certified with its on-board PCB antenna; changing the antenna (esp. on -02U) beyond the certified gain may require re-testing.

---

## 7. Design Validation Checklist

- [ ] 3V3 supply within 3.0–3.6 V at the pin under all loads; **regulator + bulk cap rated for ≥0.5 A / 345 mA TX bursts** without sag below 3.0 V.
- [ ] C2 = 0.1 µF placed at pin 1 with minimal loop; C1 = 10 µF bulk nearby.
- [ ] EN not floating; RC delay (≈10 kΩ / 1 µF) present and sized to the supply ramp; t_STBL ≥ 50 ms before EN goes high.
- [ ] Strapping pins GPIO2 (pull-up), GPIO8 (high for SPI boot), GPIO9 (pull-up; BOOT button low for download) — no external circuit corrupts boot state.
- [ ] All GPIO loads respect 3.3 V logic (no 5 V back-drive; module is **not** 5 V tolerant).
- [ ] USB D+/D− (GPIO19/18) routed as 90 Ω diff pair with series-R footprints; UART0 (GPIO20/21) accessible for flashing if no USB.
- [ ] Antenna keep-out area (-02) clear of copper/metal on all layers; module at board edge. For -02U: 50 Ω U.FL trace, antenna gain ≤ certified value.
- [ ] GND pins 9/19 and EPAD stitched to ground plane; thermal-via array under EPAD per land pattern.
- [ ] Correct land pattern / footprint (18 × 20 mm, 0.9 mm pitch) and the right flash/temperature variant ordered.
- [ ] Operating ambient within the ordered temperature grade (85 °C vs 105 °C version).

---

*Sources: ESP32-C3-WROOM-02 & WROOM-02U Datasheet v1.7 (Espressif, 2026-03-05) — Features, Pin Definitions (Table 3-1), Boot Configurations (Tables 4-1…4-6), Electrical Characteristics (Tables 6-1…6-7), RF Characteristics (§7), Module Schematics (Fig. 8-1), Peripheral Schematics (Fig. 9-1 + notes), and PCB Layout Recommendations (§11, Fig. 11-1). Internal-only and chip-level details cross-referenced to the ESP32-C3 Series Datasheet and Hardware Design Guidelines as cited in the module datasheet.*
