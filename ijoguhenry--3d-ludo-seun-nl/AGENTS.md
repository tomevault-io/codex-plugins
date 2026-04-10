## Copilot Instructions — ludo-3d-master (concise)

This file contains the most important, discoverable patterns and developer workflows to help an AI agent contribute productively.

## Quick start ✅
- Install: `npm install`
- Dev server: `npm run dev` (Vite)
```instructions
# Copilot Instructions — ludo-3d-master (concise)

Purpose: short, actionable notes to get an AI agent productive in this repo.

## Quick start ✅
- Install: `npm install`
- Dev server: `npm run dev` (Vite, serves on port 3000)
- Build: `npm run build`
- Preview production build: `npm run preview`
- Note: There is **no** test script in `package.json` currently; `playwright` is installed as a devDependency (useful for adding E2E tests).

## Environment 🧩
- Set `GEMINI_API_KEY` in `.env.local` (README references this). Vite exposes it as `process.env.GEMINI_API_KEY` (see `vite.config.ts`).
- `index.html` loads Tailwind via CDN and uses an importmap for runtime ESM—do not assume a Tailwind build step.

## High-level architecture 🔧
- Tech: React + TypeScript + three.js via `@react-three/fiber` + `@react-three/drei`.
- Store: `store.ts` (Zustand) is the single source of truth — contains players, turn flow, dice, AI, and side-effects (audio, timeouts).
- Visual components: `/components` (notable files: `Scene.tsx`, `Board.tsx`, `Piece.tsx`, `UI.tsx`, `Dice.tsx`).
- Geometry/constants: `constants.ts` contains `PATH_MAP`, `HOME_PATHS`, `YARD_POSITIONS`, `PLAYER_CONFIG`, `SAFE_SQUARES`.

## Key domain rules & examples 🔍
- Steps encoding: `-1` = yard; `0–50` = main track; `51–56` = home/goal. TOTAL_STEPS_TO_WIN = `56` (see `store.ts`).
- Global track position example: `globalIndex = (PLAYER_CONFIG[playerIndex].startPos + steps) % 52` (used for captures & positioning).
- Safe squares: `SAFE_SQUARES = [0, 13, 26, 39]` — captures are ignored here.
- AI flow: `handleAITurn()` rolls, waits, scores candidate moves, and calls `movePiece()`; change behavior by editing scoring or wait times in `store.ts`.

## Where to make changes & what to test 🔧
- Rules (moves/captures/win): edit `store.ts` — add unit tests when changing non-trivial logic.
- Board geometry/positions: edit `constants.ts` and re-check `components/Piece.tsx` mapping.
- Audio behavior: `utils/audio.ts` — remember browsers may require user interaction to resume AudioContext.
- UI toggles & debug controls: `components/UI.tsx` exposes `numAI`, `setNumAI`, `setCameraEnabled`, `setAnimationsEnabled`, and `resetGame()`.

## Debugging tips 🐞
- Use `npm run dev` and React DevTools to inspect component props/state.
- Quick programmatic state access: temporarily expose the store in `App.tsx` (e.g., `(window as any).__GAME = useGameStore;`) then call `__GAME.getState()` in console.
- Rendering/animation issues: inspect `useFrame` loops and lerp logic in `components/Piece.tsx` (positioning + hop animation lives here).

## Tests & automation 🧪
- Add focused Playwright E2E tests for: rolling a dice, selecting/moving a piece, capture flow, and the winner modal.
- For rule changes, prefer unit tests (extract pure logic from `store.ts` where helpful) and a short E2E to validate visible behavior.

## Conventions & gotchas ⚠️
- `store.ts` mixes core rules and side-effects (sound/timeouts). Refactor carefully and add tests during changes.
- `PATH_MAP` is handcrafted and may be brittle — changing it requires coordinated updates to `HOME_PATHS` and `Piece` mapping.
- Tailwind and imports are loaded at runtime; offline/CI environments may behave differently.

## PR etiquette for AI agents ✅
- Small, focused PRs; mention files changed and include a screenshot/GIF for UI changes.
- Add tests for non-trivial logic updates; document the UX impact in the PR description.

---
If you want, I can also add a Playwright test template or a short example refactor for extracting pure move validation from `store.ts` — tell me which you'd prefer. 🙌
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ijoguhenry)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ijoguhenry)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
