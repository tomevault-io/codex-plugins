# GitHub Copilot Instructions

Lumihub / Khaos Kontrol is a high-end CRM/ERP for the wedding industry with an "Industrial Noir" aesthetic. React SPA backed by Supabase (PostgreSQL, Auth, Storage, Edge Functions).

## Stack
- React 18 + TypeScript (strict) + Vite
- Tailwind CSS + Shadcn/Radix UI
- TanStack Query v5 (server state) + Zustand (client state)
- Supabase (PostgreSQL + Auth + Storage + Edge Functions)
- Zod + React Hook Form (validation)
- Sentry + custom `logger` service (observability)

## Package Manager
npm only — never suggest yarn, pnpm, or bun.

## Path Alias
`@/` maps to `src/`. Always use this alias for internal imports.

## Directory Structure

```
src/
  features/       # Domain-driven modules — main business logic
  components/     # Shared components (UI atoms, layouts, AI elements)
  hooks/          # Shared hooks
  stores/         # Zustand stores (UI/client state only)
  lib/            # queryClient, i18n, sentry, env, utils
  integrations/supabase/  # client.ts + generated types.ts
  contexts/       # React contexts (Auth, AI, Language, Analytics)
  services/       # Business services (stripe, logger, analytics, etc.)
  pages/          # Public/marketing pages
  types/          # Shared TypeScript interfaces
supabase/
  functions/      # Deno Edge Functions
  migrations/     # SQL migrations
```

## Feature Modules (`src/features/`)

`admin`, `ai`, `assistant-portal`, `assistants`, `auth`, `calendar`, `clients`, `contracts`, `dashboard`, `financial`, `microsite`, `pipeline`, `portal`, `projects`, `public-booking`

Each follows the same structure: `pages/`, `components/`, `hooks/`, `api/`.

## TypeScript
- No `any`. Define explicit interfaces for all data shapes.
- Before writing Supabase queries, verify columns exist in `src/integrations/supabase/types.ts`.
- Regenerate types after schema changes: `npm run type-gen`

## Data Fetching
- Always use TanStack Query (`useQuery`/`useMutation`) for Supabase data.
- Never use `useEffect` to fetch data.

### Query pattern
```ts
const { data, isLoading, error } = useQuery({
  queryKey: ['resource', id],
  queryFn: async () => {
    const { data, error } = await supabase.from('table').select('*').eq('id', id).single()
    if (error) throw error
    return data
  },
  enabled: !!id,
})
```

### Mutation pattern
```ts
const mutation = useMutation({
  mutationFn: async (payload) => {
    const { data, error } = await supabase.from('table').insert(payload).select().single()
    if (error) throw error
    return data
  },
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['resource'] })
    toast.success('...')
  },
  onError: (error: Error) => {
    logger.error(error, { message: '...', showToast: true })
  },
})
```

## Error Handling & Observability
- Use `logger` service (`src/services/logger.ts`), not `console.error`. Integrates with Sentry.
- All async functions require `try/catch` with toast feedback via `sonner`.
- Critical actions (delete, financial updates) must write to `system_logs` or `audit_logs`.
- Gate debug-only UI behind `import.meta.env.DEV`.

## Components
- All data-driven components require Loading (skeleton), Error, and Empty states.
- All pages use `React.lazy()` + `<Suspense>` for code splitting.

## Auth & Roles
- `useAuth()` — current user
- `useRole()` — checks `makeup_artists` and `assistants` tables
- `usePlanAccess()` — feature flags and usage limits via Supabase RPC
- `<ProtectedRoute>` / `<AdminRoute>` for route protection

## Supabase
- Import client from `@/integrations/supabase/client`
- RLS enabled on all user-data tables
- Edge Functions are Deno/TypeScript, invoked via `supabase.functions.invoke()`
- Two environments: dev (`nqufpfpqtycxxqtnkkfh`) and prod (`pymdkngcpbmcnayxieod`)

## Testing
- Unit tests: Vitest + jsdom (`npm test`)
- E2E: Playwright (`npm run test:e2e`)
- Run single test: `npx vitest run <path>`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paulalarosa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
