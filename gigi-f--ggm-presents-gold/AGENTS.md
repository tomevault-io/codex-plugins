# Gold Hunter - Game Development Plan

## Project Overview
A top-down 2D action-adventure game where players explore a vast world collecting gold ingots while trading, fighting, and discovering secrets.

## Development Phases

Moved to: docs/development-phases.md

## Development Guidelines
1. Iterative slices first
  - Ship tiny, vertical slices; integrate end-to-end before expanding.
  - Prefer a working, small feature over an incomplete, large one.

2. Modular, composable architecture
  - Keep systems in small modules with narrow responsibilities (inventory, economy, enemies, world, UI).
  - Design for composition over inheritance; prefer plain objects and functions wired together.
  - Define tiny contracts: inputs, outputs, side-effects, and failure modes.

3. Stable boundaries and minimal coupling
  - Isolate cross-cutting concerns behind helpers (e.g., transitions freeze enemies via a single function).
  - Use data-driven configs (JSON/objects) to avoid hard-wiring logic.
  - Pass dependencies explicitly as parameters; avoid hidden globals.

4. Extensibility by convention
  - New enemy types implement the same minimal lifecycle: create -> update -> destroy.
  - New items declare a data shape (type, subtype, stats) that UI and systems can render.
  - Add optional capability flags (e.g., persistentAcrossMaps) for varied behaviors without branching everywhere.

5. Testability and observability
  - Keep pure helpers pure; side-effecting code should be thin wrappers.
  - Prefer deterministic updates (time/delta inputs) so behaviors are sim-testable.
  - Add lightweight logging hooks or event emitters for key actions (purchases, hits, transitions).

6. UI layout discipline
  - Centralize layout metrics (viewport, HUD height, padding) and compute positions; don’t sprinkle constants.
  - Always word-wrap and clamp text to safe bounds; paginate when needed.
  - Block conflicting inputs when a modal UI is open; clean up all UI nodes on close.

7. Performance as a habit
  - Avoid per-frame allocations; reuse objects/arrays where practical.
  - Cap per-frame loops and bail early when not visible/active.
  - Prefer simple math and cheap physics; profile before adding complexity.

8. Data and save safety
  - Validate external/data files against schemas before loading.
  - Version save data; include migrations for breaking changes.
  - Keep save writes atomic and infrequent; batch when possible.

9. Code style and docs
  - Keep module headers with AI-INDEX tags and purpose summaries.
  - Name functions for intent; include JSDoc on public helpers.
  - Update README/docs alongside features; document assumptions and limits.
  - Maintain the AI Index: update `docs/ai/index.json` anchors and tags when modules/functions change.
  - Keep AI-INDEX tags in source headers in sync with actual responsibilities.
  - Reflect any directory structure changes in docs and contributor guides.

10. Feature flags and dev ergonomics
  - Gate WIP systems behind flags; toggle via a central dev config.
  - Provide debug overlays (grid, bounds, state) to speed iteration.
  - Add small smoke tests or scripts to assert no syntax errors after edits.

11. Documentation workflow reminders
  - When adding or moving systems, update:
    - `docs/development-phases.md` if scope/timeline shifts
    - `.github/copilot-instructions.md` guidelines if conventions change
    - `README.md` links and quickstart if dev flow changes
    - `CONTRIBUTING.md` for contributor steps and checklists
    - `docs/ai/index.json` and source AI-INDEX tags for discoverability
  - Assistant progress summaries (Copilot): keep extremely brief; prefer lists and concrete data (files, diffs, counts, statuses); avoid semantic prose; report only deltas.

## Technical Stack
- Phaser 3 for game engine
- JavaScript/ES6+ for programming
- Tiled for map creation
- Asset creation tools (to be determined)
- Version control with Git
- Build system (Webpack/Vite)

## Quality Standards
- Consistent 60 FPS performance
- Cross-browser compatibility
- Responsive design for different screen sizes
- Clean, maintainable code
- Comprehensive documentation
- Intuitive user interface
- Engaging gameplay mechanics

## Appendix: Module Contracts (stubs)

These lightweight contracts make extension safer. Keep interfaces tiny and stable; expand with optional fields/flags rather than breaking changes.

1) Inventory (`src/lib/inventory.js`)
- Responsibilities: store items, equip/unequip, render inventory UI.
- Inputs:
  - addToInventory(scene, item)
    - item: { type: 'weapon'|'shield'|..., subtype: string, name: string, [stats], [size], [color] }
  - equipWeapon(scene, item), equipShield(scene, item)
- Outputs/Side-effects:
  - Updates scene.equippedWeapon/equippedShield, HUD via UIScene.
  - Emits logs/hooks (optional) for analytics.
- Failure modes: inventory full → returns false; invalid index → no-op.

2) Economy (`src/lib/economy.js`)
- Responsibilities: currency specs, wallet management, pricing helpers.
- Inputs:
  - addToWallet(scene, type, amount)
  - spendFromWallet(scene, costPence)
  - getItemPrice(type, subtype)
- Outputs/Side-effects:
  - Updates HUD currency counters.
- Failure modes: unknown currency → no-op; insufficient funds → false.

3) Enemies (`src/lib/enemies.js`)
- Responsibilities: spawn/update/destroy enemies; shared combat helpers.
- Inputs:
  - createEnemy(scene, type, x, y, opts)
  - updateEnemies(scene, time, delta)
  - damageEnemy(scene, enemy, amount, opts?)
- Outputs/Side-effects:
  - Adds sprites to scene.worldLayer/enemiesGroup; applies knockback/stun; kill FX.
- Failure modes: unknown type → warn; missing physics → no-op.
- Extensibility: implement create<Enemy>, update<Enemy>; keep fields: enemyType, health, damage, flags.

4) World/Maps (`src/lib/world.js`)
- Responsibilities: place objects on grid, build map layers, doors, shop.
- Inputs:
  - createMapObjects(scene, opts?)
  - placeObjectOnGrid(scene, gx, gy, type, group?, data?)
- Outputs/Side-effects:
  - Creates containers/sprites, sets collisions/overlaps, wires transitions.
- Failure modes: blocked cell → returns null; unknown type → no-op.
- Extensibility: add new object types via placeObjectOnGrid; keep data-driven flags.

5) UI (`src/scenes/UIScene.js` and dialog builders)
- Responsibilities: HUD, modal dialogs (shop), prompts/toasts.
- Inputs:
  - updateHealthBar(hp, max), updateCurrency(total, copper, silver)
  - Shop dialog: takes list of items with name/type/subtype/price
- Outputs/Side-effects:
  - Blocks conflicting inputs when open; cleans up nodes on close.
- Failure modes: UI scene not active → silently skip updates.
- Extensibility: compute layout from centralized metrics; paginate when overflow.

6) Transitions (`src/lib/transitions.js`)
- Responsibilities: map scroll transitions, input/physics lock, enemy freeze.
- Inputs:
  - scrollTransitionToMap(scene, direction, newMapId, playerX, playerY)
  - beginTransition(scene), endTransition(scene)
- Outputs/Side-effects:
  - Builds new layer, slides containers, re-homes persistent enemies.
- Failure modes: transition already running → ignore.
- Extensibility: keep enemy freeze/unfreeze via a single helper; move/destroy only old-layer entities.

7) Combat (`src/lib/combat.js`)
- Responsibilities: melee swings, shield raise/lower, basic hit logic.
- Inputs:
  - swingMeleeWeapon(scene)
  - raiseShield(scene), lowerShield(scene), updateShieldPosition(scene)
- Outputs/Side-effects:
  - Damages enemies in a front arc with cooldown/knockback; chops bushes; shield blocks.
- Failure modes: no weapon/shield → no-op with log.
- Extensibility: factor hitboxes later; keep shared damage helpers in enemies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gigi-f)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gigi-f)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
