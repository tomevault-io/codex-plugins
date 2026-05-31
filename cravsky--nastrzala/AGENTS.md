# Nastrzala AI Coding Agent Instructions

These guidelines make AI agents immediately productive in this repo. Focus on existing, observable patterns – do not introduce speculative architectures.

## Project Structure & Purpose
- Monorepo with `nastrzala-app` (React + Vite + 3D visualization) and `nastrzala-be` (Express/TypeScript minimal API).
- Core domain logic: 3D van cargo packing solver in `nastrzala-app/src/solvers` (pure functions except debug logging).
- Data definitions (cargo & vehicles) as JSON in `src/data/{cargo,vehicles}`; each JSON consumed via `index.js` arrays.

## Frontend Conventions
- React entry: `nastrzala-app/src/main.jsx` mounts `App.jsx`.
- Components grouped per feature under `src/components/<FeatureName>` using `Component.jsx` + `Component.module.css` (CSS Modules).
- 3D visualization lives in `VanVisualization/` (likely Three.js via `@react-three/fiber`). Keep solver pure; visualization handles rendering only.
- Prefer TypeScript for solver/core logic (`*.ts/tsx`) while existing UI is JS/JSX; match file extension of surrounding code when adding.

## Solver Architecture (Packing Pipeline)
Stages executed in `solve()` (`solver.ts`):
1. Expand items → pieces (`preprocessing/cargo-expander.ts`). Derives meta (long/heavy/light) using: long when ratio of the two largest dims ≥ 4× (ignore thinnest); density thresholds heavy ≥300 kg/m³, light ≤150 kg/m³ (LIGHT suppressed for pallets/platforms).
2. Priority sort (`preprocessing/priority-sorter.ts`): buckets 0 LONG, 1 VERTICAL, 2 HEAVY, 3 STANDARD, 4 LIGHT/FRAGILE; sort by bucket, then weight desc, then volume desc.
3. Initialize free space (`preprocessing/space-initializer.ts`): start full cargo box, carve wheel arches + `vehicle.obstacles` into multiple FreeBox regions.
4. Trip packing loop (`packing/trip-packer.ts`): iteratively `placePieceInFreeSpace` until no progress; supports multiple trips (`max_trips`).
Placement rules (`placement/`):
- Orientation (`orientation.ts`): vertical flag restricts to indices [0,2]; allowRotations=false restricts to [0]. Else all 6 axis-aligned permutations.
- Free space selection (`free-space.ts`): iterate free boxes sorted by Z,Y,X (low, rear-left first). After placement split consumed box into up to 3 residual boxes (right/front/above).
- Stacking (`stacking.ts`): upper can stack only if lower `stackable && !fragile` and not (upper heavy over lower light). Above-floor placements require full footprint support (aggregate overlap area).

## Purity & Side Effects
- Solver functions are pure (no I/O) except conditional console logging guarded by `DEBUG` constant in `solver.ts`. Preserve this flag logic; avoid introducing async.
- Each `CargoPiece` gets stable `piece_id` as `<cargo_id>#<index>`; maintain this convention when generating new pieces.

## Units & Geometry
- All dimensions currently treated as millimeters (MVP simplification). Vehicle `cargo_space` and obstacle positions/sizes share this unit; do not mix units.
- Anchors `[x,y,z]` always represent minimum corner (rear-left-bottom). Sizes `[dx,dy,dz]` are extents along +X,+Y,+Z.

## Vehicle Space & Obstacles
- Wheel arches defined in vehicle JSON → carved out in initialization. Additional `obstacles` optional. Carving splits intersecting FreeBoxes into slices along each axis; do not mutate algorithm ordering unless required.

## Adding Features Safely
- To add new packing heuristics: implement a new preprocessing stage function and call it inside `solve()` maintaining order before space init unless it depends on free space.
- For new stacking criteria: extend `canStackOn` or support area logic; keep overlap + area validation.
- For optimization (e.g., box merging, packing retries) create pure helpers; never embed mutable global state.

## Scripts & Workflows
Frontend (`nastrzala-app`):
```bash
npm run dev      # Vite dev server
npm run build    # Production build
npm run preview  # Serve built assets
npm run lint     # ESLint (config in eslint.config.js)
```
Backend (`nastrzala-be`):
```bash
npm run dev      # nodemon + ts-node
npm run build    # tsc compile to dist/
npm start        # run compiled server
```
No test suite present; avoid inventing test commands—add tests only if requested.

## Extension Points / Integrations
- External libs: React 19, Three.js stack (`three`, `@react-three/fiber`, `@react-three/drei`). Use their idiomatic component patterns for visualization (e.g., suspense, useFrame) without modifying solver purity.
- Backend currently only health/index endpoints; any solver exposure would require adding an API route returning `SolverResponse` shape from `types/solver-types.ts`.

## Example Usage
```ts
import { solve } from "src/solvers/solver";
const response = solve({ unit: "mm", vehicle, items, max_trips: 2 });
// response.status: "ok" | "partial" | "no_fit"
```

## Code Style & Patterns
- Prefer descriptive function names: verb-noun (e.g., `initializeFreeSpace`, `expandCargoItems`).
- Keep new solver modules under appropriate subfolder (`preprocessing`, `placement`, `packing`) to preserve mental model.
- Avoid one-letter vars; re-use existing naming (e.g., `freeBoxes`, `placements`, `remaining`).

## Logging
- Stage logs use emojis + separators. Maintain formatting for consistency if extending (`console.log("\n📦 STAGE X: ...")`).

Feedback welcome: indicate unclear sections (e.g., units, stacking) or missing workflows to refine.

---
> Source: [cravsky/NaStrzala](https://github.com/cravsky/NaStrzala) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
