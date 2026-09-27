# Adafruit MAX17048 LiPoly / LiIon Fuel Gauge — KiCad port

A **KiCad 10 conversion of Adafruit's original EagleCAD project** for the MAX17048
LiPoly / LiIon Fuel Gauge and Battery Monitor breakout (STEMMA QT form factor).

This repository contains **only the converted design**. It is not an Adafruit product
and is not maintained by Adafruit.

## Upstream

| | |
|---|---|
| Original project | <https://github.com/adafruit/Adafruit-MAX17048-PCB> |
| Product page | <https://www.adafruit.com/product/5580> |
| Converted from commit | `cc7d6e2` — *"Adding cookiecutter output"*, 2022-08-31 |
| Source format | EagleCAD 9.6.2 (`.sch` + `.brd`) |
| Designed by | Limor Fried / Ladyada for Adafruit Industries |

## Board

| | |
|---|---|
| Fuel gauge IC | **MAX17048G+T10** — TDFN-8, 2 x 2 mm, 0.5 mm pitch (package outline 21-0168) |
| Size | 25.40 x 20.32 mm (1.00 x 0.80 in) |
| Stackup | 2 layer |
| Interface | I2C, two STEMMA QT / Qwiic connectors (JST SH 4-pin) |
| Battery | Two JST PH 2-pin ports |
| Converted with | KiCad 10.0.5 |

`MAX17048G+T10` decodes as: `MAX17048` 1-cell ModelGauge fuel gauge, `G` TDFN package,
`+` lead-free, `T10` tape and reel. Datasheet is in [`docs/max17048-max17049.pdf`](docs/max17048-max17049.pdf).

## Opening it

Open `Adafruit-MAX17048-STEMMA.kicad_pro` in KiCad 10.0.5 or newer.

Footprints are stored **inline in the `.kicad_pcb`**, and symbols live in the
project-local `Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym`. There are no external
library dependencies — the project opens standalone.

## Deviations from upstream

One intentional change. Everything else is a faithful conversion.

- **MAX17048 symbol pin labels corrected.** Adafruit's Eagle symbol had `SCL` and `SDA`
  transposed, compensated by deliberately crossed net wiring. The labels now match the
  datasheet. **The netlist is byte-identical to upstream** — this is purely cosmetic.
  Full explanation: [`docs/max17048-symbol.md`](docs/max17048-symbol.md).

3D models were not inherited — the Eagle source contained none. The MAX17048 model has
since been added; the remaining parts still have empty 3D slots. See
[`docs/3d-models.md`](docs/3d-models.md).

## Documentation

| Document | Contents |
|---|---|
| [`docs/conversion.md`](docs/conversion.md) | How the conversion was performed, and what the importer did or didn't carry over |
| [`docs/max17048-symbol.md`](docs/max17048-symbol.md) | The upstream SCL/SDA pin-label defect and why the board is still correct |
| [`docs/footprints.md`](docs/footprints.md) | Footprint provenance and the real manufacturer part behind each one |
| [`docs/3d-models.md`](docs/3d-models.md) | 3D model status, stock matches, and what needs downloading |
| [`docs/known-issues.md`](docs/known-issues.md) | DRC/ERC output, and which findings are false positives |

## License

The original design is released by Adafruit under
**Creative Commons Attribution-ShareAlike 3.0 Unported** — full text in
[`license.txt`](license.txt), copied verbatim from upstream. This conversion inherits
that license.

> Adafruit invests time and resources providing this open source design, please support
> Adafruit and open-source hardware by purchasing products from
> [Adafruit](https://www.adafruit.com)!
>
> Designed by Limor Fried/Ladyada for Adafruit Industries.
>
> Creative Commons Attribution/Share-Alike, all text above must be included in any
> redistribution.
