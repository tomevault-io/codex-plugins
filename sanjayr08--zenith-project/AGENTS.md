# Zenith Pharmacy Inventory System - AI Agent Instructions

## Project Overview
**Zenith** is a full-stack, AI-driven pharmacy inventory management system combining:
- **Backend**: Node.js/Express (TypeScript) with PostgreSQL + Prisma ORM
- **Frontend**: React 19 + Vite + Tailwind CSS
- **AI Service**: Python FastAPI with LangChain + Anthropic Claude
- **Payment**: PayPal subscription integration
- **Real-time**: Socket.IO for live updates

**Core Value**: Implements strict FEFO (First-Expiry-First-Out) inventory tracking with AI-powered analytics, expiry prediction, and demand forecasting.

---

## Architecture Essentials

### Three-Tier Backend Design
1. **Routes** (`src/routes/*.ts`): Define endpoints, validate input
2. **Controllers** (`src/controllers/*.ts`): Handle requests, orchestrate services
3. **Services** (`src/services/*.ts`): Business logic, database transactions, external APIs

Example flow: `POST /api/v1/sales` → `sales.controller.ts` → `SalesService.ts` computes FEFO logic → `InventoryService.ts` deducts batch.

### FEFO (First-Expiry-First-Out) Engine
**Critical pattern in this codebase.** When processing a sale:
1. `SalesService.processSale()` retrieves all available batches for a drug
2. Sorts by `expiryDate` ascending
3. Deducts quantity from earliest expiring batch first
4. Creates `InventoryMovement` records for audit trail

See: `src/services/InventoryService.ts:35-80` (batch retrieval with expiry sorting)

### Database Schema (Prisma)
**Key models for FEFO**:
- `Drug`: Product catalog
- `PurchaseBatch`: Inventory batch with `expiryDate`, `quantityRemaining`, `receivedAt`
- `Sale` + `SaleItem`: Transaction records
- `InventoryMovement`: Immutable audit trail (PURCHASE, SALE, ADJUSTMENT)

**Critical**: Always use Prisma transactions for atomic multi-model operations (e.g., sale + movement + batch update).

See: `prisma/schema.prisma` for full schema; `src/services/SalesService.ts:45-100` for transaction example.

---

## Core Patterns & Conventions

### 1. Error Handling
Use custom `AppError` (extends Error) with `statusCode` and `message`:
```typescript
// src/utils/AppError.ts
class AppError extends Error {
  statusCode: number;
  constructor(message: string, statusCode: number) { ... }
}

// Usage in controllers (wrapped with catchAsync):
if (!inventory) throw new AppError("Not found", 404);
```
**All async route handlers** must wrap in `catchAsync()` middleware. See: `src/app.ts:global error handler` and `src/utils/catchAsync.ts`.

### 2. Middleware Stack (in order)
1. `helmet()`: Security headers
2. `cors()`: Cross-origin config
3. `express.json()`: Body parser
4. **Routes** (decorated with auth, RBAC)
5. `jwtMiddleware()`: Token verification (if route protected)
6. `rbacMiddleware()`: Role-based access (ADMIN vs STAFF)
7. Global error handler

See: `src/app.ts:Express configuration`

### 3. Authentication & Authorization
- **JWT**: Tokens stored client-side, verified per request
- **Roles**: `ADMIN`, `STAFF` in User model
- **RBAC Middleware**: `src/middleware/rbac.middleware.ts` checks `req.user.role`

Protected route example: `inventory.routes.ts` → `router.post("/", rbacMiddleware("ADMIN"), ...)`

### 4. TypeScript Patterns
- Strict mode enabled; leverage types for safety
- **Controllers return**: `{ status, data, message }`
- **Services return**: Typed models (Prisma types auto-generated)
- **No `any`**: Use discriminated unions for varying response types

### 5. Real-time Events (Socket.IO)
Socket events broadcast inventory/sales changes:
```typescript
// src/socket.ts
io.emit("inventoryUpdated", { batchId, quantityRemaining });
io.emit("saleProcessed", { saleId, totalPrice, itemsCount });
```
**Frontend listens** in `SocketContext.tsx` → Updates UI instantly without polling.

---

## Critical Developer Workflows

### Database Setup
```bash
npm install                  # Install deps
npx prisma migrate dev      # Run migrations (auto-create `dist/`)
npm run seed                # Populate test data (if seed.ts exists)
```

### Local Development
**Backend** (`npm run dev`):
- Runs `nodemon` with `ts-node` transpilation
- Watches `src/` for changes; recompiles
- Listens on `http://localhost:5000`

**Frontend** (`client/ npm run dev`):
- Vite dev server with HMR
- Listens on `http://localhost:5173`
- Proxies API calls to `http://localhost:5000` (via vite.config.ts)

**Python AI Service** (manual start):
```bash
cd python_ai && pip install -r requirements.txt
python app.py  # Listens on http://localhost:8000
```

### Building for Production
**Backend**: `npm run build` → TypeScript → `dist/` → `npm start`
**Frontend**: `npm run build` → Vite → `client/dist/` (static files)

### PayPal Webhook Integration
- `paypal.controller.ts` receives `POST /api/v1/paypal/webhook`
- Verifies signature via `paypal.service.ts`
- Creates `Subscription` + `Payment` records
- No manual confirmation needed; webhook-driven

---

## Integration Points & External Services

### PayPal API (`src/services/paypal.service.ts`)
- **Environment**: `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `PAYPAL_MODE` (sandbox/live)
- **Endpoints used**: Subscription creation, payment verification
- **Error handling**: PayPal errors wrapped in `AppError`

### Google Generative AI (Predictions)
- **Environment**: `GOOGLE_API_KEY`
- **Used in**: `src/services/PredictionService.ts` for demand/expiry forecasts
- **Fallback**: If API fails, return null; frontend shows "data unavailable"

### Anthropic Claude (Python AI Service)
- **Used in**: `python_ai/agent.py` with LangChain
- **Purpose**: Natural language chat about inventory, expiry alerts
- **Database**: Stores conversation history in PostgreSQL

### PostgreSQL + Prisma
- **Connection**: Via `DATABASE_URL` env var (handled in `src/config/db.ts`)
- **Migrations**: Tracked in `prisma/migrations/`
- **Client**: `src/prismaClient.ts` exports singleton (`new PrismaClient()`)
- **Note**: IPv4 forced for Windows compatibility (see `config/db.ts`)

---

## Project-Specific Patterns

### Data Upload & Analysis
`src/services/DatasetAnalysisService.ts`:
- Accepts CSV/XLSX files via `multipart/form-data`
- Stores in `uploads/` directory (created on first upload)
- Parses with `exceljs`/`papaparse`
- AI analyzes trends; creates `DatasetAnalysis` record

### Analytics Services (3 flavors)
1. **AnalyticsService**: Basic metrics (velocity, expiry risk per drug)
2. **SalesAnalyticsService**: Advanced (daily trends, cohort analysis)
3. **FeatureEngineeringService**: ML feature extraction (moving averages, seasonal trends)

Use the one matching your query complexity. Example: Sales velocity uses `AnalyticsService.calculateSalesVelocity()`.

### Subscription Validation Middleware
`src/middleware/subscription.middleware.ts`:
- Checks if `Subscription` is active (not expired)
- Used on premium features (e.g., predictions, advanced analytics)
- Throws 403 if subscription inactive

### Email System (Nodemailer + SMTP)
**New feature for international deployments:**
- `src/services/EmailService.ts`: Handles Nodemailer setup, email sending, logging, **and email retry logic**
- `src/config/email.ts`: SMTP configuration (reads from `.env`)
- `src/templates/`: HTML email templates (expiryAlert, lowStockAlert, authEmail, analyticsDigest)
- `src/services/StockAlertService.ts`: Monitors inventory, triggers low stock alerts to admins
- `src/jobs/analyticsEmailJob.ts`: Scheduled weekly digest (Monday 9 AM via node-cron)
- `src/controllers/admin.controller.ts`: Admin dashboard endpoints for email monitoring **and retry management**
- Database: `EmailLog` model tracks all outgoing emails with status/failure reasons/retry counts

**When emails are sent**:
- User registration → Welcome email
- After each sale → Low stock alert (if any drug drops below reorder point)
- Every Monday 9 AM → Analytics digest to all admins
- Manual retries possible via admin API

**Retry System** (NEW):
- Failed emails automatically scheduled for retry with exponential backoff (5min, 15min, 60min)
- Maximum 3 retry attempts per email
- Manual retry endpoints: `POST /api/v1/admin/email-logs/:id/retry` and `POST /api/v1/admin/email-logs/retry/trigger`
- See `EMAIL_RETRY_GUIDE.md` and `EMAIL_RETRY_IMPLEMENTATION.md` for details

**Configuration**: Set `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM` in `.env`. If not set, emails log to console (dev mode friendly).

**Admin Dashboard**: `GET /api/v1/admin/*` endpoints provide email metrics, logs, failure analysis, and retry management. See `ADMIN_EMAIL_DASHBOARD.md`, `EMAIL_RETRY_GUIDE.md` for full API docs.

---

## File Organization & Discovery

| **What** | **Where** |
|---|---|
| Add new API endpoint | Create route in `src/routes/`, controller in `src/controllers/`, service in `src/services/` |
| Add new database model | Edit `prisma/schema.prisma`, run `npx prisma migrate dev` |
| Add real-time event | Emit in service/controller, listen in `client/src/context/SocketContext.tsx` |
| Fix authentication issue | Check `src/middleware/jwt.middleware.ts` or `src/services/auth.service.ts` |
| Email system changes | Edit templates in `src/templates/`, service logic in `src/services/EmailService.ts` or `StockAlertService.ts` |
| Admin email metrics | Check `src/controllers/admin.controller.ts` and `/api/v1/admin/*` endpoints |
| Frontend API call | Use `client/src/services/api.ts` (Axios instance) |
| React context state | See `client/src/context/` (AuthContext, SocketContext) |
| Frontend components | Browse `client/src/components/` by feature (dashboard, sales, etc.) |

---

## Key Command Reference

**Backend**
- `npm install` - Install dependencies
- `npm run dev` - Development server (hot reload)
- `npm run build` - Compile TypeScript
- `npm start` - Run compiled backend
- `npm run seed` - Populate test data
- `npx prisma studio` - Interactive DB browser
- `npx prisma migrate dev --name <name>` - Create migration

**Frontend**
- `cd client && npm install`
- `npm run dev` - Vite dev server with HMR
- `npm run build` - Optimized production build
- `npm run lint` - ESLint check

---

## Common Gotchas & Patterns

✅ **DO**:
- Use `catchAsync()` wrapper for all async routes
- Validate input in controllers before calling services
- Use Prisma transactions for multi-model updates
- Check `req.user.role` for RBAC before processing
- Emit Socket.IO events after state changes for live UI updates
- Handle null/undefined batch quantities in FEFO logic (edge case: zero stock)

❌ **DON'T**:
- Make direct database calls in controllers (use services)
- Throw generic Errors (use AppError with statusCode)
- Modify request body directly; always validate first
- Forget to update `InventoryMovement` audit trail on stock changes
- Mix PayPal business logic in controllers (isolate in paypal.service.ts)
- Hardcode environment variables (always use `.env` injected via process.env)

---

## Testing & Quality Assurance
- **Unit tests** for AI features exist in `python_ai/tests/` (pytest)
- **E2E manual testing**: Seeded data in `prisma/seed.ts` provides dummy drugs/batches
- **Frontend builds**: Run `npm run build` to catch TypeScript errors
- **API validation**: Use Joi/custom validators in controllers (see `auth.controller.ts`)

---

## Getting Help
- **FEFO Logic**: Read `src/services/InventoryService.ts` + `SalesService.ts`
- **PayPal Integration**: Check `src/services/paypal.service.ts` + webhook handler
- **Database schema**: `prisma/schema.prisma` is the source of truth
- **Real-time updates**: Follow Socket.IO logic from `src/socket.ts` → Frontend `SocketContext.tsx`
- **Python AI**: See `python_ai/README.md` (if exists) or `python_ai/app.py` entry point

---

**Last Updated**: Feb 2025 | **Architecture**: Full-Stack (Node.js + React + Python) | **DB**: PostgreSQL via Prisma

---
> Source: [sanjayr08/zenith-project](https://github.com/sanjayr08/zenith-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
