---
topic: Apps System
type: codebase
last_compiled: 2026-05-16
source_count: 5
status: active
---

# Apps System

## Purpose [coverage: high -- 5 sources]
The apps system provides the content display framework for AWTRIX 3. It manages two categories of apps: native apps (Time, Date, Temperature, Humidity, Battery) that run with built-in logic, and custom apps that display externally-provided content via MQTT/HTTP. Apps rotate in a configurable loop with transition effects, and the system supports notifications that overlay on top of the current app.

## Architecture [coverage: high -- 5 sources]
Apps are implemented as callback functions with the signature:
```cpp
void AppCallback(FastLED_NeoMatrix *matrix, MatrixDisplayUiState *state, 
                 int16_t x, int16_t y, GifPlayer *gifPlayer)
```

The app loop is a `std::vector<std::pair<String, AppCallback>>` stored globally. Each entry maps an app name to its render function.

Key files:
- `src/Apps.h` / `.cpp` -- native app implementations and custom app rendering
- `src/Overlays.h` / `.cpp` -- notification overlay, status overlay, menu overlay

**Native apps** (built-in):
- `TimeApp` -- displays time with 7 configurable modes (TMODE 0-6), weekday bar, calendar icon, binary clock
- `DateApp` -- displays date with 9 format options
- `TempApp` -- temperature from integrated sensor with configurable decimal places
- `HumApp` -- humidity from integrated sensor
- `BatApp` -- battery percentage (Ulanzi builds only, excluded in `awtrix2_upgrade`)

**Custom apps** are stored in `std::map<String, CustomApp>` and rendered by `ShowCustomApp()`, which delegates to one of 20 hardcoded callback wrappers (CApp1-CApp20).

The `CustomApp` struct holds extensive state:
- Display content: text, icon, colors, fragments, gradients, drawing instructions
- Behavior: scroll position/speed/delay, bounce, repeat count, duration, lifetime
- Visuals: rainbow, fade, blink, effect, overlay, progress bar, bar/line chart data
- Icon handling: push icon modes (0=static, 1=push once, 2=push and return), icon position tracking

## Talks To [coverage: medium -- 3 sources]
- **DisplayManager** (direct integration) -- registers app callbacks, called during frame rendering
- **MQTTManager** (topic subscription) -- receives custom app content via `[PREFIX]/custom/[appname]`
- **ServerManager** (HTTP POST) -- receives custom app content via `/api/custom?name=[appname]`
- **LittleFS** (file I/O) -- loads saved custom apps from CUSTOMAPP/ folder, loads icons from ICONS/

## API Surface [coverage: high -- 5 sources]
Custom app JSON properties (all optional):
- **Content**: `text`, `icon`, `color`, `gradient`, `rainbow`, `background`
- **Text control**: `textCase` (0=global, 1=uppercase, 2=as-sent), `topText`, `textOffset`, `center`, `noScroll`, `scrollSpeed`
- **Text effects**: `blinkText`, `fadeText`
- **Charts**: `bar` (up to 16 values), `line` (up to 16 values), `autoscale`, `barBC`
- **Progress**: `progress` (0-100), `progressC`, `progressBC`
- **Icon**: `pushIcon` (0/1/2), icon as ID, filename, or base64 8x8 JPG
- **Timing**: `duration`, `lifetime`, `lifetimeMode` (0=delete, 1=stale indicator), `repeat`
- **Drawing**: `draw` array of primitives (dp, dl, dr, df, dc, dfc, dt, db)
- **Effects**: `effect` (background effect name), `effectSettings`, `overlay` (weather overlays)
- **Sound**: `sound`, `rtttl`, `loopSound` (notifications only)
- **Behavior**: `hold` (notification only), `wakeup`, `stack`, `save`, `pos`, `clients` (forwarding)
- **Colored fragments**: text as array of `{"t": "text", "c": "HEXCOLOR"}` objects

MQTT placeholder feature: `{{topic/path}}` in text values gets replaced with live MQTT payload data.

Multiple custom apps can be sent simultaneously as a JSON array, auto-suffixed as `name0`, `name1`, etc.

## Data [coverage: medium -- 3 sources]
- **CustomApp struct** -- per-app state including scroll position, icon file handle, chart data buffers (16 values each), JPEG buffer (1000 bytes), color vectors, text fragments
- **Notification struct** -- similar to CustomApp with additional fields: `hold`, `wakeup`, `sound`, `rtttl`, `loopSound`, `startime`
- **notifications vector** -- queue of pending notifications, processed FIFO with optional stacking

## Key Decisions [coverage: medium -- 3 sources]
- **External logic model** -- custom apps are "dynamic pages" without on-device logic; all intelligence lives in the external smart home system. This saves flash memory and avoids firmware updates for API changes.
- **20 custom app limit** -- due to function pointer array approach with individual CApp1-CApp20 wrapper functions (comment in code: "Unattractive to have a function for every customapp which does the same, but currently still no other option found TODO")
- **MQTT placeholder substitution** -- allows direct sensor display without smart home middleware (e.g., `"Solar: {{inverter/total/P_AC}} W"`)

## Gotchas [coverage: medium -- 2 sources]
- Deleting one app from a multi-app set (test0, test1...) can disrupt loop ordering since there are no placeholder slots
- `save` flag persists custom apps to flash; avoid for frequently-updated apps due to flash write cycle limits
- `lifetime` of 0 means the app persists indefinitely; non-zero values auto-delete after N seconds without update
- The `pos` parameter for loop positioning is experimental and only applies on first push

## Sources
- [src/Apps.h](../src/Apps.h)
- [src/Apps.cpp](../src/Apps.cpp)
- [src/Overlays.h](../src/Overlays.h)
- [src/Overlays.cpp](../src/Overlays.cpp)
- [docs/apps.md](../docs/apps.md)
