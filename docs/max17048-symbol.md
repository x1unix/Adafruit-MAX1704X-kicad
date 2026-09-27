# The MAX17048 SCL/SDA pin-label defect

**Summary: Adafruit's Eagle symbol labels SCL and SDA backwards, and the schematic
wiring compensates. The copper is correct. Do not "fix" this by rewiring.**

## The datasheet

`MAX17048G+T10`, TDFN-8 2 x 2 mm (package outline 21-0168):

```
[ 1 ] CTG      SDA [ 8 ]
[ 2 ] CELL     SCL [ 7 ]
[ 3 ] VDD    QSTRT [ 6 ]
[ 4 ] GND     ALRT [ 5 ]
                  [ EP ] thermal pad
```

Pin 7 is SCL. Pin 8 is SDA. (`docs/max17048-max17049.pdf`, Pin Configuration.)

## Error 1 — the symbol

In Adafruit's `adafruit_sensor` Eagle library, `MAX17048/T` maps pin *names* to pads
like this:

```
SCL   -> pad 8      (datasheet says pad 8 is SDA)
SDA   -> pad 7      (datasheet says pad 7 is SCL)
VDD   -> pad 3      CELL  -> pad 2
!ALRT -> pad 5      QSTRT -> pad 6
VSS   -> pads 1, 4, THERMAL
```

Both I2C labels are transposed.

## Error 2 — the wiring, which cancels it

The schematic then connects the nets to the *wrong-looking* pins, on purpose:

```
net SCL  ->  symbol pin "SDA"  ->  pad 7   = SCL   correct
net SDA  ->  symbol pin "SCL"  ->  pad 8   = SDA   correct
```

The Eagle board file confirms the signals land on the right copper:

```
net SDA -> CONN3.3, X3.8
net SCL -> CONN3.4, X3.7
```

That also matches the STEMMA QT / Qwiic standard (1 = GND, 2 = V+, 3 = SDA, 4 = SCL),
so the whole signal chain is sound. This is why the shipping product works.

## What was changed in this port

Only the two pin **names** on `MAX17048/T`, in both places KiCad stores a symbol:

- `Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym` (the project library)
- the `lib_symbols` cache embedded in `Adafruit-MAX17048-STEMMA.kicad_sch`

Both are required — the schematic embeds its own copy of every symbol, and that copy is
what the canvas renders. Editing one alone leaves the two out of sync, which ERC reports
as `lib_symbol_issues`.

No wires were touched. Wires attach to pin positions and numbers, not names, so net
`SCL` stayed on pin 7 and simply reads correctly now. The `.kicad_pcb` was not modified.

### Verification

Exporting the full `(ref, pin)` mapping for every net before and after gave an
identical result:

```
FULL netlist identical before vs after?  YES - electrically unchanged
```

Resulting X3 assignment, every pin matching the datasheet:

```
net SCL    -> pin 7        name=SCL_7         datasheet=SCL
net SDA    -> pin 8        name=SDA_8         datasheet=SDA
net VDD    -> pin 3        name=VDD_3         datasheet=VDD
net VBAT   -> pin 2        name=CELL_2        datasheet=CELL
net INT    -> pin 5        name=~{ALRT}_5     datasheet=ALRT
net QSTART -> pin 6        name=QSTRT_6       datasheet=QSTRT
net GND    -> pins 1, 4, THERMAL              CTG + GND + EP
```

Reproduce with:

```sh
kicad-cli sch export netlist --format kicadxml \
  -o /tmp/net.xml Adafruit-MAX17048-STEMMA.kicad_sch
```

## Related non-issues

- **There is no `CTG` pin on the symbol.** The datasheet defines pin 1 as CTG,
  "connect to ground", so Adafruit folded it into `VSS`.
- **`VSS` shows no single pin number** because it carries three pads — 1, 4 and
  THERMAL. KiCad imports that as stacked pins; all three are present in the netlist.
