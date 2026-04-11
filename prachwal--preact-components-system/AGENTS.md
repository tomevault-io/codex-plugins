# Preact Components System - AI Agent Instructions

## Architecture Overview

This is a Preact component library built with Vite and TypeScript. The codebase follows a modular architecture with clear separation of concerns:

- **Components**: Organized in `src/components/` by category (`common/`, `layout/`, `ui/`)
- **State Management**: Theme context with provider pattern in `src/contexts/` and `src/providers/`
- **Types**: Centralized type definitions in `src/types/`
- **Hooks**: Custom hooks in `src/hooks/`

## Build & Development

- **Build**: `npm run build` (runs TypeScript declarations, CSS compilation, then Vite library build)
- **Dev**: `npm run dev` (Vite dev server)
- **TypeScript**: Uses project references with separate configs:
  - `tsconfig.app.json`: App compilation (noEmit: true)
  - `tsconfig.lib.json`: Library declarations (emitDeclarationOnly: true)
  - `tsconfig.node.json`: Node tooling (Vite config)
- **Styling**: SCSS compiled to `dist/index.css` with source maps
- **Library Output**: ES module + UMD builds in `dist/`

## Component Patterns

- Functional components with explicit TypeScript interfaces
- Props destructuring with default values: `const Component = ({ prop = default }: Props) =>`
- Class name concatenation: `const className = \`base-class ${additionalClasses}\`.trim()`
- Wrapper component pattern: `as?: 'article' | 'section' | 'div'` prop for semantic flexibility
- SVG icons using `currentColor` for theme compatibility
- Theme cycling logic: light → dark → system → light

## Theme System

Custom theme implementation supporting 'light', 'dark', 'system':

- Persists to localStorage with key `'app:theme'`
- Applies `data-theme` attribute to `<html>` and `is-dark` class for system preference handling
- Use `useTheme()` hook from `src/contexts/ThemeContext`

## Responsive Design

- Mobile-first approach with breakpoint constants: `MOBILE_BREAKPOINT = 768`, `TABLET_BREAKPOINT = 1024`
- Responsive sidebar with collapse states based on viewport width
- Touch-friendly hamburger menu with accessibility controls

## Accessibility Patterns

- ARIA labels and controls: `aria-label`, `aria-controls`, `aria-expanded`
- Focus management: programmatic focus on menu open/close
- Keyboard navigation: Escape key handling, Tab trapping
- Semantic HTML with configurable wrapper elements
- Skip links for screen reader navigation

## Navigation & State

- Hash-based routing for section navigation
- Active section tracking via `window.location.hash`
- Responsive sidebar state management with body scroll locking

## Development Workflows

- **Storybook**: `npm run storybook` for component development and testing
- **Documentation**: `npm run docs` generates TypeDoc documentation
- **Type Checking**: `npm run type-check` for strict TypeScript validation
- **Bundle Analysis**: `npm run build:analyze` opens bundle visualizer
- **Testing**: `npm run test:coverage` for coverage reports (77/77 tests passing)

## Testing Setup

- **Framework**: Vitest with @testing-library/preact and @testing-library/jest-dom
- **Test Files**: Colocated with components (e.g., `Button.test.tsx`)
- **Coverage**: Comprehensive component testing with 100% pass rate
- **UI Testing**: `npm run test:ui` launches Vitest UI for interactive testing
  App version injected via Vite's `define` from `package.json` as `__APP_VERSION__` global constant. Access via `useAppVersion()` hook or `APP_VERSION` from `src/config/constants.ts`.

## Key Files

- `src/App.tsx`: Root component wrapped in ThemeProvider
- `src/components/layout/AppShell.tsx`: Main layout with sidebar state management
- `src/providers/ThemeProvider.tsx`: Theme logic and system preference detection
- `src/components/layout/Sidebar.tsx`: Responsive navigation with accessibility features
- `tsconfig.app.json`: React path mapping for Preact compatibility (`"react": ["./node_modules/preact/compat/"]`)
- `index.ts`: Library entry point exporting components

## Documentation & Planning

- `plans/STATUS.md`: Current project status, achievements, and metrics
- `plans/IMPLEMENTATION_SUMMARY.md`: Development summary and accomplishments
- `plans/README.md`: Guide to planning documents and archive
- `plans/archive/`: Completed planning documents for reference

## Build Configuration

- **Vite**: Configured for library mode with external dependencies (`preact`, `preact/hooks`)
- **CSS**: Compiled from `src/index.scss` using Sass
- **Declarations**: Generated from `index.ts` entry point, excluding runtime constants
- **Package**: Published as dual ES+UMD with TypeScript declarations

## Common Patterns

- Component composition over inheritance
- Context + hooks for shared state
- Semantic HTML with configurable wrapper elements
- Strict TypeScript with no unused variables/parameters
- Runtime constants with fallback values for build compatibility
- Responsive breakpoints defined as named constants
- Hash-based navigation with automatic active section detection

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prachwal)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/prachwal)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
