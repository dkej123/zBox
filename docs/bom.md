# zBox bill of materials

This BOM follows the **current KiCad schematic**. It is not a claim that the legacy v2.0
PCB/Gerbers contain those parts; see [hardware status](hardware-status.md). `PAD_*` references
are PCB wire pads, not separate purchased connectors. Verify module pinouts and footprints
against the exact parts you buy.

## Required modules and electrical parts

| Qty | Item | Notes |
| ---: | --- | --- |
| 1 | Lolin D32 Pro ESP32 board | Includes microSD slot and LiPo support; schematic `U1` |
| 1 | PN532 NFC module | Must support SPI; wired at `J3` |
| 1 | NS4168 I2S amplifier module | Wired at `J13` |
| 1 | 4–8 Ω speaker | Must fit the printed holder and amplifier rating |
| 1 | MT3608 boost module | Schematic `U2`; adjust to 5 V before LEDs are connected |
| 1 | 12-pixel WS2812B strip/panel | Wired at `J11` |
| 1 | GCT USB4085 USB-C receptacle | `J1`, exact `Connector_USB:USB_C_Receptacle_GCT_USB4085` footprint |
| 1 | 3.7 V LiPo, about 4000 mAh | Current enclosure fits about 8 × 50 × 80 mm; wired at `J2` |
| 4 | 12 × 12 × 7.3 mm momentary buttons | Four-pin TACT switches with caps; wired at `J7` |
| 2 | AO3401A P-MOSFET | `Q2`, `Q3`; `Package_TO_SOT_SMD:SOT-23` |
| 1 | AO3400A N-MOSFET | `Q4`; `Package_TO_SOT_SMD:SOT-23` |

## Exact current-schematic passives

| References | Qty | Value | KiCad footprint |
| --- | ---: | --- | --- |
| `R1`, `R2` | 2 | 5.1 kΩ | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` |
| `R8`, `R11`, `R13` | 3 | 100 kΩ | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` |
| `R9`, `R10`, `R12` | 3 | 10 kΩ | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` |
| `R14` | 1 | 100 Ω | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` |
| `C1` | 1 | 470 µF polarized | `Capacitor_THT:CP_Radial_D8.0mm_P3.50mm` |
| `C2`, `C4` | 2 | 10 µF polarized | `Capacitor_THT:CP_Radial_D5.0mm_P2.50mm` |
| `C3`, `C5`, `C6` | 3 | 100 nF ceramic | `Capacitor_THT:C_Disc_D5.0mm_W2.5mm_P5.00mm` |

The base electronics total is 9 resistors, 6 capacitors, and 3 MOSFETs, plus the modules,
receptacle, buttons, battery, LEDs, speaker, and board/wiring. Buy spares for small parts.

## PCB, wiring, and consumables

- one PCB made from the legacy v2.0 Gerbers **or** protoboard/custom wiring reviewed against
  the current schematic; the v2.0 board needs the documented manual additions
- insulated hookup wire, pin headers as required by the modules, solder, flux, heat-shrink,
  cable ties, double-sided tape or hot glue, and strain relief
- a safe USB power supply and USB data cable; use the D32 Pro's supported LiPo charging path

## Mechanical parts

- all seven STL designs in [`hardware/stl/`](../hardware/stl/): shell, wall, front and rear
  lids, NFC card slot, and left/right speaker holders
- enough PETG/PLA or another suitable filament for one enclosure
- M3 heat-set inserts and M3 screws for the rear cover
- M4 screws for the side wall and ESP32 mounting, adjusted to suit the printed parts
- button protoboard and colored caps

## Runtime requirements

- one FAT32 microSD card (4–16 GB is ample)
- at least one compatible NFC card/tag; printable owner-created designs are in
  [`cards_images/`](../cards_images/) and [`karty.pdf`](../karty.pdf)
- a Docker-capable host on the same network, such as a Raspberry Pi
- optional Bluetooth headphones or speaker for A2DP output

## Future/unverified power-latch upgrade — excluded from the base total

These parts describe a **planned circuit only**. No repository PCB contains it, firmware still
uses deep sleep, values are provisional, and the design requires schematic review and bench
validation before use.

| Qty | Proposed part | Intended role |
| ---: | --- | --- |
| 1 | SPST mechanical switch rated for battery current | Hard travel disconnect in series with `BAT+` |
| 1 | AO3415A or suitable low-RDS(on) P-MOSFET | Main high-side latch switch |
| 2 | Small NPN transistors or suitable N-MOSFETs | Self-hold and `GPIO2` kill stages |
| 3 | ~100 kΩ resistors | Gate pull-up, feedback, and base/gate pull-down |
| 2 | ~10 kΩ resistors | Gate/base current limiting |
| 1 | Momentary SET path reusing `BTN_D` with isolation | Power-on request without exposing GPIO26 to battery voltage |

The target behavior is a self-holding latch that survives MCU resets and turns off only after
an active-HIGH kill pulse on `GPIO2`. Do not count these parts, connect them to v2.0, or market
the feature as implemented until reset survival, shutdown, leakage, low-battery behavior, and
switch current have been measured on hardware.
