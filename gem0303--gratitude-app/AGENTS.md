# Gratitude App - AI Coding Agent Instructions

## Project Overview
Client-side React gratitude journal built with React 19, TypeScript, and Vite. No backend, no routing library—state-based page navigation with manual rendering.

## Architecture

### State Management
- **No state library**: All state lives in [App.tsx](../src/App.tsx) and flows down via props
- **Draft persistence**: SessionStorage with 500ms debounce auto-save (see `DRAFT_KEY` in App.tsx)
- **Entries**: Seeded from [entries.json](../src/data/entries.json), managed in App state, newest-first ordering
- Navigation state controls which page component renders

### Component Patterns
Components follow a consistent structure:
```tsx
type ComponentProps = {
  customProp: string
} & HTMLAttributes<HTMLDivElement>  // Extend native HTML props

export function Component({ customProp, className = '', ...props }: ComponentProps) {
  return <div className={`${styles.base} ${className}`} {...props} />
}
```

**Key examples:**
- [Button.tsx](../src/components/Button.tsx) - Variant pattern with HTML button attributes
- [SegmentedControl.tsx](../src/components/SegmentedControl.tsx) - Generic type parameter `<T extends string>`
- [Container.tsx](../src/components/Container.tsx) - Width variants using predefined CSS classes

### Styling System
- **CSS Modules only**: Every component has `.module.css` file with scoped styles
- **Vite config**: Custom `generateScopedName: '[name]_[local]_[hash:base64:5]'`
- **Design tokens**: CSS custom properties defined in [index.css](../src/index.css) (--coral, --sunshine, --width-med, etc.)
- **No utility classes**: Manual composition via `className` prop
- Component styles import: `import styles from './Component.module.css'` → use `styles.className`

### Type System ([types.ts](../src/types.ts))
- Union types for navigation: `type Page = 'Log' | 'Entries' | 'Grains'`
- Core domain types: `Entry`, `DraftEntry`, `EntryTheme`
- Import types with `type` keyword: `import type { Entry } from './types'`

## File Organization
```
src/
├── components/     # Reusable UI components (Button, TextArea, SegmentedControl, EntryCard)
├── pages/          # Page components (Log, Entries, Grains)
├── layouts/        # Layout wrappers (MainView, NavBar)
├── data/           # Static JSON data files
├── styles/         # Global CSS (utilities.css)
└── types.ts        # Central type definitions
```

## Development Workflow

### Running the App
```bash
npm run dev      # Vite dev server (default: http://localhost:5173)
npm run build    # TypeScript check + production build
npm run lint     # ESLint check
```


### Adding New Features
1. **New page**: Add union variant to `Page` type → add component to pages/ → update App.tsx page object
2. **New component**: Create `Component.tsx` + `Component.module.css` in components/
3. **State changes**: Lift state changes to App.tsx, pass callbacks down

### Code Conventions
- **JSDoc headers**: Add `/** Component description */` comments to component files
- **MARK comments**: Use `// MARK: return` for navigation landmarks
- **TODOs**: Left inline where functionality incomplete (e.g., Button variants, maxLength limits)
- **Commented code**: Sometimes left in place (see Log.tsx form fields, App.tsx conditional rendering)

## Critical Patterns

### Page Navigation
No React Router. Page switching via state:
```tsx
const [currentPage, setCurrentPage] = useState<Page>('Log')
// Render based on currentPage value in page object passed to PageTransition
```

### Form Handling ([Log.tsx](../src/pages/Log.tsx))
- Controlled inputs with local state
- Debounced draft saving via `useEffect` with cleanup timer
- Submit handler calls parent `onAddEntry` callback, generates `id` and `createdAt` in parent

### CSS Module Usage
```tsx
import styles from './Component.module.css'
// Apply: className={styles.button}
// Combine: className={`${styles.button} ${styles.primary} ${className}`}
```

## What to Avoid
- Adding state management libraries (keep prop drilling pattern)
- Installing routing libraries (navigation is intentionally state-based)
- Using utility CSS classes (Tailwind, etc.) - use CSS Modules
- Creating context providers (state lives in App.tsx) - but this can be overriden for certain features
- Adding backend/API calls (client-only design)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gem0303)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gem0303)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
