# Copilot Instructions - Kartomat

## Project Overview

Kartomat is a client-side classroom seating solver using Constraint Satisfaction Problem (CSP) algorithms. It's a React 19 + TypeScript app built with Vite 7, using Zustand for state management and i18next for internationalization.

## Build, Test, and Lint Commands

### Package Manager

This project **requires pnpm** (enforced via preinstall script). Using npm or yarn will fail.

```bash
# Install dependencies
pnpm install

# Development
pnpm run dev              # Start dev server on localhost:5173

# Build
pnpm run build            # Production build
pnpm run preview          # Preview production build

# Linting
pnpm run lint             # Run ESLint

# Type checking
pnpm run tsc              # Run TypeScript compiler

# Testing
pnpm test:unit            # Run Vitest in watch mode
pnpm test:unit:run        # Run Vitest once
pnpm test:unit:ui         # Open Vitest UI

pnpm test                 # Run Playwright E2E tests
pnpm test:ui              # Run Playwright with UI mode
pnpm test:headed          # Run Playwright in headed mode
```

### Running Single Tests

- **Vitest**: `pnpm test:unit -- <filename>` or use `.only` in test files
- **Playwright**: `pnpm test -- <spec-file>` or `pnpm test -- --grep "test name"`

## Architecture

### State Management - Zustand Slices Pattern

State is organized into domain slices (`src/store/`):

- `studentsSlice.ts` - Student roster management
- `classroomSlice.ts` - Grid dimensions and layout
- `constraintsSlice.ts` - Seating constraints
- `solverSlice.ts` - Solution state and error messages

**Slices are combined** in `src/store/index.ts` using Zustand middleware:

- `persist` middleware for localStorage persistence
- Only specific fields are persisted (see `partialize` config)
- localStorage key: `kartomat-storage`

### Feature-Based Organization

Features are co-located in `src/features/` with their components and logic:

```
features/
  ├── students/        # Student management UI
  ├── classroom/       # Grid configuration UI
  ├── constraints/     # Constraint creation/editing UI
  ├── seating/         # Solution visualization
  └── solver/          # Solve button and solver invocation
```

Each feature exports through `index.ts` for clean imports.

### CSP Solver Architecture

The constraint solver (`src/cspSolver.ts`) implements:

- **Backtracking algorithm** with Most Constrained Variable (MCV) heuristic
- **Constraint types** defined as const object (`CONSTRAINT_TYPES`)
- **Smart placement order**: Absolute constraints → most constrained students → least constrained
- **Constraint validation**: All constraints checked after each placement

The solver is **pure** - no side effects, only returns `SeatingResult` object.

### Internationalization

i18next setup in `src/i18n/`:

- Translation files in `locales/` (JSON format)
- Language switching persisted to localStorage
- Use `useTranslation()` hook in components, `t()` function for keys

## Key Conventions

### TypeScript Usage

- **Strict mode enabled** in tsconfig.json
- Use explicit types for constraint interfaces (see `cspSolver.ts`)
- TypeScript-first: all new files should be `.ts` or `.tsx`

### Component Patterns

- Functional components only (React 19)
- Use Zustand selectors: `useStore((state) => state.fieldName)`
- Keep components focused on presentation; business logic in solver or slices

### Constraint Type System

When adding/modifying constraints:

1. Add type to `CONSTRAINT_TYPES` const object
2. Create corresponding TypeScript interface extending `BaseConstraint`
3. Add to discriminated union: `export type Constraint = ...`
4. Update solver's constraint checking logic
5. Update UI constraint manager

### Testing Structure

- **Unit tests** (`*.test.ts` in `src/`): Test CSP solver logic, constraints, algorithms
- **E2E tests** (`tests/*.spec.ts`): Test user workflows per feature/constraint type
- Playwright runs against dev server (auto-started via `webServer` config)
- Two browser configs: `chrome-desktop` (1280x720) and `chrome-mobile` (Pixel 5)

### Styling

- **Tailwind CSS 4** via PostCSS
- Responsive-first: mobile → tablet → desktop breakpoints
- Utility classes in JSX, minimal custom CSS

### ESLint Configuration

- Flat config format (`eslint.config.js`)
- Separate rule sets for `.js/.jsx` and `.ts/.tsx` files
- `varsIgnorePattern: '^[A-Z_]'` - allows unused constants/types starting with uppercase or underscore

### Security

- 1-week package release cooldown (`minimumReleaseAge: 2880` in pnpm-workspace.yaml)
- Protects against supply chain attacks on newly published packages

## MCP Servers

This project is configured with MCP servers (`.mcp.json`) for enhanced tooling:

- **playwright** - Browser automation, test debugging, trace analysis
- **react** - Component inspection, hooks debugging, React DevTools integration
- **axe** - Accessibility testing with axe-core for WCAG compliance
- **tailwindcss** - Utility class suggestions and design token reference

MCP servers run on-demand via `npx -y` to avoid adding dependencies to package.json.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/almyy)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/almyy)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
