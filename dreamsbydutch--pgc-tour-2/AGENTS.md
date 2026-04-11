# GitHub Copilot Instructions (PGC)

You are working in the **PGC** repo (Vite/React + Convex backend). Follow these instructions when generating or modifying code.

## General

- Prefer small, surgical changes.
- Don’t edit generated code in `convex/_generated/*`.
- Don’t edit generated code in `src/routeTree.gen.ts`.
- Keep TypeScript `strict` compatibility.
- Do not use explicit `any` anywhere (no `any`, `any[]`, `as any`, or `: any`).
- Don’t introduce new libraries unless asked.

## Comments (Important)

- Do **not** add comments inside function bodies (no `// ...`, `/* ... */`, or JSX `{/* ... */}` within a function).
- If a function needs explanation, add a **single JSDoc header** immediately **above** the function.
- Prefer JSDoc that works with IntelliSense: include a short summary and (when helpful) `@param`, `@returns`, and an `@example`.
- Do not add block-by-block narration comments within a function.

## Frontend: Components (Project Convention)

### Component Taxonomy (Folder Structure)

All app components live under `src/components/*` and are organized into four buckets:

- `src/components/ui/*` (import from `@/ui`)
  - UI primitives/composites only (shadcn/ui-style).
  - No Convex hooks, no auth/role hooks, no router reads/writes, no business logic.
- `src/components/displays/*` (import from `@/displays`)
  - Presentational components: props in, UI out.
  - Avoid data fetching and orchestration.
- `src/components/widgets/*` (import from `@/widgets`)
  - Leaf components that may own local state and do small, self-contained data flows.
  - Can use Convex hooks and other app hooks when appropriate.
- `src/components/facilitators/*` (import from `@/facilitators`)
  - Orchestration components and page-level composition.
  - Compose displays/widgets; handle routing glue and feature-level state.

### Import Rules (Important)

- All component imports must come only from: `@/ui`, `@/displays`, `@/widgets`, `@/facilitators`.
- Do not import components from `@/components`.
- Do not import from deep paths like `@/components/displays/...` or `@/components/widgets/...`.
- If something isn’t exported yet, add it to the appropriate barrel index:
  - `src/components/ui/index.ts`
  - `src/components/displays/index.ts`
  - `src/components/widgets/index.ts`
  - `src/components/facilitators/index.ts`

- **One component per file**: each component lives in a single `.tsx` file and exports exactly one React component (the UI).
- **Optional internals**: if the component needs local business logic or a loading state, define them in the same file as:
  - an unexported custom hook (business logic + fetching; called only by the exported component), and/or
  - an unexported `Skeleton` component (loading state; used only by the exported component).
- **Exports**: only the main UI component is exported. Any hook/skeleton in the file must be unexported.
- **Documentation**:
  - The exported component must have an in-depth JSDoc header immediately above its declaration describing the full file behavior, including `@param`, `@returns`, data fetching sources, and the purpose/major render states.
  - If present, the unexported hook must have a thorough doc comment immediately above its declaration explaining what it fetches/derives, its inputs, and what it returns to the UI component.
  - If present, the unexported `Skeleton` must have a short doc comment immediately above its declaration that explains its purpose.
- **No shared helpers/types/constants**: do not define shared types, shared utility functions, or shared constants inside component files.
  - Put shared/reused TypeScript types in `src/lib/types.ts`.
  - Put shared/reused utility functions in `src/lib/utils.ts`.
  - Put shared/reused constants in `src/lib/constants.ts`.
  - Components should import from `@/lib/types`, `@/lib/utils`, and `@/lib/constants`.
- **Skeleton UI**: use existing primitives from `src/components/ui/*` (e.g. `Skeleton`) rather than inventing new loading styles.

## Frontend: `src/components/ui/*` (UI Primitives)

`src/components/ui/*` is reserved for **UI primitives**: reusable, prop-driven components that are the end of the chain for _app concerns_.

- Components in `src/components/ui/*` may use **local UI state and DOM effects** strictly for presentation/interaction (e.g., Escape key handling, outside click handling, focus management, measuring/layout observers).
- Still forbidden in `src/components/ui/*`:
  - Convex hooks (`useQuery`, `useMutation`, `useAction`).
  - Auth/role hooks (Clerk, `useUser`, `useRoleAccess`, etc.).
  - Navigation and URL syncing (`useNavigate`, router reads/writes).
  - Business logic / app orchestration.
  - Side effects that change app data (mutations, analytics events, etc.).
- Prefer moving app data fetching and view-model construction into:
  - hooks under `src/hooks/*`, and/or
  - “smart/orchestration” components under `src/components/facilitators/*`.
- UI components may compose other UI primitives (Button/Card/Table/Skeleton) and use `@/lib/*` utilities/types.
- **Skeleton UI**: use existing primitives from `src/components/ui/*` (e.g. `Skeleton`) rather than inventing new loading styles.

## Frontend: How src/ works in this repo

This repo’s frontend is **TanStack Start** + **TanStack Router** (file-based routes), built with Vite.

Key rules:

- This is **not** a Next.js project.
- Routes live in `src/routes/*` and are defined with `createFileRoute(...)`.
- The global app shell/layout is `src/routes/__root.tsx`.
- The router is created in `src/router.tsx` using the generated `src/routeTree.gen.ts`.
- Prefer the `@/*` import aliases (`@/ui`, `@/displays`, `@/widgets`, `@/facilitators`, `@/hooks`, `@/lib`, `@/convex`).
- Use existing UI primitives in `src/components/ui/*` (shadcn/ui). Don’t invent new design tokens.
- Use `cn()` from `src/lib/utils.ts` for className composition.

## Frontend: Routes (Project Convention)

- Think of `src/routes/*` as **middleware**: routes may read/validate URL state, gate access, and keep the URL in sync — but they should not own UI or business logic.
- Routes in `src/routes/*` should be thin and consistent.
- Route files should contain only:
  - minimal routing concerns (`createFileRoute`, `validateSearch`, reading params/search via `Route.useParams()` / `Route.useSearch()`),
  - minimal auth gating (signed-in / role checks) when needed,
  - minimal navigation wiring (`Route.useNavigate()` to keep URL state in sync),
  - and rendering a single page component from `@/facilitators`.
- Do **not** put business logic, complex view-model construction, or data fetching (e.g. `useQuery`) in route files.
- Move non-routing logic into the component the route renders (or into hooks under `src/hooks/*` used by that component).
- `src/routes/__root.tsx` is the app shell and may include layout/provider wiring.

Data & hooks:

- Prefer composing view models in hooks under `src/hooks/*`.
- For Convex reads, use `useQuery(api.functions.<module>.<query>, args | "skip")`.
- Keep route components thin: fetch/compose in hooks, render via components.

Auth & providers:

- Providers are wired in `src/components/facilitators/providers/Providers.tsx` (Convex + Clerk + PostHog) and imported from `@/facilitators`.
- Preserve existing `"use client"` directives where present; don’t add them broadly unless required by the surrounding pattern.

## Convex: How it works in this repo

## Convex Direction (Important)

We prefer:

- **Convex functions = basic CRUD for each table** (create/get/update/delete).
- **A small set of Convex convenience queries** for hot screens (tournament view, leaderboard, standings) to reduce round trips and avoid client-side N+1.
- **React hooks = presentation/derived UI logic** (grouping, formatting, lightweight client-only composition).

Avoid adding lots of one-off server endpoints. If it’s not a reused, hot-path view model, keep it in hooks.

### File layout → API paths

Convex function modules map to `api` paths by file path:

- `convex/functions/seasons.ts` exports are referenced as `api.functions.seasons.<exportName>`
- `convex/functions/tournaments.ts` → `api.functions.tournaments.<exportName>`
- etc.

All app functions live under `convex/functions/*.ts` (there is no root-level `convex/functions.ts`).

### Function types

Use the correct server function type:

- `query` for reads only
- `mutation` for writes
- `action` for external APIs / non-deterministic work (e.g. `fetch`, third-party calls)

### Schema + validation

- Tables, validators, and indexes are defined in `convex/schema.ts`.
- Use `v` validators for all public function args.
- This codebase uses:
  - plural table names (e.g. `members`, `tournaments`)
  - `oldId?: string` on many tables for Supabase migration mapping
  - money amounts as **cents** (`number`)
  - `_creationTime` (built-in) + `updatedAt` (manual)

### Query performance

- Prefer `.withIndex("...")` when an index exists; avoid `.filter(...)` if it can be indexed.
- When adding a new frequently-used access pattern, update `convex/schema.ts` with a new index.
- Avoid unbounded `.collect()` on large tables unless necessary; prefer indexed reads + pagination.

### CRUD pattern (project convention)

Most domain modules follow a consistent “single CRUD function with options” shape:

- `createXxx` (mutation)
- `getXxx` (query)
- `updateXxx` (mutation)
- `deleteXxx` (mutation)

Conventions:

- `args.data` holds the core payload.
- `args.options` is optional and controls behavior, validation, response shaping.
- Common flags:
  - `skipValidation?: boolean`
  - `returnEnhanced?: boolean`
  - `includeStatistics?: boolean`

Keep these conventions when adding new entities/functions unless the user requests otherwise.

### “Enhanced” docs pattern

Most modules implement an internal helper:

- `async function enhanceXxx(ctx, doc, enhanceOptions)`

This helper:

- adds computed fields (formatted money, derived status)
- optionally includes related docs (season → tours, tournament → teams)
- optionally computes statistics

When asked for “more data in the response”, prefer either:

- Adding a small, reusable Convex convenience query for a hot screen, or
- Adding a hook that composes existing queries for presentation/derived UI.

### Migration helpers

For Supabase migration/import flows, follow the established patterns:

- Keep `oldId` mapping.
- Upsert by `oldId` (patch if exists, insert otherwise).
- Use `skipValidation` or dedicated `*WithMigration` helpers.

### External API actions (DataGolf)

- DataGolf integration is in `convex/functions/datagolf.ts`.
- It requires `DATAGOLF_API_KEY` in Convex env.
- Use `action` for fetch calls.

## Auth and identity (important)

Current Convex functions largely accept `clerkId` as an input and do not consistently enforce authorization.

- Do **not** introduce breaking auth changes without the user explicitly asking.
- If asked to “secure” endpoints, prefer using Convex auth patterns (`ctx.auth`) and derive the user identity server-side rather than trusting a client-provided `clerkId`.

## Testing / regen

- Do **not** run `npm run lint` or `npm run typecheck` after every small change.
  - Only run them when the user explicitly asks, or when finishing a larger/risky refactor where verification is important.
  - If unsure, ask whether to run checks instead of running them automatically.
- If you add/remove Convex exports, remind the user to regenerate types by running `npx convex dev`.
- Never hand-edit the generated `api` definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dreamsbydutch)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dreamsbydutch)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
