# luminar - AI Coding Agent Guide

**luminar** is a TypeScript/Three.js visualization tool for 2D vector fields with particle flow and bloom effects. It's deployed as both a CLI tool (`npx @brandonlukas/luminar`) and local dev environment.

## Quick Start

```bash
npm install && npm run dev                       # Dev server at localhost:5173
npx @brandonlukas/luminar                        # Launch CLI (drag-drop CSV in browser)
npm run visualize:csv -- --file data.csv         # Dev-mode with CSV upload
```

**CSV format:** 4 columns `(x, y, dx, dy)`. Headers optional, auto-detected. Whitespace or comma-separated.
**CLI interface:** Pre-built static app served instantly via `sirv`. Drag & drop CSV files into browser to visualize. Accepts `--port`, `--host` flags.

## Architecture Overview

**Modular design with clear separation:**
- [src/main.ts](../src/main.ts): ~130-line orchestrator - wires modules, sets up Three.js scene, handles resize/layout
- [src/modules/](../src/modules/): Feature modules with callback-based communication (no direct coupling)
  - `particle-system.ts`: Particle simulation, field sampling, buffer management
  - `field-loader.ts`: CSV→JSON transform, coordinate normalization, auto-calibration
  - `controls.ts`: UI panel - sliders, selects, parameter synchronization
  - `recording.ts`: WebM export with resolution selection
- [src/lib/](../src/lib/): Shared utilities
  - `types.ts`: TypeScript types (`ParticleParams`, `ColorPreset`, `FieldTransform`)
  - `constants.ts`: All configuration constants and defaults
  - `csv-parser.ts`: Shared CSV parser (used by CLI and dev scripts)

**Dual-field visualization:** Main app supports two simultaneous vector fields (A and B) with independent color presets. Layout adapts based on which fields are loaded.

## Core Modules

### ParticleSystem ([src/modules/particle-system.ts](../src/modules/particle-system.ts))

**Purpose:** Encapsulates particle simulation, field sampling, and buffer management.

**Key methods:**
- `init()`: Initialize Float32Array buffers and set up BufferAttributes
- `update(dt, palette)`: Main loop - sample field, update positions, apply colors
- `resizeBuffers(count)`: Dynamically adjust particle count and reallocate buffers
- `reseedLifetimes()`: Randomize particle lifetimes when slider changes
## luminar — AI agent notes

- **What it is:** TypeScript + Three.js 2D vector-field visualizer with bloom. Dual fields (A/B) render side-by-side with independent color presets and optional trails/recording.
- **Run it:** `npm install && npm run dev` (Vite dev at 5173). CLI: `npx @brandonlukas/luminar [--port 5173 --host 0.0.0.0]` serves prebuilt `dist` via `sirv` (no runtime build). Dev CSV upload helper: `npm run visualize:csv -- --file data.csv`.
- **CSV contract:** 4 cols `(x, y, dx, dy)`, commas or whitespace; header auto-skipped. Drag/drop CSV onto left/right half of the canvas to load Field A/B.

## Architecture map

- [src/main.ts](../src/main.ts): ~130-line orchestrator. Builds renderer/camera/composer, instantiates modules, wires callbacks, manages layout via `computeViewOffset()` + `updateLayout()` based on which fields are loaded, and caches color presets for per-frame updates.
- Modules ([src/modules](../src/modules)) communicate only via callbacks and shared `params` object—no cross-imports:
  - `particle-system.ts`: Buffers, spatial grid, sampling, noise, color application, view offsets for dual layout. Resets particles when out of bounds (`WORLD_EXTENT`) or lifetime expires.
  - `field-loader.ts`: Fetches `/vector-field.json` on start; `computeFieldTransform()` scales/centers data into world space; `updateStatus()` updates HUD.
  - `controls.ts`: Builds UI; callbacks drive params, lifetimes, buffer resizing, trails/bloom updates, color preset cache; reset rehydrates defaults and hides advanced panel.
  - `recording.ts`: MediaRecorder wrapper with temporary canvas resize for 1080p/1440p/4K; restores render size after stop; bitrate = `pixelCount * 4` capped at 25 Mbps.
- Shared libs ([src/lib](../src/lib)): `constants.ts` (tuning: `WORLD_EXTENT`, `FLOW_SCALE`, `SPEED_TO_GLOW`, `JITTER`, field border min/max, color presets, `defaultParams`), `types.ts` (ParticleParams, ColorPreset, FieldTransform), `csv-parser.ts` (header-tolerant, comma/whitespace split).

## Key patterns & conventions

- **CLI strategy:** Static-prebuilt serving. Always rebuild `npm run build` before publishing so `dist` is fresh; `/dist` must ship with the package.
- **Shared params:** Single `params` object mutated by controls; modules read directly each frame. Cache color presets in main to avoid lookup in hot paths.
- **Buffer discipline:** After writing Float32Arrays, set `geometry.attributes.*.needsUpdate = true` and call `computeBoundingSphere()` (done in `ParticleSystem.updateBuffers()`).
- **Layout logic:** Dual fields offset with `setViewOffset()`; `computeViewOffset()` accounts for control panel width and window aspect. Clearing a field reseeds lifetimes to avoid stale particles.
- **Noise/field sampling:** Spatial grid cell size auto-derives from data density; sampling searches 3×3 neighboring cells and rejects points beyond `fieldValidDistance` (scaled by transform).
- **TypeScript strictness:** Explicit property declarations (no `constructor(private foo)`). Target ES2022 with `erasableSyntaxOnly`.

## Common edits

- Add a param: update `ParticleParams` + `defaultParams`, add control in `controls.ts`, wire to `main.ts` callback if needed, and read in `particle-system.ts` if simulation-related.
- Tweak visuals: adjust `constants.ts` tunables or `ParticleSystem.update()`/`applyColor()`; bloom via `bloomPass` init in `main.ts` or `ControlPanel.updateBloom()`.
- New color preset: append to `COLOR_PRESETS`; note legacy mixing when key is `luminous-violet`.
- Recording tweaks: edit bitrate/size logic in `recording.ts`; keep restore-on-stop behavior.

## Build/test workflow

- Dev: `npm run dev`; Preview prod: `npm run preview`; Build: `npm run build` (tsc + Vite). CLI entry: [bin/luminar.mjs](../bin/luminar.mjs) (sirv static server).

## Quick gotchas
- `onTrailToggle(enabled)` / `onTrailDecayChange(value)`: Trail controls

- `onClearFieldA()` / `onClearFieldB()`: Clear field buttons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brandonlukas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/brandonlukas)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
