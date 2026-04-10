# Copilot Instructions for Soc Ops

## Pre-commit Checklist
- [ ] **Lint:** `npm run lint` passes
- [ ] **Build:** `npm run build` succeeds (TypeScript + Vite)
- [ ] **Test:** `npm run test` passes (or no new utils added)

---

## Quick Start

**Soc Ops** is a Social Bingo game: 5×5 grid with icebreaker questions, win with 5 in a row.

**Stack:** React 19 + TypeScript, Vite, Vitest, Tailwind v4, GitHub Pages auto-deploy

**Key:** Center square (index 12) = FREE SPACE, always marked.

---

## Architecture

```
App → (useBingoGame hook)
  ├── StartScreen
  ├── GameScreen
  │   └── BingoBoard → BingoSquare[]
  └── BingoModal
```

**State:** `useBingoGame` manages board, marks, localStorage (version: 1), win detection. Components stateless, props-only.

**State Values:** `'start'` | `'playing'` | `'bingo'`

**Win Check:** All 12 lines (5 rows, 5 cols, 2 diagonals) via `checkBingo()` → `getWinningSquareIds()`

---

## File Organization & Conventions

```
src/
  types/          BingoSquareData, GameState, BingoLine
  utils/          generateBoard(), toggleSquare(), checkBingo()
  data/           questions array (24 items), FREE_SPACE constant
  hooks/          useBingoGame (state + persistence)
  components/     App, GameScreen, BingoBoard, BingoSquare, etc.
```

**Type Safety:** Strict nulls, domain types in `types/index.ts`

**Styling:** Tailwind v4 utilities only (no CSS file). Grid: `grid grid-cols-5 gap-1 aspect-square`

**Questions:** `src/data/questions.ts` → shuffled via Fisher-Yates during board generation

---

## Board Lifecycle

1. **Generate:** `generateBoard()` shuffles 24 questions, injects FREE SPACE at center (index 12)
2. **Mutate:** `toggleSquare()` immutably flips `isMarked` (non-free only)
3. **Persist:** localStorage key `bingo-game-state` + `validateStoredData()` type guard
4. **Detect:** `checkBingo()` on each state change → modal on win

**localStorage Format:** `{ version, gameState, board, winningLine }`

---

## Testing & Common Tasks

**Tests:** Vitest globals, setup at `src/test/setup.ts` (jest-dom matchers + cleanup)
- Existing: `bingoLogic.test.ts` (80+ tests)
- Components: no unit tests yet
- Pattern: `*.test.ts` files, mock localStorage with `vi.spyOn()`

**Add Questions:** Edit `src/data/questions.ts` (24+ items), no code changes needed

**Modify Rules:** `checkBingo()` for win logic, `BOARD_SIZE`/`CENTER_INDEX` for board size

**Bug Fixes:** Understand expected behavior in tests → add test for bug → fix → test localStorage round-trip

**Styling:** Direct Tailwind utilities in component files

---

## Workflows

```bash
npm run dev        # Dev server (auto-reload)
npm run build      # tsc -b && vite build → dist/
npm run test       # Vitest (single run)
npm run lint       # ESLint (React + TypeScript)
```

**Env:** Node.js 22+, `VITE_REPO_NAME` for GitHub Pages base path (CI sets this)

**Deploy:** Push to `main` → auto-deploy via GitHub Actions to GitHub Pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VictorMalcov)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/VictorMalcov)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
