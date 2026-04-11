# Iridium — Project Guidelines

Full-stack AI chat app built with React Router v7, Better Auth, Prisma, and Tambo AI.

## Tech Stack

- **Framework**: React Router v7 (SSR, `v8_middleware` future flag)
- **Auth**: Better Auth with Prisma adapter, admin plugin (roles: USER, EDITOR, ADMIN)
- **Database**: PostgreSQL via Prisma (schema at `prisma/schema.prisma`, client generated to `app/generated/prisma/`)
- **AI**: Tambo AI (`@tambo-ai/react`, `@tambo-ai/typescript-sdk`) — client-side chat via `useTambo`, server-side thread CRUD via TypeScript SDK
- **Styling**: Tailwind CSS v4 + DaisyUI v5, CVA with tailwind-merge
- **Runtime**: Bun (local dev), Node 20 Alpine (Docker/prod)
- **Validation**: Zod
- **Icons**: lucide-react

## Build & Dev Commands

| Command             | Purpose                        |
| ------------------- | ------------------------------ |
| `bun install`       | Install dependencies           |
| `bun run dev`       | Start dev server (port 5173)   |
| `bun run build`     | Production build               |
| `bun run typecheck` | Generate route types + run tsc |
| `bun run seed`      | Seed database                  |
| `bun run studio`    | Open Prisma Studio             |

Prisma CLI: always use `bunx --bun prisma <command>`.

## Architecture

### Routing (config-based)

Routes are defined in `app/routes.ts` using `@react-router/dev/routes` helpers (`index`, `route`, `prefix`) — **not** file-system routing.

Route files export: `middleware` array → `loader` → `action` → `default` component.

Auto-generated types: `import type { Route } from './+types/<routeName>'`.

API routes live under `/api` prefix and export only `loader`/`action` (no UI component).

### Data Access Layer

Plain exported async functions in `app/models/*.server.ts` — no classes, no ORM wrapper.

- `thread.server.ts`: Tambo SDK wrapper — `createThread`, `getAllThreadsByUserKey`, `getThread`, `deleteThread` (threads & messages live in Tambo's cloud, not Prisma)
- `session.server.ts`: `getUserFromSession`, `requireUser`, `requireAnonymous`, `hasRole`, `requireRole`
- `note.server.ts`: CRUD for notes (stored in Prisma/PostgreSQL)

### Auth Flow

- Server config: `app/lib/auth.server.ts` (Better Auth + Prisma adapter)
- Client config: `app/lib/auth.client.ts` (`createAuthClient` + `adminClient` plugin)
- API passthrough: all `/api/auth/*` → `auth.handler` in `app/routes/api-auth.ts`
- Middleware: `app/middleware/auth.ts` checks session, redirects to `/login`, stores user in `userContext`
- Protect routes by exporting: `export const middleware: Route.MiddlewareFunction[] = [authMiddleware]`

### AI Chat

- **Tambo is the source of truth** for threads and messages — no local DB storage for chat
- Server-side: `app/lib/tambo.server.ts` creates a singleton `TamboAI` client (from `@tambo-ai/typescript-sdk`)
- Client-side: `TamboProvider` in `app/root.tsx` with `apiKey` passed via loader at runtime
- Chat UI uses `useTambo()` and `useTamboThreadInput()` hooks from `@tambo-ai/react`
- Thread CRUD in loaders/actions goes through `app/models/thread.server.ts` → Tambo SDK
- Custom tools defined in `app/lib/tambo.ts` (note CRUD via `/api/notes`)
- Custom components registered with `TamboProvider`: `InfoCard`, `StepList`, `ProsCons`, `NotesGallery`, `DataTable`

## Conventions

### Imports

- Use `~/` path alias for all app imports (maps to `./app/*`)
- Server-only files use `.server.ts` suffix

### Components

- Use CVA from `cva.config` (not the raw `cva` package) — it integrates `tailwind-merge`
- Export variant definitions AND a named function component
- Type props with `PropsWithChildren<Props>`
- Use DaisyUI v5 class names (`card`, `btn`, `chat-bubble`, `drawer`, `badge`, etc.)

### Routes

- Pages set `<title>` and `<meta>` inline in JSX — no `meta` export
- Use `<Form>` with `intent` hidden fields for action disambiguation
- Export `ErrorBoundary` using `isRouteErrorResponse` for error handling
- Use `tiny-invariant` for runtime assertions

### Context

- `app/context.ts`: `userContext` via React Router's `createContext<User | null>`
- `app/shared.ts`: shared className helpers (`listItemClassName`, `navLinkClassName`)

### Layout

- Root layout in `app/root.tsx`: header nav → 3/9 grid (sidebar + content) → footer
- Auth-conditional nav items, right-side DaisyUI `Drawer`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sethdavis512)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sethdavis512)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
