---
topic: Periphery & Hardware
type: codebase
last_compiled: 2026-05-16
source_count: 6
status: active
---

# Periphery & Hardware

## Purpose [coverage: high -- 6 sources]
The PeripheryManager handles all hardware peripherals on the AWTRIX 3 device: physical buttons, temperature/humidity sensors, light dependent resistor (LDR) for auto-brightness, battery monitoring, passive buzzer for RTTTL melodies, and optional DFPlayer Mini for MP3 playback. It abstracts hardware differences between the Ulanzi TC001 and custom DIY builds.

## Architecture [coverage: high -- 5 sources]
PeripheryManager is a singleton class (`PeripheryManager_`) using the EasyButton library for debounced button handling. Sensor readings are smoothed using rolling averages (10 readings for battery, 30 for LDR).

Key files:
- `src/PeripheryManager.h` / `.cpp` -- button handling, sensor reading, sound playback
- `src/MelodyPlayer/melody_player.h` / `.cpp` -- RTTTL melody parser and buzzer output
- `src/MelodyPlayer/melody_factory.h` / `.cpp` -- melody file loading
- `src/MelodyPlayer/melody_factory_rtttl.cpp` -- RTTTL string parsing
- `src/MelodyPlayer/notes.h` / `notes_array.h` -- note frequency definitions
- `lib/LightResistor/LightDependentResistor.h` / `.cpp` -- LDR reading library
- `lib/DFMiniMp3-1.0.7-noChecksums/` -- DFPlayer Mini communication (checksum validation removed)

GPIO pinout (Ulanzi TC001):
| GPIO | Function |
|------|----------|
| GPIO34 (ADC6) | Battery sensor |
| GPIO35 (ADC7) | LDR (GL5516) light sensor |
| GPIO32 | LED matrix data |
| GPIO26 | Left button |
| GPIO27 | Middle button (inverted/NC) |
| GPIO14 | Right button |
| GPIO15 | Passive buzzer |
| GPIO21/22 | I2C SDA/SCL (temperature/humidity sensors) |
| GPIO23/18 | DFPlayer RX/TX (software serial) |

## Talks To [coverage: medium -- 3 sources]
- **DisplayManager** (button callbacks) -- left/right/select/long-press events for app navigation and menu
- **MQTTManager** (button publishing) -- button press events published to MQTT
- **ServerManager** (HTTP callback) -- button events sent to configured callback URL
- **I2C sensors** (I2C bus) -- reads from BME280, BMP280, HTU21DF, or SHT31
- **DFPlayer Mini** (software serial) -- MP3 playback commands

## API Surface [coverage: medium -- 3 sources]
- `setup()` / `tick()` -- lifecycle, called from main loop
- `playBootSound()` -- plays configured boot melody
- `playFromFile(String)` -- play RTTTL file from MELODIES folder
- `playRTTTLString(String)` -- play RTTTL string directly
- `parseSound(const char *json)` -- parse sound JSON command
- `isPlaying()` -- check if sound is currently playing
- `stopSound()` -- stop current playback
- `r2d2(const char *msg)` -- play R2D2-style sound from text
- `setVolume(uint8_t)` -- set buzzer/DFPlayer volume (0-30)
- `readUptime()` -- get device uptime in milliseconds
- `getMatrixPin()` -- returns the GPIO pin for the LED matrix

Buttons (via EasyButton):
- `buttonL`, `buttonR`, `buttonS` -- left, right, select button instances
- `buttonRST` -- reset button (if available)
- Supports: press, double-press (toggle matrix), long-press (enter/exit menu)
- `SWAP_BUTTONS` setting reverses left/right button assignments

## Data [coverage: medium -- 3 sources]
- **Sensor readings** (global variables): `CURRENT_TEMP`, `CURRENT_HUM`, `CURRENT_LUX`, `LDR_RAW`, `BATTERY_PERCENT`, `BATTERY_RAW`
- **Sensor type detection** -- auto-detected at boot: `TEMP_SENSOR_TYPE` enum (NONE, BME280, HTU21DF, BMP280, SHT31)
- **Rolling averages** -- battery: 10 samples, LDR: 30 samples for smooth readings
- **Calibration offsets** -- `TEMP_OFFSET`, `HUM_OFFSET` applied to raw sensor readings

## Key Decisions [coverage: medium -- 3 sources]
- **Auto-detection of temperature sensors** -- firmware probes I2C bus for supported sensors at boot, no manual configuration needed
- **Rolling average smoothing** -- prevents display flickering from noisy ADC readings
- **DFPlayer checksum removal** -- custom fork of DFMiniMp3 library with checksum validation disabled for compatibility
- **Configurable LDR parameters** -- `LDR_GAMMA`, `LDR_FACTOR`, `MIN_BRIGHTNESS`, `MAX_BRIGHTNESS` allow tuning auto-brightness curve
- **LDR_ON_GROUND option** -- supports alternative LDR wiring configuration

## Gotchas [coverage: high -- 5 sources]
- Temperature readings are significantly affected by internal heat; `temp_offset` of -5 to -9 is typical
- Battery calibration requires reading `bat_raw` at empty and full charge, then setting `min_battery` / `max_battery` in dev.json
- The middle button is wired inverted (NC) on the Ulanzi TC001
- DFPlayer requires MP3 files in a `MP3` folder on the SD card, named with 4-digit prefixes (0001.mp3)
- `SENSORS_STABLE` flag indicates when sensor readings have settled after boot

## Sources
- [src/PeripheryManager.h](../src/PeripheryManager.h)
- [src/PeripheryManager.cpp](../src/PeripheryManager.cpp)
- [src/MelodyPlayer/melody_player.h](../src/MelodyPlayer/melody_player.h)
- [docs/hardware.md](../docs/hardware.md)
- [docs/sounds.md](../docs/sounds.md)
- [docs/faq.md](../docs/faq.md)
