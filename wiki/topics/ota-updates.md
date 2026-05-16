---
topic: OTA Updates & Power Management
type: codebase
last_compiled: 2026-05-16
source_count: 3
status: active
---

# OTA Updates & Power Management

## Purpose [coverage: medium -- 3 sources]
The UpdateManager handles over-the-air firmware updates from GitHub releases. The PowerManager provides deep sleep functionality for battery conservation. Together they manage the device lifecycle beyond normal operation.

## Architecture [coverage: medium -- 3 sources]
Both are singleton classes with minimal interfaces.

Key files:
- `src/UpdateManager.h` / `.cpp` -- firmware update check and download from GitHub
- `src/PowerManager.h` / `.cpp` -- ESP32 deep sleep management
- `src/timer.h` / `src/timer.cpp` -- periodic timer for scheduled checks

UpdateManager:
- `setup()` -- initializes update checking (if `UPDATE_CHECK` is enabled)
- `checkUpdate(bool)` -- queries GitHub releases for newer firmware; sets `UPDATE_AVAILABLE` flag
- `updateFirmware()` -- downloads and flashes new firmware binary

PowerManager:
- `sleepParser(const char*)` -- parses sleep JSON command
- `sleep(uint64_t)` -- enters ESP32 deep sleep for specified microseconds
- Wake sources: timeout expiry or middle button press

## Talks To [coverage: medium -- 2 sources]
- **GitHub Releases** (HTTPS) -- checks for and downloads firmware updates
- **DisplayManager** (sleep animation) -- `showSleepAnimation()` before entering deep sleep
- **MQTT/HTTP API** -- update and sleep commands received via API
- **Onscreen menu** -- update check available as menu item

## API Surface [coverage: medium -- 2 sources]
- MQTT `[PREFIX]/doupdate` / HTTP POST `/api/doupdate` -- trigger firmware update
- MQTT `[PREFIX]/sleep` / HTTP POST `/api/sleep` -- `{"sleep": N}` deep sleep for N seconds
- MQTT `[PREFIX]/reboot` / HTTP POST `/api/reboot` -- device reboot
- Onscreen menu "UPDATE" -- check and download updates

## Gotchas [coverage: medium -- 2 sources]
- Deep sleep can only be woken by timeout or middle button press; no API wake-up available
- OTA updates require the full AWTRIX flasher partition table for sufficient flash space
- `UPDATE_CHECK` can be disabled to prevent automatic update polling

## Sources
- [src/UpdateManager.h](../src/UpdateManager.h)
- [src/PowerManager.h](../src/PowerManager.h)
- [docs/api.md](../docs/api.md)
