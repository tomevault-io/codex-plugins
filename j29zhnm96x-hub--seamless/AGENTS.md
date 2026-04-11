# AI Coding Agent Guidelines for `SeamlessPlayer`

> Status: Static web app for seamless audio loops using Web Audio API.

## Purpose
Enable fast, accurate changes to the gapless loop player with iOS-friendly background/lock playback.

## Project Map
- `index.html`: Minimal UI (Play/Stop/Volume), hidden `<audio id="audioOut" playsinline>`.
- `app.js`: Audio graph, buffer decode/cache, loop point detection, play/stop, file/clipboard import.
- `audio/`: Place loops here (e.g. `your_loop.mp3`). Not required for local file/clipboard loads.
- `README.md`: Run instructions and implementation notes.

## Core Architecture
- `AudioContext` → `GainNode master` → `MediaStreamDestination` → hidden `<audio>` via `srcObject`.
- Loop playback uses a single `AudioBufferSourceNode` with `loop=true`, `loopStart`, `loopEnd`.
- Smoothness via linear ramps on both `master.gain` and a per-loop `loopGain`.
- iOS resilience: audio flows only through `MediaStreamDestination`; do not connect to `audioCtx.destination`.

## Loop Points Algorithm (in `computeLoopPoints`)
- Head/tail threshold scan (~−60 dB) to skip near-silence.
- Add ~2 ms margins around boundaries.
- Within ~10 ms windows at start/end, choose lowest average |amplitude| samples.
- Fallback to full buffer if degenerate.

## Developer Workflows
- Serve locally (pick one):
	- VS Code Live Server: open `index.html` with Live Server.
	- Node: `npx http-server . -p 5173` or `npx serve -l 5173 .` then open `http://localhost:5173`.
- Test iOS behavior on device; ensure first play occurs via user gesture.
- Place an example file at `audio/your_loop.mp3` or load via file picker/clipboard.

## UI/Controls
- `#play` starts playback using the loaded buffer or `audio/your_loop.mp3` if present.
- `#stop` ramps down and stops.
- `#volume` adjusts `master.gain` with a short target time.
- File import via `<input type="file" accept="audio/*">`, drag-and-drop, and clipboard (`navigator.clipboard.read()` or paste event).
- URL input and paste/drop of links supported; loads via `fetch` + `decodeAudioData` (mind CORS).
- Waveform preview canvas (`#wave`) shows buffer and loop markers.
- Settings panel (`#s-threshold`, `#s-margin`, `#s-window`, `#recompute`) tunes loop detection.

## iOS/Background Notes
- Call `audioOut.play()` after a user gesture. Code nudges on `visibilitychange` and resumes context when suspended.
- Background persistence is OS-dependent; this routing maximizes success within platform limits.
- Media Session (`navigator.mediaSession`) is used for metadata and play/pause/stop handlers.

## Coding Conventions
- Keep patches surgical; avoid unrelated refactors.
- No direct connection to `audioCtx.destination` when `MediaStreamDestination` is in use.
- Prefer descriptive function names; avoid large inline comments; document behavior in `README.md`.
- UI styling: mobile-first with `style.css` using CSS variables (`--accent`, `--bg`, etc.), avoid introducing frameworks unless requested.
- Accessibility: maintain `aria-label`, `aria-live` for status elements; preserve large tap targets (≥44px height) on buttons.
- Responsive redraw: call `drawWaveform()` on resize/orientation change to keep waveform crisp on iPhone retina.

## Typical Changes (Examples)
- Add a new control: implement handler in `index.html`, wire in `bindUI()` in `app.js`.
- Tweak loop sensitivity: adjust `threshold`, `margin`, or `win` in `computeLoopPoints`.
- Add alternative source loading: extend `loadBufferFromUrl` or add blob/file decode path.

## What NOT To Do
- Do not introduce frameworks/build tooling without request; this is a static app.
- Do not connect graph to `audioCtx.destination` alongside `MediaStreamDestination`.
- Do not remove the hidden `<audio>` or its `playsinline` attribute.

## PR/Change Report Template
- Intent: brief summary
- Files touched: list
- Verification: how it was served and tested (device/browser)
- Follow-ups: risks, limitations, next steps

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j29zhnm96x-hub)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/j29zhnm96x-hub)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
