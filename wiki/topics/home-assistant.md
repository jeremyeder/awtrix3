---
topic: Home Assistant Integration
type: codebase
last_compiled: 2026-05-16
source_count: 5
status: active
---

# Home Assistant Integration

## Purpose [coverage: medium -- 5 sources]
AWTRIX 3 integrates with Home Assistant via MQTT auto-discovery and a bundled Arduino HA library. When `HA_DISCOVERY` is enabled, the device registers itself with Home Assistant automatically, exposing sensors (temperature, humidity, battery, LDR, uptime, WiFi signal), controls (buttons, switches for power/display), and update entities.

## Architecture [coverage: medium -- 4 sources]
The HA integration uses two layers:
1. **MQTT Discovery** -- MQTTManager publishes HA discovery config messages to `[HA_PREFIX]/[component]/[device]` topics
2. **ArduinoHA library** (bundled in `lib/home-assistant-integration/`) -- provides device type abstractions for HA entity registration

Key files:
- `lib/home-assistant-integration/src/ArduinoHA.h` -- main include, aggregates all device types
- `lib/home-assistant-integration/src/HADevice.h` / `.cpp` -- HA device representation
- `lib/home-assistant-integration/src/HAMqtt.h` / `.cpp` -- HA-specific MQTT wrapper
- `lib/home-assistant-integration/src/device-types/` -- 16 device type implementations:
  - HASensor, HASensorNumber -- numeric/text sensor entities
  - HABinarySensor -- on/off sensor entities
  - HAButton -- button entities
  - HASwitch -- toggle entities
  - HALight, HAFan, HACover, HALock, HANumber, HASelect -- control entities
  - HACamera -- camera entity (used for liveview)
  - HAScene -- scene entities
  - HAHVAC -- thermostat entities
  - HADeviceTracker, HADeviceTrigger, HATagScanner -- tracking entities
- `lib/home-assistant-integration/src/utils/` -- serialization helpers (HASerializer, HANumeric, HADictionary, HAUtils)

## Talks To [coverage: medium -- 3 sources]
- **Home Assistant** (MQTT Discovery) -- auto-registers device entities
- **MQTTManager** (MQTT publishing) -- sends sensor values, receives commands
- **MQTT Broker** (TCP) -- all communication flows through the broker

## API Surface [coverage: medium -- 3 sources]
Configuration:
- `HA_DISCOVERY` (boolean) -- enable/disable HA auto-discovery
- `HA_PREFIX` (string) -- discovery topic prefix (default: `homeassistant`)

Exposed HA entities typically include:
- Sensors: temperature, humidity, battery level, LDR/lux, uptime, WiFi signal, free RAM
- Controls: matrix power, auto-transition, display brightness
- Update: firmware update availability and trigger
- Buttons: reboot, factory reset

The library supports device availability, last will testament, and entity categories.

## Data [coverage: low -- 1 source]
- **HADevice** -- represents the AWTRIX device with unique ID, name, manufacturer, model, firmware version
- **Entity state** -- each registered entity maintains its current value for HA state reporting

## Key Decisions [coverage: medium -- 2 sources]
- **Bundled library** -- the ArduinoHA library is included directly in `lib/` rather than as a PlatformIO dependency, allowing custom modifications
- **Optional integration** -- HA discovery is opt-in; the device works standalone or with any MQTT system
- **Configurable prefix** -- allows multiple HA instances or non-standard setups

## Gotchas [coverage: medium -- 2 sources]
- HA discovery must be enabled in the web interface before entities appear in Home Assistant
- The HA prefix defaults to `homeassistant`; change it if your HA uses a different discovery prefix
- Notification dismissal can be triggered from Home Assistant when `hold: true` is set

## Sources
- [lib/home-assistant-integration/README.md](../lib/home-assistant-integration/README.md)
- [lib/home-assistant-integration/src/ArduinoHA.h](../lib/home-assistant-integration/src/ArduinoHA.h)
- [lib/home-assistant-integration/src/HADevice.h](../lib/home-assistant-integration/src/HADevice.h)
- [src/Globals.h](../src/Globals.h)
- [docs/api.md](../docs/api.md)
