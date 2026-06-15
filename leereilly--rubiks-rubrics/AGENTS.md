# Copilot instructions for Rubik's Rubrics

A self-contained 3D Rubik's Cube visualizer/solver built with three.js. **Everything lives in a single `index.html`** (~1350 lines): inline `<style>`, the HTML UI, an importmap, and one `<script type="module">` holding all logic.

## Running locally

There is **no build step, no package manager, and no tests or linters** — no `package.json` exists. three.js (r160) is loaded from a CDN via the importmap, so the page must be served over HTTP, not opened as `file://`:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

To verify a change, load it in a browser and watch the cube scramble then solve. There is no automated test to run.

## Architecture (the big picture)

- **No solver library.** A solved cube is scrambled by a random sequence of quarter turns; the solution is simply that sequence **reversed and inverted** (`scramble.slice().reverse().map(inverse)` in `setScramble`). Every solve is therefore valid by construction.
- **Logical model drives rendering.** Each cubie is `{ obj, home, pos, quat, stickers }` where `pos` is a grid `Vector3` and `quat` an orientation `Quaternion`. `applyMoveInstant(move)` mutates `pos`/`quat` (rounding `pos` to integers to prevent drift); `renderFromModel()` copies the model onto the three.js objects. Scrubbing/stepping work by replaying instant moves, which keeps the timeline frame-perfect and drift-free — **do not derive cube state from the three.js scene graph.**
- **Move engine.** A move is `{ axis:'x'|'y'|'z', layers:number[], dir:1|-1 }` (one quarter turn of one or more layers, where `layers` holds slice indices `-1|0|1`). `animateMove` does the eased visual turn by reparenting affected cubies under a temporary pivot `Group`, then commits the result with `applyMoveInstant` + `renderFromModel` on completion.
- **Notation round-trips.** `BASE` maps Singmaster tokens → moves; `parseScramble(str)` turns a notation string into moves (throwing a friendly error on bad input) and `moveToNotation(m)` does the reverse. Custom-scramble input and the copy buttons both flow through these.
- The module is organized by banner-comment sections (`// ---` blocks): Renderer/scene/camera, Build the cube, Move engine, Animation, UI wiring, Mobile bottom sheet, theme toggle, Background helper, Resize + render loop, Go. Add new code to the matching section.

## Conventions specific to this repo

- **Singmaster / WCA notation is authoritative.** Follow the standard already encoded in `BASE`: M follows L, E follows D, S follows F; wide turns (`r u f` / `Rw…`) add the adjacent slice; `x y z` rotate the whole cube the way `R U F` turn. The in-app help and `parseScramble` regex must stay consistent with the WCA reference linked in the notation modal.
- **Sticker colors are two parallel 6-entry palettes** indexed by `faceId` (0–5): `CLASSIC` and `GITHUB` (contribution-graph greens). The "GitHub contributions" toggle swaps the active `palette` and recolors every sticker material — keep both arrays the same length and order.
- **Theme** is the `data-theme` attribute on `<html>`, persisted in `localStorage` under `rr-theme`, and applied by an inline script **before first paint** to avoid a flash. Light-mode styling is written as `[data-theme="light"] …` CSS overrides.
- **UI is wired by fixed element IDs** via `getElementById` in the "UI wiring" section. Adding a control = add the markup in `#panel` (or elsewhere in `<body>`) **and** its listener in that section; keep the IDs in sync.
- **three.js is pinned to 0.160.0** in the importmap. Bumping it means editing both `imports` entries (`three` and `three/addons/`) together; addons (`OrbitControls`, `RoomEnvironment`, `RoundedBoxGeometry`, `UnrealBloomPass`, `EffectComposer`) are imported from `three/addons/`.
- Rendering goes through an `EffectComposer` with `UnrealBloomPass`; the render loop calls `composer.render()`, not `renderer.render()`. The background is a generated gradient regenerated on theme change.
- On screens `≤ 640px` the control panel becomes a draggable/swipeable bottom sheet, and the camera frustum is shifted up so the cube stays visible above it — preserve this when touching panel or camera code.

---
> Source: [leereilly/rubiks-rubrics](https://github.com/leereilly/rubiks-rubrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-15 -->
