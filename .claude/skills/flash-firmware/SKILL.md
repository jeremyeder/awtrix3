---
name: flash-firmware
description: Build and flash AWTRIX firmware to a connected ESP32 device, then open serial monitor
disable-model-invocation: true
---

Build, flash, and monitor the AWTRIX 3 firmware.

## Steps

1. Ask the user which build environment to use:
   - **ulanzi** (default) — Ulanzi TC001 Smart Pixel Clock
   - **awtrix2_upgrade** — Legacy AWTRIX 2 mainboard on Wemos D1 Mini32

2. Check that PlatformIO is installed:
   ```bash
   pio --version
   ```

3. Build and upload:
   ```bash
   pio run -e <environment> -t upload
   ```

4. If upload succeeds, ask if the user wants to open the serial monitor:
   ```bash
   pio device monitor
   ```
   Monitor runs at 115200 baud with ESP32 exception decoder filter.

## Troubleshooting

- **Upload failed**: Check USB connection, try `pio device list` to find the port
- **Build failed on Globals.cpp**: This file is gitignored. Ensure a local copy exists in `src/`
- **Permission denied on serial port**: May need `sudo chmod 666 /dev/ttyUSB0` (Linux) or add user to `dialout` group
