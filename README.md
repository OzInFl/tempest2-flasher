# TEMPEST 2 — Firmware Flasher

Public web flasher and SD card files for **TEMPEST 2**.
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
| `firmware/tempest2-v0.92-r2.factory.bin` | Merged image: bootloader `0x1000`, partitions `0x8000`, boot_app0 `0xe000`, app `0x10000` |
| `firmware/firmware.bin` | App-only image for the on-device SD card update (`/firmware.bin` on the card) |
| `manual/TEMPEST2_User_Manual_v0.92.pdf` | User manual (built in `TEMPESTV2` by `tools/build_manual.py` from real device screenshots) |
| `.nojekyll` | Serve `firmware/` and `.json` verbatim |

SD card files (`customwords.txt` + `af/1.wav`..`af/4409.wav`, 222 MB zip) are a
[release asset](https://github.com/OzInFl/tempest2-flasher/releases/tag/sd-files-v1), not committed,
to keep the repo and Pages site small. SHA-256 `62e69b633ed8eacd9283daa5a5ea495cfcfccfc29cb289d51f2d49e7a456b205`.

## Current release

| Version | Built from | factory.bin SHA-256 |
|---|---|---|
| 0.92 (r2) | `TEMPESTV2@398ace7` | `77ec24319834afb3e85a44c79b6f6eba878560290ce5db68f7526e863f791f8e` |

r2 fixes the first upload (r1), whose bootloader header said 4 MB flash: units flashed from the page stayed black.

`firmware.bin` SHA-256: `192657cac6fa64f6f253ea677e3fc2e2e1721e93fc2618a034ef02ef99ba6e0d`

## Publishing a new release

1. In `TEMPESTV2`, bump `FIRMWARE_VERSION`, then `pio run`.
2. Build the merged image (board settings: DIO, 40 MHz, 16 MB):
   ```
   esptool.py --chip esp32 merge_bin -o firmware/tempest2-vX.YZ.factory.bin \
     --flash_mode dio --flash_freq 40m --flash_size 16MB \
     0x1000  <framework>/tools/sdk/esp32/bin/bootloader_dio_40m.bin \
     0x8000  .pio/build/m5stack-core2/partitions.bin \
     0xe000  <framework>/tools/partitions/boot_app0.bin \
     0x10000 .pio/build/m5stack-core2/firmware.bin
   ```
   `<framework>` = `~/.platformio/packages/framework-arduinoespressif32@3.20003.220626`
   (Arduino core 2.0.3, platform `espressif32@5.0.0`).

   **Then fix the flash-size byte.** The prebuilt bootloader's header says 4 MB and `merge_bin` will
   not change it (the image carries a hash). PlatformIO's uploader patches it on the fly, the web
   flasher does not, and with 4 MB the 16 MB partition table is rejected: the unit stays black and silent.
   ```
   python -c "p='firmware/tempest2-vX.YZ.factory.bin'; b=bytearray(open(p,'rb').read()); assert b[0x1003]==0x20; b[0x1003]=0x40; open(p,'wb').write(b)"
   ```
   (`0x40` = 16 MB, 40 MHz.) Verify by flashing one unit **from the page** before announcing a release.
3. Copy `.pio/build/m5stack-core2/firmware.bin` to `firmware/firmware.bin`.
4. Update `manifest-tempest2.json` (version + path), the version shown in `index.html`, and this table.
