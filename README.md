# zBox

![zBox audio box](assets/photos/zbox-hero.jpg)

zBox is an offline-first ESP32 NFC audio box: tap a card to play an MP3 from microSD, use
four physical buttons for playback and modes, and manage music and mappings from a small
FastAPI web portal. Wi-Fi is needed only for provisioning, synchronization, and maintenance.

> [!CAUTION]
> The manufactured PCB and Gerbers are legacy **v2.0 (April 2026)**. They predate the current
> schematic's July load-switch changes and are not a turnkey version of that schematic. A
> build from these Gerbers needs manual off-board wiring; read
> [Hardware status](docs/hardware-status.md) before ordering boards.

## What you need

- [ ] Lolin D32 Pro (ESP32), PN532 SPI NFC module, NS4168 I2S amplifier, and 4–8 Ω speaker
- [ ] MT3608 boost module, 12-pixel WS2812B strip/panel, and four momentary buttons
- [ ] 3.7 V LiPo, USB-C power/charging connection, hookup wire, PCB/protoboard, and solder
- [ ] Printed enclosure parts, M3 heat-set inserts, M3/M4 screws, and suitable filament
- [ ] FAT32 microSD card, NFC cards/tags, and optional Bluetooth headphones or speaker
- [ ] USB cable, soldering tools, multimeter, and a computer with PlatformIO
- [ ] Docker-capable host on the same network (a Raspberry Pi is a good fit)

See the [complete BOM](docs/bom.md) for values, footprints, quantities, and revision notes.

## Quick start

### 1. Clone everything

```bash
git clone --recursive https://github.com/dkej123/zBox.git
cd zBox
```

If you already cloned without submodules, run `git submodule update --init --recursive`.

### 2. Assemble the hardware

Print the [STL enclosure parts](hardware/stl/) and follow the
[printing notes](hardware/printing_tips.md). Wire the modules using the
[hardware reference](docs/hardware.md), accounting for the
[v2.0 PCB limitations](docs/hardware-status.md). Check polarity and shorts before power-up.

### 3. Prepare storage and flash

Format a microSD card as FAT32, insert it into the Lolin D32 Pro, connect USB, then run:

```bash
cd esp32
pio run -e lolin_d32_pro -t upload
cd ..
```

There is no compile-time server address. The device exposes its own sync service over Wi-Fi.

### 4. Start the server

```bash
docker compose up -d --build
curl http://localhost:8000/api/health
```

Open `http://<server-host>:8000`. The optional `ZBOX_IP` environment value is only a fallback;
the preferred setup is saving the device address in the portal.

### 5. Provision Wi-Fi

Hold buttons `A+B` for about two seconds to enter Sync Mode. On first use, join the
`zBox-Sync` Wi-Fi network from a phone or laptop and complete its captive portal with the
credentials for the same network as the server. Note the device IP shown by your router or
serial monitor.

### 6. Add the device and content

In the admin portal:

1. Open **Device**, enter the zBox IP address or `zbox.local`, and save it.
2. Open **Songs** and upload MP3 files.
3. Open **Tags**, scan or enter an NFC UID, name it, and assign a track.
4. Return to **Device** and start synchronization.

When sync finishes, hold `A+B` to return to normal operation. Present the registered card;
playback should start locally from microSD through the NS4168 speaker.

## Controls

| Input | Action |
| --- | --- |
| `A` or `B`, click | Play/pause |
| `A`, double-click | Previous track |
| `B`, double-click | Next track |
| `A`, hold ~2 s | Toggle optional Bluetooth output |
| `B`, hold ~2 s | Toggle NFC Card/Music playback mode |
| `C` / `D`, click | Volume down / up |
| `C`, hold ~2 s | Show battery level |
| `D`, hold ~1 s | Enter deep sleep |
| `A+B`, hold ~2 s | Enter or leave Sync Mode |
| Sleeping: hold `D` ≥400 ms | Wake normally |
| Sleeping: hold `C+D` ≥400 ms | Wake into night-light mode |

The device currently powers down with ESP32 deep sleep. The proposed zero-standby-current
soft latch and mechanical travel switch are design work only; they have not been built or
bench-validated. See [Hardware status](docs/hardware-status.md).

## Repository map

| Path | Contents |
| --- | --- |
| [`esp32/`](esp32/) | PlatformIO firmware and native tests |
| [`server/`](server/) / [`web/`](web/) | FastAPI server and browser admin |
| [`hardware/pcb/kicad/`](hardware/pcb/kicad/) | Current schematic plus legacy v2.0 PCB layout |
| [`hardware/pcb/gerbers/`](hardware/pcb/gerbers/) | Legacy v2.0 manufacturing outputs |
| [`hardware/stl/`](hardware/stl/) | Printable enclosure parts |
| [`hardware/cad/fusion360/step/`](hardware/cad/fusion360/step/) | Editable mechanical STEP export |

## Documentation

- [Build guide](docs/build-guide.md)
- [Bill of materials](docs/bom.md)
- [Hardware status and revision gap](docs/hardware-status.md)
- [Hardware wiring](docs/hardware.md)
- [Firmware](docs/esp32-firmware.md)
- [Server](docs/server.md)

`karty.pdf` and [`cards_images/`](cards_images/) are optional printable card assets. The project
owner holds the rights needed to publish the owner-created card graphics and other media here.

## License

The [MIT license](LICENSE) covers the repository's code, hardware design files, documentation,
and owner-created media unless a file states otherwise. Third-party submodules and dependencies
retain their own licenses. This is a DIY design: independently review electrical and battery
safety before manufacturing or giving a completed device to a child.
