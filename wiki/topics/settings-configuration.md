---
topic: Settings & Configuration
type: codebase
last_compiled: 2026-05-16
source_count: 5
status: active
---

# Settings & Configuration

## Purpose [coverage: high -- 5 sources]
AWTRIX 3 has a layered configuration system: runtime settings adjustable via API/onscreen menu, persistent network/MQTT settings in `DoNotTouch.json`, and advanced developer settings in `dev.json`. Over 60 configurable parameters control display behavior, sensor calibration, network setup, and hardware options.

## Architecture [coverage: medium -- 3 sources]
Settings are stored as global variables in `Globals.h/cpp`, loaded from LittleFS JSON files at boot, and modifiable at runtime via the settings API.

Key files:
- `src/Globals.h` / `.cpp` -- all configuration variables, `loadSettings()` / `saveSettings()`
- `src/MenuManager.h` / `.cpp` -- onscreen menu for direct hardware adjustment
- `docs/dev.md` -- developer settings documentation
- `docs/onscreen.md` -- onscreen menu documentation
- `docs/webinterface.md` -- web interface documentation

Configuration layers:
1. **DoNotTouch.json** -- WiFi, MQTT, NTP, static IP, HA discovery, auth (set via web interface)
2. **dev.json** -- advanced settings applied at boot only (manually created via file manager)
3. **Settings API** -- runtime-adjustable settings (MQTT `[PREFIX]/settings` or HTTP `/api/settings`)
4. **Onscreen menu** -- subset of settings adjustable via physical buttons

## API Surface [coverage: high -- 5 sources]

### Runtime Settings (via API)
| Key | Type | Description | Default |
|-----|------|-------------|---------|
| `ATIME` | number | App display duration (seconds) | 7 |
| `TEFF` | number | Transition effect (0-10) | 1 |
| `TSPEED` | number | Transition speed (ms) | 500 |
| `TCOL` | color | Global text color | N/A |
| `TMODE` | int | Time app mode (0-6) | 1 |
| `BRI` | number | Brightness (0-255) | N/A |
| `ABRI` | boolean | Auto brightness | N/A |
| `ATRANS` | boolean | Auto transition | N/A |
| `UPPERCASE` | boolean | Force uppercase | true |
| `SSPEED` | int | Scroll speed (% of default) | 100 |
| `VOL` | int | Sound volume (0-30) | N/A |
| `OVERLAY` | string | Global weather overlay | N/A |
| `TIM/DAT/HUM/TEMP/BAT` | boolean | Enable native apps | true |
| `BLOCKN` | boolean | Block physical navigation | false |
| Color settings | color | Per-app text colors, calendar colors, weekday colors | various |

### Developer Settings (dev.json, boot-only)
| Key | Type | Description | Default |
|-----|------|-------------|---------|
| `hostname` | string | Device hostname | uniqueID |
| `matrix` | int | Matrix layout type (0/1/2) | 0 |
| `rotate_screen` | boolean | Flip display 180 degrees | false |
| `mirror_screen` | boolean | Mirror display | false |
| `temp_offset` | float | Temperature calibration | -9 |
| `hum_offset` | float | Humidity calibration | 0 |
| `min/max_brightness` | int | Auto-brightness range | 2/180 |
| `min/max_battery` | int | Battery calibration (raw ADC) | 475/665 |
| `background_effect` | string | Global background effect | N/A |
| `dfplayer` | boolean | Enable DFPlayer | false |
| `debug_mode` | boolean | Serial debug output | false |
| `ap_timeout` | int | WiFi AP fallback timeout (sec) | 15 |
| `new_year` | boolean | Fireworks at midnight | false |
| `swap_buttons` | boolean | Swap left/right buttons | false |

### Onscreen Menu Items
Brightness, Color, Auto-switch, Transition speed, App time, Time format, Date format, Week start, Temperature unit, App toggles, Sound toggle, Update check.

## Data [coverage: medium -- 3 sources]
- **Globals.h** -- 80+ extern variables covering all configuration state
- **DoNotTouch.json** -- persisted to LittleFS, regenerated if deleted
- **dev.json** -- manually managed, optional file

## Key Decisions [coverage: medium -- 2 sources]
- **Separate config files** -- critical network settings isolated in DoNotTouch.json to prevent accidental API modification; dev settings in a separate file because they rarely change
- **Boot-only dev settings** -- dev.json is read once at startup, avoiding runtime complexity for hardware-level settings
- **Web-first initial setup** -- WiFi and MQTT must be configured through the web interface; API access requires network connectivity first

## Gotchas [coverage: high -- 5 sources]
- Dev.json settings require a reboot to take effect
- Enabling/disabling native apps (TIM, DAT, HUM, TEMP, BAT) requires a reboot
- Deleting DoNotTouch.json resets all network and MQTT settings
- Auth credentials cannot be recovered if forgotten; full reset required
- `max_brightness` above 180 can cause overheating from the LED matrix

## Sources
- [src/Globals.h](../src/Globals.h)
- [src/MenuManager.h](../src/MenuManager.h)
- [docs/dev.md](../docs/dev.md)
- [docs/onscreen.md](../docs/onscreen.md)
- [docs/webinterface.md](../docs/webinterface.md)
