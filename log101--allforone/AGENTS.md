# Project Guidelines

## Code Style

- **TypeScript & React 19**: Modern functional components with strict typing.
- **Tailwind CSS 4**: Utility-first styling. Check `vite.config.ts` for Tailwind integration.
- **Shadcn UI**: Follow Radix-based patterns in [src/components/ui/](src/components/ui/).
- **Immer**: Use for immutable updates in complex logic (e.g., [src/modules/layout/utils.ts](src/modules/layout/utils.ts)).

## Architecture

- **State Management**: **Zustand** store in [src/modules/store.ts](src/modules/store.ts). Use the `createSelectors` utility in [src/modules/utils.ts](src/modules/utils.ts) to minimize re-renders.
- **Logic Boundaries**:
  - `src/modules/layout/`: Pure layout engines (Sugiyama, Grid).
  - `src/modules/import/`: CSV/JSON normalization logic.
  - `src/modules/custom-nodes-edges/`: React Flow specific extensions.
- **Data Flow**: Data flows from import/store -> Layout Engine (`getLayoutedElements`) -> React Flow renderer.

## Build and Test

- **Commands**:
  - `npm install`: Standard install.
  - `npm run dev`: Launch Vite dev server.
  - `npm run build`: Production build (runs `tsc` first).
  - `npm run test`: Execute **Vitest** unit tests.
- **Testing Patterns**: Unit tests for layout logic are prioritized. See [src/modules/layout/horizontalLayout.test.ts](src/modules/layout/horizontalLayout.test.ts).

## Project Conventions

- **React Flow**: Extensive use of `@xyflow/react`. Refer to [src/modules/custom-nodes-edges/collapsibleNode.tsx](src/modules/custom-nodes-edges/collapsibleNode.tsx) for custom node patterns.
- **Custom Hooks**: Prefer `useStore` for global state (e.g., node types, gaps, layout styles).
- **Tooling**: Built-in devtools panels for layout debugging are in [src/modules/devtools/](src/modules/devtools/).

## Integration Points

- **Visualization**: **@xyflow/react** (React Flow 12).
- **Exporting**: `html-to-image` and `jspdf` for PDF and PNG generation in [src/modules/exportUtils.ts](src/modules/exportUtils.ts).
- **Parsing**: `PapaParse` for CSV handling in [src/modules/import/index.ts](src/modules/import/index.ts).

## Security

- **Data Privacy**: Local processing only. No sensitive data should be sent to external services.
- **Local Files**: Sample datasets are served from `public/` and `src/assets/sampleData/`.

---
> Source: [log101/allforone](https://github.com/log101/allforone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-30 -->
