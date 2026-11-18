# Claude Code task: Salinometer v2 (Rev C) — schematic and PCB in EasyEDA via MCP

You are building a compact 2-layer PCB (target **≤ 60 × 40 mm**) for a handheld conductivity salinometer for the Science Olympiad 2027 Water Quality (Div C) event. Use the EasyEDA MCP tools available in this session. The electrical design below is final and has been reviewed; every part was in stock at JLCPCB on 9 Nov 2026. Your job is to enter it exactly, make the schematic very readable, lay out a small and very clean board, and produce JLCPCB fabrication + assembly files.

Work in two phases. **Stop after Phase 1 (schematic) and wait for the user to approve** before starting Phase 2 (PCB).

Rev C changes from Rev B, for context: ICL7106 + LCD replaced by ICL7107 + four LED digits (the LCD-drive chips are out of stock); the whole board now runs from a single +5 V rail (ICL7107 single-supply mode, datasheet Figure 16), so the 9 V regulator, the decimal-point inverter and the LCD are gone; op-amps are now TLV9062; switches/counter are now 74HC4053/74HC4060.

---

## 0. Ground rules (read first)

1. **No programmable parts.** The event rule (3.Part IV.a) bans programmable devices and anything that communicates over wifi. Do not add any microcontroller, EEPROM, programmable logic, Bluetooth or wifi part, and do not add USB data lines. If you think something needs a programmable part, stop and ask.
2. **Do not change the circuit.** Do not change part values, topology or part numbers. If a part is out of stock, unavailable in the EasyEDA/LCSC library, or a pin name does not match this spec, **stop and report it** with two or three equivalent alternatives (same package and pinout preferred). Do not silently substitute.
3. **Re-check stock first.** Before placing anything, look up every JLC part number in Section 3 and report current stock. The tightest are U10 ICL7107 (576 on 9 Nov) and DS1–DS4 (783). Tell the user to add 3× C2857176 and 8× C10690 to their JLC parts library immediately so the stock is reserved.
4. **Verify every pin from the datasheet, not from memory.** For every IC and the LED digits, open the LCSC datasheet linked from the part's library entry and confirm the pin numbers you wire. Where this document says "verify", it means the pin numbers have not been confirmed. When the library symbol's pin names differ from the datasheet, the datasheet wins.
5. **Prefer JLCPCB Basic parts, then "Preferred Extended" parts.** For every passive, pick a **Basic** part of the stated value and package. For each Extended part below, check whether a pin-compatible Basic or Preferred Extended equivalent exists with the same ratings and with **"Economic and Standard"** PCBA type; propose it in your Phase 1 report, do not swap without approval. Reject any part marked "Standard Only" (it forces the $25 Standard assembly).
6. **Use net labels, not long wires.** Every net named in this document must carry exactly that name on the schematic.
7. Before you start, call the MCP's tool list and tell the user which EasyEDA operations you can and cannot do through MCP (place library part by LCSC number, create custom symbol/footprint, draw wire, net label, run ERC, place/route PCB, run DRC, export Gerber/BOM/CPL). If something essential is missing, say exactly which steps the user must do by hand.

---

## 1. What the circuit does

- A 4-electrode conductivity probe (outer electrodes C1, C2 carry current; inner P1, P2 sense voltage) sits in salt water.
- A 74HC4060 makes a ~1.2 kHz clock. A 74HC4053 switch chops a ±0.42 V reference into a square wave around 2.5 V; an op-amp drives probe electrode C1 through a DC-blocking capacitor.
- A transimpedance amp turns the current returning from C2 into **I_SIG**. An INA826 reads P1/P2; a thermistor-gain stage turns that into **VT** (temperature-compensated to 25 °C).
- A trimmer (lineariser) mixes VT and I_SIG into **REF_SIG**.
- The same 74HC4053 demodulates I_SIG and REF_SIG synchronously onto four capacitors.
- An ICL7107 shows 1000 × (I_SIG p-p) / (REF_SIG p-p) on four 7-segment LED digits. The display reads % salinity directly: 1 count = 0.01 %, decimal point fixed so it reads X.XX and 1X.XX.
- The ICL7107 runs in its **single +5 V mode** (Intersil/Renesas datasheet Figure 16): V+ = 5 V, V− = GND, COMMON driven to 2.5 V, external ratiometric reference, input signals small (< ±1.5 V) and centred on 2.5 V. All three datasheet conditions are met by this circuit.
- Power: one 1S Li-ion cell, charged over a **power-only 6-pin USB-C** (no data pins exist on the connector), boosted to 5.4 V by an MT3608 and regulated to **V5 = 5.0 V** by an ME6211 LDO.
- **AGND is a 2.5 V signal reference net, not ground.** GND (0 V, battery negative) is the copper pour. Never connect AGND to the pour.

---

## 2. Schematic organisation (readability requirements)

Make 4 sheets (or, if the MCP only supports one sheet, 4 large titled boxes on one A3 sheet):

| Sheet | Title | Contents |
|---|---|---|
| 1 | POWER | J1, U1, LED1, J2, Q1, SW1, U2, L1, D1, FB1, U3, V5D filter, all power caps |
| 2 | REFERENCE, AGND & CLOCK | Reference divider, U5A AGND buffer, U6 74HC4060, U7 74HC4053 power/select pins |
| 3 | PROBE FRONT END | J3, J4, JP1, U7 section A (chopper), U5B drive, U8A TIA, U9 INA826, U8B temperature stage, RV2 lineariser |
| 4 | DEMODULATOR, ADC & DISPLAY | U7 sections B/C, demod caps, U10 ICL7107 and its integrator/auto-zero/reference/oscillator parts, DS1–DS4, R37 |

Drawing rules:
- Signal flows left → right. Positive supplies at the top, GND at the bottom. Power-port symbols for V5, V5D, VBAT, VBUS, VSW, GND, and a distinct net flag for **AGND**.
- Each IC's decoupling capacitor drawn directly next to its supply pin.
- Split dual op-amps into units (U5A/U5B, U8A/U8B); split U7 into sections A/B/C if the symbol allows (A on sheet 3, B/C on sheet 4, power/select on sheet 2).
- Every R/C shows designator, value and key spec when it matters ("100nF C0G", "22µF 25V").
- Title block on every sheet: "SALINOMETER v2 Rev C", sheet name, date, "No programmable devices — SciOly WQ 3.Part IV.a".
- Text notes at: AGND ("2.5 V signal reference — NOT ground"), USB-C ("Charge only — no data lines"), C14 ("DC block, no DC through probe"), JP1 ("1-2 = thermistor (default), 2-3 = fixed 10k for bench test"), RV1 ("SPAN"), RV2 ("LIN"), U10 ("Single +5 V mode, datasheet Fig. 16; COMMON driven to V+/2 — intentional").
- 50 mil grid, no wires over part bodies, no 4-way junctions, junction dots on every T.

---

## 3. Bill of materials (Rev C)

Stock figures are from the JLCPCB parts library on 9 Nov 2026. All listed parts were "Economic and Standard" PCBA type unless noted.

### 3.1 ICs, semiconductors, connectors, display

| Ref | Part (MPN) | Package | JLC # | Stock 9 Nov | Notes |
|---|---|---|---|---|---|
| U10 | **ICL7107CM44** (JSMSEMI) | QFP-44 10×10 | **C2857176** | 576, $3.86 | 3½-digit ADC + LED driver (ICL7107 clone). Use the EasyEDA symbol/footprint attached to C2857176 and the JSMSEMI datasheet pin table. Wire by **pin name**; QFP pin numbers differ from the 40-pin DIP. |
| DS1–DS4 | **FJ3161BH** (Zhihao) | 10-pin through-hole, 0.36" single digit, 7.5 × 14 mm | **C10690** | 783, $0.26 | Red, **common anode**. Assembly type "Wave Soldering", Economic OK. **Pinouts of 0.36" single digits vary by maker** (common anode on pins 3/8 or on pins 1/6): take the pin map from the FJ3161BH datasheet / its EasyEDA symbol and confirm it before wiring. |
| U5, U8 | **TLV9062IDR** (TI) | SOIC-8 | **C398355** | 182,640 | Dual RRIO op-amp, 1.8–5.5 V, 10 MHz. Pins: 1 OUTA, 2 −INA, 3 +INA, 4 V−, 5 +INB, 6 −INB, 7 OUTB, 8 V+. |
| U9 | INA826AIDR (TI) | SOIC-8 | C38433 | 10,631 | In-amp, 3–36 V. Pins: 1 −IN, 2 RG, 3 RG, 4 +IN, 5 −VS, 6 REF, 7 OUT, 8 +VS. |
| U7 | **74HC4053D,653** (Nexperia) | SOIC-16 | **C5647** | 6,492 | Triple SPDT analog switch, 80 Ω at 5 V. Pins: 1 2Y1, 2 2Y0, 3 3Y1, 4 3Z, 5 3Y0, 6 E̅ (enable, active low), 7 VEE, 8 GND, 9 S3, 10 S2, 11 S1, 12 1Y0, 13 1Y1, 14 1Z, 15 2Z, 16 VCC. Y0 is connected when its select is LOW, Y1 when HIGH. Verify. |
| U6 | **74HC4060D,653** (Nexperia) | SOIC-16 | **C5649** | 1,708 | Oscillator/14-stage divider. Pins: 7 Q4 (÷16), 8 GND, 9 φO̅/C_TC side, 10 φO/R_TC side, 11 RS (series R), 12 MR (reset), 16 VCC. Verify the RC-oscillator figure in the Nexperia datasheet and wire R_TC, C_TC, R_S exactly as shown there. |
| U1 | TP4054 (UMW) | SOT-23-5 | C668215 | 90,683 | Li-ion linear charger, 4.2 V. Pins: 1 CHRG, 2 GND, 3 BAT, 4 VCC, 5 PROG. I_BAT = 1000 × V_PROG / R_PROG, V_PROG = 1 V. Verify. |
| U2 | MT3608 (Aerosemi) | SOT-23-6 | C84817 | 34,660 | Boost. Pins: 1 SW, 2 GND, 3 FB, 4 EN, 5 IN, 6 NC. V_FB = 0.6 V. |
| U3 | **ME6211C50M5G-N** (Microne) | SOT-23-5 | **C236670** | 8,725 | 5.0 V 500 mA LDO, 6.5 V max input. Pins: 1 VIN, 2 GND, 3 EN, 4 NC, 5 VOUT. Verify. |
| Q1 | AO3401A | SOT-23 | C15127 (Basic) | 1,676,254 | P-MOSFET load switch. 1 G, 2 S, 3 D. |
| D1 | SS14 | SMA | Basic | — | Boost Schottky. |
| LED1 | Red LED | 0603 | Basic | — | Charging indicator. |
| L1 | SWPA4030S220MT (Sunlord) | 4×4 mm | C83472 | 24,120 | 22 µH, 1 A. |
| FB1 | Ferrite bead 600 Ω@100 MHz | 0603 | Basic | — | Between boost and LDO. |
| RV1 | 3224W-1-201E (Bourns) | SMD 4.8×3.5 | C116321 | 2,515 | 200 Ω 12-turn, "SPAN". |
| RV2 | 3224W-1-502E (**BOCHEN**) | SMD 5×4.8 | **C27974760** | 3,014 | 5 kΩ 10-turn, "LIN". (The Bourns 5 k, C81349, is "Standard Only" — do not use it.) |
| J1 | TYPE-C 6P (SHOU HAN) | SMD | C456012 | 988,199 | Power-only USB-C: A5 CC1, B5 CC2, A9/B9 VBUS, A12/B12 GND, shell. |
| J2, J4 | S2B-PH-SM4-TB(LF)(SN) (JST) | SMD right-angle | C295747 | 3,094 | 2-pin PH 2.0 mm. J2 battery, J4 thermistor. |
| J3 | S5B-PH-SM4-TB(LF)(SN) (JST) | SMD right-angle | C265104 | 7,071 | 5-pin PH, probe. |
| SW1 | MSK12C02 (SHOU HAN) | SMD 8×2.8 | C431540 | 177,879 | SPDT slide, 50 mA rated — it switches only Q1's gate. |
| JP1 | 3-pad solder jumper | 0603-size pads | — | — | Default bridge 1-2 (thin cuttable trace). |
| TP1–TP10 | Test pads | 1.0 mm round SMD | — | — | V5, AGND, GND, CLK, I_SIG, VT, IN HI, IN LO, REF HI, REF LO. |
| H1–H4 | M3 mounting holes | 3.2 mm plated, 6 mm pad | — | — | Pads tied to GND. |

### 3.2 Passives (0603 1 % thick-film Basic unless stated)

Resistor tolerance: 1 % is enough everywhere. The measurement is ratiometric and calibrated with two trimmers.

| Ref | Value | Function |
|---|---|---|
| R1, R2 | 5.1 k | USB-C CC1, CC2 pull-downs (one each) |
| R3 | 1 k | LED1 current |
| R4 | 2.4 k | TP4054 PROG (≈420 mA; for a ~2000 mAh cell) |
| R5 | 100 k | Q1 gate pull-up to VBAT |
| R6 | 1 k | Q1 gate to SW1 |
| R7 | 120 k | MT3608 FB top |
| R8 | 15 k | MT3608 FB bottom → 0.6 × (1 + 120/15) = 5.4 V |
| R9 | 10 Ω | V5 → V5D filter |
| R10, R13 | 10 k | Reference divider outer |
| R11, R12 | 2 k | Reference divider inner (±0.417 V about MID = 2.5 V) |
| R14 | 10 Ω | AGND buffer isolation |
| R15 | 100 Ω | Drive output series |
| R16 | 220 k | 74HC4060 R_S |
| R17 | 22 k | 74HC4060 R_TC |
| R18, R19 | 10 k | Demodulator input resistors |
| R20 | 100 Ω | TIA input series |
| R21 | 47 Ω | TIA fixed feedback (in series with RV1) |
| R22 | 68 k | Temperature stage feedback |
| R23 | 11 k | Temperature-compensation slope (≈2.1 %/°C with a 10 k B3950 NTC) |
| R24, R25 | 1 k | INA input protection |
| R26, R27 | 1 M | INA input bias to AGND |
| R28 | DNP | INA826 RG footprint (open = gain 1) |
| R29 | 100 Ω | Thermistor chain series |
| R30 | 10 k | JP1 bench-test substitute for the thermistor |
| R31 | 5.1 k | Lineariser end resistor |
| R32, R33 | 100 k | ICL7107 IN HI / IN LO series |
| R34 | 180 k | ICL7107 integrator resistor (0.93 V swing at 10 %, 1.85 V at 19.99) |
| R35 | 100 k | ICL7107 oscillator |
| R37 | 680 Ω | Decimal-point LED current (≈4.5 mA) |

| Ref | Value | Spec | Function |
|---|---|---|---|
| C1 | 4.7 µF | X5R ≥10 V | VBUS |
| C2 | 10 µF | X5R 25 V 0805 | VBAT |
| C3 | 22 µF | X5R 25 V 0805 | Boost input (VSW) |
| C4 | 22 µF | X5R 25 V 0805 | Boost output (VBOOST) |
| C5 | 10 µF | X5R 25 V 0805 | VBOOST_F (LDO input) |
| C6 | 10 µF | X5R 16 V 0805 | V5 at LDO output |
| C7 | 100 nF | X7R | V5 at LDO output |
| C8 | 1 µF | X7R | V5D |
| C9, C10 | 100 nF | X7R | P_REF–MID, MID–N_REF |
| C11 | 10 µF | X5R 0805 | MID–GND |
| C12 | 1 µF | X7R | AGND–GND at U5A |
| C13 | 100 nF | X7R | AGND–GND at U10 COMMON |
| C14 | 22 µF | X5R 10 V 0805 | Drive DC block (series, U5B → probe C1) |
| C15 | 100 pF | C0G | DRV_IN hold cap to AGND |
| C16 | 1 nF | C0G (X7R OK) | 74HC4060 C_TC |
| C17–C20 | 1 µF | X7R | Demod caps I_L, I_H, REF_L, REF_H to AGND |
| C21 | 1 nF | C0G/X7R | TIA feedback |
| C22 | 10 pF | C0G | Temperature stage feedback |
| C23 | 100 nF | X7R | U10 V+ |
| C24 | 10 µF | X5R 0805 | U10 V+ bulk (LED current) |
| C25, C26, C27, C28 | 100 nF | **C0G 1206 50 V, JLC C170182 (1206N104J500CT, 145,787 in stock)** | C_REF, C_INT, C_AZ ×2 (one part number → one Extended fee) |
| C29 | 100 pF | C0G | ICL7107 oscillator |
| C30–C35 | 100 nF | X7R | Decoupling: U1, U5, U6, U7, U8, U9 |

---

## 4. Netlist (wire exactly this)

Format: `Ref.pin (name) → NET`. "→ X → NET" means a series part.

### 4.1 Sheet 1 — POWER
- J1.A9, J1.B9 → **VBUS**. J1.A12, J1.B12, shell → **GND**. J1.A5 → R1 → GND. J1.B5 → R2 → GND. C1 VBUS–GND.
- U1.4 (VCC) → VBUS, C30 at pin. U1.2 → GND. U1.3 (BAT) → **VBAT**; C2 VBAT–GND. U1.5 (PROG) → R4 → GND. U1.1 (CHRG) → LED1 cathode; LED1 anode → R3 → VBUS.
- J2.1 → VBAT, J2.2 → GND. Silkscreen "+" at pin 1 and "CHECK BATT POLARITY".
- Q1: S → VBAT; D → **VSW**; G → **Q1_G**. R5 Q1_G–VBAT. R6 Q1_G → **SW_COM**. SW1 common → SW_COM; one throw → GND (= ON); other throw unconnected. Silkscreen "ON" on the grounding side.
- U2.5 (IN) → VSW; U2.4 (EN) → VSW; U2.2 → GND; U2.6 NC; U2.1 (SW) → **SW_NODE**. L1 VSW–SW_NODE. D1 anode SW_NODE, cathode **VBOOST**. R7 VBOOST–**FB**; R8 FB–GND; U2.3 → FB. C3 VSW–GND; C4 VBOOST–GND.
- FB1 VBOOST → **VBOOST_F**; C5 VBOOST_F–GND.
- U3.1 (VIN) → VBOOST_F; U3.3 (EN) → VBOOST_F; U3.2 → GND; U3.4 NC; U3.5 (VOUT) → **V5**; C6, C7 V5–GND at the pin.
- R9 V5 → **V5D**; C8 V5D–GND. (V5D feeds only U6 and U7.)

### 4.2 Sheet 2 — REFERENCE, AGND & CLOCK
- Divider: V5 → R10 → **P_REF** → R11 → **MID** → R12 → **N_REF** → R13 → GND. C9 P_REF–MID, C10 MID–N_REF, C11 MID–GND.
- U5A: pin 3 → MID; pin 2 → pin 1; pin 1 → R14 → **AGND**. C12 AGND–GND near R14. U5 pin 8 → V5 (C31), pin 4 → GND.
- U6 74HC4060: pin 16 → V5D (C32), pin 8 → GND, pin 12 (MR) → GND. RC oscillator per the Nexperia datasheet figure, with R_S = R16 (220 k), R_TC = R17 (22 k), C_TC = C16 (1 nF), all meeting at node **OSC_X**. Pin 7 (Q4) → **CLK**. Other Q outputs no-connect. Expect f_osc ≈ 18–21 kHz, CLK ≈ 1.1–1.3 kHz, 50 % duty (exact value does not matter).
- U7 74HC4053: pin 16 (VCC) → V5D (C33), pin 8 (GND) → GND, pin 7 (VEE) → GND, pin 6 (E̅) → GND, pins 11, 10, 9 (S1, S2, S3) → CLK.

### 4.3 Sheet 3 — PROBE FRONT END
- U7 section 1 (chopper): pin 12 (1Y0, CLK low) → N_REF; pin 13 (1Y1, CLK high) → P_REF; pin 14 (1Z) → **DRV_IN**. C15 DRV_IN–AGND.
- U5B (drive): pin 5 → DRV_IN; pin 6 → pin 7; pin 7 → R15 → **DRV_OUT**; C14 DRV_OUT → **PROBE_C1**.
- J3: 1 → PROBE_C1, 2 → **PROBE_P1**, 3 → **PROBE_P2**, 4 → **PROBE_C2**, 5 (shield) → GND. Silkscreen "C1 P1 P2 C2 SH".
- U8A (TIA): pin 3 → AGND; PROBE_C2 → R20 → **TIA_IN** = pin 2; feedback pin 1 → R21 → RV1 (rheostat: one end + wiper tied) → TIA_IN; C21 between pin 1 and TIA_IN. Pin 1 = **I_SIG**.
- U9 INA826: pin 1 (−IN) ← R24 ← PROBE_P1; R26 pin 1 → AGND. Pin 4 (+IN) ← R25 ← PROBE_P2; R27 pin 4 → AGND. Pins 2–3: R28 DNP. Pin 5 → GND. Pin 6 (REF) → AGND. Pin 8 → V5 (C35). Pin 7 → **INA_OUT**.
- Thermistor chain: INA_OUT → R29 → **NTC_A**. JP1 pad 2 (centre) = NTC_A; pad 1 → **NTC_EXT** = J4 pin 1; pad 3 → R30 → **NTC_B**. J4 pin 2 → NTC_B.
- U8B (temperature stage): pin 5 → AGND; NTC_B → R23 → **TS_IN** = pin 6; R22 ∥ C22 from pin 7 to pin 6. Pin 7 = **VT**. U8 pin 8 → V5 (C34), pin 4 → GND.
- Lineariser RV2: end (pin 1) → VT; other end (pin 3) → R31 → I_SIG; wiper (pin 2) → **REF_SIG**.

Signal polarity (keep exactly as wired): CLK HIGH selects P_REF → drive positive → I_SIG goes below AGND, VT goes above AGND.

### 4.4 Sheet 4 — DEMODULATOR, ADC & DISPLAY
- U7 section 2: pin 15 (2Z) ← R18 ← I_SIG; pin 2 (2Y0, CLK low) → **I_L**; pin 1 (2Y1, CLK high) → **I_H**.
- U7 section 3: pin 4 (3Z) ← R19 ← REF_SIG; pin 5 (3Y0, CLK low) → **REF_L**; pin 3 (3Y1, CLK high) → **REF_H**.
- C17 I_L–AGND, C18 I_H–AGND, C19 REF_L–AGND, C20 REF_H–AGND.
- U10 ICL7107 (wire by **pin name**):
  - V+ → V5 (C23 + C24 at the pin). **V− → GND. GND (digital/display ground) → GND.**
  - COMMON → AGND (C13 AGND–GND at the pin).
  - IN HI ← R32 ← I_L. IN LO ← R33 ← I_H.
  - REF HI → REF_H. REF LO → REF_L.
  - CREF+ / CREF−: C25 between them.
  - Integrator: BUFF → R34 → **INT_X**; INT → C26 → INT_X; A-Z → C27 ∥ C28 → INT_X.
  - Oscillator: OSC1 → node **OSC1_X**; R35 OSC1_X–OSC2; C29 OSC1_X–OSC3 (f ≈ 0.45/RC ≈ 45 kHz).
  - TEST → TP only (lamp test; pulling it to V+ lights all segments). Do not connect anything else.
  - NC pins → no-connect flags.
- Display (digit order left → right: DS1 = "1"/sign, DS2 = hundreds, DS3 = tens, DS4 = units). All four common-anode pins of each digit → **V5**. No series resistors on segments: ICL7107 segment outputs are current-limited sinks.
  - DS4 (units) cathodes a–g ← U10 A1, B1, C1, D1, E1, F1, G1.
  - DS3 (tens) cathodes a–g ← U10 A2…G2.
  - DS2 (hundreds) cathodes a–g ← U10 A3…G3.
  - DS1: cathodes b **and** c ← U10 AB4 (both segments on one pin); cathode g ← U10 POL (minus sign); cathodes a, d, e, f, dp → no-connect.
  - DS2 dp cathode → R37 → GND (decimal point permanently on, so 1000 counts reads "10.00" and 350 reads "3.50").
  - DS1, DS3, DS4 dp → no-connect.

### 4.5 Test pads
TP1 V5, TP2 AGND, TP3 GND, TP4 CLK, TP5 I_SIG, TP6 VT, TP7 U10 IN HI, TP8 U10 IN LO, TP9 REF_H, TP10 REF_L, plus an unnumbered pad on U10 TEST.

---

## 5. Phase 1 deliverables (then STOP)

1. ERC with zero errors; explain every remaining warning.
2. Export every net with its pin list and compare it line by line against Section 4. Report any difference.
3. Per part: LCSC number, Basic/Preferred/Extended, PCBA type (Economic/Standard Only), current stock, unit price, and the datasheet link you used to confirm the pinout. Flag any shortage, pin-name mismatch, or Basic/Preferred alternative.
4. Export a PDF of all sheets and give the user the path or link.
5. Ask the user to approve before Phase 2.

---

## 6. Phase 2 — PCB (only after approval)

### 6.1 Stack-up and rules
- 2 layers, FR-4, 1.6 mm, 1 oz copper. Ask the user for solder-mask and silkscreen colours before export (they may want them to match a colour scheme used across their builds).
- Clearance 0.2 mm minimum everywhere, 0.3 mm around SW_NODE and VBUS.
- Trace widths: signals 0.25 mm; AGND 0.4 mm; V5 0.5 mm (it carries up to ~200 mA of LED current); V5D 0.3 mm; VBUS, VBAT, VSW, VBOOST, VBOOST_F, SW_NODE 0.6 mm.
- Vias 0.3 mm drill / 0.6 mm pad; GND stitching vias every ~5 mm along pour edges and around the boost section.

### 6.2 Board size and mechanical
- Outline **≤ 60 × 40 mm**, 1 mm corner radius.
- Four **M3 mounting holes**: 3.2 mm plated, 6 mm GND pads, centres 3.5 mm in from each edge at the corners; no other copper, parts or traces within 3.5 mm of a hole centre.
- **All parts on TOP** (Economic PCBA, single side). The four digits (DS1–DS4, 4 × 7.5 mm = 30 mm wide, 14 mm tall) sit in a row centred along one long edge on 7.6 mm pitch so they read as one display.
- If everything does not fit on one side at 60 × 40 mm, first shrink passives to 0402 (Basic 0402 parts exist for all values here except the 1206 C0G and the 0805 bulk caps), then ask the user before moving parts to the bottom (that forces Standard PCBA).
- Edges: J1 USB-C on one short edge, J3 probe and J4 thermistor on the opposite short edge, J2 and SW1 on the long edge opposite the display. SW1's actuator flush with or overhanging the edge. RV1 and RV2 near an edge, adjust screws reachable through a case.

### 6.3 Placement zones
1. **Power zone** (corner by J1/J2/SW1): J1, U1, LED1, C1, C2, J2, Q1, SW1, then U2, L1, D1, C3, C4 in a tight loop (input cap → IC → inductor → diode → output cap, minimum loop area), FB1, C5, U3, C6, C7, R9, C8. Keep SW_NODE copper tiny. At least 8 mm from TIA_IN and U10's integrator parts.
2. **Display zone**: DS1–DS4 row, with U10 directly behind it so the 23 segment lines run as a short, straight, evenly spaced bus.
3. **ADC zone**: U10 with C25–C29, R34, R35 within 4 mm of their pins; demod caps C17–C20 between U7 and U10.
4. **Analog zone** (by J3/J4): U5, U8, U9, RV1, RV2, JP1, reference divider. J3 → R20 → U8 pin 2 as short as possible. P1/P2 traces from J3 to R24/R25 to U9 as a parallel, equal-length pair with GND pour on both sides.
5. **Clock/logic**: U6, U7 between the analog and ADC zones; CLK short.

### 6.4 Routing quality (the user's priority — clean and aligned)
- 45° corners only; no 90° corners, no acute angles, no stubs.
- Parallel groups (segment bus; demod lines; P1/P2) as evenly spaced buses with identical pitch (0.25 mm at 0.5 mm pitch), aligned and turning together.
- Top mainly horizontal, bottom mainly vertical; few layer changes, placed in neat rows.
- **GND** pour on both layers (0.3 mm clearance, thermal reliefs), stitched. **Do not pour AGND.**
- LED return current: route U10's GND pin and the V5 feed to the digits with their own wide traces back to the LDO, so display current does not flow under the analog zone.
- AGND guard trace (0.3 mm) around the TIA_IN node (U8 pin 2, R20, R21/RV1, C21) and around U10's INT_X, CREF± and A-Z nodes.
- CLK, OSC_X and all 74HC control traces ≥ 2 mm from TIA_IN, PROBE_P1/P2 and INT_X; never parallel to them.
- Every decoupling cap within 2 mm of its supply pin with its GND via at the pad.

### 6.5 Silkscreen
"SALINOMETER v2 Rev C", "USB-C CHARGE ONLY", "+" and "CHECK BATT POLARITY" at J2, "C1 P1 P2 C2 SH" at J3, "NTC" at J4, "SPAN" at RV1, "LIN" at RV2, "ON" at SW1, "NTC / 10k" at JP1, test pad names, pin-1 marks on every IC, "%" next to DS4.

### 6.6 Checks and outputs
1. DRC: zero errors; connectivity: zero unrouted nets. Confirm AGND is not connected to GND anywhere.
2. 3D view: digits clear of every part, USB-C opening at the board edge, trimmer screws accessible.
3. Export Gerber, drill, **BOM and CPL in JLCPCB format** with LCSC numbers. DS1–DS4 are included (through-hole, wave soldering). Check CPL rotations in the JLC preview for U1, U2, U3, U5–U10, D1, LED1, Q1, J1, J3, SW1, RV1, RV2, DS1–DS4.
4. Give the user: board size, layer count, Extended-part list with fees, and top/bottom/3D screenshots.

---

## 7. Ask the user rather than guess
- If the FJ3161BH pin map in the datasheet and in the EasyEDA symbol disagree.
- If anything does not fit at 60 × 40 mm on one side.
- Board colours.
- Whether to add a TVS on VBUS or reverse-battery protection (not in Rev C).

---

## 8. Design notes the build must preserve (do not "fix" these)
- ICL7107 V− tied to GND and COMMON at 2.5 V is the datasheet's single-supply configuration; it is valid only because the reference is external (REF_H/REF_L) and the inputs are small and centred on 2.5 V. Do not add a negative supply or connect REF LO / IN LO to GND.
- No resistors in series with the display segments.
- C14 must stay in series with probe electrode C1 (no DC through the probe).
- Expected signal levels for bring-up (probe in 1–10 % salt, 15–30 °C): VT 2.1–2.9 V, I_SIG 2.4–2.6 V square waves around AGND = 2.50 V; integrator swing at U10 INT ≤ 1.9 V from 2.5 V.
- Power: about 100–200 mA from V5 depending on lit segments; a 2000 mAh cell gives roughly 6–11 h. Charge with the switch off.
