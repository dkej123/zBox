# zBox ESP32 Firmware

Firmware for the ESP32 Lolin D32 Pro using PlatformIO and the mixed `arduino, espidf` framework setup.

## Quick start

```bash
pio run
pio run -t upload
pio device monitor
```

## Before flashing

- review `ENABLE_LEDS`
- verify the board and partition settings match your hardware
- read the [hardware revision status](../docs/hardware-status.md); the legacy v2.0 PCB needs
  manual wiring for the current firmware's load switches

There is no compile-time server address. Enter Sync Mode with a two-second `A+B` hold,
provision Wi-Fi through the `zBox-Sync` captive portal, and save the device address in the
server's **Device** page. `ZBOX_IP` is only a server-side fallback.

The main project constants live in [`src/zbox_config.h`](src/zbox_config.h).

## Documentation

- [Firmware architecture](../docs/esp32-firmware.md)
- [Hardware notes](../docs/hardware.md)
- [Build guide](../docs/build-guide.md)
- [Server API and deployment](../docs/server.md)
