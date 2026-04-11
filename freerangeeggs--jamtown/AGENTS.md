# Copilot Instructions for Jamtown (LetterJam Game Tracker)

## Project Overview

React/TypeScript SPA for tracking LetterJam game state. Built with Vite and persists to browser localStorage. No external state management library - uses React hooks and prop drilling.

## Game Context

LetterJam is a cooperative word game where players deduce their secret letters by observing clues from other players. This app tracks:
- **10 rounds maximum** (`MAX_ROUND_LINES`) - typical game length
- **9 letter inputs per round** - one per player position (supports 2-6 players, some positions may be empty)
- **1 guess per round** - player's deduced letter for that round
- **Derived letters section** (7 letters, `MAX_WORD_LENGTH`) - confirmed letters from successful guesses
- **Guess letters section** (7 letters) - working hypotheses for remaining letters
- **Inactive letters** (J/Q/V/X/Z) - not used in the base game

Understanding this context explains why arrays are fixed-size and why the UI has specific input counts.

## Architecture

### State Management Pattern

- **Single source of truth**: `GameSheetV1` type in [GameSheet.ts](../src/GameSheet.ts) defines entire game state
- **Persistence layer**: [Persistance.ts](../src/Persistance.ts) wraps localStorage with type-safe methods (key: `'gameV1'`)
- **Top-level state**: [App.tsx](../src/App.tsx) holds state via `useState` and passes callbacks down
- **Immutable updates**: Always spread existing state, modify fields, then call both `setSheet()` and `saveState()` sequentially

### Component Hierarchy

```
App (state holder)
├── CharacterList (static alphabet, marks J/Q/V/X/Z inactive)
├── WordSection (aggregates 10 RoundLine updates)
│   └── WordLine × 10 (9 letter inputs + 1 guess input per round)
└── LetterSection × 2 (derived letters section + guess letters section)
```

### Data Flow

1. User types in input → component's `onChange` fires
2. Component updates local state (e.g., `setLetters()`)
3. Component immediately calls parent callback with new data
4. Parent reconstructs `GameSheetV1` via spread operator
5. Parent calls `setSheet(newSheet)` then `saveState(newSheet)`

## Critical Conventions

### Fixed-Size Array Pattern

All arrays are pre-sized (see [GameSheet.ts](../src/GameSheet.ts)):
- 10 round lines (`MAX_ROUND_LINES`)
- 7 letters per word (`MAX_WORD_LENGTH`)  
- 9 letters max per round line

Initialize with `.fill()` or `.map()` to ensure consistent indexes.

### Component Props Pattern

Every stateful component uses this signature:

```typescript
type ComponentProps = {
  data: DataType;
  onUpdated: (newData: DataType) => void;  // immediate callback
};
```

Example: [WordLine.tsx](../src/components/WordLine.tsx) receives `line: RoundLine` and calls `onUpdated()` on every keystroke.

### Controlled Inputs + Local State

Components like [WordLine](../src/components/WordLine.tsx) maintain local state for inputs AND sync immediately with parent:

```typescript
const [letters, setLetters] = useState(/* from props */);
// On change: setLetters(new) then props.onUpdated(new)
```

This creates "local state buffer + instant sync" pattern throughout codebase.

### CSS Class Naming

Input styling uses position-based classes:
- `.line1` through `.line9` for round-specific colors (see [App.css](../src/App.css#L38-L78))
- `.singleInput` for letter inputs, `.big` for result sections
- `.darkBg` for guess letters section

## Development Workflow

**IMPORTANT**: This project uses Vite, not Create React App (migrated).

- **Start dev**: `yarn dev` (opens http://localhost:3000 automatically)
- **Build**: `yarn build` (TypeScript compile + Vite build → `dist/`)
- **Preview**: `yarn preview` (test production build locally)
- **Package manager**: Use Yarn exclusively, not npm

## Known Issues

1. **newGame() reload hack**: [App.tsx](../src/App.tsx#L27) uses `window.location.reload()` because `setSheet()` doesn't trigger re-render properly. Root cause unknown - state update should work but doesn't.

2. **Console logging**: Debug logs present in [App.tsx](../src/App.tsx#L43) and [WordSection.tsx](../src/components/WordSection.tsx#L15) for tracking state updates. Not production-ready.

## Key Files

- [App.tsx](../src/App.tsx) - State orchestration, persistence calls, component composition
- [GameSheet.ts](../src/GameSheet.ts) - Type definitions (`GameSheetV1`, `RoundLine`) and `blankSheet` factory
- [Persistance.ts](../src/Persistance.ts) - localStorage abstraction (load/save/clear)
- [WordSection.tsx](../src/components/WordSection.tsx) - Aggregates 10 WordLine updates, demonstrates array mapping pattern
- [WordLine.tsx](../src/components/WordLine.tsx) - Best example of controlled input + callback pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freerangeeggs)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/freerangeeggs)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
