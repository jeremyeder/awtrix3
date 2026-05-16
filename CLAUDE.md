# AWTRIX 3

ESP32 firmware for the Ulanzi Smart Pixel Clock (TC001) and AWTRIX 2 upgrade boards. Built with PlatformIO + Arduino framework.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:7510c1e2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->

## Build

```bash
pio run                          # Build default env (ulanzi)
pio run -e ulanzi                # Build for Ulanzi TC001
pio run -e awtrix2_upgrade       # Build for AWTRIX 2 mainboard
pio run -t upload                # Build and flash via USB
pio device monitor               # Serial monitor (115200 baud)
```

CI builds on push to `main` when `src/` or `lib/` changes. Creates a GitHub Release with firmware binaries.

## Architecture

Manager-based singleton pattern. Each manager has `setup()` + `tick()` called from `main.cpp`:

| Manager | File | Role |
|---------|------|------|
| DisplayManager | `src/DisplayManager.cpp` | LED matrix rendering, app lifecycle, notifications |
| ServerManager | `src/ServerManager.cpp` | WiFi, HTTP API, web server |
| MQTTManager | `src/MQTTManager.cpp` | MQTT client, Home Assistant discovery |
| PeripheryManager | `src/PeripheryManager.cpp` | Buttons, sensors (BME280/BMP280/SHT31/HTU21D), LDR, buzzer |
| UpdateManager | `src/UpdateManager.cpp` | OTA firmware updates |
| PowerManager | `src/PowerManager.cpp` | Battery monitoring, power management |

Supporting modules:
- `Apps.cpp` — Native and custom app rendering logic
- `Overlays.cpp` — Clock, date, notification overlays
- `Functions.cpp` — Utility functions (color conversion, text helpers)
- `effects.cpp` — Background LED effects
- `icons.cpp` — Icon storage and retrieval (LittleFS)
- `timer.cpp` — Timer/alarm functionality
- `MatrixDisplayUi.cpp` — Display UI framework (transitions, app rotation)
- `ArtnetWifi.cpp` — Art-Net DMX protocol support
- `Dictionary.cpp` — Key-value settings persistence

## Key Files

- `platformio.ini` — Build environments, library dependencies, board config
- `src/Globals.h` / `src/Globals.cpp` — All global state, settings, VERSION string
- `src/htmls.h` — Embedded HTML for web interface
- `src/cert.h` — SSL certificates for OTA
- `awtrix_partition.csv` — Custom ESP32 partition table
- `version` — Current firmware version (updated by CI)

## Board Targets

| Environment | Board | Define | Use Case |
|-------------|-------|--------|----------|
| `ulanzi` | esp32dev | `-DULANZI` | Ulanzi TC001 Smart Pixel Clock |
| `awtrix2_upgrade` | wemos_d1_mini32 | `-Dawtrix2_upgrade` | Legacy AWTRIX 2 mainboard |

## Gotchas

- `src/Globals.cpp` is **gitignored** — contains the VERSION string and default settings. CI generates it. You need a local copy to build.
- MQTT packet size is set to 8192 bytes via `-DMQTT_MAX_PACKET_SIZE=8192` build flag.
- "Apps" in AWTRIX are not standalone — they're dynamic display pages driven by external systems via MQTT/HTTP. The device has no app logic itself.
- The display is an 8x32 LED matrix (256 LEDs) using FastLED + NeoMatrix.
- Custom fonts in `src/AwtrixFont.h` — not standard Arduino fonts.
- LittleFS is used for icon and file storage on the ESP32 flash.
- Boot sequence runs a FreeRTOS task for the boot animation on core 0.

## Code Style

- C++ with Arduino framework conventions
- Singleton managers via static `getInstance()` pattern
- Global state in `Globals.h/cpp` (extern declarations + definitions)
- `DEBUG_PRINTLN` / `DEBUG_PRINTF` macros for serial debug output
- No unit tests — validation is done by flashing and testing on hardware

## Dependencies

Key libraries (managed by PlatformIO):
- ArduinoJson 6.x — JSON parsing for API and MQTT payloads
- FastLED + NeoMatrix — LED matrix control
- PubSubClient — MQTT client
- EasyButton — Button debouncing
- Adafruit sensor libraries — BME280, BMP280, HTU21D, SHT31
- EspSoftwareSerial — DFPlayer Mini communication
