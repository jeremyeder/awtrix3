# Wiki Schema

This file defines the structure and conventions for the AWTRIX 3 codebase wiki. It is generated on first compile and co-evolved between human and LLM on subsequent runs.

**Human:** You can edit this file to rename topics, merge them, add conventions, or change the article structure. The compiler will respect your changes on the next run.

**Compiler:** Read this file before classifying sources. Follow its conventions. Add new topics here when discovered. Never remove topics without human approval.

## Topics

- project-overview: High-level architecture, boot sequence, build system, and project context for the AWTRIX 3 ESP32 firmware
- display-manager: LED matrix rendering engine, app loop management, transitions, drawing API, and notification display
- apps-system: Native apps (Time, Date, Temp, Humidity, Battery), custom app framework, notification system, and app loop management
- mqtt-integration: MQTT client, topic routing, Home Assistant discovery, placeholder substitution, and bidirectional messaging
- http-server: WiFi management, HTTP API server, web configuration interface, AP mode, file manager, and liveview
- periphery-hardware: Physical buttons, temperature/humidity sensors, LDR, battery monitoring, buzzer, DFPlayer, and GPIO pinout
- effects-overlays: 19 animated background effects, 7 weather overlay effects, color palettes, and effect settings
- games: Game framework with Slot Machine and AWTRIX Says implementations
- home-assistant: HA MQTT auto-discovery, ArduinoHA library integration, and entity registration
- api-reference: Comprehensive dual-protocol (MQTT/HTTP) API documentation with all endpoints, JSON schemas, and drawing instructions
- sound-system: RTTTL melody playback via passive buzzer and optional DFPlayer Mini MP3 module
- settings-configuration: Layered configuration system (DoNotTouch.json, dev.json, Settings API, onscreen menu) with 60+ parameters
- ota-updates: Over-the-air firmware updates from GitHub and ESP32 deep sleep power management

## Concepts

Cross-cutting patterns that span 3+ topics. Interpretive, not just factual.
- singleton-manager-pattern: Every subsystem uses C++ singletons with setup()/tick() lifecycle -- connects [project-overview, display-manager, mqtt-integration, http-server, periphery-hardware, games, ota-updates]
- dual-protocol-api-parity: MQTT and HTTP APIs accept identical JSON payloads for nearly all features -- connects [api-reference, mqtt-integration, http-server, apps-system]
- external-logic-model: Custom apps have no on-device logic; all intelligence lives in external smart home systems -- connects [apps-system, api-reference, mqtt-integration, project-overview]

## Article Structure

Each topic article follows the codebase template:
- **Purpose** [coverage] -- what this module does, standalone briefing
- **Architecture** [coverage] -- internal structure, key files, entry points
- **Talks To** [coverage] -- dependencies and communication patterns
- **API Surface** [coverage] -- exposed interfaces and endpoints
- **Data** [coverage] -- owned state, databases, caches, queues
- **Key Decisions** [coverage] -- why it was built this way
- **Gotchas** [coverage] -- known issues, edge cases, common mistakes
- **Sources** -- backlinks to every contributing source file

Coverage tags: `[coverage: high -- N sources]`, `[coverage: medium -- N sources]`, `[coverage: low -- N sources]`

## Naming Conventions

- Topic slugs: lowercase-kebab-case (e.g., `display-manager`, `mqtt-integration`)
- Files: `{topic-slug}.md` in `topics/`
- Concept files: `{concept-slug}.md` in `concepts/`
- Dates: YYYY-MM-DD format everywhere
- Links: Markdown `[text](path)` with relative paths from `topics/` or `concepts/`

## Evolution Log

- 2026-05-16: Initial schema generated from 11 topics, 3 concepts (first codebase compilation of AWTRIX 3 firmware)
