# Copilot Instructions — Telepharmacy ERP

## Architecture

Turborepo + pnpm monorepo: NestJS 11 API (`apps/api` :3000), Next.js 15 admin dashboard (`apps/admin` :3001), Next.js 15 + LINE LIFF shop (`apps/shop` :3002). Shared packages: `@telepharmacy/db` (Drizzle ORM), `@telepharmacy/shared` (Zod validators, types, Thai constants/utils), `@telepharmacy/ai` (Gemini OCR, chatbot, drug-checker). Infrastructure: PostgreSQL 16, Redis 7 (BullMQ), Meilisearch, MinIO, Traefik.

## Commands

```bash
pnpm install                 # Install all
docker compose up -d         # Start Postgres, Redis, Meilisearch, MinIO
pnpm db:push                 # Push schema to DB (dev)
pnpm dev                     # All apps via Turborepo
pnpm dev:api                 # API only
pnpm --filter @telepharmacy/api test -- --testPathPattern=<pattern>  # Single test
pnpm db:generate && pnpm db:migrate  # Production migration workflow
pnpm db:seed                 # Seed drugs, staff, allergy groups
```

## NestJS API Conventions (`apps/api/src/`)

- **Global prefix `/v1`** is set in `main.ts` — do NOT add `v1/` in `@Controller()` decorators
- **Module pattern:** `modules/<name>/` → `module.ts`, `controller.ts`, `service.ts`, `dto/`, `index.ts` (barrel)
- **Auth decorators** imported from sibling `../auth` barrel:
  - `@Public()` — skip JWT guard
  - `@Roles('pharmacist', 'admin')` — restrict to staff roles
  - `@PatientOnly()` — restrict to LINE-authenticated patients
  - `@CurrentUser()` — inject `RequestUser` from JWT payload
  - `@SkipResponseWrap()` — return raw response (LINE webhook)
- **3 global guards** (registered via `APP_GUARD` in AuthModule, order matters): `JwtAuthGuard` → `RolesGuard` → `PatientOnlyGuard`
- **Response envelope** — `ResponseInterceptor` wraps all returns as `{ success: true, data, message }`. Errors via `HttpExceptionFilter` as `{ success: false, error: { code, message, details } }`
- **Drizzle injection:** `@Inject(DRIZZLE)` — `DatabaseModule` is global, no need to import. Schema tables imported directly from `@telepharmacy/db`
- **Dual controllers pattern:** Patient-facing uses `@PatientOnly()` + `@CurrentUser()`, staff-facing uses `@Roles(...)`. Prefix staff routes with `staff/` (e.g., `@Controller('staff/orders')`)
- **Validation:** Global `ValidationPipe` (whitelist + transform) for class-validator DTOs; Zod schemas via `@anatine/zod-nestjs` for specific endpoints
- **Thai error messages** in user-facing validators from `@telepharmacy/shared/validators`

## Database Schema (`packages/db/src/schema/`)

- **IDs:** `uuid("id").primaryKey().defaultRandom()` — never use serial/integer PKs
- **Columns:** `snake_case` in SQL, `camelCase` in Drizzle TS (e.g., `created_at` → `createdAt`)
- **Timestamps:** Every table has `createdAt`/`updatedAt` with `{ withTimezone: true }.defaultNow()`. Soft-delete tables add `deletedAt`
- **Money/quantities:** Use `numeric()` (never float). Counts use `integer()`
- **Business IDs:** Human-readable codes (`orderNo`, `rxNo`, `patientNo`) are `varchar` + `unique`, separate from UUID PK
- **Enums:** Defined in `schema/enums.ts` as Postgres `pgEnum`. Mirrored in `@telepharmacy/shared/types/enums.ts` as `as const` tuples + union types
- **Relations:** Centralized in `schema/relations.ts`, separate from table definitions
- **Indexes:** Convention `{table}_{column}_idx` in table's third argument

## Frontend — Admin (`apps/admin/src/`)

- **Auth:** Cookie-based JWT (`access_token` cookie). Middleware reads cookie, base64-decodes JWT, checks `type === 'staff'`. No server-side verification — delegated to API
- **API calls:** Use `apiFetch(path)` from `lib/api-client.ts` — auto-injects Bearer token, auto-refreshes on 401 with dedup, unwraps `data` from envelope. SWR hook: `useApi<T>(path)` from `lib/use-api.ts`
- **Route structure:** `/login` (public), `/dashboard/**` (protected). Middleware redirects to `/login?from=<path>`

## Frontend — Shop (`apps/shop/src/`)

- **Auth:** LINE LIFF SDK → exchange LIFF token for JWT via API → stored in Zustand `authStore` (persisted to `reya-auth` localStorage key)
- **State:** Zustand stores in `store/` — `cart.ts` (persisted `reya-cart`, save-for-later), `auth.ts` (persisted `reya-auth`), `address.ts`
- **API calls:** `shopFetch(path, token)` from `lib/api-client.ts` — manual token param, no auto-refresh
- **LIFF helpers:** `lib/liff.ts` — lazy-loaded singleton. Use `liffLogin()`, `liffLogout()`, `getLiffProfile()`, `isInLiff()`
- **UI language:** Thai (ภาษาไทย) — all user-facing text in Thai

## Shared Package Conventions (`@telepharmacy/shared`)

Sub-path imports — `@telepharmacy/shared/types`, `/validators`, `/constants`, `/utils`:
- **types/enums.ts** — mirrors DB pgEnums as `as const` tuples + union types (source of truth for both API and frontend)
- **validators/** — Zod schemas with Thai error messages for auth, patient, order, prescription, product
- **constants/** — Thai drug classification labels, order status flow machine, membership tiers, business constants
- **utils/** — Thai date formatting (Buddhist Era +543), ID generators (`REYA-YYYYMMDD-###`), THB currency formatter

## Key Patterns

- **Dockerfile build args:** `NEXT_PUBLIC_*` vars must be declared as both `ARG` and `ENV VARNAME=${VARNAME}` (not `ENV VARNAME=`) for Next.js build-time baking
- **Config namespaces:** `registerAs()` pattern in `apps/api/src/config/` — access via `this.configService.get('jwt.secret')`
- **BullMQ queues:** `ocr-queue` (prescription OCR), `adherence-queue` (medication reminders) — require Redis
- **Soft deletes:** Patient queries must check `deletedAt IS NULL`
- **Thai Pharmacy Act compliance:** Drug classification (ยาอันตราย/ยาควบคุมพิเศษ) affects dispensing rules — see `spec/telepharmacy-deep-dive.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reyatelehealth2026-crypto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
