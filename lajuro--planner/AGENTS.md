# Copilot Instructions for Planner Semanal

## Project Overview

This is a **single-page React + TypeScript web app** for creating, customizing, and printing weekly planners. The entire application lives in `src/App.tsx` (~3,250 lines) with a monolithic architecture—no routing, no backend, no component splitting. Portuguese is used for UI labels and data structures.

## Architecture & Key Patterns

### Single-File Component Architecture

- **All logic in `src/App.tsx`**: React component, TypeScript interfaces, template data, icon SVGs, and rendering logic.
- **No component extraction**: Keep additions in the same file unless explicitly breaking apart the architecture.
- **Inline SVG icons**: Custom icon components (e.g., `IconTasks`, `IconMeal`) defined as functional components returning SVG.

### Template System (`PlannerTemplate` interface)

Templates are the core data model. Each contains:

- **Multiple layouts**: `layouts: Partial<Record<Orientation, LayoutDefinition>>` supports `portrait` and `landscape` variants.
- **Sections with CSS Grid positioning**: Each `SectionDefinition` includes `style: CSSProperties` with explicit `gridColumn`/`gridRow`.
- **Section types**:
  - Bullet lists: `bulletLabels?: string[]`
  - Lined notes: `lines?: number`
  - Blank sections (no special properties)
- **Accent styling**: `accent?: SectionAccent` applies visual variants (`'default' | 'muted' | 'highlight' | 'outline'`).

**Example template structure**:

```typescript
{
  id: 'weekly-essentials',
  name: 'Lista semanal',
  tags: ['tarefas', 'familia', 'rotina'],
  icon: 'tasks',
  defaultTitle: 'Lista de Afazeres',
  palette: { accent: '#f7af8b', neutral: '#f6f4f2', ink: '#1e2933' },
  layouts: {
    portrait: {
      gridTemplateColumns: 'repeat(3, 1fr)',
      sections: [
        { id: 'monday', label: 'Segunda', style: { gridColumn: '1 / 2', gridRow: '1 / 2' } }
        // ... more sections
      ]
    }
  }
}
```

### State Persistence with localStorage

- **Storage key**: `STORAGE_KEY = 'planner-studio-preferences'`
- **Stored data**: Template selection, orientation, custom text, zoom settings, and user-created templates (`customTemplates`).
- **Hydration pattern**: Load from `localStorage` on mount via `useEffect`, skip saves until `hydrationRef.current = true` to prevent race conditions.
- **Custom templates**: User-created templates stored as `customTemplates: PlannerTemplate[]` array, IDs prefixed with `'custom-'`.

### CSS Grid Layout Engine

Sections render into a CSS Grid defined by `LayoutDefinition`:

- **Grid template**: `gridTemplateColumns` and `gridTemplateRows` define the grid structure.
- **Section placement**: Each `SectionDefinition.style` contains `gridColumn` and `gridRow` for explicit positioning.
- **Responsive gap**: `gap` property controls spacing between sections.

## Development Workflow

### Running the App

```bash
npm run dev        # Start Vite dev server
npm run build      # TypeScript check + production build
npm run lint       # ESLint with React Hooks + TypeScript rules
npm run preview    # Preview production build
```

### Key Files

- `src/App.tsx`: Main component with all logic
- `src/App.css`: Application styles (1,569 lines) with print media queries
- `src/index.css`: Global resets and root styles
- `vite.config.ts`: Vite config with React plugin

## Code Conventions

### TypeScript Strictness

- **Strict mode enabled**: `strict: true` with additional linting (`noUnusedLocals`, `noUnusedParameters`).
- **No implicit any**: Always type function parameters and return types.
- **Explicit type guards**: Use `typeof` checks and custom guards (e.g., `isOrientation`, `isZoomMode`).

### Naming & Language

- **Portuguese for domain logic**: Template names, section labels, and UI text use Portuguese (e.g., `'Segunda'`, `'Lista de Afazeres'`).
- **English for code**: Variable names, function names, and comments in English.
- **Type naming**: Use descriptive union types (`type Orientation = 'portrait' | 'landscape'`) and interface names with suffix (`PlannerTemplate`, `LayoutDefinition`).

### React Patterns

- **useState for UI state**: All component state managed with `useState`.
- **useMemo for derived data**: Filtered templates, computed styles, and combined template lists use `useMemo`.
- **useEffect for side effects**: localStorage sync, window resize listeners, and scale calculations.
- **useRef for DOM access**: Preview stage (`previewStageRef`) for scale calculations.

### CSS Patterns

- **CSS Variables for theming**: `--accent`, `--neutral`, `--ink` computed from template palette.
- **Print-specific styles**: `@media print` rules hide UI controls and adjust page layout.
- **BEM-like naming**: Class names like `.workspace-shell`, `.topbar`, `.preview-stage`.

## Adding New Templates

1. **Define the template object** in the `templates` array following the `PlannerTemplate` interface.
2. **Specify both orientations** if applicable: `layouts: { portrait: {...}, landscape: {...} }`.
3. **Use Portuguese labels**: Match existing naming conventions (`'Segunda'`, not `'Monday'`).
4. **Include CSS Grid coordinates**: Each section needs `gridColumn` and `gridRow` in `style`.
5. **Test print layout**: Verify rendering at different zoom levels and in print preview.

## Custom Template Creation (Builder UI)

- **Builder state**: `creatorDraft: TemplateCreatorDraft` with sections array.
- **Section types**: `'notes' | 'list' | 'blank'` mapped to `lines`, `bulletLabels`, or neither.
- **Conversion**: `creatorDraft` transforms into `PlannerTemplate` on save with `id: 'custom-${timestamp}'`.
- **Persistence**: Saved custom templates merge into `customTemplates` array and sync to localStorage.

## Common Pitfalls

- **Don't split App.tsx**: The monolithic structure is intentional—keep related logic together.
- **Respect localStorage hydration**: Always check `hydrationRef.current` before persisting state.
- **CSS Grid coordinates are 1-indexed**: `gridColumn: '1 / 2'` spans the first column.
- **Print styles matter**: Changes to layout affect physical printing—test with browser print preview.
- **Portuguese in data, English in code**: Keep this separation consistent across the codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lajuro)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Lajuro)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
