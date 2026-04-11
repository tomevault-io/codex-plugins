# Copilot Instructions for tc-animator

## Project Overview

- **Type:** React + TypeScript app, built with Vite
- **Structure:**
    - Main source: `src/` (components, lib, schemas, stores)
    - UI components: `src/components/` (with subfolders for properties, three.js, and UI primitives)
    - State management: `src/stores/` (e.g., `ProjectStore.ts`, `TrainsStore.ts`)
    - Data schemas: `src/schemas/`
    - Utilities: `src/lib/utils.ts`
    - Test data: `test-data/`
    - Tests: `src/App.test.tsx`, `src/test/`

## Build & Test

- **Install:** `pnpm install`
- **Dev server:** `pnpm dev`
- **Build:** `pnpm build`
- **Preview:** `pnpm preview`
- **Test:** `pnpm test` (uses [Vitest](https://vitest.dev/))
- **Lint:** `pnpm lint` (ESLint, see `eslint.config.js`)

## Key Patterns & Conventions

- **Component organization:**
    - ShadCN UI primitives in `src/components/ui/` (e.g., `button.tsx`, `dialog.tsx`)
    - Three.js-related components in `src/components/three/`
    - Properties panels in `src/components/properties/`
- **State management:**
    - Uses Zustand stores in `src/stores/` (not Redux)
    - Stores are TypeScript classes or modules, not React context
- **Data flow:**
    - Project and train data are loaded from schemas in `src/schemas/`
    - Test data in `test-data/` can be used for development
- **Styling:**
    - Global styles in `src/index.css`
    - All other styles are Tailwind CSS
- **Type safety:**
    - All code is TypeScript; prefer strict typing
    - Schemas define data shapes for project/train objects

## Integration & External Dependencies

- **Three.js** is used for 3D rendering (see `src/components/three/`)
- **Vite** for build/dev
- **Vitest** for testing
- **ESLint** for linting (see `eslint.config.js`)
- **pnpm** as the package manager

## Examples

- To add a new UI component, create it in `src/components/ui/` and use it in other components
- To add a new Three.js component, place it in `src/components/three/`
- To add a new store, create a file in `src/stores/` and import it where needed
- To add a new schema, define it in `src/schemas/` and use it for type validation

## Special Notes

- Do not use Redux, Zustand, or Context for state unless refactoring
- Use Tailwind CSS for styling; avoid custom CSS where possible
- Ensure all new code is covered by tests
- Prefer functional React components
- Use strict TypeScript types and interfaces
- Keep test data in `test-data/` up to date with schema changes

---

For more details, see `README.md` and the `src/` directory structure. Use context7 for more up-to-date docs and code examples.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wondervale)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Wondervale)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
