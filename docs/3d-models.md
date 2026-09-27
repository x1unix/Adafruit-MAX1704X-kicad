# 3D models

**Status: 1 of 9 attached.** X3 (the MAX17048) has a model; the rest still have empty
3D slots.

This is deliberate, not an oversight. The Eagle source contains **zero 3D data** — no
`<package3d>` elements, no `packages3d` section, no STEP references. Eagle 9.6.2 kept
3D packages in Autodesk-managed cloud libraries, and Adafruit's published files carry
only `urn:adsk.eagle:` handles for two symbols. Nothing was lost in conversion; there
was nothing to inherit.

Models can be attached at any time without redoing the import.

## Attached

### X3 — MAX17048G+T10

`3dmodels/MAX17048G-T10--3DModel-STEP-510211.STEP` — AP214 STEP exported from
SolidWorks 2021, internally named `MAX17048G+T10.STEP`. Geometry verified at
2.000 x 2.000 x 0.800 mm, matching the TDFN-8 2 x 2 mm body.

Attached to the `TDFN8_2X2MM` footprint in `Adafruit-MAX17048-STEMMA.kicad_pcb` as:

```
(model "${KIPRJMOD}/3dmodels/MAX17048G-T10--3DModel-STEP-510211.STEP"
    (offset (xyz 0 0 0))
    (scale  (xyz 1 1 1))
    (rotate (xyz -90 0 0))
)
```

**The -90 degree X rotation is required.** The source model is authored **Y-up** — the
2 x 2 mm body lies in the X-Z plane with height running along +Y from 0 to 0.8 mm.
KiCad expects Z-up, with the part sitting on the board at Z = 0. Models exported from
SolidWorks and 3D ContentCentral commonly have this convention; check the axis extents
before assuming a model drops in at zero rotation.

Verified by rendering: body centred in the silkscreen outline, all 8 leads landing on
their pads, pin-1 dot over pad 1.

```sh
kicad-cli pcb render --side top --zoom 3.4 --pivot '0,-0.165,0' \
  --quality high -o /tmp/x3.png Adafruit-MAX17048-STEMMA.kicad_pcb
```

## Where to put them

Project-local, so they survive KiCad upgrades and travel with the repository:

```
3dmodels/
```

Reference them from each footprint's 3D tab as:

```
${KIPRJMOD}/3dmodels/<file>.step
```

## Still needs downloading — 1 part

Ships with a KiCad **footprint** but no STEP file. Verified absent across all 7245
stock models in KiCad 10.0.5.

### X1, X2 — JST S2B-PH-SM4-TB

PH series, 2 mm pitch, 2-pin, side-entry SMT with mounting tabs.
KiCad ships `JST_PH_S2B-PH-SM4-TB_1x02-1MP_P2.00mm_Horizontal.kicad_mod` but not the
`.step`. Only the through-hole variants have models (`B2B-PH-K`, `S2B-PH-K`).

- [SnapMagic Search](https://www.snapeda.com/parts/S2B-PH-SM4-TB/JST%20Sales%20America%20Inc./view-part/)
- [Ultra Librarian](https://app.ultralibrarian.com/details/c0ea14ac-1ee6-11e9-ab3a-0a3560a4cccc/JST/S2B-PH-SM4-TB-LF-SN-)
- [Digi-Key](https://www.digikey.com/en/models/926655)
- [3D ContentCentral](https://www.3dcontentcentral.com/download-model.aspx?catalogid=171&id=817113)

## Available in stock — 7 parts

Paths relative to `${KICAD10_3DMODEL_DIR}`.

| Ref | Model | Fit |
|---|---|---|
| C1, C2 | `Capacitor_SMD.3dshapes/C_0603_1608Metric.step` | exact |
| R2 | `Resistor_SMD.3dshapes/R_0603_1608Metric.step` | exact |
| D1 | `LED_SMD.3dshapes/LED_0603_1608Metric.step` | exact |
| JP1 | `Connector_PinHeader_2.54mm.3dshapes/PinHeader_1x06_P2.54mm_Vertical.step` | exact |
| JP2 | `Connector_PinHeader_2.54mm.3dshapes/PinHeader_1x02_P2.54mm_Vertical.step` | exact |
| CONN3, CONN4 | `Connector_JST.3dshapes/JST_SH_SM04B-SRSS-TB_1x04-1MP_P1.00mm_Horizontal.step` | pad geometry identical to KiCad-official, but the footprint origin differs by ~0.5 mm in Y — nudge in the 3D viewer |
| R3 | `Resistor_SMD.3dshapes/R_Array_Convex_4x0603.step` | approximate — needs a **90 degree Z rotation** (Adafruit's array runs along X, the model along Y), and Adafruit's outer pads are 0.9 mm vs the model's 0.8 mm |

For an exact R3, download a 4x0603 array model (Panasonic EXB-38V, Yageo YC164) instead.

## Needs no model

Mechanical or artwork only: `SJ1`, `SJ2` (solder jumpers, copper only), `FID3`, `FID4`
(fiducials), `U$19`, `U$21` (bare plated mounting holes), `U$22` (Adafruit logo),
`U$25` (rev marker), `U$30`, `U$31` (STEMMA QT silkscreen), `UNK_HOLE_0/1` (free holes).

## Note on sourcing

There is **no official Adafruit KiCad library** — Adafruit publishes Eagle libraries
only. Don't spend time looking for one.
