---
topic: Games
type: codebase
last_compiled: 2026-05-16
source_count: 3
status: active
---

# Games

## Purpose [coverage: medium -- 3 sources]
AWTRIX 3 includes a simple game framework with two built-in games: Slot Machine and AWTRIX Says (a Simon-style memory game). Games run as a separate mode that takes over the display from the normal app loop.

## Architecture [coverage: medium -- 3 sources]
Games are managed by the `GameManager_` singleton, which uses a state machine to track the current game. Each game is implemented as its own singleton class.

Key files:
- `src/Games/GameManager.h` / `.cpp` -- game state management, input routing
- `src/Games/SlotMachine.h` / `.cpp` -- slot machine game implementation
- `src/Games/AwtrixSays.h` / `.cpp` -- Simon-says memory game implementation

Game state enum:
```cpp
enum GameState { None, Slot_Machine, AWTRIX_Says };
```

## Talks To [coverage: medium -- 2 sources]
- **DisplayManager** (rendering) -- games render directly to the matrix during their tick
- **PeripheryManager** (button input) -- receives button presses for game control
- **MQTTManager** (controller input) -- `ControllerInput()` accepts external commands, `sendPoints()` publishes scores

## API Surface [coverage: medium -- 2 sources]
- `setup()` / `tick()` -- lifecycle methods
- `start(bool active)` -- enter/exit game mode
- `ControllerInput(const char *cmd)` -- process external game input
- `selectPressed()` -- handle select button during games
- `ChooseGame(short game)` -- switch to a specific game
- `sendPoints(int points)` -- publish game score

The `GAME_ACTIVE` global flag indicates whether a game is currently running.

## Data [coverage: low -- 1 source]
- **Game state** -- current game type and per-game internal state (slot positions, Simon sequence, score)

## Key Decisions [coverage: low -- 1 source]
- **Singleton per game** -- each game is a separate singleton, selected by the GameManager
- **External controller support** -- games can be controlled via MQTT in addition to physical buttons

## Gotchas [coverage: low -- 1 source]
- Games take over the display entirely; the normal app loop is suspended while `GAME_ACTIVE` is true

## Sources
- [src/Games/GameManager.h](../src/Games/GameManager.h)
- [src/Games/SlotMachine.h](../src/Games/SlotMachine.h)
- [src/Games/AwtrixSays.h](../src/Games/AwtrixSays.h)
