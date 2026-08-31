# zBox hardware wiring

This is the wiring reference for the **current schematic and firmware**. It is newer than the
manufactured v2.0 PCB/Gerbers. Read [hardware status](hardware-status.md) before building and
use the [BOM](bom.md) for exact values and footprints.

## Main blocks

```text
NFC card -> PN532 -> ESP32 Lolin D32 Pro -> microSD
                         |       |
                         |       +-> WS2812B LEDs through MT3608
                         +-> I2S -> NS4168 -> 4–8 ohm speaker
                         +-> optional Bluetooth A2DP output
```

Playback is offline. Wi-Fi is active in Sync Mode for provisioning, content transfer,
diagnostics, and maintenance.

## ESP32 pin assignments

| GPIO | Function | Notes |
| ---: | --- | --- |
| `0` | PN532 MOSI | Software SPI |
| `4` | microSD CS | D32 Pro onboard slot |
| `5` | PN532 SS | Software SPI |
| `12` | `NFC_EN` | Q4 low-side switch; active HIGH |
| `13` | NS4168 DIN | I2S data |
| `14` | WS2812B DIN | 12 LEDs by default |
| `15` | `NS_EN` | Q3 high-side switch; active LOW/Hi-Z off |
| `21` | PN532 MISO | Software SPI |
| `22` | PN532 SCK | Software SPI |
| `25` | Button C | Internal pull-up, volume down/battery preview |
| `26` | Button D | Internal pull-up, volume up/sleep/deep-sleep wake |
| `27` | `LED_EN` | Q2 high-side switch; active LOW |
| `32` | NS4168 BCLK | I2S bit clock |
| `33` | NS4168 LRCK/WS | I2S word select |
| `35` | Battery ADC | D32 Pro onboard divider |
| `36` | Button A | Input-only; external 10 kΩ pull-up required |
| `39` | Button B | Input-only; external 10 kΩ pull-up required |

All buttons are active LOW. Current behavior is documented in the root [README](../README.md)
and verified by the native button-decoder tests.

## D32 Pro header positions

With USB at the bottom, count each 16-pin header from top to bottom as `L1..L16` and
`R1..R16`.

| Position | Connection | Position | Connection |
| --- | --- | --- | --- |
| `L1` | `+3V3` | `R1` | `GND` |
| `L3` | `GPIO36 / BTN_A` | `R3` | `GPIO22 / PN532_SCK` |
| `L4` | `GPIO39 / BTN_B` | `R6` | `GPIO21 / PN532_MISO` |
| `L6` | `GPIO32 / I2S_BCLK` | `R9` | `GPIO5 / PN532_SS` |
| `L7` | `GPIO33 / I2S_LRCK` | `R12` | `GPIO4 / SD_CS` |
| `L8` | `GPIO25 / BTN_C` | `R13` | `GPIO0 / PN532_MOSI` |
| `L9` | `GPIO26 / BTN_D` | `R14` | `GPIO2` (unused today) |
| `L10` | `GPIO27 / LED_EN` | `R15` | `GPIO15 / NS_EN` |
| `L11` | `GPIO14 / LED data` | `R16` | `GND` |
| `L13` | `GPIO13 / I2S_DOUT` |  |  |
| `L15` | `USB / 5 V` |  |  |
| `L16` | `BAT / VBAT` |  |  |

## Audio and LEDs

Connect NS4168 `BCLK`, `LRCK/WS`, and `DIN` to GPIO32, GPIO33, and GPIO13 respectively. Its
VDD comes from switched `+3V3_NS`, not directly from `+3V3`. Fit the bulk and ceramic
decoupling at the module and keep I2S/speaker wiring short.

The MT3608 takes the battery rail and supplies 5 V to the LED strip. Adjust it to 5 V with
the LEDs disconnected. Q2 gates the boost input with `LED_EN=GPIO27`; `R8=100k` holds the
P-MOSFET off. WS2812B data is GPIO14 and the configured count is 12.

## Current-schematic load switches

These circuits exist in the July schematic and firmware but **not** on the legacy v2.0 PCB.
They must be wired off-board when v2.0 is used.

### NS4168 high-side switch

| Part | Connection |
| --- | --- |
| `Q3` AO3401A | source -> `+3V3`; drain -> `+3V3_NS`; gate -> `NS_EN_G` |
| `R11` 100 kΩ | `NS_EN_G` -> `+3V3` (default off) |
| `R12` 10 kΩ | GPIO15 -> `NS_EN_G` |
| `C6` 100 nF | `NS_EN_G` -> `+3V3` (soft-start) |
| `C2` 10 µF, `C3` 100 nF | `+3V3_NS` -> GND, close to NS4168 |
| `J13` | NS4168 pad: VDD=`+3V3_NS`, GND, BCLK=32, LRCK=33, DIN=13 |

GPIO15 is active LOW. Firmware uses Hi-Z for off so `R11` can close Q3 without a boot glitch.
The `R12+C6` soft-start is important: switching the amplifier abruptly can collapse the ESP32
3.3 V rail and cause a brownout loop.

### PN532 low-side switch

| Part | Connection |
| --- | --- |
| `Q4` AO3400A | drain -> `NFC_GND_SW`; source -> GND; gate -> `NFC_EN_G` |
| `R13` 100 kΩ | `NFC_EN_G` -> GND (default off and safe boot strap) |
| `R14` 100 Ω | GPIO12 -> `NFC_EN_G` |
| `C4` 10 µF, `C5` 100 nF | `+3V3` -> `NFC_GND_SW`, close to PN532 |
| `J3` | PN532 VDD=`+3V3`; GND=`NFC_GND_SW`; SPI pins as listed above |

GPIO12 is active HIGH. Before switching Q4 off, firmware makes the PN532 SPI lines high
impedance so the floating module ground cannot back-power the ESP32 through protection diodes.

## Power and safety notes

- `C1` is a 470 µF polarized capacitor in the current schematic; observe polarity.
- `R1` and `R2` are the two 5.1 kΩ USB-C CC resistors.
- `R9` and `R10` are the external 10 kΩ pull-ups required by input-only GPIO36/39.
- The checked-in PCB has older JBL wiring and different Q2 labeling/value; use the current
  schematic as the wiring authority but never assume its additions are routed in v2.0.
- The project uses deep sleep, not a physical power disconnect. The proposed latch is
  [future and unverified](hardware-status.md#planned-power-latch-improvement).
- Review LiPo charging, wire gauge, insulation, polarity, and enclosure fire/impact risk for
  your parts. Do not leave a first assembly charging unattended.
