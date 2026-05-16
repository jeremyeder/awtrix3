---
concept: Dual-Protocol API Parity
last_compiled: 2026-05-16
topics_connected: [api-reference, mqtt-integration, http-server, apps-system]
status: active
---

# Dual-Protocol API Parity

## Pattern
Nearly every AWTRIX 3 feature is accessible through both MQTT and HTTP with identical JSON payloads. The MQTT topic `[PREFIX]/custom/[name]` maps directly to HTTP POST `/api/custom?name=[name]`. Settings, indicators, power control, sound, and navigation all follow this pattern. The JSON schema is the same regardless of transport.

This parity is maintained by having both MQTTManager and ServerManager route incoming payloads to the same handler functions in DisplayManager. The API documentation (docs/api.md) presents both protocols side-by-side in tables.

## Instances
- **Custom apps** in [apps-system](../topics/apps-system.md): identical JSON format works via `[PREFIX]/custom/test` (MQTT) or `POST /api/custom?name=test` (HTTP)
- **Settings** in [api-reference](../topics/api-reference.md): same JSON keys work via `[PREFIX]/settings` (MQTT) or `GET/POST /api/settings` (HTTP)
- **Exceptions**: a few endpoints are HTTP-only (`/api/erase`, `/api/resetSettings`) where MQTT access would be inappropriate for destructive operations

## What This Means
This design choice dramatically lowers the barrier to entry for smart home integrations. Users of Node-RED, Home Assistant, ioBroker, or any system with either MQTT or HTTP capabilities can control AWTRIX without adapters. It also means the API documentation serves double duty -- learn one payload format, use either transport. The tradeoff is that both managers must be kept in sync when new features are added, creating a maintenance burden for the developer.

## Sources
- [api-reference](../topics/api-reference.md)
- [mqtt-integration](../topics/mqtt-integration.md)
- [http-server](../topics/http-server.md)
- [apps-system](../topics/apps-system.md)
