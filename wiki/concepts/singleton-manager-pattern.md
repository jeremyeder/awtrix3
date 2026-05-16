---
concept: Singleton Manager Pattern
last_compiled: 2026-05-16
topics_connected: [project-overview, display-manager, mqtt-integration, http-server, periphery-hardware, games, ota-updates]
status: active
---

# Singleton Manager Pattern

## Pattern
Every major subsystem in AWTRIX 3 is implemented as a C++ singleton class with a consistent `setup()` / `tick()` lifecycle interface. The main loop calls `tick()` on each manager every frame, and `setup()` runs once during boot in a fixed order. This pattern appears in DisplayManager, MQTTManager, ServerManager, PeripheryManager, UpdateManager, PowerManager, MenuManager, and GameManager -- 8 singletons total.

The pattern uses `static getInstance()` with a default-deleted constructor, and each class is exposed as an `extern` global reference (e.g., `extern DisplayManager_ &DisplayManager`).

## Instances
- **All managers** in [project-overview](../topics/project-overview.md): Boot sequence initializes managers in dependency order (Periphery -> Server -> Display -> Update -> MQTT)
- **DisplayManager** in [display-manager](../topics/display-manager.md): The largest singleton, managing LED matrix, app loop, notifications, and rendering pipeline
- **GameManager** in [games](../topics/games.md): Delegates to per-game singletons (SlotMachine_, AwtrixSays_), creating a tree of singletons

## What This Means
The singleton pattern provides simplicity -- any manager can call any other manager directly without dependency injection or event bus complexity. On an embedded system with 520KB RAM, this is a pragmatic choice. However, it creates tight coupling: every manager implicitly depends on every other manager's global state via `Globals.h`. The 80+ global variables in `Globals.h` act as a shared state bus, making it difficult to reason about state changes in isolation. This architecture works well for a single-developer project at this scale but would be a refactoring bottleneck if the codebase grew significantly.

## Sources
- [project-overview](../topics/project-overview.md)
- [display-manager](../topics/display-manager.md)
- [mqtt-integration](../topics/mqtt-integration.md)
- [http-server](../topics/http-server.md)
- [periphery-hardware](../topics/periphery-hardware.md)
- [games](../topics/games.md)
- [ota-updates](../topics/ota-updates.md)
