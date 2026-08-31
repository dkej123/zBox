# zBox build guide

This guide takes a new checkout from parts to the first NFC-triggered MP3. Read
[hardware status](hardware-status.md) first: the available PCB/Gerbers are legacy v2.0 and do
not contain every circuit in the current schematic.

## Prerequisites

- all required parts in the [BOM](bom.md), including a FAT32-capable microSD card and NFC tag
- enclosure parts from [`hardware/stl/`](../hardware/stl/) and the
  [printing notes](../hardware/printing_tips.md)
- soldering tools, multimeter, USB data cable, and a safe LiPo charging arrangement
- Git, [PlatformIO Core](https://docs.platformio.org/en/latest/core/installation/index.html),
  Docker Engine, and the Docker Compose plugin
- a Docker-capable host and a 2.4 GHz Wi-Fi network reachable by both host and ESP32

## 1. Check out the repository

```bash
git clone --recursive https://github.com/dkej123/zBox.git
cd zBox
git submodule status --recursive
```

For an existing non-recursive checkout, run `git submodule update --init --recursive`.

## 2. Print and assemble

Print each STL separately. Test-fit the lids, NFC slot, speaker holders, buttons, and board
before installing heat-set inserts. The STEP assembly in
[`hardware/cad/fusion360/step/`](../hardware/cad/fusion360/step/) is useful for fit changes.

Wire the four buttons on protoboard and assemble the electronics using the pin table and
load-switch circuits in [hardware wiring](hardware.md). The PN532 must be configured for SPI.
Mount the NFC antenna away from metal and keep the I2S/audio wiring short.

If using the v2.0 PCB/Gerbers, do not fit it as though it matched the current schematic. The
amplifier/NFC load switches, current button pull-ups, and NS4168 header require manual
off-board implementation. Use a continuity meter to verify every affected net listed in
[hardware status](hardware-status.md). Never connect the proposed future power latch as part
of the base build.

Before applying power:

- verify LiPo and electrolytic capacitor polarity;
- verify there is no short between power and ground;
- set the MT3608 output to 5 V before connecting the LEDs;
- confirm the USB-C footprint/receptacle match; and
- inspect all switched-power MOSFET orientation and module pinouts.

## 3. Prepare the microSD card

Format the card as FAT32 and insert it into the Lolin D32 Pro slot. A 4–16 GB card is ample.
Leave it empty: the first synchronization creates the required directories and metadata.

## 4. Build and flash firmware

From the repository root:

```bash
cd esp32
pio run -e lolin_d32_pro
pio run -e lolin_d32_pro -t upload
pio device monitor --baud 115200
```

The production build intentionally compiles most logging out. For diagnosis, flash
`lolin_d32_pro_debug` and monitor at 921600 baud. No compile-time server address is
required; the server connects to the HTTP service exposed by zBox in Sync Mode.

## 5. Start the server

On the Docker host, from the repository root:

```bash
docker compose config
docker compose up -d --build
docker compose ps
curl http://localhost:8000/api/health
```

Open `http://<server-host>:8000`. Compose persists the database/settings in `./data`, tracks
and system sounds in `./music`, and mounts `./web` into the container. Back up `data/` and
`music/`; both are intentionally ignored by Git.

You may put `ZBOX_IP=zbox.local` (or an IP address) in a local `.env`, but this is only the
fallback. The preferred configuration is stored through **Device** in the admin portal.

## 6. Provision Wi-Fi

Power on zBox and hold `A+B` for about two seconds. The device restarts into Sync Mode. If it
has no saved network, join the `zBox-Sync` access point and use its captive portal to select
the same 2.4 GHz network used by the server. The credentials are stored on the ESP32.

Find the assigned address in the router's client list or the serial log. In the server portal,
open **Device**, enter that IP or `zbox.local`, save it, and check that the device becomes
online. The portal value takes precedence over `ZBOX_IP`.

## 7. First playback

1. In **Songs**, upload an MP3.
2. In **Tags**, choose scan, present a card to the PN532, and assign the uploaded track.
3. In **Device**, start synchronization and wait for all files and mappings to complete.
4. Hold `A+B` to leave Sync Mode, then present the registered card in NFC Card Mode.

Audio should play locally from microSD through the NS4168. Bluetooth is optional: hold `A`
for about two seconds, then select a target from **Device** if needed.

## Troubleshooting

- **No `zBox-Sync` network:** enter Sync Mode with a two-second `A+B` hold; erase stored Wi-Fi
  only if reprovisioning is required, then retry close to the access point.
- **Portal shows offline:** confirm zBox is in Sync Mode, verify both hosts are on the same
  network, save the current device address under **Device**, and test `http://<zbox-ip>/diag/status`.
- **SD failure:** reformat FAT32, reseat the card, and verify `SD_CS=GPIO4` and the D32 Pro slot.
- **NFC failure:** set the PN532 to SPI and verify `SS=5`, `SCK=22`, `MISO=21`, and `MOSI=0`.
- **No local audio:** verify NS4168 `BCLK=32`, `LRCK=33`, `DIN=13`, speaker polarity, and the
  manually wired `NS_EN=15` load switch described in [hardware wiring](hardware.md).
- **Unexpected resets when audio starts:** disconnect power immediately and recheck the Q3
  soft-start parts, especially `R12=10k`, `C6=100nF`, and local decoupling.
- **Won't wake:** hold `D` for at least 400 ms; hold `C+D` to request night-light mode.
