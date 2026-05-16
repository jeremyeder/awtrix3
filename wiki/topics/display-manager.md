---
topic: Display Manager
type: codebase
last_compiled: 2026-05-16
source_count: 6
status: active
---

# Display Manager

## Purpose [coverage: high -- 6 sources]
The DisplayManager is the central rendering engine for AWTRIX 3. It controls the 32x8 WS2812B LED matrix via FastLED and FastLED_NeoMatrix, manages the app loop (native and custom apps), handles notifications, processes drawing instructions, and coordinates transitions between apps. It is the largest and most complex subsystem in the firmware.

## Architecture [coverage: high -- 6 sources]
DisplayManager is a singleton class (`DisplayManager_`) instantiated globally. It wraps around `MatrixDisplayUi`, which provides the app-switching framework with configurable transition effects.

The display pipeline per frame:
1. Background effects are drawn (if configured)
2. The current app callback renders its content
3. Overlays are drawn on top (status indicators, notifications, menu)
4. The frame is pushed to the LED matrix

Key files:
- `src/DisplayManager.h` / `.cpp` -- main class with rendering, app management, settings
- `src/MatrixDisplayUi.h` / `.cpp` -- UI framework handling app transitions, timing, and state
- `src/GifPlayer.h` -- GIF animation playback on the matrix

The class exposes a comprehensive drawing API:
- Text rendering: `printText()`, `HSVtext()`, `GradientText()`, colored fragments
- Shapes: `drawRect()`, `drawFilledRect()`, `drawLine()`, `drawPixel()`, `drawCircle()`, `fillCircle()`
- Charts: `drawBarChart()`, `drawLineChart()`, `drawProgressBar()`
- Images: `drawJPG()`, `drawBMP()`, `drawRGBBitmap()`
- Custom drawing instructions: `processDrawInstructions()` -- parses JSON draw commands (dp, dl, dr, df, dc, dfc, dt, db)

## Talks To [coverage: high -- 5 sources]
- **MatrixDisplayUi** (direct call) -- delegates app transitions, overlay management, and frame timing
- **Apps** (callback functions) -- each app registers an `AppCallback` function pointer
- **MQTTManager** (direct call) -- publishes stats, app state, and screen data
- **ServerManager** (HTTP endpoints) -- serves liveview, screenshots, and API responses
- **PeripheryManager** (button events) -- receives button press notifications for navigation

## API Surface [coverage: high -- 6 sources]
Core management:
- `setup()` / `tick()` -- lifecycle methods called from main loop
- `loadNativeApps()` / `loadCustomApps()` -- populate the app loop
- `generateCustomPage()` / `parseCustomPage()` -- create/update custom apps from JSON
- `generateNotification()` -- queue a notification overlay
- `switchToApp()` / `nextApp()` / `previousApp()` / `forceNextApp()` -- app navigation
- `setAutoTransition()` -- enable/disable automatic app cycling
- `reorderApps()` -- change app order via JSON

Display control:
- `setBrightness()` -- set matrix brightness (0-255)
- `setPower()` -- turn matrix on/off with animation
- `setNewSettings()` -- apply settings from JSON
- `startArtnet()` -- enable Artnet/DMX mode

Status indicators (3 colored pixels at right edge):
- `setIndicator1Color/State()`, `setIndicator2Color/State()`, `setIndicator3Color/State()`
- `indicatorParser()` -- parse JSON for indicator config including blink/fade effects

Data output:
- `getStats()` -- JSON string of device statistics
- `getSettings()` -- JSON string of current settings
- `getAppsAsJson()` / `getAppsWithIcon()` -- app loop information
- `ledsAsJson()` -- current matrix state as color array (for liveview)

## Data [coverage: medium -- 3 sources]
- **App vector** -- `std::vector<std::pair<String, AppCallback>>` storing the ordered list of active apps
- **Custom apps map** -- `std::map<String, CustomApp>` keyed by app name
- **Notification queue** -- `std::vector<Notification>` for stacked notifications
- **LED buffer** -- 256 CRGB values (32x8 matrix) managed by FastLED

## Key Decisions [coverage: medium -- 4 sources]
- **11 transition effects** -- Random, Slide, Fade, Zoom, Rotate, Pixelate, Curtain, Ripple, Blink, Reload, Crossfade; each implemented as a separate method in MatrixDisplayUi
- **Configurable FPS** -- default ~30 FPS (33ms update interval), adjustable via `MATRIX_FPS`
- **3 matrix layout types** -- configurable via `dev.json` to handle different LED wiring patterns
- **Gamma correction** -- applied to LED output for perceptual brightness linearity
- **Color correction and temperature** -- FastLED CRGB corrections applied globally

## Gotchas [coverage: medium -- 2 sources]
- Drawing instructions consume significant RAM depending on complexity; too many objects can cause crashes or reboots
- The `appIsSwitching` flag must be checked before modifying app state to avoid race conditions during transitions
- Custom apps are limited to 20 due to hardcoded callback functions (CApp1-CApp20)
- GIFs used in BigTime mode (TMODE=5) cannot be replaced while the mode is active because the file handle is held open

## Sources
- [src/DisplayManager.h](../src/DisplayManager.h)
- [src/DisplayManager.cpp](../src/DisplayManager.cpp)
- [src/MatrixDisplayUi.h](../src/MatrixDisplayUi.h)
- [src/MatrixDisplayUi.cpp](../src/MatrixDisplayUi.cpp)
- [src/GifPlayer.h](../src/GifPlayer.h)
- [docs/effects.md](../docs/effects.md)
