# Copilot Instructions for FEDF Frontend

## Project Overview
FEDF Frontend is a React + TypeScript + Vite educational dashboard application. It tracks user learning activities, maintains streaks, displays skills, and provides insights. The app uses React Router for client-side routing with authentication-protected routes via `AuthContext`.

## Architecture & Key Patterns

### Authentication & State Management
- **AuthContext** (`src/context/AuthContext.tsx`) is the single source of truth for user authentication
- Uses React Context API (no Redux) for simplicity; persists user to `localStorage` under `'ghostwrite_user'` key
- Protected routes wrap authenticated pages via `<ProtectedRoute>` component that redirects unauthenticated users to `/signin`
- Auth methods: `signUp()`, `signIn()`, `signOut()` - currently use simulated delays, replace with real API calls when backend is ready

### Routing & Page Structure
- **App.tsx** configures all routes with BrowserRouter; pages live in `src/app/` following a file-based routing convention
- Protected routes: `/dashboard`, `/profile/:username`, `/quiz`, `/settings`
- Public routes: `/` (landing), `/signin`, `/signup`
- Dynamic routes use `[param]` naming (e.g., `[username]` for user profiles)

### Data Flow & API Layer
- **api.ts** (`src/services/api.ts`) exports mock data functions; currently returns hardcoded objects with 1-second delays
- Dashboard page loads data with `Promise.all()` for 4 concurrent API calls: `getDashboardStats()`, `getActivityData()`, `getSkills()`, `getInsights()`
- When integrating real backend, replace mock functions in `api.ts` with actual HTTP calls (fetch or axios)
- Types centralized in `src/types/dashboard.ts`; import from here for consistency

### Component Patterns
- **Dumb components** (ActivityChart, ConsistencyCard, SkillChart, InsightCard) accept data as props and focus on rendering
- **Smart components** (Navbar, Sidebar) manage state and hook into AuthContext
- **Exports** centralized via `src/components/index.ts` for cleaner imports
- Use CSS variables for styling (defined in `App.css`): `--accent-primary`, `--text-secondary`, `--space-2xl`, etc.

### TypeScript & Type Safety
- Types are strict; all API responses must match types in `src/types/dashboard.ts`
- Use `type` for data structures, keep interfaces minimal
- Page components default export pattern: `export default function PageName() { ... }`

## Development Workflow

### Build & Run
```bash
npm run dev        # Start dev server with HMR on localhost:5173
npm run build      # TypeScript compile + Vite build → dist/
npm run lint       # ESLint check
npm run preview    # Preview production build locally
```

### Key Commands
- ESLint checks both TypeScript and React; fix with `npm run lint` (may require `--fix` flag for auto-fixes)
- TypeScript config split: `tsconfig.json` (base), `tsconfig.app.json` (app), `tsconfig.node.json` (build tools)
- HMR works automatically; no manual refresh needed during `npm run dev`

## Project-Specific Conventions

### Naming & Organization
- Components use PascalCase (e.g., `ActivityChart.tsx`)
- Types use singular forms (User, Skill, DashboardStats)
- Page files are `page.tsx` following Next.js convention
- Utility services go in `src/services/`, contexts in `src/context/`

### Styling Approach
- CSS variables in `App.css` for consistent theming; **do not hardcode colors**
- Inline styles for dynamic behavior (e.g., hover effects with `onMouseEnter`/`onMouseLeave`)
- Use CSS classes for static styling, reserve inline styles for event-driven changes

### Loading & Error States
- Loading states managed locally in component (e.g., `setLoading(true)` on mount in `DashboardPage`)
- Show "Loading..." message to user during data fetch; use `loading` state to guard render
- No global error boundary yet; add error handling to API calls as app grows

## Integration Points & Dependencies

### External Libraries
- **react-router-dom** v6: client-side routing with hooks like `useNavigate()`, `useParams()`
- **React** v19: latest features, hooks only (no class components)
- **Vite**: fast dev server, instant HMR, tree-shaking at build time
- **TypeScript** 5.9: strict mode enabled; use `!` sparingly (null assertions)

### Backend Integration
- Currently all API calls in `api.ts` use mock data with `setTimeout()` delays
- When backend is ready, replace function bodies with `fetch()` or axios calls
- Maintain same function signatures and return types to avoid breaking page components
- Consider auth token storage (JWT in localStorage or httpOnly cookie strategy)

## Common Tasks

### Adding a New Page
1. Create `src/app/newpage/page.tsx` (or `[param]/page.tsx` for dynamic routes)
2. Import in `App.tsx` and add `<Route>` entry
3. Wrap with `<ProtectedRoute>` if authentication required
4. Import data from `api.ts` and types from `src/types/dashboard.ts`

### Adding a New Component
1. Create file in `src/components/ComponentName.tsx` with named export
2. Add to `src/components/index.ts` exports
3. Use in pages: `import { ComponentName } from '../../components'`
4. Accept data as props; minimize internal state

### Integrating Real API
1. Identify endpoint requirements from `api.ts` mock functions
2. Replace mock function body with `fetch()` or axios call
3. Handle errors with try-catch; surface to user via loading/error state
4. Test with real backend; verify response types match TypeScript interfaces

## Avoid Common Pitfalls
- **Don't hardcode theme values**: use CSS variables (`var(--accent-primary)`)
- **Don't bypass AuthContext**: always use `useAuth()` hook for auth state, not localStorage directly
- **Don't mix pages with layouts**: layout components (Navbar, Sidebar) stay separate from pages
- **Don't forget type imports**: use `import type { ... }` for TypeScript-only imports to reduce bundle size

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kiran8658)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Kiran8658)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
