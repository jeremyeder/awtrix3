---
topic: API Reference
type: codebase
last_compiled: 2026-05-16
source_count: 4
status: active
---

# API Reference

## Purpose [coverage: high -- 4 sources]
AWTRIX 3 exposes a dual-protocol API (MQTT and HTTP) for controlling all device functions: custom app creation, notifications, display settings, sound playback, mood lighting, colored indicators, power management, and firmware updates. Both protocols accept identical JSON payloads, allowing users to choose based on their infrastructure.

## Architecture [coverage: medium -- 3 sources]
The API is split across two managers:
- **MQTTManager** -- subscribes to `[PREFIX]/[command]` topics, routes payloads to handlers
- **ServerManager** -- registers HTTP routes at `/api/[command]`, routes payloads to the same handlers

This dual routing means every feature is accessible via both protocols with the same JSON format.

Key files:
- `docs/api.md` -- comprehensive API documentation (the authoritative reference)
- `src/MQTTManager.cpp` -- MQTT topic handlers
- `src/ServerManager.cpp` -- HTTP route handlers
- `src/DisplayManager.cpp` -- actual command execution

## API Surface [coverage: high -- 4 sources]

### Device Status
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/stats` | GET `/api/stats` | Device stats (battery, RAM, uptime, WiFi, etc.) |
| `[PREFIX]/stats/effects` | GET `/api/effects` | Available effect names |
| `[PREFIX]/stats/transitions` | GET `/api/transitions` | Available transition names |
| `[PREFIX]/stats/loop` | GET `/api/loop` | Current app loop listing |

### Display Control
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/power` | POST `/api/power` | `{"power": true/false}` |
| `[PREFIX]/sleep` | POST `/api/sleep` | `{"sleep": N}` deep sleep for N seconds |
| `[PREFIX]/moodlight` | POST `/api/moodlight` | Solid color: `{"brightness":170,"kelvin":2300}` or `{"color":"#FF00FF"}` |

### Custom Apps & Notifications
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/custom/[name]` | POST `/api/custom?name=[name]` | Create/update custom app |
| `[PREFIX]/notify` | POST `/api/notify` | One-time notification |
| `[PREFIX]/notify/dismiss` | POST `/api/notify/dismiss` | Dismiss held notification |
| Empty payload to custom topic | Empty body to custom endpoint | Delete custom app |

### Navigation
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/nextapp` | POST `/api/nextapp` | Next app |
| `[PREFIX]/previousapp` | POST `/api/previousapp` | Previous app |
| `[PREFIX]/switch` | POST `/api/switch` | `{"name":"Time"}` switch to named app |

### Sound
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/sound` | POST `/api/sound` | `{"sound":"alarm"}` play RTTTL file |
| `[PREFIX]/rtttl` | POST `/api/rtttl` | Play RTTTL string directly |

### Indicators
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/indicator1-3` | POST `/api/indicator1-3` | `{"color":[255,0,0],"blink":500,"fade":1000}` |

### Settings
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/settings` | GET/POST `/api/settings` | Read/write all settings |

Key settings keys: `ATIME`, `TEFF`, `TSPEED`, `TCOL`, `TMODE` (0-6), `BRI`, `ABRI`, `ATRANS`, `UPPERCASE`, `SSPEED`, `TIM`/`DAT`/`HUM`/`TEMP`/`BAT` (enable/disable native apps), `VOL`, `OVERLAY`, `WD` (weekday), `SOM` (start on Monday), `CEL` (Celsius), `BLOCKN` (block navigation), plus color settings for each native app.

### System
| MQTT | HTTP | Description |
|------|------|-------------|
| `[PREFIX]/doupdate` | POST `/api/doupdate` | Trigger firmware update |
| `[PREFIX]/reboot` | POST `/api/reboot` | Reboot device |
| N/A | POST `/api/erase` | Factory reset |
| N/A | POST `/api/resetSettings` | Reset settings only |

### Drawing Instructions
Custom apps and notifications accept a `draw` array with primitives:
- `dp` [x, y, color] -- pixel
- `dl` [x0, y0, x1, y1, color] -- line
- `dr` [x, y, w, h, color] -- rectangle outline
- `df` [x, y, w, h, color] -- filled rectangle
- `dc` [x, y, r, color] -- circle outline
- `dfc` [x, y, r, color] -- filled circle
- `dt` [x, y, text, color] -- text
- `db` [x, y, w, h, [bitmap]] -- RGB888 bitmap array

Colors accept hex strings (`"#FF0000"`) or RGB arrays (`[255,0,0]`).

## Key Decisions [coverage: medium -- 2 sources]
- **Protocol parity** -- identical JSON payloads work via both MQTT and HTTP, minimizing learning curve
- **Built-in app names** -- Time, Date, Temperature, Humidity, Battery are reserved names for `switch` command
- **Stacking notifications** -- `stack: true` (default) queues notifications; `stack: false` replaces current
- **Client forwarding** -- `clients` array in notifications can forward to other AWTRIX devices

## Sources
- [docs/api.md](../docs/api.md)
- [src/MQTTManager.h](../src/MQTTManager.h)
- [src/ServerManager.h](../src/ServerManager.h)
- [src/DisplayManager.h](../src/DisplayManager.h)
