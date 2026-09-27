# How this conversion was performed

Source: Adafruit's EagleCAD 9.6.2 project, commit `cc7d6e2` (2022-08-31).
Target: KiCad 10.0.5.

## Why the GUI was required

KiCad exposes its **board** importers to the command line but not its **schematic**
importers:

- `kicad-cli pcb import` — converts Eagle / Altium / PADS / CADSTAR / pcad boards headlessly.
- `kicad-cli sch` — offers only `erc`, `export`, `upgrade`. There is **no** `import`.

There is no standalone `eeschema` binary on macOS either; the editors ship as separate
`.app` bundles with no CLI entry point.

So the schematic had to go through the GUI, and doing the whole project there is
preferable anyway — a full project import links symbols to footprints, so the board
netlist is cross-referenced to the schematic. A CLI board-only import gives you nets
but no schematic association.

```
File -> Import Non-KiCad Project -> Import EAGLE Project...
```

Select the `.sch`; the matching `.brd` is picked up automatically from the same
basename and folder.

## What carried over cleanly

Eagle `.sch`/`.brd` files are **self-contained** — every footprint and symbol
definition is embedded in the file itself. All 16 packages used by the board were
present, with zero used-but-undefined packages, so no external library had to be
installed before converting.

Import statistics:

| | |
|---|---|
| Footprints | 24 |
| Tracks | 126 |
| Vias | 13 |
| Zones | 6 |
| Pads | 65 |
| Nets | 11 — GND, VCC, VDD, VBAT, SCL, SDA, INT, QSTART, N$1, N$2, N$4 |

Footprints are written **inline into the `.kicad_pcb`**, so the geometry is Adafruit's
original and the board needs no `fp-lib-table`. Symbols went into the project-local
`Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym`, registered via `sym-lib-table`.

## What did not carry over

- **Design rules and net classes.** Board Setup starts at KiCad defaults. Upstream used
  looser clearances than KiCad's defaults — see [`known-issues.md`](known-issues.md).
- **3D models.** The Eagle project contained none to begin with. See
  [`3d-models.md`](3d-models.md).
- **Eagle-specific layers** needed review: `tRestrict`/`bRestrict`, `tKeepout`,
  `tGlue`, `Measures`, `tDocu`. The GUI importer resolved these; a raw
  `kicad-cli pcb import` instead leaves items on an `UNDEFINED` layer, which makes the
  board unloadable until remapped (`Measures` dimensions belong on `Dwgs.User`,
  keepout graphics on a `User.n` layer).

## Import artifacts

- **`UNK_HOLE_0` / `UNK_HOLE_1`** at (2.54, 17.78) and (22.86, 17.78) are Eagle *free
  holes* (`<hole drill="2.54">`), not components. The importer wraps each bare NPTH in
  a dummy footprint, so they appear as unannotated parts.
- **`${REV}` text variable** on the `PCBFEAT-REV-040` rev marker (U$25) is unresolved
  unless defined. It is set to `040` in `Adafruit-MAX17048-STEMMA.kicad_pro`, taken
  from the Eagle package name.

## Verification commands

```sh
# board loads, and DRC
kicad-cli pcb drc --format json -o /tmp/drc.json Adafruit-MAX17048-STEMMA.kicad_pcb

# schematic ERC
kicad-cli sch erc --format json -o /tmp/erc.json Adafruit-MAX17048-STEMMA.kicad_sch

# netlist, for comparing against the Eagle source
kicad-cli sch export netlist --format kicadxml -o /tmp/net.xml Adafruit-MAX17048-STEMMA.kicad_sch

# visual check
kicad-cli pcb render --side top -o /tmp/top.png Adafruit-MAX17048-STEMMA.kicad_pcb

# symbol library parses
kicad-cli sym export svg -o /tmp/syms Adafruit-MAX17048-STEMMA-eagle-import.kicad_sym
```
