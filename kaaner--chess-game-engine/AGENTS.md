## Purpose

This file gives concise, actionable guidance to AI coding assistants working on the Antigravity Chess project. Focus on the actual code layout, data shapes, build/debug commands, and concrete places to change behavior (not general style advice).

## Quick start / developer workflow

- Install and run the dev server: this project uses Vite.
  - npm install
  - npm run dev   # starts vite dev server; index.html loads /src/main.js
- Build for production: npm run build
- Preview production build: npm run preview

package.json (root) contains these scripts and lists `vite` as a devDependency.

## High-level architecture (why / what)

- Single-page browser app using ES modules (package.json `type: "module"`).
- Clear separation of concerns:
  - `src/game/Board.js` — board state and cloning/mutations.
  - `src/game/Piece.js` — piece types and color enums, Piece class.
  - `src/game/Rules.js` — all move generation logic (pseudo-legal + legal filtering).
  - `src/game/GameState.js` — game lifecycle: turn, history, makeMove(), game-over detection.
  - `src/main.js` — UI bootstrap and simple example usage (creates GameState and writes to DOM).

This separation means changes to move logic should happen in `Rules.js`; UI changes in `main.js` / assets.

## Project-specific conventions and patterns

- Board coordinates: rows 0..7, cols 0..7. Row 0 is Black's back rank; Row 7 is White's back rank.
- Piece encoding: `PieceType` uses lowercase letters (`'p','r','n','b','q','k'`) and `PieceColor` uses `'w'|'b'` (see `src/game/Piece.js`).
- Move flow:
  1. `GameState.makeMove(fromRow, fromCol, toRow, toCol)` (validates turn and legality)
  2. `Rules.getLegalMoves(board, row, col)` returns legal moves by simulating pseudo-moves and using `isSquareAttacked` to filter.
  3. If legal, `Board.movePiece()` executes and `GameState` updates history and switches turn.
  4. `GameState.updateGameStatus()` checks for checkmate/stalemate using `Rules.isSquareAttacked`.
- Board cloning: `Board.clone()` creates fresh Piece objects — used by `Rules.getLegalMoves` to simulate moves.

## Integration points & where to change behavior

- Add or modify piece movement rules: edit `src/game/Rules.js`.
  - Pseudo-legal moves are generated in `getPseudoLegalMoves`; check and then simulate in `getLegalMoves`.
- Change board representation or persistence: `src/game/Board.js`.
- UI rendering or controls: `src/main.js` and `index.html` (the app mounts to the element with id="app"; a placeholder container with id="chess-board" is included).

## Data shapes / examples

- Piece: `{ color: 'w'|'b', type: 'p'|'r'|'n'|'b'|'q'|'k', hasMoved: boolean }`
- Move history entry: `{ from: { r, c }, to: { r, c }, piece }`
- Example usage (from `src/main.js`):
  - `const game = new GameState(); game.makeMove(6,4,4,4);`  // e2 -> e4 in friendly coordinates

## Performance notes (observed patterns)

- `getLegalMoves` clones the board for every pseudo-move to check king safety. This is simple and correct but may be a hotspot if you add AI or heavy search. Profile or replace with incremental move/undo if optimizing.

## Files to inspect first when working here

- `src/game/Rules.js` — rules and move generation
- `src/game/GameState.js` — game flow and state transitions
- `src/game/Board.js` — board layout, clone, movePiece, findKing
- `src/main.js` & `index.html` — how the app boots and where UI hooks mount

## What I won't assume

- There are no tests or CI configuration discovered — don't assume test frameworks exist. Keep changes small and manual-run friendly.

## If you need to change AI/engine integration

- Add a separate engine module (e.g., `src/engine/`) that reads `GameState.getBoard()` and returns moves; keep engine pure (no DOM access).

---
If any of these sections are unclear or you'd like more examples (e.g., pinpoint modifications to implement castling, en-passant, or add a minimax AI), tell me which area and I will iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaaner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
