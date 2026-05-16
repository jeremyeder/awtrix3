---
topic: Sound System
type: codebase
last_compiled: 2026-05-16
source_count: 4
status: active
---

# Sound System

## Purpose [coverage: medium -- 4 sources]
AWTRIX 3 supports two sound output methods: a built-in passive buzzer for monophonic RTTTL melodies, and an optional DFPlayer Mini module for MP3 playback. Sounds can be triggered via API calls, attached to notifications, or played automatically at boot and on New Year.

## Architecture [coverage: medium -- 4 sources]
The sound system is part of PeripheryManager with a dedicated MelodyPlayer subsystem for RTTTL parsing and playback.

Key files:
- `src/MelodyPlayer/melody_player.h` / `.cpp` -- buzzer tone generation, note timing
- `src/MelodyPlayer/melody_factory.h` / `.cpp` -- loads RTTTL from files in MELODIES/
- `src/MelodyPlayer/melody_factory_rtttl.cpp` -- parses RTTTL format strings
- `src/MelodyPlayer/melody.h` -- melody data structure
- `src/MelodyPlayer/notes.h` / `notes_array.h` -- musical note frequency tables
- `lib/DFMiniMp3-1.0.7-noChecksums/src/DFMiniMp3.h` -- DFPlayer Mini driver (modified)

Sound output paths:
1. **Buzzer (default)**: GPIO15, plays RTTTL melodies parsed note-by-note
2. **DFPlayer Mini (optional)**: communicates via software serial (GPIO23 RX, GPIO18 TX), plays MP3 files from SD card

## Talks To [coverage: medium -- 2 sources]
- **PeripheryManager** (direct integration) -- `parseSound()`, `playFromFile()`, `playRTTTLString()`
- **Notification system** (JSON `sound`/`rtttl` keys) -- auto-plays when notification is displayed
- **MQTT/HTTP API** (`/sound`, `/rtttl` endpoints) -- direct playback triggers

## API Surface [coverage: medium -- 3 sources]
Playback:
- `POST /api/sound` or MQTT `[PREFIX]/sound` -- `{"sound":"alarm"}` plays RTTTL file from MELODIES folder
- `POST /api/rtttl` or MQTT `[PREFIX]/rtttl` -- plays RTTTL string directly
- DFPlayer: use `/sound` with 4-digit number (e.g., `{"sound":"0001"}`)

Notification integration:
- `"sound": "filename"` -- play RTTTL file with notification
- `"rtttl": "rtttl_string"` -- play inline RTTTL with notification
- `"loopSound": true` -- loop sound for notification duration

Settings:
- `VOL` (0-30) -- volume for buzzer and DFPlayer
- `SOUND_ACTIVE` -- enable/disable sound globally
- `buzzer_volume` in dev.json -- enables volume control for buzzer (not compatible with all tones)

File storage:
- RTTTL files: `/MELODIES/[name].txt` containing RTTTL format strings
- DFPlayer MP3s: `MP3/` folder on DFPlayer's SD card, named `0001.mp3`, `0002.mp3`, etc.

Pre-defined sound constants:
- `DFMINI_MP3_BOOT` ("1"), `DFMINI_MP3_ALARM` ("2"), `DFMINI_MP3_TIMER` ("2"), `DFMINI_MP3_CLICK` ("5"), `DFMINI_MP3_CLICK_ON` ("3"), `DFMINI_MP3_ENTER` ("4")

## Data [coverage: low -- 1 source]
- **Melody struct** -- parsed RTTTL data with note sequence and timing
- **Note frequency tables** -- mapping of note names to frequencies in Hz

## Key Decisions [coverage: medium -- 2 sources]
- **RTTTL format** -- widely available format with thousands of melodies online; monophonic fits the passive buzzer limitation
- **DFPlayer as optional add-on** -- keeps the base hardware simple while allowing MP3 capability for custom builds
- **Boot sound** -- configurable via `bootsound` in dev.json, using an RTTTL melody

## Gotchas [coverage: medium -- 2 sources]
- DFPlayer is only enabled via `dfplayer: true` in dev.json (for awtrix2_upgrade builds)
- MP3 files must follow the 4-digit naming convention in the MP3 folder
- Buzzer volume control (`buzzer_volume` in dev.json) does not work with all tone types
- Long RTTTL strings in notification JSON may exceed the MQTT packet buffer

## Sources
- [src/MelodyPlayer/melody_player.h](../src/MelodyPlayer/melody_player.h)
- [src/MelodyPlayer/melody_factory.h](../src/MelodyPlayer/melody_factory.h)
- [docs/sounds.md](../docs/sounds.md)
- [docs/api.md](../docs/api.md)
