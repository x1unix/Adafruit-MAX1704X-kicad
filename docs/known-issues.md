# Known issues and false positives

Both DRC and ERC report violations on this design. **Most are artifacts of the
conversion or of upstream design choices, not defects.** This document separates them
so nobody "fixes" a working board.

## DRC — 115 violations, 20 unconnected

Measured on the CLI-imported board with KiCad default rules. Counts shift slightly
after the GUI import and after filling zones.

### Not bugs

| Finding | Count | Why |
|---|---|---|
| `shorting_items`, `clearance` on SJ1 / SJ2 | 1 + 4 | **The solder jumpers working as designed.** `SOLDERJUMPER_CLOSEDWIRE`'s `WIRE` pad deliberately bridges pads 1 and 2 — that is what makes it a normally-closed jumper. KiCad has no concept of an Eagle solder jumper. |
| `silk_overlap`, `silk_over_copper` | 68 | Eagle tolerated silkscreen over pads; KiCad's defaults do not. Upstream shipped this way. |
| `solder_mask_bridge` | 22 | Fine-pitch parts against KiCad's default mask rules. |
| `copper_edge_clearance` | 8 | Connector mounting tabs at 0.32–0.36 mm against KiCad's default 0.5 mm. Adafruit's own rules were looser. |
| `items_not_allowed` | 2 | The mounting-hole pads sitting inside their own imported keepout. |
| `via_dangling` | 6 | Stitching vias. |
| `unconnected_items` | 20 | **Zones are unfilled.** Press `B` in the PCB editor; most clear. |

### Worth attention

- Design rules and net classes did not carry over. Board Setup is at KiCad defaults,
  which are *stricter* than what this board was manufactured to. Set real
  trace/via/clearance minimums before using DRC output to judge the layout.

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
