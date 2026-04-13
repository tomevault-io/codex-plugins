
# YouBeatAI Workspace Instructions

**Project:** YouBeatAI / Ritme Robot — A music-creation app in Godot 4.6 for Dutch children (~10 years old) to make drum loops with a visual beat-ring interface and AI companion.

**Full Documentation:** See [STYLE_GUIDE.md](../STYLE_GUIDE.md) and [Ritme Robot Architecture.md](../Ritme%20Robot%20Architecture.md) for complete conventions and design philosophy.

---

## Quick Facts

| Aspect | Detail |
|--------|--------|
| **Engine** | Godot 4.6 with GDScript (not C#) |
| **Architecture** | Pure event-driven via `EventBus` autoload |
| **Typing** | Static typing mandatory (`func foo(x: int) -> String`) |
| **Indentation** | Tabs (Godot default) |
| **Singletons** | `EventBus`, `GameState`, `SongState`, `FFT` |
| **State Pattern** | `GameState` = runtime/playback; `SongState` = persistent model |
| **Data** | All data classes extend `Resource` (serializable) |
| **Main Scene** | `Scenes/main.tscn` |

---

## Architecture Essentials

### Event-Driven Communication

**No direct coupling.** All inter-system communication flows through:

```gdscript
# Emit signals across the bus
EventBus.beat_triggered.emit(beat_index)
EventBus.section_switched.emit(section_data)
EventBus.recording_started.emit()

# Listen to signals
EventBus.beat_triggered.connect(_on_beat_triggered)
```

### GameState vs SongState

- **`GameState`** (Scripts/Global/game_state.gd) — Runtime state
  - `playing: bool`, `current_beat: int`, `beat_progress: float`
  - `is_recording: bool`, `microphone_volume: float`
  - User settings (clap_bias, metronome, audio_settings, etc.)

- **`SongState`** (Scripts/Global/song_state.gd) — Persistent song model
  - Wraps a live `SongData` resource (saved/loaded from disk)
  - Delegates: `sections`, `bpm`, `total_beats`, `swing`, `song_track`
  - Runtime-only: `current_section`, `selected_track_index`, `selected_soundbank`

### Data Classes (Resource-based)

All persistent data extends `Resource` and lives in `Scripts/DataClasses/`:

```gdscript
extends Resource
class_name SongData

@export var bpm: int = 120
@export var sections: Array[SectionData] = []
@export var song_track: TrackData
```

This makes the entire song tree serializable and testable.

---

## GDScript Conventions

### Static Typing (Always)

```gdscript
# ✅ DO: Explicit types everywhere
func get_section(index: int) -> SectionData:
    var sections: Array[SectionData] = []
    var bpm: int = 120
    return sections[index]

# ❌ DON'T: Implicit types
func get_section(index):
    var sections = []
    return sections[index]
```

### Type-Inferred Variables (`:=` when obvious)

```gdscript
var copy := SynthTrackData.new(index, knob_position)  # Type is obvious
var bus_idx := AudioServer.get_bus_index(bus_name)   # Type is obvious
var value: float = 0.5  # When type is not obvious, use explicit annotation
```

### Constants

```gdscript
const BEATS_AMOUNT_DEFAULT: int = 16
const SILENT_DB: float = -80.0
const TRACK_COUNT: int = 6
```

### Functions

- **Public functions:** `func play_beat() -> void:`
- **Private functions:** `func _calculate_swing() -> float:` (prefix with `_`)
- **Signal callbacks:** `func _on_section_switched(section: SectionData) -> void:` (use `_on_<signal_name>` pattern)
- **One responsibility per function** — keep them short and focused

### Properties with Getters/Setters

```gdscript
var bpm: int:
    get: return data.bpm
    set(value): data.bpm = value
```

### Section Headers (organize large files)

```gdscript
# ── Playback ──────────────────────────────────────────────────────────

func play():
    pass

# ── Recording ─────────────────────────────────────────────────────────

func start_recording():
    pass
```

---

## File & Naming Conventions

| Type | Convention | Location | Example |
|------|-----------|----------|---------|
| Regular scripts | `snake_case.gd` | `Scripts/` subdirs | `beat_manager.gd` |
| **Class files** | `PascalCase.gd` + `class_name` | `Scripts/` subdirs | `SampleTrackPlayer.gd` |
| Scene instances | `snake_case.tscn` | `Scenes/` | `main.tscn` |
| **Prefab scenes** | `PascalCase.tscn` | `Scenes/Prefab/` | `BeatButton.tscn` |
| Resource files | `snake_case.tres` | `Resources/` | `runtime_bus_layout.tres` |
| UIDs | Auto `.uid` files | Alongside source | `beat_manager.gd.uid` |

### Directory Structure

```
Scripts/
├── Global/          # Autoload singletons (EventBus, GameState, SongState)
├── Managers/        # Core game logic (BeatManager, SectionManager, etc.)
├── Audio/           # Audio playback, recording, buses
├── DataClasses/     # Pure data resources (SongData, TrackData, etc.)
├── UI/              # Visual controllers and buttons
├── Klappy/          # Robot companion logic
├── Achievements/    # Tutorial/achievement system
├── ChaosPad/        # Chaos pad mixer
└── soundfont/       # Voice processing, note sequencing

Scenes/
├── main.tscn                     # Main application
├── main_menu.tscn                # Menu screen
├── soundbank.tscn                # Soundbank browser
├── Prefab/                        # Reusable prefabs
├── UI_Components/                 # UI scenes
├── Klappy_components/             # Robot model & animation
└── Work_in_progress_scenes/       # ⚠️ Will be deleted

Experimental/                      # ⚠️ Prototype work — most will be removed
Resources/                         # Audio, soundfonts, textures, icons
```

### Avoid/Don't Reference

- ❌ `Scenes/Work_in_progress_scenes/` — will be deleted
- ❌ `Experimental/` — prototypes, most to be removed
- ❌ `Scenes/OLD/` — legacy scenes

---

## Signal Patterns & EventBus

### Defining Signals (in EventBus)

```gdscript
# Scripts/Global/event_bus.gd
signal beat_triggered(beat_index: int)
signal section_switched(section: SectionData)
signal recording_started()
signal recording_stopped(audio: AudioStream)
signal set_recorded_stream_requested(track_index: int, audio: AudioStream)
signal mute_all_requested(mute: bool)
```

### Emitting Signals

```gdscript
# From any manager or scene
EventBus.beat_triggered.emit(current_beat)
EventBus.section_switched.emit(new_section)
```

### Listening to Signals

```gdscript
func _ready():
    EventBus.beat_triggered.connect(_on_beat_triggered)
    EventBus.section_switched.connect(_on_section_switched)

func _on_beat_triggered(beat_index: int) -> void:
    print("Beat %d triggered" % beat_index)

func _on_section_switched(section: SectionData) -> void:
    print("Section switched to %s" % section.name)
```

### No Direct Manager References

```gdscript
# ❌ DON'T: Direct coupling
beat_manager.play_section(section)

# ✅ DO: Signal-based
EventBus.section_switched.emit(section)
```

---

## Common Development Patterns

### Adding a New Feature

1. **Define data** → Create a new class in `Scripts/DataClasses/` extending `Resource`
2. **Create signals** → Add to `EventBus` if cross-system communication needed
3. **Implement logic** → New manager in `Scripts/Managers/` (or extend existing)
4. **Connect UI** → Add controller/button script in `Scripts/UI/`
5. **Emit & listen** → Use EventBus signals for communication

### Recording Audio (Reference)

See [/memories/repo/voice_recording_architecture.md](/memories/repo/voice_recording_architecture.md) for the complete recording flow.

**Key: TrackRecorder** → MicrophoneRecorder → AudioPlayerManager pipeline:

```gdscript
# User clicks record button
EventBus.recording_sample_button_toggled.emit(true)
  ↓
# TrackRecorder starts recording
EventBus.start_recording_requested.emit()
  ↓
# MicrophoneRecorder records audio
EventBus.recording_stopped.emit(audio)
  ↓
# AudioPlayerManager sets the recorded stream
EventBus.set_recorded_stream_requested.emit(track_index, audio)
```

### Testing Architecture

- Managers are independently testable (no direct coupling)
- Data classes are pure resources (serializable for snapshotting)
- Use `emit()` to simulate EventBus signals during tests

---

## Autoloads (Singletons)

| Name | Path | Purpose |
|------|------|---------|
| `EventBus` | `Scripts/Global/event_bus.gd` | Central signal hub |
| `GameState` | `Scripts/Global/game_state.gd` | Runtime state |
| `SongState` | `Scripts/Global/song_state.gd` | Persistent song |
| `FFT` | `fft/Fft.gd` | Spectrum analysis |

Access anywhere:
```gdscript
EventBus.beat_triggered.emit(beat)
GameState.playing = true
SongState.bpm = 120
FFT.compute_spectrum(data)
```

---

## Build & Project Setup

**Engine:** Godot 4.6 (GDScript)

**Main Scene:** `res://Scenes/main.tscn` (set in project.godot)

**Build Task:** Available in VS Code — run via task `build` (requires Godot executable path configured)

**Audio Setup:**
- Default bus layout: `uid://cc7g4ekfye5tq` (runtime_bus_layout.tres)
- Text-to-speech: Enabled
- Microphone input: Enabled

---

## Key Insights

1. **No direct coupling** — All systems talk via EventBus signals
2. **Static typing everywhere** — Catch errors at edit-time, not runtime
3. **Managers are encapsulated** — Each has one job, one EventBus connection point
4. **Data is serializable** — Everything extends `Resource` for easy save/load
5. **Runtime vs persistent** — `GameState` ephemeral, `SongState` on-disk
6. **Signal naming** — Listeners follow `_on_<signal_name>` pattern
7. **Tab indentation** — Godot default; don't mix spaces

---

## Common Tasks

### Add a new track type setting
1. Create resource class in `Scripts/TrackSettingsRes/`
2. Register in `Scripts/Managers/track_settings_registry.gd`
3. Emit via `EventBus.track_settings_changed`

### Modify beat ring behavior
1. Look in `Scripts/Managers/beat_manager.gd`
2. Emit `EventBus.beat_triggered` at appropriate times
3. Update UI listeners in `Scripts/UI/`

### Add audio effect
1. Create effect profile in `Scripts/Audio/AudioBanks/`
2. Reference in `SoundBankMatrix` resource
3. Apply via track's `EffectProfile` reference

---

## Documentation Links

- [STYLE_GUIDE.md](../STYLE_GUIDE.md) — Detailed code conventions
- [Ritme Robot Architecture.md](../Ritme%20Robot%20Architecture.md) — Design philosophy & project layout
- [Todo.md](../Todo.md) — Feature roadmap

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CatmanJunior)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/CatmanJunior)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
