# CLAUDE.md - AI Assistant Guide for EventLayer

## Project Overview

EventLayer is a full-service event management platform built as a monorepo using **pnpm workspaces** and **Turborepo**. It enables organizers to create and manage events with features like attendee management, scheduling, ticketing, venue mapping, sponsor management, and a PWA-based mobile app experience.

## Tech Stack

- **Package Manager**: pnpm (v9.1.1)
- **Monorepo Tool**: Turborepo
- **Frontend**: SvelteKit 2.x with Svelte 4.x
- **Styling**: TailwindCSS with PostCSS
- **Database**: PostgreSQL with Drizzle ORM
- **API**: tRPC for type-safe API calls
- **Authentication**: Lucia Auth
- **Caching**: Redis (ioredis/Upstash)
- **Email**: Resend
- **Payments**: Stripe
- **File Storage**: AWS S3
- **Testing**: Vitest (unit) + Playwright (e2e)
- **Language**: TypeScript (strict mode)

## Repository Structure

```
eventlayer/
├── apps/
│   └── web/                    # Main SvelteKit application
│       ├── src/
│       │   ├── routes/         # SvelteKit file-based routing
│       │   │   ├── (api)/      # API endpoints (Stripe webhooks, etc.)
│       │   │   ├── (checkin)/  # Event check-in functionality
│       │   │   ├── (manage)/   # Admin/staff management dashboard
│       │   │   └── (pwa)/      # Main PWA app for attendees
│       │   ├── lib/
│       │   │   ├── components/ # Svelte components
│       │   │   ├── server/     # Server-side utilities
│       │   │   └── state/      # State management
│       │   ├── hooks.server.ts # Server hooks (auth, CORS, tRPC)
│       │   └── hooks.client.ts # Client hooks
│       ├── static/             # Static assets
│       └── tests/              # Playwright tests
├── packages/
│   ├── api/                    # tRPC router and API logic
│   │   └── src/
│   │       ├── router/         # tRPC procedures by domain
│   │       ├── models/         # Business logic functions
│   │       ├── core/           # Auth, Redis, errors
│   │       └── media/          # Media handling (S3 uploads)
│   ├── db/                     # Database schema and configuration
│   │   └── schema/             # Drizzle schema files by table
│   ├── ui/                     # Shared UI component library
│   │   └── src/components/     # Reusable Svelte components
│   └── util/                   # Shared utilities (lodash wrappers, dayjs, etc.)
└── tooling/
    ├── eslint/                 # Shared ESLint configuration
    ├── prettier/               # Shared Prettier configuration
    ├── tailwind/               # Shared Tailwind configuration
    └── typescript/             # Shared TypeScript configuration
```

## Package Naming Convention

All internal packages use the `@matterloop/` scope:
- `@matterloop/api` - API/tRPC package
- `@matterloop/db` - Database package
- `@matterloop/ui` - UI components
- `@matterloop/util` - Utilities
- `@matterloop/eslint-config` - ESLint config
- `@matterloop/prettier-config` - Prettier config
- `@matterloop/tsconfig` - TypeScript config

## Development Commands

```bash
# Install dependencies
pnpm install

# Start development server (web app only)
pnpm dev

# Build all packages
pnpm build

# Run linting
pnpm lint

# Format code
pnpm format

# Database operations (from packages/db or apps/web)
pnpm db:push        # Push schema changes to database
pnpm db:gen         # Generate migrations
pnpm db:introspect  # Introspect existing database

# Testing (from apps/web)
pnpm test              # Run all tests
pnpm test:unit         # Run Vitest unit tests
pnpm test:integration  # Run Playwright tests
```

## Code Conventions

### TypeScript

- Strict mode enabled (`noUncheckedIndexedAccess: true`)
- Use type inference where possible
- Export types alongside implementations
- Use Zod schemas for runtime validation

### Svelte Components

- Use `lang="ts"` in script blocks
- Follow component naming: `PascalCase.svelte`
- Use `$lib/` alias for imports from `src/lib/`
- Context API for global state (`setContext`/`getContext`)
- Svelte stores with `writable`/`persisted`

### Styling

- TailwindCSS for styling (utility-first)
- Use `tw()` utility from `@matterloop/ui` for conditional classes
- PostCSS for processing
- Dark mode via class strategy
- CSS variables for theming (HSL colors)

### Prettier Configuration

- No semicolons
- Tabs for indentation
- Single quotes
- Trailing commas
- 100 character print width
- Import sorting with `@ianvs/prettier-plugin-sort-imports`

Import order:
1. React/Next (if applicable)
2. Third-party modules
3. `@matterloop/*` packages
4. Relative imports

### File Structure Patterns

**SvelteKit Routes:**
```
routes/
├── +page.svelte          # Page component
├── +page.server.ts       # Server load function
├── +layout.svelte        # Layout component
├── +layout.server.ts     # Layout server load
└── +server.ts            # API endpoint
```

**tRPC Procedures:**
```typescript
// packages/api/src/router/entityProcedures.ts
export const entityProcedures = t.router({
  list: procedureWithContext.query(async ({ ctx }) => { ... }),
  get: procedureWithContext.input(z.object({ id: z.string() })).query(...),
  create: procedureWithContext.input(schema).mutation(...),
  update: procedureWithContext.use(verifyMe()).input(schema).mutation(...),
})
```

**Database Schema:**
```typescript
// packages/db/schema/entity.ts
export const entityTable = pgTable('entity', {
  id: uuid('id').default(sql`extensions.uuid_generate_v4()`).primaryKey(),
  // ... columns
})
export const entityRelations = relations(entityTable, ({ many, one }) => ({ ... }))
export const entitySchema = createInsertSchema(entityTable)
export type Entity = typeof entityTable.$inferSelect
```

## Key Patterns

### Authentication Flow

1. Server hooks (`hooks.server.ts`) validate sessions via Lucia
2. `event.locals.me` contains the authenticated user
3. `event.locals.event` contains the current event context (multi-tenant)
4. tRPC procedures use middleware: `verifyMe()`, `verifyAdmin()`, `verifyEvent()`

### Multi-Tenant Architecture

- Events are identified by subdomain or custom domain
- `handleEventContext` hook resolves event from hostname
- All queries are scoped to `event.locals.event.id`

### tRPC Integration

- Router defined in `packages/api/src/root.ts`
- Procedures in `packages/api/src/router/*.ts`
- Context created via `createContext` in `procedureWithContext.ts`
- Client calls via `trpc-sveltekit`

### Database Access

```typescript
import { db, eq, and } from '@matterloop/db'
import { userTable, eventUserTable } from '@matterloop/db'

// Query example
const user = await db.query.userTable.findFirst({
  where: eq(userTable.id, userId),
  with: { photo: true }
})

// Insert example
await db.insert(userTable).values({ ... }).returning()

// Update example
await db.update(userTable).set({ ... }).where(eq(userTable.id, id))
```

### Caching Pattern

```typescript
import { redis } from '@matterloop/api/src/core/redis'

// Cache invalidation after mutations
redis.del(`event_users:${eventId}`)
redis.del(`event_heavy:${eventId}`)

// Cache read with fallback
let data = await redis.get<Entity>(key)
if (!data) {
  data = await db.query.entityTable.findFirst(...)
  redis.set(key, data)
}
```

## Route Groups

- `(api)` - Webhook endpoints (Stripe, etc.)
- `(checkin)` - Event check-in interface for staff
- `(manage)` - Admin dashboard for event management
- `(pwa)` - Main attendee-facing PWA application

## Environment Variables

Create a `.env` file with:

```env
DB_URL=postgresql://...
REDIS_URL=redis://...
PUBLIC_BASE_URL=eventlayer.io
BASE_HOST=eventlayer.io
NODE_ENV=development

# AWS S3
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_BUCKET=...

# Auth
LUCIA_SECRET=...

# Services
RESEND_API_KEY=...
STRIPE_SECRET_KEY=...
STRIPE_WEBHOOK_SECRET=...
```

## Common Tasks

### Adding a New tRPC Procedure

1. Create/update file in `packages/api/src/router/`
2. Add to router in `packages/api/src/root.ts`
3. Use appropriate middleware (`verifyMe`, `verifyAdmin`, etc.)
4. Add Zod input validation

### Adding a New Database Table

1. Create schema file in `packages/db/schema/`
2. Export from `packages/db/index.ts`
3. Add to db schema object in `packages/db/index.ts`
4. Run `pnpm db:push` to sync

### Adding a New Page

1. Create directory in appropriate route group
2. Add `+page.svelte` for component
3. Add `+page.server.ts` for data loading
4. Use layouts from parent `+layout.svelte`

### Adding a New Shared Component

1. Create in `packages/ui/src/components/`
2. Export from `packages/ui/index.ts`
3. Import as `import { Component } from '@matterloop/ui'`

## Testing Guidelines

- Unit tests: Place next to source files as `*.test.ts`
- Integration tests: Place in `apps/web/tests/`
- Use Vitest for unit tests, Playwright for e2e

## Important Notes

- The dev server runs on port 8884
- CSRF checking is disabled in svelte.config.js
- PWA functionality via `@vite-pwa/sveltekit`
- Redis caching should be invalidated after database mutations
- Event context is resolved from subdomain/domain in hooks

## REST API Documentation (llms.txt)

The project includes an `llms.txt` file in the root directory that documents the public REST API. This file is designed to be readable by both humans and LLMs.

### IMPORTANT: Maintaining llms.txt

**When making changes to the REST API, you MUST update `llms.txt` to reflect those changes.**

This includes:
- Adding new endpoints → Add full documentation with request/response examples
- Modifying existing endpoints → Update the documentation accordingly
- Changing authentication → Update the authentication section
- Adding query parameters → Document the new parameters
- Changing response formats → Update the example responses

### REST API Structure

```
apps/web/src/routes/(api)/rest/
├── pages/
│   ├── +server.ts              # GET /rest/pages
│   └── [pageId]/
│       └── +server.ts          # GET /rest/pages/:pageId
└── users/
    └── +server.ts              # GET /rest/users

apps/web/src/lib/server/rest/
└── auth.ts                     # validateApiKey(), getCorsHeaders()
```

### Adding a New REST Endpoint

1. Create directory under `apps/web/src/routes/(api)/rest/`
2. Add `+server.ts` with handlers (GET, POST, etc.)
3. Use `validateApiKey(request)` for authentication
4. Use `getCorsHeaders(origin)` for CORS
5. **Update `llms.txt` with complete endpoint documentation**
6. Add changelog entry at the bottom of `llms.txt`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickyhajal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
