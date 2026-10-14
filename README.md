# TEMPEST 2 — Firmware Flasher

Public web flasher for **TEMPEST 2** on the **M5Stack Core2 for AWS v1.3**.
Flashes the device from Chrome/Edge over USB (Web Serial) via
[ESP Web Tools](https://esphome.github.io/esp-web-tools/).

**Live site:** https://ozinfl.github.io/tempest2-flasher/

> This repo is intentionally **public** (so the flasher is reachable) and contains
> **only** the flasher page and release firmware binaries. Firmware **source** lives
> in the private `OzInFl/TEMPESTV2` repo.

## What's here

| File | Purpose |
|---|---|
| `index.html` | Flasher page: install button, flashing steps, SD card setup/troubleshooting |
| `manifest-tempest2.json` | ESP Web Tools manifest (ESP32, one merged image at `0x0`) |
| `firmware/tempest2-core2-v0.92.factory.bin` | Merged image: bootloader `0x1000`, partitions `0x8000`, boot_app0 `0xe000`, app `0x10000` |
| `firmware/firmware.bin` | App-only image for the on-device SD card update (`/firmware.bin` on the card) |
| `.nojekyll` | Serve `firmware/` and `.json` verbatim |

## Current release

| Version | Built from | factory.bin SHA-256 |
|---|---|---|
| 0.92 | `TEMPESTV2@398ace7` | `b18009bdf3f1768ae61ec3a4b78a9cdb2d87a55c3c041c6e5524903aad4d98cb` |

`firmware.bin` SHA-256: `192657cac6fa64f6f253ea677e3fc2e2e1721e93fc2618a034ef02ef99ba6e0d`

## Publishing a new release

1. In `TEMPESTV2`, bump `FIRMWARE_VERSION`, then `pio run`.
2. Build the merged image (board settings: DIO, 40 MHz, 16 MB):
   ```
   esptool.py --chip esp32 merge_bin -o firmware/tempest2-core2-vX.YZ.factory.bin \
     --flash_mode dio --flash_freq 40m --flash_size 16MB \
     0x1000  <framework>/tools/sdk/esp32/bin/bootloader_dio_40m.bin \
     0x8000  .pio/build/m5stack-core2/partitions.bin \
     0xe000  <framework>/tools/partitions/boot_app0.bin \
     0x10000 .pio/build/m5stack-core2/firmware.bin
   ```
   `<framework>` = `~/.platformio/packages/framework-arduinoespressif32@3.20003.220626`
   (Arduino core 2.0.3, platform `espressif32@5.0.0`).
3. Copy `.pio/build/m5stack-core2/firmware.bin` to `firmware/firmware.bin`.
4. Update `manifest-tempest2.json` (version + path), the version shown in `index.html`, and this table.
