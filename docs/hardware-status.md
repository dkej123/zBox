# Hardware status and revision gap

The repository intentionally keeps both the manufactured files and the newer design work.
They are not the same revision and must not be presented as interchangeable.

## Status matrix

| Artifact or feature | Status | Builder impact |
| --- | --- | --- |
| PCB layout and Gerbers | **Legacy v2.0, April 2026; manufactured** | Available for reference/reproduction, but older than the current schematic and not turnkey |
| Current KiCad schematic | **Newer design, July 2026 additions** | Documents the intended modules, button pull-ups, and NFC/amplifier load switching |
| Current firmware | **Implements July load-switch control** | Expects `NS_EN=GPIO15`, `NFC_EN=GPIO12`, current button mapping, and deep-sleep sequencing |
| v2.0 manual build | **Possible with off-board wiring** | Builder must add and verify the missing current-schematic circuits manually |
| Soft power latch / travel switch | **Future, unimplemented, unverified** | Excluded from the base build and BOM total; do not assume zero-current off |

## What v2.0 does not contain

The routed [`zbox.kicad_pcb`](../hardware/pcb/kicad/zbox.kicad_pcb) and files in
[`hardware/pcb/gerbers/`](../hardware/pcb/gerbers/) do **not** contain the current schematic's:

- `Q3`, `Q4`;
- `R9` through `R14`;
- `C2` through `C6`;
- `J13`; or
- `NS_EN`, `+3V3_NS`, `NFC_EN`, and `NFC_GND_SW` nets.

The v2.0 PCB instead retains older JBL-era references such as `Q1`, `R3`, `R6`, `R7`, `J10`,
and `J12`. Those are not the current firmware's default NS4168 audio design.

Practically, manufacturing the checked-in Gerbers produces legacy v2.0—not a board matching
the current schematic. The two load switches, button pull-ups, NS4168 connection, decoupling,
and affected nets must be wired off-board and continuity-checked against
[hardware wiring](hardware.md). A future PCB revision should synchronize schematic, layout,
Gerbers, BOM, firmware pins, and a physically tested assembly before it is called turnkey.

## Current power-off behavior

Today, all shutdown paths end in ESP32 deep sleep. Firmware gates the LED supply and, when
the July off-board circuits are present, the NS4168 and PN532 supplies. The D32 Pro itself is
still powered, so this is not a physical battery disconnect and not a zero-current-off state.

## Planned power-latch improvement

The proposed upgrade combines:

- a mechanical SPST kill switch in series with `BAT+` for travel/storage;
- a self-holding soft latch, set by the existing `BTN_D`, that remains on across software,
  watchdog, panic, and brownout resets;
- an active-HIGH, roughly 100–120 ms kill pulse from `GPIO2` after graceful shutdown; and
- firmware changes replacing deep-sleep wake/off tails while preserving the current LED
  hold-to-confirm behavior and `C+D` night-light selection.

This is design intent only. It is absent from the schematic, PCB, Gerbers, and firmware.
Before adoption it needs electrical review plus bench tests for latch SET/HOLD/KILL behavior,
GPIO2 boot strapping, reset survival, button isolation from battery voltage, off-state leakage,
switch/MOSFET current and thermal margin, idle timeout, low-battery cutoff, and every shutdown
path. Proposed parts are isolated in the optional section of the [BOM](bom.md).

## Release policy

The existing files remain available as legacy v2.0. Do not publish a v1.0 hardware release
until a synchronized successor PCB has been manufactured and physically validated.
