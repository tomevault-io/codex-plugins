# Audio-Programming Project Instructions

## Project Overview
This project simulates the Yamaha DX7 FM synthesizer using SuperCollider, with Python used to parse original DX7 SysEx (`.SYX`) patch files. The goal is to recreate the authentic sound of the DX7 by accurately mapping its internal parameters to SuperCollider's synthesis engine.

## Architecture & Data Flow
1.  **Input**: Binary `.SYX` files (Yamaha DX7 SysEx).
2.  **Parsing (Python)**: `syx_parser.py` reads `.SYX` files, decodes the 128-byte voice data, and converts DX7-specific values (0-99) into "Real Units" (Hz, Seconds, dB, Radians).
    - **Output**: A JSON file containing patch metadata, a pre-calculated 6x6 modulation matrix, and operator parameters.
3.  **Synthesis (SuperCollider)**: `project.scd` defines a `\subSynth` SynthDef that implements the 6-operator FM engine.
    - **Input**: Receives patch data via OSC (`/update_synth`) or direct instantiation.
    - **Engine**: Uses `SinOsc`, `EnvGen`, and `LocalIn`/`LocalOut` (for feedback loops).

## Key Components

### Python Parser (`syx_parser.py`)
- **`parse_voice_to_spec`**: Main logic. Converts raw bytes to a structured dictionary.
- **`get_dx7_algorithm`**: Crucial function. Maps the DX7 Algorithm ID (1-32) to a **6x6 Wiring Matrix** (flattened to 36 floats) for SuperCollider.
    - **Matrix Logic**: `Row = Destination (Carrier)`, `Col = Source (Modulator)`.
    - **Values**: Modulation Index in Radians (approx 0.0 - 12.57).
- **`dx7_level_to_amp` / `dx7_rate_to_duration`**: Helper functions for value conversion.

### SuperCollider Engine (`project.scd`)
- **`\subSynth`**: The FM Voice.
    - **`opWiringMatrix`**: 36-float array controlling FM routing and depth.
    - **`opEnvLevels` / `opEnvTimes`**: Flattened arrays for 6 operators * 4 envelope stages.
- **OSC Listeners**:
    - `/gate`: Note On/Off (1/0).
    - `/update_synth`: Expects a large array of arguments to update the current synth parameters.

## Developer Workflow
1.  **Parse Patches**:
    ```bash
    python syx_parser.py "path/to/patches.SYX" "output.json"
    ```
2.  **Run Synth**:
    - Open `project.scd` in SuperCollider (or VS Code with SC extension).
    - Boot server (`s.boot`).
    - Execute the `SynthDef(\subSynth, ...)` block.
    - Execute the `OSCdef` blocks to enable control.

## Conventions & Patterns
- **Operator Indexing**: DX7 uses 1-6. Python converts this to 0-5 internally.
- **Feedback**: Implemented in SC using `LocalIn.ar(6)` and `LocalOut.ar`. The feedback connection is baked into the `opWiringMatrix` by the Python parser.
- **Envelopes**: DX7 envelopes (R1-R4, L1-L4) are converted to SC `Env` objects. Note that DX7 "Rate" is inverse to "Duration" (99=fast/0s, 0=slow).
- **Matrix Flattening**: The 6x6 matrix is always passed as a flat 36-element array to SuperCollider.

## Integration Notes
- **Missing Link**: Currently, `syx_parser.py` outputs JSON, but `project.scd` listens for OSC. An intermediate step (or script update) is required to bridge `output.json` -> OSC Message -> SuperCollider.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PlayCreatively) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
