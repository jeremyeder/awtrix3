---
topic: Effects & Overlays
type: codebase
last_compiled: 2026-05-16
source_count: 5
status: active
---

# Effects & Overlays

## Purpose [coverage: high -- 5 sources]
The effects system provides 19 animated background effects (BrickBreaker, Plasma, Matrix, Fireworks, etc.) and 7 weather overlay effects (rain, snow, storm, etc.) for AWTRIX 3. Effects can be applied as backgrounds behind custom apps/notifications, as global backgrounds across all apps, or as per-app/per-notification overlays. Each effect supports configurable speed, color palettes, and blending modes.

## Architecture [coverage: high -- 5 sources]
Effects are defined as an array of `Effect` structs, each containing a name, function pointer, and default settings. They are rendered as a layer in the display pipeline -- either behind content (background) or on top (overlay).

Key files:
- `src/effects.h` / `.cpp` -- effect definitions, function registry, overlay rendering

Core types:
```cpp
struct EffectSettings {
    double speed;
    CRGBPalette16 palette;
    bool blend;
};

typedef void (*EffectFunc)(FastLED_NeoMatrix *, int16_t, int16_t, EffectSettings *);

enum OverlayEffect { NONE, DRIZZLE, RAIN, SNOW, STORM, THUNDER, FROST };
```

19 built-in effects: Fade, MovingLine, BrickBreaker, PingPong, Radar, Checkerboard, Fireworks, PlasmaCloud, Ripple, Snake, Pacifica, TheaterChase, Plasma, Matrix, SwirlIn, SwirlOut, LookingEyes, TwinklingStars, ColorWaves.

7 weather overlays: clear, drizzle, rain, snow, storm, thunder, frost. These draw weather particles on top of existing content.

## Talks To [coverage: medium -- 2 sources]
- **DisplayManager** (called during rendering) -- `callEffect()` invoked for background effects, `EffectOverlay()` for weather overlays
- **MatrixDisplayUi** (background callback) -- global background effects rendered via `setBackground()`
- **Custom apps / notifications** (JSON `effect` key) -- per-app effect assignment

## API Surface [coverage: high -- 5 sources]
- `callEffect(matrix, x, y, index)` -- render an effect by array index
- `getEffectIndex(name)` -- look up effect index by name string
- `updateEffectSettings(index, json)` -- update speed/palette/blend from JSON
- `EffectOverlay(matrix, x, y, effect)` -- render a weather overlay
- `getOverlay(string)` -- parse overlay name string to enum

Effect settings JSON:
```json
{
  "speed": 3,
  "palette": "Rainbow",
  "blend": true
}
```

Built-in palettes: Cloud, Lava, Ocean, Forest, Stripe, Party, Heat, Rainbow.

Custom palettes: create a `.txt` file in `/PALETTES/` with 16 hex colors (one per line, `#RRGGBB` format).

Effect usage modes:
1. **Per-app/notification**: `{"effect": "Plasma", "effectSettings": {"speed": 3}}` in custom app JSON
2. **Global background**: set `background_effect` in dev.json
3. **Weather overlay (per-app)**: `{"overlay": "snow"}` in custom app JSON
4. **Weather overlay (global)**: `OVERLAY` setting via settings API

## Data [coverage: medium -- 2 sources]
- **effects[]** -- fixed array of 19 Effect structs with default settings
- **Palette files** -- custom palettes loaded from `/PALETTES/` directory at runtime

## Key Decisions [coverage: medium -- 2 sources]
- **Effects as first layer** -- background effects render behind text/icons, allowing content to remain readable
- **Separate overlay system** -- weather overlays are distinct from background effects and render on top
- **Global vs per-app** -- overlays cannot mix global and per-app modes simultaneously
- **Custom palettes from files** -- avoids hardcoding color schemes; users can create unlimited palette variations

## Gotchas [coverage: medium -- 2 sources]
- Do not use comments in custom palette `.txt` files (despite the documentation example showing them)
- High-complexity effects with many drawing instructions can consume significant RAM
- Global overlays and per-app overlays are mutually exclusive
- Effect names must match exactly (case-sensitive) when referenced by name

## Sources
- [src/effects.h](../src/effects.h)
- [src/effects.cpp](../src/effects.cpp)
- [src/Overlays.h](../src/Overlays.h)
- [src/Overlays.cpp](../src/Overlays.cpp)
- [docs/effects.md](../docs/effects.md)
