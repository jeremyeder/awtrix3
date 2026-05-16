---
topic: Project Overview
type: codebase
last_compiled: 2026-05-16
source_count: 8
status: active
---

# Project Overview

## Purpose [coverage: high -- 8 sources]
AWTRIX 3 is a custom open-source firmware for the Ulanzi Smart Pixel Clock (TC001), an ESP32-based device with a 32x8 WS2812B LED matrix. It serves as a smart home companion, displaying time, date, temperature, humidity, and battery status out of the box, while providing a rich API (MQTT and HTTP) for custom apps driven by external systems like Home Assistant, ioBroker, or Node-RED. The firmware is developed by Stephan Muehl (Blueforcer) and licensed under CC BY-NC-SA 4.0.

The project targets the ESP32 platform using PlatformIO with the Arduino framework, running at 240MHz. It supports both the stock Ulanzi TC001 hardware and custom DIY builds using ESP32-WROOM boards.

## Architecture [coverage: high -- 8 sources]
The firmware follows a singleton manager pattern. Each major subsystem is implemented as a C++ singleton class with `setup()` and `tick()` methods called from the main loop. The boot sequence in `main.cpp` initializes managers in order: PeripheryManager, ServerManager, DisplayManager, UpdateManager, and MQTTManager.

The main loop runs continuously, calling `tick()` on each manager:
```
loop() -> timer_tick() -> ServerManager.tick() -> DisplayManager.tick() 
       -> PeripheryManager.tick() -> MQTTManager.tick()
```

Key files:
- `src/main.cpp` -- entry point, boot sequence, main loop
- `src/Globals.h` / `Globals.cpp` -- all global configuration variables and settings persistence
- `src/DisplayManager.h` / `.cpp` -- LED matrix rendering, app management, notifications
- `src/ServerManager.h` / `.cpp` -- WiFi and HTTP server
- `src/MQTTManager.h` / `.cpp` -- MQTT client and message routing
- `src/PeripheryManager.h` / `.cpp` -- buttons, sensors, buzzer, DFPlayer
- `src/Apps.h` / `.cpp` -- native and custom app definitions
- `src/MatrixDisplayUi.h` / `.cpp` -- UI framework for app transitions and overlays

Build environments (platformio.ini):
- `ulanzi` (default) -- ESP32dev with `-DULANZI` flag, espressif32@6.3.0
- `awtrix2_upgrade` -- for upgrading legacy AWTRIX 2.0 hardware (Wemos D1 Mini32)

## Talks To [coverage: medium -- 4 sources]
- **MQTT Broker** (MQTT: configurable host/port) -- primary communication channel for custom apps, notifications, settings, and stats
- **Home Assistant** (MQTT Discovery: configurable prefix) -- auto-discovery of device entities
- **NTP Server** (UDP: configurable) -- time synchronization
- **GitHub Releases** (HTTPS) -- OTA firmware update checks
- **Artnet Controllers** (UDP: 2 universes, 384 channels each) -- DMX/Artnet LED control

## Data [coverage: medium -- 3 sources]
- **LittleFS** -- on-chip flash filesystem storing settings, icons (ICONS/), melodies (MELODIES/), custom app configs (CUSTOMAPP/), color palettes (PALETTES/), and the `DoNotTouch.json` settings file
- **Global state** -- extensive set of runtime variables in `Globals.h` covering display settings, sensor readings, network config, MQTT config, and app state
- **Settings persistence** -- `loadSettings()` / `saveSettings()` read/write JSON to flash

## Key Decisions [coverage: medium -- 4 sources]
- **Singleton pattern for managers** -- each subsystem is a single global instance, simplifying cross-module communication at the cost of tight coupling
- **Custom apps as external data** -- apps do not store logic on-device; they display content sent via MQTT/HTTP, keeping flash usage low and avoiding firmware rewrites for API changes
- **Fixed 32x8 matrix** -- optimized for the Ulanzi TC001 form factor; other sizes are not supported
- **Dual-core ESP32 usage** -- boot animation runs on core 0 via FreeRTOS task while setup completes on core 1
- **Max 20 custom apps** -- hardcoded limit with individual callback functions (CApp1 through CApp20)

## Gotchas [coverage: medium -- 3 sources]
- Flash memory may show only 192KB if flashed via Ulanzi's web updater instead of the AWTRIX online flasher (partition table difference)
- The `DoNotTouch.json` file contains critical settings; deleting it requires full reconfiguration
- Temperature sensor readings are affected by internal heat from the matrix and battery; calibration via `temp_offset` in `dev.json` is usually necessary
- GIF icons must be 8-bit without transparency to avoid rendering glitches

## Sources
- [src/main.cpp](../src/main.cpp)
- [src/Globals.h](../src/Globals.h)
- [src/DisplayManager.h](../src/DisplayManager.h)
- [platformio.ini](../platformio.ini)
- [docs/README.md](../docs/README.md)
- [docs/quickstart.md](../docs/quickstart.md)
- [docs/faq.md](../docs/faq.md)
- [docs/hardware.md](../docs/hardware.md)
