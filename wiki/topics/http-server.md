---
topic: HTTP Server & Web Interface
type: codebase
last_compiled: 2026-05-16
source_count: 6
status: active
---

# HTTP Server & Web Interface

## Purpose [coverage: high -- 6 sources]
The ServerManager handles WiFi connectivity, the HTTP API server, and the web-based configuration interface. It provides an alternative to MQTT for controlling AWTRIX 3 via REST-like HTTP endpoints, serves the onboard web interface for initial setup and ongoing configuration, and manages WiFi connection including AP mode fallback.

## Architecture [coverage: medium -- 4 sources]
ServerManager is a singleton class (`ServerManager_`) built on the `esp-fs-webserver` library (bundled in `lib/webserver/`). It manages WiFi connection, serves static files from LittleFS, and routes API requests.

Key files:
- `src/ServerManager.h` / `.cpp` -- WiFi, HTTP server, API routing
- `lib/webserver/esp-fs-webserver.h` / `.cpp` -- underlying web server framework
- `lib/webserver/setup_htm.h` -- WiFi setup portal HTML
- `lib/webserver/edit_htm.h` -- file manager HTML
- `src/htmls.h` -- additional embedded HTML (liveview, backup pages)

Boot sequence:
1. `loadSettings()` -- reads WiFi and network config from `DoNotTouch.json`
2. `setup()` -- connects to WiFi (with AP fallback after configurable timeout), starts HTTP server, registers API routes
3. `tick()` -- handles incoming HTTP requests each loop iteration

AP mode: If WiFi connection fails within `ap_timeout` seconds (default 15), AWTRIX creates an access point named `awtrix_XXXXX` with password `12345678` for initial configuration at `192.168.4.1`.

## Talks To [coverage: medium -- 3 sources]
- **WiFi** (802.11: saved SSID/password) -- connects to local network
- **DisplayManager** (direct call) -- forwards API requests for custom apps, notifications, settings, power control
- **MQTTManager** (shared API) -- many HTTP endpoints mirror MQTT topics
- **PeripheryManager** (direct call) -- forwards sound commands
- **LittleFS** (file system) -- serves static files, file manager operations

## API Surface [coverage: high -- 5 sources]
All HTTP API endpoints are under `http://[IP]/api/`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/stats` | GET | Device statistics (battery, RAM, uptime, etc.) |
| `/api/effects` | GET | List available effects |
| `/api/transitions` | GET | List available transitions |
| `/api/loop` | GET | List apps in current loop |
| `/api/screen` | GET | Current matrix as 24-bit color array |
| `/api/power` | POST | Toggle matrix on/off |
| `/api/sleep` | POST | Deep sleep for N seconds |
| `/api/sound` | POST | Play RTTTL sound file |
| `/api/rtttl` | POST | Play RTTTL string directly |
| `/api/moodlight` | POST | Set matrix to solid color/temperature |
| `/api/indicator1-3` | POST | Set colored indicator state |
| `/api/custom` | POST | Create/update custom app (query: `name=`) |
| `/api/notify` | POST | Send notification |
| `/api/notify/dismiss` | POST | Dismiss held notification |
| `/api/switch` | POST | Switch to named app |
| `/api/nextapp` | POST | Navigate to next app |
| `/api/previousapp` | POST | Navigate to previous app |
| `/api/settings` | GET/POST | Read/write settings |
| `/api/doupdate` | POST | Trigger firmware update |
| `/api/reboot` | POST | Reboot device |
| `/api/erase` | POST | Factory reset (formats flash) |
| `/api/resetSettings` | POST | Reset settings only |

Additional web pages:
- `/screen` -- browser-based liveview of the matrix
- `/fullscreen` -- fullscreen liveview with configurable FPS
- File manager for uploading/downloading icons, melodies, and configs
- Backup/restore functionality (zip of entire flash)

Authentication: Optional basic auth via `AUTH_USER` / `AUTH_PASS` protecting all pages and API endpoints.

## Data [coverage: medium -- 2 sources]
- **DoNotTouch.json** -- persistent storage for WiFi settings, MQTT config, NTP settings, static IP config, HA discovery flag, auth credentials
- **Connection state** -- `isConnected` flag and `myIP` address available to other managers

## Key Decisions [coverage: medium -- 3 sources]
- **AP mode fallback** -- ensures device remains configurable even without saved WiFi credentials
- **Configurable web port** -- `WEB_PORT` allows changing from default 80, displayed in boot scroll
- **Static IP support** -- optional static IP/gateway/subnet/DNS configuration via web interface
- **HTTP button callbacks** -- `button_callback` setting in dev.json sends button press events to an external HTTP URL

## Gotchas [coverage: medium -- 2 sources]
- WiFi connection failure is indicated by a blinking LED in the top-left corner
- Losing the auth password requires a full device reset
- The web file manager includes a `DoNotTouch.json` file that should not be manually edited (deleting it forces reconfiguration)
- Some USB cables cause flashing failures; try a different cable and port

## Sources
- [src/ServerManager.h](../src/ServerManager.h)
- [src/ServerManager.cpp](../src/ServerManager.cpp)
- [lib/webserver/esp-fs-webserver.h](../lib/webserver/esp-fs-webserver.h)
- [docs/webinterface.md](../docs/webinterface.md)
- [docs/api.md](../docs/api.md)
- [docs/quickstart.md](../docs/quickstart.md)
