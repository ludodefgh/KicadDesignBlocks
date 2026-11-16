# TPS563208 buck block: 5 V → 3.3 V / 1 A

A self-contained KiCad 10 project with just the TPS563208 and its passives, as laid
out in the MultiSensors sensor node (a private repository). It
exists so the power stage can be reviewed on its own, without the rest of the board.

The folder and files keep the name `TPS563201`: the block started with that part and
switched to the TPS563208 (same package, pinout and passives) in November 2026.

![3D top view](docs/TPS563201-3d-top.png)

| File | What it is |
|---|---|
| `TPS563201.kicad_pro` / `.kicad_sch` / `.kicad_pcb` | the project (open the `.kicad_pro`) |
| `docs/TPS563201-schematic.pdf` | schematic, readable without KiCad |
| `docs/TPS563201-layout.svg` | F.Cu, B.Cu, silkscreen and outline, readable without KiCad |
| `docs/webench/` | TI WEBENCH exports the part values came from (BOM CSV, layout images) |
| `sym/Block.kicad_sym` | project-local symbols, so the project opens on any machine |

## Operating point

| | Value |
|---|---|
| Input | 4.75–5.25 V, USB-C VBUS (the connector is outside this block) |
| Output | 3.32 V (0.768 V × (1 + 33.2 k / 10 k)) |
| Design current | 1 A (expected load: about 100 mA average, about 570 mA peak) |
| Switching frequency | 580 kHz (TPS563208, D-CAP2, forced continuous conduction (FCCM): no pulse skipping at light load) |
| Main loads on 3.3 V | MCU radio bursts, SCD41 CO₂ sensor (asks for a low-noise supply) |

## Parts

| Ref | Part | Value | LCSC |
|---|---|---|---|
| U11 | TI TPS563208DDCR, SOT-23-6 | | C468250 |
| L1 | Sunlord SWPA4030S3R3MT, 4 × 4 × 3 mm | 3.3 µH, Isat 3.6 A, DCR 52 mΩ | C15269 |
| C5, C6 | Samsung CL21A106KAYNNNE, 0805 | 10 µF 25 V X5R (input) | C15850 |
| C7 | 0805 | 100 nF (input, high-frequency) | |
| C8, C9 | Samsung CL21A226MAQNNNE, 0805 | 22 µF 25 V X5R (output) | C45783 |
| C4 | Yageo CC0603KRX7R9BB104, 0603 | 100 nF 50 V X7R (bootstrap) | C14663 |
| R5 | UNI-ROYAL 0603WAF3322T5E, 0603 | 33.2 k 1 % (feedback, upper) | C23003 |
| R6 | UNI-ROYAL 0603WAF1002T5E, 0603 | 10 k 1 % (feedback, lower) | C25804 |

- **L1**: at 4.75–5.25 V in, the ripple is 0.52–0.64 A peak-to-peak, so the inductor
  peaks at about 1.3 A at 1 A out.
- **Output capacitors**: two 22 µF give about 29.6 µF at 3.3 V DC bias (Samsung
  DC-bias curve), inside TI's 20–68 µF range. The Samsung part is marked NRND; its
  replacement CL21A226MAYNNN (C602037) gives about 27.4 µF.
- **C7**: optional in TI's Table 7-2, kept because the TPS563201EVM-715 fits it (C1)
  and WEBENCH reserves a spot for it.
- **U11, TPS563208 rather than TPS563201**: the TPS563201 runs in Eco-mode (pulse
  skipping) below about 0.29 A here, which gives a low-frequency ripple on 3.3 V.
  The SCD41 asks for at most 30 mV peak-to-peak on its unloaded supply (SCD4x
  datasheet, Table 4). The TPS563208 is the FCCM variant of the same datasheet
  (SLVSD90): it switches at a fixed frequency at any load, for about 3 mV of
  calculated ripple with the output capacitors here. The cost is a higher quiescent
  current and a lower light-load efficiency, which do not matter on USB power.
- **No 0402**: every passive is 0603 or larger, for hand assembly on a hot plate.
  R5 and R6 are pinned to 1 % parts, as TI asks for the divider (§7.2.2.2).

## Board and layout

- **4 layers: F.Cu / In1 = GND plane / In2 = +3.3 V plane / B.Cu.** The inner planes
  are cut to this block's outline. In the full board they cover the whole PCB.
- All parts are on the top side, for hot-plate assembly.
- The layout follows TI SLVSD90B §7.4 (fig. 7-18) and the TPS563201EVM-715 (SLVUAK7):
  - C7 sits across U11 pins 1 (GND) and 3 (VIN), at the same 1.9 mm pitch. The SW
    node leaves between its pads, under its body.
  - C5 and C6 sit right behind C7, on shared GND / +5 V copper areas.
  - L1 is above U11. The SW node is a small island: a copper area on F.Cu, plus a
    small area on B.Cu that crosses under the GND path through vias.
  - The input and output capacitors share one GND copper area that ties into U11
    pin 1.
  - The feedback divider sits on the quiet side of U11, away from L1 and SW.
- Design rules come from MultiSensors: 0.15 mm clearance, 0.4 mm minimum on power
  nets, and hand-assembly margins (0.5 mm between courtyards, with the pairs inside
  this block exempted because TI wants them tight) (`TPS563201.kicad_dru`).

## Where a review would help

1. **SW copper on B.Cu.** The SW pin reaches L1 through 6 vias and a 6.7 mm² area
   on B.Cu, as in TI's fig. 7-18; there is no direct path on F.Cu. It faces the
   +3.3 V plane at about 0.2 mm, which estimates to about 1 pF onto a heavily
   decoupled plane. A reviewer suggested keeping SW on the top layer instead; that
   needs a redraw of the block. Moving it onto In1 would cut the GND plane under the
   hot loop, so it is not an option; In2 (3.3 V plane) would be.
2. **Bootstrap path.** C4 stands upright to the right of L1, above U11 pin 6. VBST
   reaches it with a 0.6 mm trace, about 1 mm between the two pad edges. Its SW pad
   picks up SW from L1's SW pad on F.Cu: about 1.6 mm of trace between the two pad
   edges, over the GND plane.
3. **Feedback sense.** R5 measures the output through a via into the +3.3 V plane,
   not through a dedicated trace from the output capacitors (TI rule 6). R6 returns
   to GND through its own via, not a Kelvin connection to U11 pin 1 (rule 7).
4. **No series resistor on the bootstrap capacitor.** The TI EVM fits a 0 Ω (R5
   there) so the SW edges can be slowed for EMC. There is none here.

## Status

Not fabricated, not measured. The schematic passes ERC with 0 errors and 0 warnings.
The PCB passes DRC with 0 errors, 0 warnings, 0 unconnected items and 0
schematic/PCB mismatches.

## Source

This project is **generated** from MultiSensors with
`kicad/_generators/export_tps563201_block.py <this folder>`. The script takes values,
footprints and MPN/LCSC fields from the MultiSensors schematic, and the layout from
the user's hand-drawn U11 group in the MultiSensors board. Change the block in
MultiSensors and re-export; edits made only here are lost at the next export.
