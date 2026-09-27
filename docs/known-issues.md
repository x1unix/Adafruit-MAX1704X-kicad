# Known issues and false positives

Both DRC and ERC report violations on this design. **Most are artifacts of the
conversion or of upstream design choices, not defects.** This document separates them
so nobody "fixes" a working board.

## DRC — 106 violations, 1 unconnected

Measured on the GUI-imported board with zones filled and KiCad default rules.

### Not bugs

| Finding | Count | Why |
|---|---|---|
| `silk_overlap`, `silk_over_copper` | 68 | Eagle tolerated silkscreen over pads; KiCad's defaults do not. Upstream shipped this way. |
| `solder_mask_bridge` | 25 | Fine-pitch parts against KiCad's default mask rules. |
| `clearance` on SJ1 (pad 1 / WIRE, pad 2 / WIRE) | 2 | **The solder jumper working as designed.** `SOLDERJUMPER_CLOSEDWIRE` has a netless `WIRE` pad that deliberately bridges pads 1 and 2 — that is what makes it normally-closed. See *Solder jumpers* below. |
| `items_not_allowed` | 2 | The mounting-hole pads sitting inside their own imported keepout. |
| `text_thickness`, `text_height` | 4 | Silkscreen text below KiCad's default minimums. |

### Worth attention

- **`shorting_items` + 2 `clearance` involving net `N$4` — orphan copper inherited from
  upstream.** In the Eagle source, signal `N$4` has **zero contactrefs** and exactly one
  wire segment. It is a 0.635 mm dangling stub on the bottom layer that overlaps SJ2's
  pad 2 (`VDD`) and runs within 0.076 mm of the `VBAT` trace. Electrically it is just an
  appendix hanging off the VDD pad and goes nowhere, so the board works — but it is
  stray copper, not intentional design. Deleting it is safe and clears all three
  violations.

- **1 unconnected: an isolated GND island on F.Cu.** The top-layer GND pour fills as
  three islands. Two are properly stitched; the third (~2.33 mm², roughly
  X 150.4–154.6, Y 105.1–107.2, between X3 and CONN3) contains **no GND via and no GND
  pad**, so it is floating copper. Harmless at this size and frequency, but fix it by
  adding a stitching via, or set the zone's island removal to drop sub-threshold
  islands.

- **2 `starved_thermal`.** JP2 pad 2 (GND, B.Cu) and X3 pad 4 (GND, F.Cu) each get one
  thermal spoke where KiCad's default rule wants two. Upstream had no such rule. X3's
  exposed pad is also tied to GND, so that one is immaterial.

- **Design rules and net classes did not carry over.** Board Setup is at KiCad defaults,
  which are *stricter* than what this board was manufactured to. Set real trace/via/
  clearance minimums before using DRC output to judge the layout.

### Zone fill

Zones must be filled before DRC output means anything. Filling took unconnected items
from 20 to 1 and cleared all 6 `via_dangling` findings. If a checkout shows them
unfilled, press `B` in the PCB editor and re-run DRC before reading anything into the
counts above.

## Solder jumpers

KiCad models solder jumpers perfectly well — it ships 30 footprints in `Jumper.pretty`
(including `SolderJumper-2_P1.3mm_Bridged_*`) and 11 symbols in `Jumper.kicad_sym`
(`SolderJumper_2_Bridged`, `SolderJumper_3_Bridged12`, …). The everyday idiom is the
same as a **0 ohm resistor**: a two-pin part bridging two nets.

The difference is how the bridge is drawn. KiCad's bridged jumpers have **only two
pads** and no copper between them — the short is a solder blob the assembler adds, so
nothing overlaps and DRC stays quiet. Adafruit's Eagle footprint instead adds a **third
netless `WIRE` pad** physically overlapping pads 1 and 2, which KiCad reads as two
zero-clearance violations.

So the noise is a property of *this imported footprint*, not a gap in KiCad. Options:

1. **Leave it.** The violations are cosmetic; the board is correct.
2. **Give the `WIRE` pad a net** so it stops being a foreign object between two pads.
3. **Rebuild SJ1 as a KiCad solder jumper** (`SolderJumper_2_Bridged` +
   `SolderJumper-2_P1.3mm_Bridged_*`) or a 0 ohm resistor. Cleanest long term, but it
   replaces Adafruit's footprint — see [`footprints.md`](footprints.md) before doing it.

## ERC — 39 violations

| Finding | Count | Why |
|---|---|---|
| `footprint_link_issues` | 25 | Symbols' footprint fields don't resolve to a library, because footprints live **inline in the `.kicad_pcb`** and there is no `fp-lib-table`. Harmless unless you want "update PCB from schematic" to work. |
| `power_pin_not_driven` | 4 | No `PWR_FLAG` symbols. Normal for an imported schematic. |
| `unconnected_wire_endpoint` | 6 | Import artifacts. |
| `missing_bidi_pin`, `missing_unit` | 4 | Import artifacts. |

`lib_symbol_issues` should read **0**. A non-zero count means the project
`.kicad_sym` and the `lib_symbols` cache inside the `.kicad_sch` have drifted apart —
see [`max17048-symbol.md`](max17048-symbol.md).

## Things that look wrong but are correct

- **MAX17048 SCL/SDA.** Upstream had the labels transposed with compensating wiring.
  Corrected here, netlist unchanged. Full detail in [`max17048-symbol.md`](max17048-symbol.md).
- **No `CTG` pin on the MAX17048 symbol.** Datasheet pin 1 is "connect to ground";
  Adafruit folded it into `VSS`.
- **`VSS` has no single pin number** — it carries pads 1, 4 and THERMAL as stacked pins.
- **`UNK_HOLE_0` / `UNK_HOLE_1`** are Eagle free holes, not unannotated components.
- **Adafruit's footprints differ from KiCad-official ones.** Intentional. See
  [`footprints.md`](footprints.md).
