# CANNONBALL (REIL INDUSTRIAL) Schematic & PCB Review — 2026-06-26

**Board:** ESP32-C3 battery-powered IoT/dev board — USB-C in, BQ24075 charger + DW01A/FS8205A 1S protection + MAX17048 fuel gauge, AP2112K-3.3 LDO, I²C sensor suite (SHT40, VEML6030, LSM6DS3), SK6812 addressable LEDs.
**Files:** `CANNONBALL.kicad_sch`, `CANNONBALL.kicad_pcb` (KiCad 10, single sheet, 4-layer PCB 25.56 × 23.64 mm).
**Documentation catalog:** `docs_summaries/` (datasheet excerpts) + `docs/components/` extractions.
**Analyzers run:** schematic, PCB (`--full --proximity`), cross-domain, EMC (44-rule), thermal, SPICE (ngspice, 4 subcircuits). Gerber & lifecycle: see *Notes and Assumptions*.

## Summary
- Total findings: **24** (2 Critical · 9 Warnings · 8 Suggestions · 5 reviewer-overridden false positives)
- Critical: **2**
- Warnings: **9**
- Suggestions: **8**
- Pass: 30+ checks (see *Passed Checks*)

> **Verdict: NOT ready to fabricate.** Two blocking issues — a footprint/package mismatch on the IMU (U4) and a reversed Qwiic pinout on the I²C connector (J4) — will cause an unassemblable part and a power-to-signal short on a standard cable. The battery-protection circuit also deviates from the DW01A datasheet in two mandatory passives. The core architecture is otherwise sound and the digital/bus design verifies cleanly.

---

## Critical Issues

### C-1 — U4 (LSM6DS3) footprint does not match the part package
**FAIL [CRITICAL]** · Affected: `U4`, net I²C bus · Basis: raw-PCB-measured + catalog-verified
`U4` uses footprint `Package_LGA:LGA-14_2x2mm_P0.35mm_LayoutBorder3x4y`. I measured the actual pads in `CANNONBALL.kicad_pcb`: 14 pads spanning **1.525 × 1.525 mm on a 0.35 mm pitch** (pad size 0.325 × 0.20 mm). The LSM6DS3 / LSM6DS3TR-C is an **LGA-14L, 2.5 × 3.0 mm, 0.5 mm pitch** package (`docs_summaries/lsm6ds3tr-c.md` §Package: "LGA-14L, 2.5 mm × 3.0 mm × 0.83 mm").

The footprint is a *different, smaller* package (≈30 % pitch error, wrong body size). A real LSM6DS3-family IMU cannot be soldered to it — pads will not register with the device terminals. Either the footprint is wrong or the part is mislabeled (the schematic value is `LSM6DS3` with no MPN).

**Fix:** Replace with the correct LSM6DS3TR-C footprint (KiCad `Package_LGA:LGA-14_2.5x3mm_P0.5mm…`) and confirm the exact MPN. Re-verify pad-1 orientation and the SA0/CS strap pins after the swap.

### C-2 — J4 (Qwiic/STEMMA-QT I²C connector) pinout is reversed
**FAIL [CRITICAL]** · Affected: `J4`, nets `3V3`, `GND`, `GPIO 0 / SCL`, `GPIO 1 / SDA` · Basis: raw-file + README + de-facto standard
`J4` is a **JST-SH 1.0 mm 4-pin** connector (`connectors:CONN-SMD_4P-P1.00_ZX-SH1.0-4PLN`; README: "I2C expansion — JST SH 1.0 mm 4-pin"). That is the physical Qwiic / STEMMA-QT connector, whose **de-facto standard pinout is pin1→4 = GND, 3V3, SDA, SCL**.

The schematic wires it pin1→4 = **SCL, SDA, 3V3, GND** — fully reversed. A standard Qwiic cable plugged in would connect:

```
 cable wire      lands on board net      consequence
 ----------      ------------------      -----------
 1 GND      -->  SCL                     SCL pulled to remote GND
 2 3V3      -->  SDA                      3.3 V driven onto SDA  *** bus/IC damage
 3 SDA      -->  3V3                      remote SDA shorts board 3V3 rail
 4 SCL      -->  GND                      GND pulled to remote SCL
```

This shorts the 3V3 rail to a signal line and mis-powers every device on both ends. (Note: even the README's own stated order "3.3 V, GND, SDA, SCL" is non-standard and also disagrees with the schematic — the connector definition is inconsistent in three places.)

**Fix:** Re-assign J4 to the Qwiic standard:
```
J4 pin 1 → GND
J4 pin 2 → 3V3
J4 pin 3 → GPIO 1 / SDA
J4 pin 4 → GPIO 0 / SCL
```
Confirm pin-1 location against the connector drawing. If J4 is intentionally *not* meant for Qwiic cables, document the custom pinout prominently on the silkscreen and in the README — but the JST-SH choice strongly implies Qwiic compatibility.

---

## Warnings

### W-1 — DW01A VCC resistor R10 = 100 Ω, below datasheet minimum (470 Ω)
**FAIL [WARNING]** · Affected: `R10`, `U5` (DW01A) · Basis: catalog-verified
DW01A VCC (pin 5) is fed from `P+` through `R10 = 100 Ω` (`P+ → R10 → __unnamed_9 → U5.VCC`, with `C11` 100 nF to B−). `docs_summaries/DW01A.md` §3.2 specifies **R1 = 470 Ω … 1.5 kΩ, "mandatory, must be ≥ 470 Ω"** (supply current-limit / isolation; forms the VCC RC filter with C1). 100 Ω is below the floor — it weakens fault isolation of the supply pin and raises the RC corner (SPICE: R10/C11 = 15.9 kHz vs the intended sub-kHz). Use 470 Ω (1 kΩ is common on reference 1S boards).
```
P+ ──[R10 ≥470Ω]──┬── VCC (U5 pin5)
                  │
               [C11 100nF]
                  │
                 B−
```

### W-2 — DW01A CS series resistor (R2) is missing
**FAIL [WARNING]** · Affected: `U5` pin 2 (CS), net `GND`/`P−` · Basis: catalog-verified
`U5` CS (pin 2) connects **directly to GND** (= pack P−). `docs_summaries/DW01A.md` §3.2 lists **R2 = 1 kΩ … 3 kΩ (typ 2 kΩ) between CS and P− as mandatory** — it limits current into the CS pin and sets the overcurrent/short sense connection. Without it, the CS pin is unprotected during a short-circuit event (when P− swings toward B+), stressing the IC. Add a 2 kΩ resistor in series from CS to P−.

### W-3 — BQ24075 IN and OUT are tied together (non-datasheet power-path)
**FAIL [WARNING]** · Affected: `U6`, `D1`, net `5V` · Basis: catalog-verified + raw-file
The charger IN (pin 13) and the regulated system OUT (pins 10/11) sit on the **same net `5V`**, fed from VBUS through Schottky `D1` (1N5819W): `VBUS → F1 → D1 → 5V → {U6.IN, U6.OUT, U2.VIN}`. The BQ24075's defining feature is Dynamic Power-Path Management with a **regulated 5.5 V OUT** and reverse blocking between IN and OUT (`docs_summaries/bq24075.md` §1, §2.3 V_O(REG)=5.5 V). Shorting IN to OUT:
- forfeits the regulated 5.5 V system rail (rail = IN ≈ VBUS−0.35 V ≈ 4.65 V on USB, ≈ V_BAT on battery);
- bypasses the chip's internal reverse protection — `D1` is now doing that job (and costs ~0.35 V headroom);
- defeats clean DPPM load-sharing.

It likely *functions* because the downstream AP2112 LDO tolerates 3.0–6 V in and battery-supplement still works through the BAT→OUT FET. But this is not the datasheet topology. **Confirm intent.** The clean design is IN ← VBUS (direct, post-fuse — the chip is 28 V tolerant), OUT → separate system rail, no D1.

### W-4 — BQ24075 thermal pad under-stitched (1 via vs 5–6 recommended)
**FAIL [WARNING]** · Affected: `U6` exposed pad · Basis: catalog + PCB analyzer (TV-001)
The VQFN-16 exposed pad is connected by **1 thermal via**. `docs_summaries/bq24075.md` §4 calls for a **5–6-via array** under the pad as the primary heat path; under-stitching risks entering thermal regulation (charge fold-back) at higher charge currents / hot ambient. Add a 5–6 via array into the inner/bottom GND copper. (Charge current here is modest — ~0.5 A — so this is reliability margin, not an immediate failure.)

### W-5 — AP2112 (U2) worst-case junction temperature ~103 °C
**FAIL [WARNING]** · Affected: `U2` · Basis: thermal analysis
Thermal estimate: Tj ≈ **103 °C** at the USB corner (V_IN ≈ 4.65 V, V_OUT = 3.3 V, ~0.4 A sustained → ~0.52 W in SOT-23-5, no thermal vias). Tj(max) is 125 °C, so margin is ~22 °C and is optimistic — real SOT-23-5 θ_JA on limited copper is 150–220 °C/W. This is a corner case (USB-powered + sustained Wi-Fi load); on battery the drop is small and the part runs cool. Maximize copper pour around U2 and avoid placing it under the antenna keep-out.

### W-6 — Low-battery brown-out margin (LDO dropout)
**FAIL [WARNING]** · Affected: `U2`, `3V3` rail · Basis: inference + README
With the LDO fed from the system rail (= V_BAT on battery), the AP2112's ~250 mV+ dropout means the 3V3 rail begins sagging once V_BAT < ~3.55 V. ESP32-C3 Wi-Fi TX peaks (~350 mA) can momentarily brown out near the discharge tail (the README itself sets a 3.0 V floor). The DW01A cut-off (~2.4 V) is well below this, so the bottom ~0.5 V of cell capacity is effectively unusable and risky under RF load. Acceptable for the use case but worth documenting; a buck-boost or low-dropout-at-load regulator would recover it.

### W-7 — 4-layer stack has no dedicated solid ground plane
**FAIL [WARNING]** · Affected: whole board · Basis: EMC + cross-domain analyzers
All four copper layers are typed **signal** (stackup: F.Cu / 0.1 mm prepreg / In1.Cu / 1.24 mm core / In2.Cu / 0.1 mm / B.Cu) and there is a **single GND zone fragmented into ~14 islands** (cross-analysis PS-002). The EMC pass flags **20 reference-plane-gap errors (GP-001)**, adjacent-signal-layer coupling (SU-001), and missing stitching vias at layer transitions — including the USB `DP`/`DN` pair (RP-001). For a board carrying USB FS and a 2.4 GHz radio, dedicate **In1.Cu (0.1 mm under F.Cu) to a solid GND pour**, give all top-side signals a continuous reference, and add return-stitching vias near layer changes on D+/D-.

### W-8 — Numerous untented vias-in-pad
**FAIL [WARNING]** · Affected: `R9, R14, R15` (0402 passives), `U1`, `U10` EP, `Q1`, `U2` · Basis: PCB analyzer (VP-001)
Many pads contain untented/unfilled vias. On small 0402 passives this causes solder wicking → reduced joint / tombstoning; on IC and thermal/GND pads it causes voiding. Tent (or via-fill-and-cap) these, especially the 0402s and the U10/Q1 exposed pads.

### W-9 — Components placed < 0.5 mm from board edge
**FAIL [WARNING]** · Affected: `R10` (0.28 mm), `C12` (0.28 mm), `R16` (0.45 mm), `C21` (0.46 mm) · Basis: PCB analyzer (PM-002)
Several parts sit inside the typical 0.5 mm assembly/depaneling keep-in. Pull them inboard. (USB1 intentionally overhangs the edge — expected for the connector.)

---

## Suggestions

- **S-1 — MPN coverage:** only 6/33 unique parts carry an MPN (`SS-001`). Populate MPNs (including the SOT-23 transistors and all passives) before ordering; the BOM otherwise has values + footprints.
- **S-2 — Fiducials:** none present on a 45-SMD board (`FD-001`). Add ≥3 fiducials for pick-and-place.
- **S-3 — Test points:** no dedicated TPs (`TE-001`); debug is via headers J5/J6 (UART, BOOT) and J4 (I²C). Adequate, but consider TPs on `3V3`, `P+`, `5V`.
- **S-4 — Sleep current:** AP2112 EN is tied to V_IN (always-on); its 55 µA Iq dominates the sleep budget on a battery board. Consider EN control from a GPIO or a lower-Iq LDO to extend runtime.
- **S-5 — Charge-rate ceiling:** EN2 is hardwired to GND, so only USB100/USB500 modes are reachable; the resistor-programmed input limit (`R17`/ILIM, requires EN2=high) and >500 mA charging are unavailable. `R17` is still correctly populated (ILIM must not float). Confirm 500 mA max is intended.
- **S-6 — USBLC6 bypass:** the canonical 100 nF from the USBLC6 VBUS pin (U1.5) to GND is absent on the `VBUS` net (`usblc6-4.md` §5). Add a local 100 nF.
- **S-7 — ERC hygiene:** add `PWR_FLAG` to `B-` and `VBUS` to clear the rail-source warnings (`RS-001`).
- **S-8 — README vs schematic mismatch:** README reference designators do not match the schematic (README `U1`=ESP32 / `U6`=AP2112 / `U8`=LSM6DS3 vs schematic `U3`=ESP32 / `U2`=AP2112 / `U4`=LSM6DS3). Reconcile to avoid assembly/debug confusion.

---

## Connector Pin Assignment Recommendations

### J4 — I²C expansion (JST-SH 1.0 mm) — **change required** (see C-2)
Current: `1=SCL, 2=SDA, 3=3V3, 4=GND`. Re-assign to the Qwiic/STEMMA-QT standard:

| Pin | Current net | Proposed net | Rationale |
|-----|-------------|--------------|-----------|
| 1 | SCL | **GND** | Qwiic standard; prevents power/signal swap with off-the-shelf cables |
| 2 | SDA | **3V3** | |
| 3 | 3V3 | **GPIO 1 / SDA** | |
| 4 | GND | **GPIO 0 / SCL** | |

### J3 / J5 / J6 — acceptable as-is
- **J3** (JST-PH 2.0, battery): `1=P+`, `2=B−`. Correct 1S pack connection.
- **J5** (1×7): `5V, GND, GPIO20/RX, GPIO21/TX, GND, GPIO1/SDA, GPIO0/SCL` — UART + I²C + power, GND interleaved. Fine.
- **J6** (1×7): `GND, GPIO6, GPIO7, BOOT_SW, GND, 5V, 3V3` — good GND distribution and boot access. Fine.

---

## Passed Checks

**Power Integrity**
- AP2112K-3.3 (U2): VIN/GND/VOUT/EN/NC mapping correct; EN tied to V_IN = always-on per design (`AP2112.md` §1, internal 3 MΩ PD). Input & output decoupling present on `5V`/`3V3`.
- 3V3 rail decoupling: multiple 100 nF + 10 µF across the rail; SPICE PDN Z ≈ 0.01 Ω @ 1 MHz, settles to 3.3 V (inrush ≈ 0.2 A).
- BQ24075 program/control pins all datasheet-valid: `TS`=10 kΩ→GND (unused-TS config, `bq24075.md` §3.1), `ISET`=1.8 kΩ→I_CHG≈494 mA, `ILIM`=2 kΩ (defined, not floating), `TMR`=float (default timer), `CE`/`SYSOFF`=GND (charge enabled, normal), `EN2`=GND/`EN1`=GPIO10 (100/500 mA select).
- BQ24075 status: `CHG` open-drain → D4 via R11 5.1 kΩ to 5V (in range); `PGOOD` unused (open-drain, safe open).

**Battery Protection**
- DW01A + FS8205A topology correct: OD→R12(100 Ω)→Q1.G1, OC→R13(100 Ω)→Q1.G2, FET pair in the B− line (Q1 S1=B−, S2=GND/P−, common drain MF_DRAIN), GND(pin6)=B−, CS=P−. Matches `DW01A.md` §3.1–3.2 (passive values flagged in W-1/W-2).

**Bus & MCU**
- I²C bus (GPIO0=SCL, GPIO1=SDA): 10 kΩ pull-ups (R15/R14) to 3V3; devices SHT40 (0x44), VEML6030 (ADD→3V3=0x48), MAX17048 (0x36), LSM6DS3 (SA0→GND=0x6A) — **no address conflicts**.
- LSM6DS3 in I²C mode: CS→3V3, SA0→GND, aux SPI (SDX/SCX) NC — correct (analyzer's "SPI" tag is wrong; see overrides).
- ESP32-C3 strapping pins terminated: GPIO2→10 kΩ↑(R6), GPIO8→10 kΩ↑(R2), GPIO9/BOOT→10 kΩ↑(R3)+button — all default-high for normal SPI boot.
- EN reset: R7 10 kΩ↑ + C3 100 nF + SW1 (SPICE 159 Hz RC ≈ 1 ms) — standard.
- Native USB: D+=GPIO19, D−=GPIO18 (ESP32-C3 correct), both routed through USBLC6-4.

**USB-C front end**
- USBLC6-4SC6 (U1) pinout verified (`usblc6-4.md` §2): I/O1=CC1, I/O2=DN, I/O3=DP, I/O4=CC2, VBUS, GND — D+/D−/CC1/CC2 all clamped.
- CC1/CC2 each have 5.1 kΩ Rd to GND (UFP/sink advertising default current) — correct.

**LED subsystem**
- DMG2301L (Q2) P-FET high-side switch verified (`DMG2301L.md`): S=3V3, D=LED_3V3, G=GPIO5 with 10 kΩ↑ (off by default, drive low to enable; V_GS≈−3.3 V within ±8 V limit).
- SK6812 chain at 3.3 V (correct logic level for VIH≈0.7·VDD vs 3.3 V drive), 330 Ω series on data (R8), DIN→DOUT daisy-chain correct.

**SPICE:** 4/4 subcircuits pass (DW01A VCC RC, EN reset RC, 3V3 decoupling, inrush).

---

## Reviewer Overrides / Triaged False Positives

| Analyzer finding | Verdict | Reason |
|---|---|---|
| `PP-001` U5.VCC "no DC path to power rail" (error) | False positive | DC path exists: `VCC ← R10 (100 Ω) ← P+`. The BFS failed only because `P+` (battery) isn't a *declared* rail. (Value issue tracked separately in W-1.) |
| `PU-001` ×5 missing pull-ups (U4 INT1/INT2, U5 OD, U6 TMR, U8 INT) | False positive | INT1/INT2 are push-pull outputs left NC; OD is a FET gate-drive output; TMR is a timer-program pin (float = default); VEML INT is an unused open-drain. None are pull-up bus pins. |
| `DP-005` "Diff pair B+/B-" via/length asymmetry | False positive | `B+`/`B−` are battery power nets, not a differential signal pair — name-suffix mismatch. |
| LSM6DS3 classified "motion/spi" | Misclassification | The part is strapped for I²C (CS=high, SA0=GND). Does not affect the design. |
| `CP-001` / 0.002–0.007 mm² courtyard overlaps | Negligible | Sub-pad rounding on a dense small board; only U6↔R10 (0.388 mm²) is worth tidying. |

---

## Notes and Assumptions

- **Documentation basis:** pin/value checks are verified against the `docs_summaries/` datasheet-excerpt catalog (cited inline). This is catalog-level verification; for the two blocking issues (C-1, C-2) I additionally measured the raw `.kicad_pcb`/`.kicad_sch`.
- **MAX17048 (U10):** functional connections verified (VDD/CELL→P+, ALRT→GPIO3, I²C, GND, QSTRT/CTG→GND). The exact pin-number table was not independently cross-checked beyond function — the catalog pinout section is abbreviated. Connections are functionally correct.
- **FS8205A:** no entry in `docs_summaries/` (a `docs/components/fs8205a/` folder exists but no summary). Pinout assumed standard dual-N-FET SOT-23-6 (S1/G1/S2/G2/D-D); topology is consistent with the DW01A reference. Confirm against the FS8205A datasheet.
- **D1 = 1N5819W** and **F1 = MF-FSMF050X** have no catalog entries; treated by part number/standard behavior.
- **Gerber analysis: not performed** — no fabrication outputs in the project.
- **Lifecycle/temperature audit: not performed** — no network/MPN coverage; rerun after MPNs are populated (S-1).
- **GND island count (W-7)** comes from the analyzer's copper union-find; confirm visually in KiCad after *Edit → Fill All Zones* before redesigning the stackup.
- **Schematic not modified** (read-only review, per request).
