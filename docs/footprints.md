# Footprint provenance

All footprints are **Adafruit's originals**, carried over from the Eagle `.brd` and
stored inline in `Adafruit-MAX17048-STEMMA.kicad_pcb`.

**Do not swap these for KiCad-official footprints.** Adafruit's pads deliberately
differ — oversized JST mounting tabs, 0.9 mm outer pitch on the resistor array, longer
JST PH pin pads. Substituting stock footprints would change the board.

## Real parts behind each footprint

Identified by comparing pad geometry against the KiCad-official library. Eagle stores
pad rotation separately (`rot="R90"`), which has to be applied before comparing.

| Ref | Eagle footprint | Actual part | Match quality |
|---|---|---|---|
| X3 | `TDFN8_2X2MM` | **MAX17048G+T10**, TDFN-8 2x2 mm, 0.5 mm pitch, EP 0.8 x 1.38 mm (package outline 21-0168) | KiCad equivalent: `TDFN-8-1EP_2x2mm_P0.5mm_EP0.8x1.2mm` |
| CONN3, CONN4 | `JST_SH4` | **JST SM04B-SRSS-TB**, SH series 1.0 mm, 4-pin, side-entry SMT (STEMMA QT / Qwiic) | Exact match to KiCad-official: 0.6 x 1.55 pins at 1 mm pitch, 1.2 x 1.8 tabs |
| X1, X2 | `JSTPH2_BATT` | **JST S2B-PH-SM4-TB**, PH series 2.0 mm, 2-pin, side-entry SMT with tabs | Same part; Adafruit uses longer pin pads (4.6 vs 3.5 mm) and larger tabs |
| R3 | `RESPACK_4X0603` | 4 x 0603 convex resistor array, 10K | Near-match to `R_Array_Convex_4x0603`; Adafruit's outer pads sit at 0.9 mm, stock is 0.8 mm, and the array runs along X rather than Y |
| C1, C2 | `0603-NO` | 1uF 0603 MLCC | Standard 0603 |
| R2 | `0603-NO` | 10K 0603 | Standard 0603 |
| D1 | `CHIPLED_0603_NOOUTLINE` | Green 0603 LED | Standard 0603 |
| JP1 | `1X06_ROUND_70` | 1x6 2.54 mm THT header | 1 mm drill, 1.778 mm pad |
| JP2 | `1X02_ROUND` | 1x2 2.54 mm THT header | 1 mm drill, 1.778 mm pad |
| SJ1 | `SOLDERJUMPER_CLOSEDWIRE` | Solder jumper, normally closed | The `WIRE` pad intentionally bridges pads 1 and 2 |
| SJ2 | `SOLDERJUMPER_2WAY_OPEN_NOPASTE` | 3-pad 2-way solder jumper | |
| U$19, U$21 | `MOUNTINGHOLE_2.5_PLATED` | 2.5 mm plated mounting hole, 3.2 mm pad | |
| FID3, FID4 | `FIDUCIAL_1MM` | 1 mm fiducial | |
| U$22 | `ADAFRUIT_3.5MM` | Adafruit logo, silkscreen only | |
| U$25 | `PCBFEAT-REV-040` | Revision marker, silkscreen only (`${REV}` = 040) | |
| U$30, U$31 | `STEMMAQT` | STEMMA QT label, silkscreen only | |
| UNK_HOLE_0/1 | — | Eagle free holes, 2.54 mm NPTH — not components | |

## Connector pinouts

STEMMA QT / Qwiic (`JST_SH4`), matching the standard:

```
1 = GND    2 = V+ (VCC)    3 = SDA    4 = SCL
```

Both CONN3 and CONN4 are wired in parallel, as pass-through I2C ports.
