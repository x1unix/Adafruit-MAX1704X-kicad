# MAX17048 — datasheet reference

Machine-readable condensation of [`max17048-max17049.pdf`](max17048-max17049.pdf)
(Maxim/Analog Devices doc `19-6171 Rev 7, 11/16`), scoped to what matters for this board.

**The PDF is authoritative.** This file exists so agents don't have to parse 19 pages of
images. Anything safety- or tolerance-critical should be confirmed against the PDF.

## Identity

| | |
|---|---|
| Part on this board | **MAX17048G+T10** |
| Function | 3µA 1-cell Li+ fuel gauge with ModelGauge |
| Package | 8 TDFN-EP, 2 x 2 mm, 0.5 mm pitch, 0.8 mm height |
| Package code | `T822+3` |
| Package outline | `21-0168` |
| Land pattern | `90-0065` |
| Temp range | -40°C to +85°C |
| Suffix decode | `G` = TDFN, `+` = lead-free/RoHS, `T10` = tape and reel |

`MAX17048` is the 1-cell part; `MAX17049` is the 2-cell variant sharing the same
datasheet, pinout and registers. **This board uses the 1-cell MAX17048.**

> Note: Adafruit's Eagle package description cites outline `21-0268`. The datasheet's
> Package Information table gives `21-0168` for 8 TDFN-EP. Use `21-0168`.

## Pinout — TDFN-8

```
[ 1 ] CTG      SDA [ 8 ]
[ 2 ] CELL     SCL [ 7 ]
[ 3 ] VDD    QSTRT [ 6 ]
[ 4 ] GND     ALRT [ 5 ]
              [ EP ]
```

| Pin | Name | Function |
|---|---|---|
| 1 | CTG | Connect to ground. |
| 2 | CELL | Connect to positive battery terminal. **MAX17048: not internally connected.** MAX17049: voltage sense input. |
| 3 | VDD | Power-supply input, bypass 0.1µF to GND. **MAX17048: this is the voltage sense input** — connect to positive battery terminal. |
| 4 | GND | Ground, connect to negative battery terminal. |
| 5 | ALRT | Open-drain, active-low alert output. Optionally to host interrupt. |
| 6 | QSTRT | Quick-start input, hardware reset. **Connect to GND if unused.** |
| 7 | SCL | I2C clock input. Internal pulldown for disconnect sensing. |
| 8 | SDA | Open-drain I2C data I/O. Internal pulldown for disconnect sensing. |
| — | EP | Exposed pad (TDFN only). Connect to GND. |

Note that on the **MAX17048**, `VDD` — not `CELL` — is what gets measured.

### As wired on this board

| Pin | Name | Net |
|---|---|---|
| 1 | CTG | `GND` |
| 2 | CELL | `VBAT` |
| 3 | VDD | `VDD` (fed via solder jumper SJ2) |
| 4 | GND | `GND` |
| 5 | ALRT | `INT` — broken out to JP1.5 |
| 6 | QSTRT | `QSTART` — broken out to JP1.6, **not tied to GND** |
| 7 | SCL | `SCL` |
| 8 | SDA | `SDA` |
| EP | — | `GND` |

Closest datasheet configuration (Table 3): *1S host-side location, low-cell interrupt*.

> The symbol in this project had `SCL`/`SDA` labels transposed upstream. Corrected —
> see [`max17048-symbol.md`](max17048-symbol.md) before touching anything I2C.

## I2C

| | |
|---|---|
| 7-bit address | **`0x36`** — fixed, not configurable |
| 8-bit address | `0x6C` write / `0x6D` read |
| Max clock | 400 kHz (works at any speed from 0) |
| Register width | **All registers are 16-bit.** 8-bit writes have no effect. |
| Byte order | MSB first; MSB of a 16-bit register lives at the even address |

Use `0x36` with Linux/Arduino/CircuitPython APIs, which take 7-bit addresses. The
datasheet quotes the 8-bit form, which is a common source of confusion.

Address auto-increments across a multi-byte transaction. Writes beyond `0x4F` are
ignored; reads beyond `0xFF` return `0xFF`.

## Registers

| Addr | Name | R/W | LSb | Default | Purpose |
|---|---|---|---|---|---|
| `0x02` | VCELL | R | 78.125 µV/cell | — | ADC measurement of VCELL |
| `0x04` | SOC | R | 1%/256 | — | Battery state of charge |
| `0x06` | MODE | W | — | `0x0000` | Quick-start, hibernate status, sleep enable |
| `0x08` | VERSION | R | — | `0x001_` | IC production version |
| `0x0A` | HIBRT | R/W | — | `0x8030` | Hibernate entry/exit thresholds |
| `0x0C` | CONFIG | R/W | — | `0x971C` | RCOMP, sleep, alert config |
| `0x14` | VALRT | R/W | 20 mV | `0x00FF` | Voltage alert min/max |
| `0x16` | CRATE | R | 0.208 %/hr | — | Approximate charge/discharge rate |
| `0x18` | VRESET/ID | R/W | 40 mV | `0x96__` | Reset threshold + factory ID |
| `0x1A` | STATUS | R/W | — | `0x01__` | Alert cause flags |
| `0x40`–`0x7F` | TABLE | W | — | — | Custom battery model |
| `0xFE` | CMD | R/W | — | `0xFFFF` | POR command |

### Bit layouts

**MODE (`0x06`)** — MSB: `X | QuickStart | EnSleep | HibStat | X X X X`, LSB all don't-care.

**CONFIG (`0x0C`)** — MSB `RCOMP[7:0]` (default `0x97`); LSB `SLEEP | ALSC | ALRT | ATHD[4:0]`.
- `RCOMP` — temperature compensation. Host should update at least once per minute:
  ```
  if (T > 20) RCOMP = 0x97 + (T - 20) * (-0.5);
  else        RCOMP = 0x97 + (T - 20) * (-5.0);
  ```
- `ATHD` — empty alert threshold, `(32 - ATHD)%`, range 1–32%. POR `0x1C` = 4%.
- `ALSC` — enable alert on 1% SOC change.
- `ALRT` — alert status; **write 0 to clear**, which also deasserts the ALRT pin.

**STATUS (`0x1A`)** — MSB `X | EnVR | SC | HD | VR | VL | VH | RI`.
`RI` reset indicator, `VH`/`VL` voltage high/low, `VR` voltage reset, `HD` SOC low,
`SC` 1% SOC change. Clear the bit after servicing.

**HIBRT (`0x0A`)** — MSB `HibThr` (0.208 %/hr), LSB `ActThr` (1.25 mV).
`0x0000` disables hibernate, `0xFFFF` forces it always on.

**VALRT (`0x14`)** — MSB `VALRT.MIN`, LSB `VALRT.MAX`, both 20 mV/LSb.

**VRESET/ID (`0x18`)** — MSB `VRESET[7:1]` (40 mV/LSb) + `Dis` bit 0; LSB is the
read-only factory `ID`. For removable batteries set at least 300 mV below empty
voltage; for captive batteries set to 2.5 V.

**CMD (`0xFE`)** — write `0x5400` for a full POR.

**TABLE unlock** — write `0x57` to `0x3F` and `0x4A` to `0x3E`. ModelGauge registers do
not update while unlocked, so relock promptly with `0x00` to both.

## Electrical

| Parameter | Min | Typ | Max | Unit |
|---|---|---|---|---|
| VDD supply voltage | 2.5 | | 4.5 | V |
| Supply current, sleep (TA ≤ +50°C) | | 0.5 | 2 | µA |
| Supply current, hibernate (reset comp disabled) | | 3 | 5 | µA |
| Supply current, hibernate (reset comp enabled) | | 4 | | µA |
| Supply current, active | | 23 | 40 | µA |
| Voltage error (VCELL 3.6 V, +25°C) | -7.5 | | +7.5 | mV/cell |
| Voltage-measurement resolution | | 1.25 | | mV/cell |
| Voltage-measurement range (MAX17048, VDD pin) | 2.5 | | 5 | V |
| VRESET configurable range (40 mV steps) | 2.28 | | 3.48 | V |
| VRESET trimmed at 3 V | 2.85 | 3.0 | 3.15 | V |
| ADC sample period, active | | 250 | | ms |
| ADC sample period, hibernate | | 45 | | s |
| SCL clock frequency | 0 | | 400 | kHz |
| VIH (SDA, SCL, QSTRT) | 1.4 | | | V |
| VIL (SDA, SCL, QSTRT) | | | 0.5 | V |
| VOL (SDA, ALRT) at IOL = 4 mA | | | 0.4 | V |
| Bus low-detection timeout | 1.75 | | 2.5 | s |

### Absolute maximum ratings

| | |
|---|---|
| CELL to GND | -0.3 V to +12 V |
| All other pins to GND | -0.3 V to +6 V |
| Continuous sink current, SDA / ALRT | 20 mA |
| Operating temperature | -40°C to +85°C |
| Storage temperature | -55°C to +125°C |
| Reflow soldering | +260°C |
| Lead temperature (TDFN, soldering 10s) | +300°C |

## Behaviour worth knowing

- **ModelGauge needs no current-sense resistor** and does not accumulate error the way
  coulomb counters do. No periodic correction events required.
- **Battery insertion:** OCV is ready 17 ms after insertion, SOC 175 ms after that.
  First SOC update is roughly 1 s after POR.
- **Quick-start should usually not be used.** The IC handles insertion transparently.
  Only use it when the power-up sequence is noisy enough to spoil the initial SOC
  estimate, and only when VCELL is fully relaxed. POR includes a quick-start.
- **Hibernate is automatic by default** and is the recommended configuration. The IC
  drops to a 45 s ADC period below C/4 loading, cutting quiescent current under 5 µA
  without hurting accuracy.
- **Sleep mode** (< 1 µA) halts everything — the IC cannot detect self-discharge, so
  wake it before charging or discharging. Enter with `MODE.EnSleep = 1` plus either
  `CONFIG.SLEEP = 1` or holding SDA and SCL low past `tSLEEP`. Prefer hibernate if
  4 µA is tolerable.
- **Alerts** are level-driven: ALRT stays low until software writes `CONFIG.ALRT = 0`.
  Five sources — low SOC, 1% SOC change, reset, overvoltage, undervoltage.
- **Instantaneous voltage does not map to SOC.** One VCELL value can correspond to many
  SOC values (3.81 V occurs at 2%, 50% and 72% in the datasheet example). Don't
  reimplement SOC from voltage.

## Pull-ups

The IC provides none. The system must supply pull-ups on SDA, SCL and ALRT (if used).
On this board that is the `RESPACK_4X0603` array (R3, 10K).
