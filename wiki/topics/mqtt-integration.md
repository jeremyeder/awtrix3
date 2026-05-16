---
topic: MQTT Integration
type: codebase
last_compiled: 2026-05-16
source_count: 5
status: active
---

# MQTT Integration

## Purpose [coverage: high -- 5 sources]
The MQTTManager provides the primary communication channel between AWTRIX 3 and external smart home systems. It handles bidirectional messaging: receiving custom app content, notifications, settings changes, and control commands, while publishing device stats, button presses, current app state, and screen data. It also supports Home Assistant MQTT auto-discovery and MQTT topic placeholder substitution within custom app text.

## Architecture [coverage: medium -- 3 sources]
MQTTManager is a singleton class (`MQTTManager_`) using the PubSubClient library. It connects to a configurable broker with optional authentication and supports a configurable topic prefix (default: device unique ID).

The MQTT packet size is set to 8192 bytes (`-DMQTT_MAX_PACKET_SIZE=8192` in build flags) to accommodate large JSON payloads for custom apps with drawing instructions.

Key files:
- `src/MQTTManager.h` / `.cpp` -- MQTT client, topic routing, message handling
- `src/Globals.h` -- MQTT configuration variables (host, port, user, pass, prefix)

Topic structure follows `[PREFIX]/[command]` pattern:
- Inbound: `custom/[name]`, `notify`, `settings`, `power`, `sleep`, `sound`, `rtttl`, `moodlight`, `indicator1-3`, `switch`, `nextapp`, `previousapp`, `doupdate`, `reboot`, `sendscreen`
- Outbound: `stats`, `stats/effects`, `stats/transitions`, `stats/loop`, `screen`, button events, current app name

## Talks To [coverage: medium -- 3 sources]
- **MQTT Broker** (TCP: configurable host/port) -- persistent connection with auto-reconnect
- **DisplayManager** (direct call) -- forwards custom app JSON, notification JSON, settings, power/indicator commands
- **PeripheryManager** (direct call) -- forwards sound playback commands
- **Home Assistant** (MQTT Discovery) -- publishes device configuration for auto-discovery when `HA_DISCOVERY` is enabled
- **ServerManager** (shared logic) -- many commands are available via both MQTT and HTTP with identical JSON payloads

## API Surface [coverage: high -- 5 sources]
Publishing methods:
- `publish(topic, payload)` -- publish to `[PREFIX]/[topic]`
- `rawPublish(prefix, topic, payload)` -- publish with custom prefix (used for HA discovery)
- `beginPublish()` / `writePayload()` / `endPublish()` -- chunked publishing for large payloads
- `sendStats()` -- publish device statistics JSON
- `sendButton(byte, bool)` -- publish button press/release events
- `setCurrentApp(String)` -- publish current app name
- `setIndicatorState()` -- publish indicator state changes

Subscription:
- `subscribe(topic)` -- subscribe to a topic
- `getValueForTopic(topic)` -- retrieve last known value for a subscribed topic (used by MQTT placeholder feature)

Configuration (via `DoNotTouch.json` and web interface):
- `MQTT_HOST`, `MQTT_PORT` -- broker address
- `MQTT_USER`, `MQTT_PASS` -- authentication credentials
- `MQTT_PREFIX` -- topic prefix
- `HA_PREFIX` -- Home Assistant discovery prefix (default: `homeassistant`)
- `IO_BROKER` -- ioBroker compatibility mode

## Data [coverage: low -- 1 source]
- **Topic-value map** -- internal `std::map` caching the last payload received for each subscribed topic (used by MQTT placeholder substitution)
- **Connection state** -- tracks broker connectivity, reconnect timing

## Key Decisions [coverage: medium -- 2 sources]
- **8KB MQTT packet size** -- accommodates complex custom app JSON with drawing instructions, but limits very large payloads
- **Shared API parity with HTTP** -- most MQTT commands have equivalent HTTP endpoints with identical JSON format, allowing users to choose their preferred protocol
- **ioBroker compatibility flag** -- `IO_BROKER` setting adjusts MQTT behavior for ioBroker's topic handling differences
- **Placeholder substitution** -- `{{topic/path}}` syntax in custom app text enables direct MQTT data display without smart home middleware

## Gotchas [coverage: medium -- 2 sources]
- MQTT connection failure is indicated by a blinking LED in the bottom-left corner of the matrix
- The MQTT client only connects if `MQTT_HOST` is non-empty; leaving it blank skips MQTT entirely
- Large drawing instruction arrays can exceed the 8KB packet limit
- Stats are published at a configurable interval (`stats_interval` in dev.json, default 10000ms)

## Sources
- [src/MQTTManager.h](../src/MQTTManager.h)
- [src/MQTTManager.cpp](../src/MQTTManager.cpp)
- [src/Globals.h](../src/Globals.h)
- [docs/api.md](../docs/api.md)
- [docs/faq.md](../docs/faq.md)
