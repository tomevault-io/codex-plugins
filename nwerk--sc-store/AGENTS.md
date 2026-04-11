# SC-Store — Copilot Project Instructions

## Project Overview

SC-Store is a **SuperCollider** audio synthesis and effects project designed for
live performance on embedded Linux (Raspberry Pi with pisound HAT). It provides
real-time synthesizers and audio effects controlled via MIDI hardware.

## Technology Stack

- **Language:** SuperCollider (`sclang` / `.scd` files)
- **Audio Server:** `scsynth` (SuperCollider server)
- **Target Platform:** Linux (Raspberry Pi + pisound HAT), with Windows fallback
- **MIDI Hardware:** MIDI Mix controller, FS-1-WL USB-MIDI footswitch
- **No external dependencies** — all audio synthesis is native SuperCollider

## Repository Structure

```
sc-store/
├── main.scd              # Scramble FFT effect patch
├── main_sustain.scd      # Bouncing ball physical-modeling synth
├── main_tape.scd         # Tape emulation effect patch
├── classes/              # Shared SuperCollider classes
│   ├── inputtracker.scd  # Tartini pitch tracker + onset detection
│   └── master.scd        # Master output volume control
├── config/
│   └── serversetup.scd   # Platform-specific audio server config
├── launchers/
│   └── run-sc.sh         # Generic sclang launcher
├── midi/
│   ├── midiconfig.scd    # MIDI device init + CC bus management
│   └── mappings/         # Per-patch MIDI CC parameter mappings
├── synths/               # SynthDef definitions
│   ├── bouncing_ball.scd # Physical-modeling bouncing ball synth
│   ├── scramble.scd      # FFT bin scramble effect
│   └── tapeemulate.scd   # Tape saturation + reverb effect
├── run-sc-main_tape.sh   # Production launcher (tape patch)
└── run_main_bounce-sc.sh # Production launcher (bouncing ball)
```

## Architecture & Conventions

### Signal Flow

1. **Audio Input** → line-in microphone or instrument
2. **Pitch Tracking** (`inputtracker.scd`) → frequency, amplitude, and transient buses
3. **Synthesis / Effects** → one of three modes (scramble, bouncing ball, tape)
4. **Master Output** (`master.scd`) → stereo audio out

### Bus Architecture

- **CC Bus** (128 channels): all MIDI CC parameters, shared across patches
- **Frequency / Amplitude Buses**: pitch tracker outputs consumed by synths
- **Audio Buses**: inter-synth signal routing (e.g., `bouncingBallBus`)

### Key Coding Conventions

- **SynthDef naming**: lowercase with camelCase (e.g., `\bouncingBallSmooth`, `\tartiniTracker_sigmundStyle`)
- **MIDI mapping files**: one per patch, placed in `midi/mappings/`, named `<patch>_midi.scd`
- **Main patches** load dependencies via relative paths:
  ```supercollider
  ("config/serversetup.scd").loadRelative;
  ("midi/mappings/<patch>_midi.scd").loadRelative;
  ("midi/midiconfig.scd").loadRelative;
  ("synths/<synth>.scd").loadRelative;
  ```
- **CC values** are normalized to `0–1` range for continuous controls
- **Latching CCs** toggle on value `127` (for footswitch triggers)
- **Server boot** happens inside `s.waitForBoot { ... }` blocks
- **Platform detection**: `Platform.case` with `\linux` and `\windows` branches in server config

### What NOT to Do

- Do not introduce external plugins or Quarks without explicit approval
- Do not hardcode absolute paths in `.scd` files — use `loadRelative` and bus references
- Do not change the CC bus size (128 channels) without updating all mapping files
- Do not modify `serversetup.scd` platform branches without testing on both platforms
- Do not remove or rename existing SynthDef names as they may be referenced by external systems

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nwerk)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nwerk)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
