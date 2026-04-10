# KORA Engineering Guardrails & AI Development Instructions

## Project Overview

**KORA** is an AI-powered business platform scaffolded as a modular monolith.

- **Frontend**: React 18, Vite, TypeScript, Tailwind CSS, Zustand (state management)
- **Backend**: Node.js (ES2022), Express, TypeScript, PostgreSQL, Redis, BullMQ
- **Infrastructure**: Docker Compose (Postgres + Redis), environment-based configuration
- **Key Services**: Multi-AI provider SDKs (Anthropic, OpenAI, Google Gemini, Mistral), Clerk auth, payment gateways (Stripe, PayPal, Flutterwave, Paystack)

## Quick Start Commands

### Backend
```bash
cd backend
npm install
cp .env.example .env  # Configure with real API keys
npm run dev           # Start dev server (port 3000)
npm run dev:worker    # Start async workers in parallel
npm run db:migrate    # Apply pending migrations
npm run db:seed       # Seed demo data
npm run typecheck     # Type check without building
npm run test          # Run vitest suite
npm run test:watch    # Watch mode
npm run build         # TypeScript compilation
npm start             # Run compiled server
npm start:worker      # Run compiled workers
```

### Frontend
```bash
cd frontend
npm install
cp .env.example .env  # Set VITE_API_BASE_URL=http://localhost:3000
npm run dev           # Start Vite dev server (port 5173)
npm run build         # TypeScript + Vite build
npm run preview       # Preview production build
```

### Infrastructure
```bash
docker compose up -d postgres redis      # Start services
docker compose down                       # Stop services
```

### URLs (Local Development)
- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:3000`
- Health check: `http://localhost:3000/health`
- API docs placeholder: `http://localhost:3000/api/docs`

## Architecture & Module Boundaries

### Backend Module Structure

KORA follows strict module boundaries. Each module is self-contained with:
- Routes (`modules/<module>/routes.ts`)
- Service layer (`modules/<module>/service.ts`)
- Database repositories (`db/repositories/<module>Repository.ts`)

**Core Modules** (8):
- `auth` - JWT authentication, session management, RBAC
- `bookings` - Appointment scheduling, booking lifecycle
- `appointments` - Appointment management
- `clinical` - Patient records, clinical data
- `emergency` - Critical incident management
- `finance` - Invoicing, revenue tracking, budgeting
- `payments` - Multi-gateway payment processing (Stripe, PayPal, Flutterwave, Paystack)
- `ai` - AI orchestration, live ranking, anomaly detection

**Additional Modules** (20+):
- `notifications` - Email, SMS, push, WhatsApp dispatch via BullMQ
- `reporting` - Report generation and analytics
- `clients`, `crm`, `tenant`, `staff`, `providers` - Domain entities
- `media`, `reviews`, `marketplace`, `social`, `video` - Features
- `analytics`, `platform`, `availability`, `discovery`, `campaigns`, `services` - Platform features
- `automation`, `canva`, `geofence`, `chatbot` - Advanced features

### Key Architecture Patterns

**Auth Middleware**:
- All protected routes require `requireAuth` middleware (see `src/middleware/auth.ts`)
- Routes specify protection at route registration: `app.use("/api/bookings", requireAuth, bookingsRoutes)`
- Public routes use `optionalAuth` for anonymous access
- JWT bearer tokens validated via `JWT_SECRET` (must be configured in environment)
- Token lifecycle: Generated on register/login, validated on each request, revoked on logout
- Organization ID extracted from JWT and accessible via `res.locals.auth.organizationId`
- Session tracking in database enables logout/revocation and audit trails

**Async Processing**:
- BullMQ + Redis for job queues
- Producer endpoints:
  - `POST /api/notifications/dispatch` → notifications queue
  - `POST /api/reporting/generate` → reporting queue
- Worker entrypoint: `backend/src/workers.ts`
- Jobs auto-enqueued with retries (exponential backoff)

**AI Orchestration**:
- Multi-provider support: Anthropic, OpenAI, Google, Mistral
- Live ranking: `POST /api/ai/orchestrate/live` aggregates signals from 8+ modules
- Ranking factors: severity, dependencies, role-based weights, SLA risk, historical feedback
- Feedback loop: `POST /api/ai/orchestrate/feedback` (accepted/rejected/executed)
- Signal aggregator runs hourly cycle to detect anomalies

**Database**:
- PostgreSQL with migration-based schema
- Migrations auto-applied on startup via `npm run db:migrate`
- Seed data in `db/seed.ts` for demo org and user
- Repositories in `db/repositories/` providing typed queries

## Engineering Guardrails

### ✅ DO

1. **Keep strict module boundaries**
   - Each module owns its routes, service logic, and database queries
   - Cross-module communication via HTTP endpoints (not direct imports)
   - Example: `finance` module should call `POST /api/bookings/:id` instead of direct queries

2. **Never bypass authentication middleware**
   - All auth-sensitive routes must use `requireAuth`
   - Never trust incoming org IDs—always validate via `res.locals.auth.organizationId`
   - Example: Always verify user has permission for requested org

3. **Prefer typed contracts between API and UI**
   - Share TypeScript types in `frontend/src/types/` and `backend/src/shared/types.ts`
   - Define request/response shapes explicitly (not loose `any` types)
   - Example: `export interface LiveOrchestrationResult { ... }`

4. **Add tests for behavioral changes**
   - Test files in `backend/src/test/`
   - Use Vitest with supertest for HTTP contracts
   - Example: Test that `POST /api/ai/orchestrate/live` returns `prioritizedActions` array

5. **Keep feature flags & environment settings explicit**
   - Use `.env.example` as source of truth
   - Feature gates in code: check `process.env.<FLAG>` explicitly
   - Example: `if (process.env.ANTHROPIC_API_KEY) { /* use Claude */ }`

6. **Use proper error handling**
   - Return typed errors with HTTP status codes
   - Use `errorHandler` middleware for 500s
   - Include `x-request-id` in logs for tracing

7. **Cache strategically**
   - Use Redis for hot data (cache tier in `src/shared/cache.ts`)
   - Set TTLs explicitly (not indefinite)
   - Example: Cache booking summaries for 5 minutes

8. **Log structured data**
   - Use `logger` utility in `src/shared/logger.js`
   - Include organization ID for multi-tenancy context
   - Example: `logger.info("Payment processed", { organizationId, transactionId })`

### ❌ DON'T

1. **Don't bypass `requireAuth` middleware**
   - Wrong: `router.get("/action", someRoute)` on auth-sensitive path
   - Right: `app.use("/api/bookings", requireAuth, bookingsRoutes)`

2. **Don't hardcode organization IDs**
   - Wrong: Assume org from request params without validation
   - Right: Extract from `res.locals.auth.organizationId` and treat as trusted

3. **Don't create circular module dependencies**
   - Wrong: `bookings` module → `finance` module → `bookings` module
   - Right: Create `shared/` utilities or use pub/sub via queues

4. **Don't commit `.env` files**
   - Only `.env.example` should be in git
   - `.env` is in `.gitignore`

5. **Don't ignore TypeScript strict mode**
   - Always satisfy `tsconfig.json` strict mode
   - Don't use `any` types without justification

6. **Don't ship untested database migrations**
   - Test locally before merging
   - Document breaking changes in PR

7. **Don't trust user input for organization context**
   - Wrong: `req.query.orgId` or `req.headers['x-org-id']` without validation
   - Right: `res.locals.auth.organizationId` from verified token

## Development Environment Setup

### Environment Variables (Backend)

Required for local development:
```
PORT=3000
DATABASE_URL=postgresql://kora:kora@localhost:5432/kora
REDIS_URL=redis://localhost:6379
CLERK_SECRET_KEY=sk_test_xxx
CLERK_AUTHORIZED_PARTIES=http://localhost:5173
AI_PROVIDER=claude
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
GOOGLE_API_KEY=
MISTRAL_API_KEY=
AI_BUDGET_USD_MONTHLY=100
```

### Environment Variables (Frontend)

```
VITE_API_BASE_URL=http://localhost:3000
VITE_DEV_BEARER_TOKEN=      # Dev token for local testing
VITE_ORG_ID=org_placeholder
```

## Key File Locations

### Backend

| Path | Purpose |
|------|---------|
| `src/app.ts` | Express app factory, middleware setup |
| `src/server.ts` | HTTP server entry point |
| `src/workers.ts` | BullMQ worker entrypoint |
| `src/middleware/auth.ts` | Clerk token validation |
| `src/middleware/errorHandler.ts` | Global error handler |
| `src/db/client.ts` | PostgreSQL connection pool |
| `src/db/migrate.ts` | Schema migration runner |
| `src/db/migrations/` | SQL migration files (numbered order) |
| `src/db/seed.ts` | Demo data seeder |
| `src/modules/*/routes.ts` | Module HTTP routes |
| `src/modules/*/service.ts` | Business logic |
| `src/db/repositories/*.ts` | Database queries |
| `src/workers/*.ts` | Job handlers |
| `src/queues/index.ts` | BullMQ queue definitions |
| `src/services/aiClient.ts` | AI provider factory |
| `src/shared/cache.ts` | Redis caching layer |
| `src/shared/logger.ts` | Structured logging |
| `src/test/*.test.ts` | Vitest test files |

### Frontend

| Path | Purpose |
|------|---------|
| `src/App.tsx` | Root router config |
| `src/main.tsx` | React entry point |
| `src/pages/` | Route components |
| `src/components/` | Reusable UI components |
| `src/hooks/` | Custom React hooks |
| `src/stores/` | Zustand store definitions |
| `src/services/api.ts` | HTTP client & API calls |
| `src/contexts/` | React context providers (Auth, Theme) |
| `src/types/` | TypeScript interface definitions |
| `vite.config.ts` | Vite bundler config |
| `tailwind.config.js` | Tailwind CSS config |
| `tsconfig.json` | TypeScript config |

## Common Patterns

### Creating a New Route

```typescript
// src/modules/feature/routes.ts
import { Router } from "express";
import { requireAuth } from "../../middleware/auth.js";

export const featureRoutes = Router();

featureRoutes.get("/", requireAuth, async (req, res, next) => {
  try {
    const orgId = res.locals.auth.organizationId;
    // Service logic here
    res.json({ module: "feature", data: [] });
  } catch (err) {
    next(err);
  }
});

// Register in app.ts
import { featureRoutes } from "./modules/feature/routes.js";
app.use("/api/feature", requireAuth, featureRoutes);
```

### Queuing an Async Job

```typescript
// Enqueue
import { enqueueNotification } from "../../queues/index.js";
await enqueueNotification({
  organizationId: orgId,
  channel: "email",
  payload: { /* email data */ }
});

// Handle in worker
const notificationsWorker = new Worker("notifications", async (job) => {
  if (job.name === "dispatch_notification") {
    // Process job.data
  }
});
```

### Caching DB Results

```typescript
import { getCachedJson, setCachedJson } from "../../shared/cache.js";

const cacheKey = `booking:${id}`;
let booking = await getCachedJson(cacheKey);
if (!booking) {
  booking = await queryDb("SELECT * FROM bookings WHERE id = $1", [id]);
  await setCachedJson(cacheKey, booking, 300); // 5 min TTL
}
```

### Multi-Tenancy Context

```typescript
// Always extract from verified auth context
const organizationId = res.locals.auth.organizationId;

// Use in queries
const rows = await queryDb(
  "SELECT * FROM bookings WHERE organization_id = $1",
  [organizationId]
);

// NEVER do this:
// const organizationId = req.query.orgId; // ❌ UNSAFE
```

## Testing Strategy

- **Unit tests**: Service functions, utilities
- **Integration tests**: API contracts (routes + mocks)
- **Example**: See `src/test/orchestration.test.ts` for pattern
- Run: `npm run test` or `npm run test:watch`

## Performance Considerations

1. **Database**: Use connection pooling (configured in `src/db/client.ts`)
2. **Caching**: Cache expensive queries (org-level summaries, dashboards)
3. **Job queue**: Use for slow operations (reports, notifications, ML inference)
4. **Rate limiting**: Express rate-limit configured on AI endpoints
5. **Pagination**: Return limited results (typically 100 rows max)

## Anti-Patterns to Avoid

1. **Direct database access from route handlers** → Use repositories
2. **Org ID from user input** → Use from auth token
3. **Sync operations blocking event loop** → Use BullMQ for slow tasks
4. **Untyped API responses** → Define interfaces
5. **Logging without context** → Include org/user/request IDs
6. **Bypassing auth middleware** → All sensitive routes require protection
7. **Magic strings for feature gates** → Use explicit env var checks

---

**Last Updated**: March 2026 | **Phase**: 06 (Operational Stability)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erpforgeafrica-dotcom)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/erpforgeafrica-dotcom)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
