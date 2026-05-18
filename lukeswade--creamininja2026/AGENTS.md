# CreamiNinja — Copilot Instructions

## Project Overview

CreamiNinja is a full-stack Progressive Web App (PWA) for sharing Ninja CREAMi recipes. It enables users to create, share, and discover recipes with social features including friend connections ("Ninjagos"), privacy controls, and AI-powered recipe generation.

**Target deployment:** Cloudflare-native stack with Pages (frontend) and Workers (API).

## Architecture

This is a **monorepo** using pnpm workspaces with two main applications:

- **`apps/api`** — Backend API (Cloudflare Workers with Hono framework)
- **`apps/web`** — Frontend PWA (React + Vite + Tailwind CSS)

### Backend (`apps/api`)
- **Framework:** Hono (web framework for Cloudflare Workers)
- **Database:** Cloudflare D1 (SQLite-compatible)
- **Storage:** Cloudflare R2 (S3-compatible for images)
- **Auth:** Email/password with server-side sessions (httpOnly cookies)
- **Security:** CSRF protection, Turnstile bot protection, rate limiting
- **AI Integration:** OpenAI API for recipe generation (abstracted provider interface)

### Frontend (`apps/web`)
- **Framework:** React 18 with TypeScript
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Routing:** React Router v6
- **State Management:** TanStack Query (React Query)
- **PWA:** vite-plugin-pwa with service worker support
- **Icons:** Lucide React

## Tech Stack

### Common
- **Language:** TypeScript (strict mode enabled)
- **Package Manager:** pnpm 9.0.0 (pinned via packageManager field)
- **Node Version:** 20+
- **Linting:** ESLint 9 with flat config
- **Formatting:** Prettier

### API Dependencies
- `hono` — Web framework
- `@hono/zod-validator` — Request validation
- `zod` — Schema validation
- `jose` — JWT/session token handling
- `aws4fetch` — AWS signature v4 for R2 presigned URLs
- `nanoid` — ID generation
- `@cloudflare/workers-types` — TypeScript types for Workers

### Web Dependencies
- `react` & `react-dom` — UI framework
- `@tanstack/react-query` — Data fetching and caching
- `react-router-dom` — Client-side routing
- `clsx` — Conditional class names
- `lucide-react` — Icon library
- `tailwindcss` — Utility-first CSS
- `vite-plugin-pwa` — PWA capabilities

## Coding Conventions

### General Style
- **Indentation:** 2 spaces (enforced by `.editorconfig`)
- **Quotes:** Double quotes for strings
- **Semicolons:** Always use semicolons
- **Trailing commas:** None (per Prettier config)
- **Line endings:** LF (Unix-style)
- **Final newline:** Always include

### TypeScript
- **Strict mode:** Enabled in all packages
- **Type imports:** Use `import type` for type-only imports
- **No any:** Avoid `any` type; use `unknown` or proper typing
- **Unused vars:** Prefix with underscore (`_`) if intentionally unused
- **Module resolution:** Bundler mode (for Vite/Wrangler compatibility)

### React/Frontend
- **Components:** Use functional components with hooks (no class components)
- **File organization:** Group by feature/route in `src/routes` and `src/components`
- **Styling:** Use Tailwind utility classes; use `clsx` for conditional classes
- **State:** Prefer TanStack Query for server state; React hooks for local state
- **Exports:** Default exports for route components; named exports for utilities

### API/Backend
- **Framework:** Use Hono patterns (middleware, context)
- **Validation:** Use Zod schemas with `@hono/zod-validator`
- **Error handling:** Return JSON with `{ ok: false, error: { message: "..." } }`
- **Database:** Use parameterized queries to prevent SQL injection
- **Auth:** Session tokens in httpOnly cookies with CSRF protection
- **File naming:** Kebab-case for route files, camelCase for utility modules

## Development Workflow

### Setup
```bash
# Root install
pnpm install

# API setup
cd apps/api
pnpm db:local:setup    # Creates local D1 database
pnpm db:seed:local     # Optional: Add demo data

# Web setup
cd apps/web
# Optional: Create .env to override defaults
# VITE_API_BASE defaults to http://localhost:8787
# VITE_TURNSTILE_SITE_KEY is optional for local dev (API can bypass with TURNSTILE_BYPASS=true)
```

### Running Locally
```bash
# From root - run all apps
pnpm dev

# Or individually
cd apps/api && pnpm dev   # API: http://localhost:8787
cd apps/web && pnpm dev   # Web: http://localhost:5173
```

### Linting & Formatting
```bash
# Lint all packages
pnpm lint

# Format all packages
pnpm format

# Individual packages
cd apps/api && pnpm lint
cd apps/web && pnpm format
```

### Building
```bash
# Build all packages
pnpm build

# Individual packages
cd apps/api && pnpm deploy    # Deploy API to Cloudflare
cd apps/web && pnpm build     # Build for Cloudflare Pages
```

## Database & Migrations

- **Schema location:** `apps/api/migrations/`
- **Migration tool:** Wrangler D1
- **Local database:** `.wrangler/state/v3/d1/miniflare-D1DatabaseObject/`
- **Commands:**
  - `pnpm db:local:setup` — Create and migrate local DB
  - `pnpm db:remote:apply` — Apply migrations to remote D1
  - `pnpm db:seed:local` / `pnpm db:seed:remote` — Seed demo data

**Important:** Always create migration files for schema changes; never modify the database manually.

## Security Guidelines

### Authentication
- **Sessions:** Use httpOnly cookies with secure flag in production
- **CSRF:** Validate `x-csrf-token` header on mutating requests
- **Passwords:** Always hash with appropriate algorithm (implementation in `auth` routes)
- **Tokens:** Use `jose` library for JWT creation/verification

### Data Validation
- **API:** Validate all inputs with Zod schemas before processing
- **SQL:** Always use parameterized queries (D1 prepared statements)
- **File uploads:** Validate file types and sizes; use presigned URLs
- **User input:** Sanitize and validate both client and server-side

### Environment Variables
- **Secrets:** Never commit secrets; use Wrangler secrets or environment variables
- **API keys:** Store in Cloudflare Workers secrets (e.g., `OPENAI_API_KEY`, `TURNSTILE_SECRET_KEY`)
- **Client env:** Prefix with `VITE_` for frontend exposure

### Rate Limiting
- **AI endpoints:** Implement rate limiting to prevent abuse
- **Auth endpoints:** Protect login/signup from brute force
- **Implementation:** Use custom middleware in `apps/api/src/middleware/rateLimit.ts`

## Testing

**Note:** This repository does not currently have automated tests configured. When adding tests:

- **Backend:** Consider using Vitest with Cloudflare Workers test utilities
- **Frontend:** Use Vitest + React Testing Library
- **E2E:** Consider Playwright for integration testing

## File Organization

### API (`apps/api/src/`)
```
src/
├── db/              # Database utilities and queries
├── middleware/      # Hono middleware (auth, CSRF, rate limit)
├── routes/          # API endpoints by domain
│   ├── auth.ts
│   ├── recipes.ts
│   ├── friends.ts
│   ├── feed.ts
│   ├── uploads.ts
│   └── ai.ts
├── util/            # Shared utilities
├── env.ts           # Environment type definitions
└── index.ts         # App entry point
```

### Web (`apps/web/src/`)
```
src/
├── components/      # Reusable UI components
├── lib/             # Utilities, API client, types
├── routes/          # Page components (by route)
├── main.tsx         # App entry point
└── styles.css       # Global styles (Tailwind imports)
```

## Common Patterns

### API Route Pattern
```typescript
import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import type { Env } from "../env";

const router = new Hono<{ Bindings: Env }>();

const schema = z.object({
  field: z.string()
});

router.post("/endpoint", zValidator("json", schema), async (c) => {
  const body = c.req.valid("json");
  // Business logic
  return c.json({ ok: true, data: {} });
});

export default router;
```

### React Query Pattern
```typescript
// In component
const { data, isLoading } = useQuery({
  queryKey: ["resource", id],
  queryFn: () => fetchResource(id)
});

const mutation = useMutation({
  mutationFn: createResource,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["resource"] });
  }
});
```

## Deployment

### API (Cloudflare Workers)
- Deploy: `cd apps/api && pnpm deploy`
- Config: `wrangler.toml`
- Secrets: Set via `wrangler secret put <KEY>`
- Route: `api.creamininja.com/*`

### Web (Cloudflare Pages)
- **Build command:** `pnpm -C apps/web build`
- **Output directory:** `apps/web/dist`
- **Environment variables:**
  - `VITE_API_BASE` — API URL (e.g., `https://api.creamininja.com`)
  - `VITE_TURNSTILE_SITE_KEY` — Cloudflare Turnstile site key
- **Domains:** `creamininja.com`, `www.creamininja.com`

## Resources

- **Hono Documentation:** https://hono.dev/
- **Cloudflare Workers Docs:** https://developers.cloudflare.com/workers/
- **Cloudflare D1 Docs:** https://developers.cloudflare.com/d1/
- **TanStack Query Docs:** https://tanstack.com/query/latest
- **Tailwind CSS Docs:** https://tailwindcss.com/docs

## Notes for Copilot

- This is a **Cloudflare-native** stack; do not suggest Express, Next.js, or other frameworks
- Use **Hono** patterns for API routes, not Express/Fastify
- Use **D1** SQL syntax (SQLite-compatible), not PostgreSQL or MySQL specific features
- For file uploads, use **R2 presigned URLs**, not direct multipart uploads
- Frontend API calls go through **TanStack Query**; avoid fetch directly in components
- Respect the **monorepo** structure; shared utilities should be in the appropriate package
- Follow the **existing patterns** in the codebase for consistency

---
> Source: [lukeswade/creamininja2026](https://github.com/lukeswade/creamininja2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
