# Night City Stock Exchange - Copilot Instructions

## Project Overview

Cyberpunk-themed single-player stock market simulation game. Built with TanStack Start (file-based SSR), React 19, Drizzle ORM (SQLite), Zustand state, and Tailwind v4 + shadcn/ui.

## Critical Architecture Patterns

### Route/Feature Separation (MANDATORY)

Routes (`src/routes/`) handle ONLY routing concerns:

- Auth guards (`beforeLoad`)
- Data loading (`loader` functions)
- Route configuration
- Import feature components from `src/features/`

Example pattern:

```tsx
// src/routes/dashboard.tsx
import { createFileRoute, redirect } from '@tanstack/react-router'
import { Dashboard } from '@/features/dashboard'
import { useAuthStore } from '@/lib/auth-store'

export const Route = createFileRoute('/dashboard')({
  beforeLoad: ({ location }) => {
    if (!useAuthStore.getState().isAuthenticated) {
      throw redirect({ to: '/login', search: { redirect: location.href } })
    }
  },
  component: Dashboard, // Feature component
})
```

Feature components (`src/features/`) contain ALL UI logic, hooks, and presentation code. See existing files: `dashboard.tsx`, `login.tsx`, `portfolio.tsx`, `market-listing.tsx`, `company-detail.tsx`.

### Server Functions (TanStack Start Pattern)

All server functions are centralized in `src/lib/serverFn/` for easy tracking:

```tsx
// Server function definition in src/lib/serverFn/getActiveStocks.ts
import { createServerFn } from '@tanstack/react-start'

export const getActiveStocks = createServerFn({ method: 'GET' }).handler(
  async () => {
    const { getActiveStocksFromDb } = await import('@/lib/db-queries.server')
    return getActiveStocksFromDb()
  },
)
```

```tsx
// Usage in hooks (src/hooks/useGetActiveStocks.ts)
import { getActiveStocks } from '@/lib/serverFn'

export function useGetActiveStocks() {
  return useQuery({
    queryKey: ['activeStocks'],
    queryFn: () => getActiveStocks(),
  })
}
```

**Critical**: All database queries MUST be in `src/lib/db-queries.server.ts` and dynamically imported to avoid bundling server code in client.

### Config Boundary (SECURITY)

`src/lib/config.ts` exports TWO configs:

- `clientConfig`: Safe for browser (validated with Zod). Import anywhere.
- `serverConfig`: Contains secrets. Import ONLY in `.server.ts` files or server functions.

**Never** expose `serverConfig` in client components or routes.

### State Management

- **Auth**: `src/lib/auth-store.ts` - Zustand with `persist` middleware (localStorage)
- **Market/Portfolio**: Create similar stores in `src/lib/` with Zustand
- Access in routes: `useAuthStore.getState()` for synchronous checks in guards
- Access in components: `useAuthStore()` hook for reactive state

### Database Layer

- **Schema**: `src/db/schema.ts` - Drizzle ORM SQLite tables
- **Queries**: `src/lib/db-queries.server.ts` - All database operations
- **Seeding**: `src/db/seed.ts` - Initial company/price data
- **Types**: Use `createSelectSchema` from `drizzle-zod` for type-safe validation

Static company metadata (logos, affiliations) stored in `src/lib/company-data.ts`, synced with DB via seed script.

## Import Order (ESLint Strict)

```tsx
// 1. External dependencies (alphabetical)
import { createFileRoute } from '@tanstack/react-router'
import { ArrowUpIcon } from 'lucide-react'

// 2. Internal @/ imports (alphabetical)
import { Dashboard } from '@/features/dashboard'
import { useAuthStore } from '@/lib/auth-store'
```

**Array types**: Use `Array<string>` not `string[]` (ESLint rule).

## Commands

```bash
npm run dev          # Dev server on :3000
npm run check        # Prettier + ESLint auto-fix
npm test             # Vitest unit tests
npm run lint         # ESLint only (no fix)
```

## Key Files Reference

- `src/routes/__root.tsx` - App shell, QueryClient provider, layout
- `src/components/app-navigation.tsx` - Main nav (extracted from root)
- `src/lib/company-data.ts` - Static company metadata
- `vite.config.ts` - TanStack Start, Nitro, Tailwind, tsconfig-paths plugins
- `REQUIREMENTS.md` - Feature checklist (checkboxes track implementation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vossryn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
