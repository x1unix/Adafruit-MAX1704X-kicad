# AGENTS.md

Guidance for AI agents working in this repository.

## What this is

A KiCad 10 port of Adafruit's EagleCAD project for the MAX17048 fuel gauge breakout
(Adafruit 5580), converted from upstream commit `cc7d6e2`. The chip is
**MAX17048G+T10** (TDFN-8, 2 x 2 mm). Read [`README.md`](README.md) first, then
[`docs/`](docs/).

**For anything about the chip itself** — pinout, I2C address, registers, electrical
limits — use [`docs/max17048-datasheet.md`](docs/max17048-datasheet.md). It is a
condensation of the official PDF written for agents to consume, so you don't have to
parse 19 pages of scanned tables. The PDF
([`docs/max17048-max17049.pdf`](docs/max17048-max17049.pdf)) remains authoritative for
anything tolerance- or safety-critical.

This is a hardware design, not software. There is no build, no test suite, and changes
are verified with `kicad-cli` and by looking at the board.

## Rules

### 1. Never judge a pin by its label

Adafruit's upstream MAX17048 symbol had `SCL` and `SDA` transposed, with the schematic
wiring deliberately crossed to compensate. The board was always correct. "Fixing" the
wiring would have broken it.

**Before changing anything that looks mis-pinned, check the net-to-pad mapping against
the datasheet, not the pin names:**

```sh
kicad-cli sch export netlist --format kicadxml -o /tmp/net.xml Adafruit-MAX17048-STEMMA.kicad_sch
```

Full story: [`docs/max17048-symbol.md`](docs/max17048-symbol.md); authoritative pinout
in [`docs/max17048-datasheet.md`](docs/max17048-datasheet.md).

### 2. Preserve Adafruit's footprints

Footprints are stored **inline in the `.kicad_pcb`** and are Adafruit's originals. They
intentionally differ from KiCad-official equivalents (oversized JST tabs, 0.9 mm outer
pitch on the resistor array, longer JST PH pads). Do not "normalize" them to stock
KiCad footprints — that changes the board. See [`docs/footprints.md`](docs/footprints.md).

### 3. Symbol edits touch two files

A `.kicad_sch` embeds its own `lib_symbols` copy of every symbol, and that copy is what
renders. Any scripted symbol edit must change **both**:

- `Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym`
- the `lib_symbols` section inside `Adafruit-MAX17048-STEMMA.kicad_sch`

Confirm they're in sync afterwards — `kicad-cli sch erc` must report
`lib_symbol_issues: 0`.

Wires attach to pin positions and numbers, never names, so pure renames leave the
netlist untouched. Prove it by diffing the netlist before and after.

### 4. Check whether KiCad is running before editing files

```sh
ps aux | grep '[k]icad'; ls *.lck
```

`.lck` files mean KiCad holds the project open and will overwrite on-disk edits when it
saves. Ask the user to close it, or tell them explicitly to close **without saving** and
reopen.

### 5. Don't re-raise deliberately deferred work

The user scoped these out. They are known state, not pending bugs:

- **3D models are not attached.** The Eagle source had none.
  [`docs/3d-models.md`](docs/3d-models.md).
- **DRC and ERC report violations**, nearly all false positives or upstream choices.
  [`docs/known-issues.md`](docs/known-issues.md). Notably, DRC flags SJ1/SJ2 as shorting
  nets — those are solder jumpers doing their job.
- **Design rules and net classes did not carry over.** Board Setup is at KiCad defaults.

## Tooling

`kicad-cli` (10.0.5) imports **boards** headlessly but has **no schematic import** —
`kicad-cli sch` offers only `erc`, `export`, `upgrade`, and there is no standalone
`eeschema` binary on macOS. Eagle schematic conversion is GUI-only:
`File -> Import Non-KiCad Project -> Import EAGLE Project...`.

Useful checks:

```sh
kicad-cli pcb drc    --format json -o /tmp/drc.json Adafruit-MAX17048-STEMMA.kicad_pcb
kicad-cli sch erc    --format json -o /tmp/erc.json Adafruit-MAX17048-STEMMA.kicad_sch
kicad-cli pcb render --side top    -o /tmp/top.png  Adafruit-MAX17048-STEMMA.kicad_pcb
kicad-cli sym export svg           -o /tmp/syms     Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym
```

Rendering the board is a fast way to confirm a change didn't wreck the geometry.

## Repository layout

```
Adafruit-MAX17048-STEMMA.kicad_pro    project
Adafruit-MAX17048-STEMMA.kicad_sch    schematic (embeds lib_symbols)
Adafruit-MAX17048-STEMMA.kicad_pcb    board (footprints inline)
Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym   project symbol library
sym-lib-table                         registers the symbol library
3dmodels/                             project-local STEP files, ${KIPRJMOD}/3dmodels/
docs/                                 conversion knowledge + datasheet
```

Not worth committing: `*.lck`, `*.kicad_prl`, `.history/`, `fp-info-cache`.

## Conventions

- Keep documentation in `docs/`; `README.md` stays an overview with links.
- When you discover something non-obvious about the design, write it into the relevant
  `docs/` file rather than only reporting it in chat.
- Upstream is CC BY-SA 3.0 and the Adafruit attribution block in `README.md` must stay
  in any redistribution.
