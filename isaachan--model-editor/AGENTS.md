# Copilot instructions for model-editor

Purpose: Help Copilot or other AI assistants quickly understand how to build, run, and modify this repository and follow its code conventions.

## Quick commands (project root)
- Install deps: npm install
- Dev server: npm run dev  # starts Vite on http://localhost:5173
- Build: npm run build     # runs `tsc -b && vite build`
- Preview build: npm run preview
- Lint: npm run lint       # eslint .
- Format: npm run format   # prettier --write

Note: There is no `test` script in package.json. The architecture doc recommends Vitest for testing; example single-test runs if added:
- npx vitest run path/to/test.spec.ts
- npx vitest path/to/test.spec.ts --run

## High-level architecture (summary)
- Frontend-only single-page app (React + TypeScript) built with Vite.
- Canvas rendering via Konva (react-konva). UI components in `src/components/`.
- State management via Zustand: three main stores expected in `src/store/` — `useDiagramStore`, `useEditorStore`, `useHistoryStore`.
- Data model separation: semantic data (diagram elements, metadata) vs layout data (x,y,width,height). Semantic data is authoritative and persisted; layout is for rendering only.
- Persistence: localStorage-based auto-save (debounced). History store implements undo/redo with a bounded stack.
- Entry points: `src/main.tsx` (boot) → `src/App.tsx` (root component).
- Build outputs: `dist/` (vite build). TypeScript project references via tsconfig.app.json / tsconfig.node.json.

## Key conventions and patterns
- Directory layout: keep top-level features under `src/{components,store,models,utils,hooks,styles}` following ARCH_DESIGN.md.
- Stores:
  - useDiagramStore: holds diagram semantic data and actions (add/update/delete/setElements/clearAll).
  - useEditorStore: UI/editor transient state (tool mode, selection, zoom/pan, grid/snap flags).
  - useHistoryStore: push/undo/redo with maxSize (default ~100) and debounce/merge rules for drag operations.
- IDs: use `nanoid` for unique element IDs.
- Semantic vs Layout: exportable/serializable diagram payload should exclude layout fields when possible; history snapshots store both.
- Rendering: Konva layering strategy — only re-render affected layers. Canvas layers like Grid, Type, Relation, Note, Selection are used to optimize drawing.
- Geometry utilities: `src/utils/geometry.ts` and `cardinality.ts` implement deterministic math; keep them pure and unit-testable.
- Commits: repository uses Conventional Commits (see ARCH_DESIGN). Use `feat(ME-XXX): ...`, `fix(ME-XXX): ...`, `docs: ...`.
- Stories: map tasks to `docs/stories.csv` (story-driven development in ARCH_DESIGN.md).

## Where to look for authoritative docs in this repo
- README.md — developer quickstart and tech stack.
- docs/ARCH_DESIGN.md — full architecture, state/store interfaces, rendering and testing choices (Vitest, Playwright suggested for E2E).
- docs/prd.md and docs/schema/ — product requirements and the diagram schema.

## AI/assistant-specific files checked
Checked for common AI assistant instruction/config files (CLAUDE.md, .cursorrules, AGENTS.md, .windsurfrules, CONVENTIONS.md, AIDER_CONVENTIONS.md, .clinerules) — none were detected to incorporate.

---


## Common code search keywords
- useDiagramStore, useEditorStore, useHistoryStore (Zustand stores in src/store/)
- geometry.ts, cardinality.ts (geometry/math utils)
- Konva Layer, Canvas, TypeNode, RelationLine, GeneralizationBox, StickyNote (canvas rendering components)
- exportStage, exportSvg (export utilities)
- stories.csv (user stories, story-driven development)
- ARCH_DESIGN.md, prd.md, VISUAL_DESIGN.md (architecture, requirements, visual design)

---

Created by Copilot-guided audit. For follow-ups: want an MCP server configured for Playwright E2E testing or other CI helpers?

---
> Source: [isaachan/model-editor](https://github.com/isaachan/model-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-03 -->
