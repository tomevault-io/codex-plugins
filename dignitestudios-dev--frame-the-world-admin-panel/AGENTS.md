# Copilot Instructions

## Commands

```bash
npm run dev       # Start development server (http://localhost:3000)
npm run build     # Production build
npm run lint      # Run ESLint
```

No test suite is configured.

## Architecture

This is a **Next.js 16 App Router** admin panel with the following top-level structure:

- `app/auth/` — Public auth pages (login, forgot-password, reset-password, verification). Wrapped by `PublicRoute` — redirects to `/dashboard` if already authenticated.
- `app/dashboard/` — Protected pages. Wrapped by `ProtectedRoute` — redirects to `/auth/login` if not authenticated.
- `components/ui/` — shadcn/ui primitives (do not edit directly; re-add via `npx shadcn add <component>`).
- `components/charts-and-graphs/` — Feature chart components built on top of `components/ui/chart.tsx` and Recharts.
- `lib/api/` — Axios instance + per-domain API function files (`*.api.ts`).
- `lib/slices/` — Redux Toolkit slices.
- `lib/store.ts` — Redux store; add new slices here.
- `contexts/` — React context providers (sidebar config, theme).
- `hooks/` — Custom hooks that consume contexts or browser APIs.

### Auth flow

Authentication state lives in **Redux** (`state.auth.isAuthenticated`). The JWT token is stored in `localStorage` under the key `authToken`. The Axios request interceptor (`lib/api/axios.ts`) automatically attaches it as a `Bearer` token, and the response interceptor clears it and redirects to `/auth/login` on 401.

`ProtectedRoute` and `PublicRoute` are client components that read Redux auth state and redirect accordingly. They wrap the `layout.tsx` of each route group, not individual pages.

### Sidebar

The sidebar is fully configurable at runtime via `SidebarConfigProvider` (in `contexts/sidebar-context.tsx`). The config has three fields:

- `variant`: `"sidebar" | "floating" | "inset"`
- `collapsible`: `"offcanvas" | "icon" | "none"`
- `side`: `"left" | "right"`

`useSidebarConfig()` (from either `hooks/use-sidebar-config.ts` or directly from `contexts/sidebar-context.tsx`) reads and updates this config. The dashboard layout handles left/right sidebar positioning based on `config.side`.

Navigation items are defined as a static `data.navGroups` array in `components/app-sidebar.tsx` — add new routes there.

## Key Conventions

### Path aliases

`@/` maps to the project root. Always use it instead of relative imports.

```ts
import { cn } from "@/lib/utils";
import { API } from "@/lib/api/axios";
```

### Styling

Use the `cn()` utility from `@/lib/utils` (clsx + tailwind-merge) for conditional class composition. Tailwind CSS v4 is used — no `tailwind.config.js`; all theme customization lives in `app/globals.css` as CSS variables.

### shadcn/ui

- Style: `radix-vega`, base color: `stone`, icon library: `lucide-react`.
- Add new components with: `npx shadcn add <component>` — they land in `components/ui/`.
- Charts: use shadcn chart primitives (`components/ui/chart.tsx`) + Recharts. Reference: https://ui.shadcn.com/charts/area.

### Adding API calls (TanStack Query)

Server state (anything fetched from the API) is managed with **TanStack Query**. Redux is reserved for client-only global state (e.g., auth flags).

Follow the pattern in `lib/api/users.api.ts`:

1. Create `lib/api/<domain>.api.ts`.
2. Define a `<domain>Keys` object for structured query keys.
3. Write plain async functions that call the shared `API` axios instance.
4. Export `use<Domain>` query hooks and `use<Action><Domain>` mutation hooks that call `qc.invalidateQueries` on success.

```ts
// Consuming a query
const { data, isPending, isError } = useUsers();

// Consuming a mutation
const { mutate: createUser } = useCreateUser();
createUser(payload);
```

`QueryClient` config (stale time, retry) lives in `lib/query-client.ts`. ReactQueryDevtools are included in development via `components/providers.tsx`.

### Data tables

Use `@tanstack/react-table`. See `app/dashboard/users/components/data-table.tsx` for the established pattern (column definitions, row actions, toolbar).

### Forms

Use `react-hook-form`. Toast notifications use `sonner`.

### Setting the API base URL

Update `baseURL` in `lib/api/axios.ts` before connecting to a real backend. It is currently a placeholder string.

---
> Source: [dignitestudios-dev/Frame-the-World-Admin-Panel](https://github.com/dignitestudios-dev/Frame-the-World-Admin-Panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
