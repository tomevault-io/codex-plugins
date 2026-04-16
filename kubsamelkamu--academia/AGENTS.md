# GitHub Copilot Instructions - Academia

Academia is a multi-tenant academic project management platform built with Next.js App Router.

## Stack (source of truth: package.json)

- Next.js 16 + React 19 + TypeScript (strict)
- Tailwind CSS v4 + shadcn/ui (Radix primitives)
- Client API: axios instance in `src/lib/api/client.ts`
- State: Zustand (UI/auth) + TanStack Query v5 (server state)

## Repo map (high level)

```
src/
  app/                 # App Router routes/layouts
  components/
    ui/                # shadcn/ui primitives (do not edit unless required)
    layout/            # Header/sidebar/footer
    dashboard/         # Role dashboards and widgets
  config/navigation.ts # Role-based navigation config
  lib/                 # Utils, API client, themes, validations
  store/               # Zustand stores
  types/               # Shared TypeScript types
  validations/         # zod schemas
```

## Non-negotiables

- Prefer Server Components by default; add `"use client"` only when interactivity is required.
- Do not restyle by adding new hex colors or custom fonts; use Tailwind tokens and existing theme variables.
- Avoid editing `src/components/ui/*` directly; prefer composition/wrappers in feature components.
- Keep changes minimal and consistent with existing patterns (imports use `@/*` alias).

## Multi-tenant & auth

- Tenant context is derived from the request host in `src/middleware.ts` and forwarded as `x-tenant-id` for internal requests.
- Calls to the upstream backend should use the shared axios client (`src/lib/api/client.ts`), which injects:
  - `X-Tenant-Domain` (from `useAuthStore`, `tenantDomain`)
  - `Authorization: Bearer <token>` when available
- Never hardcode tenant IDs/domains in components or API handlers.

## Roles & navigation

- Roles are defined in `src/config/navigation.ts`:
  - `department_head`, `coordinator`, `advisor`, `student`, `department_committee`
- When adding a dashboard route, update navigation config and ensure RBAC matches the route structure under `src/app/dashboard/*`.

## Data fetching rules

- In Client Components: use TanStack Query (`useQuery`, `useMutation`) for server state.
- Use the shared axios client for app backend calls so tenant/auth headers are consistent.
- In Server Components / route handlers: use `fetch()` (and pass through needed headers) when appropriate.

## API routes (App Router)

- Place route handlers under `src/app/api/**/route.ts`.
- Return `NextResponse.json(...)` for JSON and avoid caching sensitive data (`cache: "no-store"` when needed).

## TypeScript & validation

- Avoid `any`; define or reuse types from `src/types/*`.
- Reuse zod schemas from `src/validations/*` when validating forms or API payloads.

## Commands

- Dev: `npm run dev`
- Lint: `npm run lint`
- Build: `npm run build`

## When unsure

- Follow existing patterns in nearby files (dashboard pages/components are the best references).
- Prefer the simplest implementation that satisfies the requirement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kubsamelkamu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
