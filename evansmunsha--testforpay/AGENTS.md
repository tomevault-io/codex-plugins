# AI Copilot Instructions for TestForPay

## Project Overview

**TestForPay** is a Next.js full-stack platform connecting app developers with paid Google Play beta testers. Two-sided marketplace with integrated payment processing, fraud detection, and role-based access control.

- **Tech Stack**: Next.js 16, React 19, TypeScript, Prisma ORM, PostgreSQL (Prisma Postgres), Stripe, Tailwind CSS, shadcn/ui
- **Key Domains**: Authentication, Job Management, Applications, Payments, Fraud Detection, PWA Support

## Architecture

### Route Groups & Layout Structure
- `app/(auth)/` — Public auth routes (login, signup, verify-email) with minimal centered layout
- `app/(dashboard)/` — Protected dashboard routes requiring authentication via `useAuth()` hook
- `app/api/` — REST API routes organized by domain (auth, jobs, applications, payments, admin, webhooks)
- No middleware.ts exists; auth protection is client-side via `useAuth()` + redirect in layout

### Data Model (Prisma Schema)
**Key Entities:**
- `User` — Roles (DEVELOPER, TESTER, ADMIN); Stripe integration (customerId/accountId); fraud tracking
- `TestingJob` — Apps needing testers; budget calculated as `paymentPerTester * testersNeeded + platformFee (15%)`
- `Application` — Tester applications to jobs; multi-stage verification (PENDING → APPROVED → VERIFIED → TESTING → COMPLETED)
- `Payment` — Escrow and transfer tracking; integrates with Stripe
- `DeviceInfo` — Tester Android device details (model, version)
- `Notification`, `PushSubscription` — Push notifications & PWA

**Fraud System:**
- `FraudLog` — Tracks suspicious activities
- `User.fraudScore` (0-100), `flagged`, `suspended` with `suspendReason`

### API Pattern (Route Handlers)

All API routes follow this pattern:
```typescript
export async function POST/GET(request: Request) {
  try {
    const currentUser = await getCurrentUser()  // JWT from cookie
    if (!currentUser) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    
    // Role check (DEVELOPER/TESTER/ADMIN)
    if (currentUser.role !== 'DEVELOPER') {
      return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
    }
    
    // Parse & validate with Zod schema
    const validated = createJobSchema.parse(body)
    
    // Business logic with Prisma
    // Return NextResponse.json()
  } catch (error) {
    if (error instanceof Error && error.name === 'ZodError') {
      return NextResponse.json({ error: 'Invalid input' }, { status: 400 })
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

**Key Points:**
- Use `getCurrentUser()` from `lib/auth.ts` (reads JWT from httpOnly cookie)
- Always validate input with Zod schemas from `lib/validators.ts`
- Include Prisma `select`/`include` to avoid N+1 queries
- Pass metadata to Stripe for tracking (jobId, developerId, etc.)

### Authentication Flow

**Token Management** (`lib/auth.ts`):
- Generate JWT: 7-day expiry, stores `userId`, `email`, `role`
- Set httpOnly cookie via `setAuthCookie()` (secure in production, lax sameSite)
- Get/verify token: `getAuthCookie()` + `verifyToken(token)`
- Current user: `getCurrentUser()` returns `TokenPayload | null`

**Client-Side Auth** (`hooks/use-auth.ts`):
- Fetches `/api/auth/session` on mount (client-side hook marked `'use client'`)
- Returns `user`, `loading`, `isDeveloper`/`isTester`/`isAdmin` computed flags
- Dashboard layout redirects to login if `!user && !loading`

**Note:** No Next.js middleware.ts; protection is layout-level redirect + hook checks.

## Key Patterns & Conventions

### Validation (Zod Schemas)
All input validated in `lib/validators.ts`. Example:
```typescript
export const createJobSchema = z.object({
  appName: z.string().min(3),
  testersNeeded: z.number().min(12).max(500),
  paymentPerTester: z.number().min(5).max(100),
})
export type CreateJobInput = z.infer<typeof createJobSchema>
```
- Use `schema.parse(input)` in API routes (throws on error)
- Reuse types with `z.infer<typeof schema>`

### Stripe Integration (`lib/stripe.ts`)
- **Job Creation Payment**: `createJobPaymentIntent()` creates PaymentIntent; metadata includes jobId, developerId
- **Tester Onboarding**: `createConnectedAccount()` creates Stripe Express account; `createAccountLink()` for onboarding flow
- **Transfers**: `transferToTester()` sends escrowed funds to tester's account
- **Refunds**: `refundPaymentIntent()` issues partial refunds to developer if job is cancelled
- **API Version**: `2025-12-15.clover` (explicitly pinned)

### Job Cancellation with Partial Refunds
Job cancellation (`app/api/jobs/[id]/route.ts`) handles:
1. **Compensation based on progress**:
   - COMPLETED: 100% payment
   - TESTING: 75% payment
   - VERIFIED: 50% payment
   - OPTED_IN: 25% payment
   - APPROVED/PENDING: 0% (no work done)
2. **Payment processing**:
   - Updates Payment records with compensation amounts
   - Sets status to `REFUNDED` for audit trail
   - Calculates platform fee (15%) on actual payouts
3. **Developer refund**:
   - Calls `refundPaymentIntent()` for unused funds
   - Returns detailed breakdown: tester payouts, platform fee, developer refund
4. **Response includes `breakdown`**:
   - `totalBudgetPaid`, `totalTesterPayouts`, `platformFeeOnPayouts`, `developerRefund`
   - `payoutDetails[]` — each tester's status and compensation

### Notification System
**Push Notifications** (`lib/notifications.ts`):
- `sendNotification(data)` creates in-app notification + sends web push to all subscriptions
- Uses web-push library with VAPID keys for PWA support
- Includes fallback for invalid subscriptions (auto-removes stale endpoints)
- All notifications include: title, body, url (navigation link), type (category)

**Notification Flow:**
1. **Job Posted** (Stripe webhook `checkout.session.completed`):
   - 🚀 Notify **developer**: "Job Published! Your testing job is now live"
   - 📱 Notify **all testers**: "New Job Available! New testing opportunity available"

2. **Tester Applies** (`/api/applications` POST):
   - Notify **developer**: "New Application - [Tester] applied to test [App]"
   - Send email via Resend (parallel)

3. **Application Approved** (`/api/applications/[id]` PATCH approve):
   - 🎉 Notify **tester**: "Application Approved! Start testing now"
   - Send email via Resend

4. **Application Rejected** (`/api/applications/[id]` PATCH reject):
   - Notify **tester**: "Application Update - Not approved this time"
   - Delete Payment record (refund escrow)

5. **Verification Approved** (`/api/applications/[id]` PATCH verify):
   - ✅ Notify **tester**: "Verification Approved! Testing period started"
   - 🎯 Notify **developer**: "Tester Verified! [Tester] started testing"
   - Update Payment status to PROCESSING
   - Send email via Resend

6. **Testing Completed** (auto-completion cron `/api/cron/auto-complete-tests`):
   - Application status: TESTING → COMPLETED
   - Payment status: PROCESSING (ready for payout)
   - Runs at 2 AM UTC (before payout cron at 3 AM)

7. **Payment Received** (Stripe webhook `transfer.created`):
   - 💰 Notify **tester**: "Payment Received! You've received $X for testing [App]"
   - Payment status: PROCESSING → COMPLETED
   - Transfer to tester's Stripe Connect account

**Notification Templates** (pre-defined in `NotificationTemplates`):
- `applicationApproved`, `applicationRejected`, `verificationApproved`
- `testingComplete`, `paymentReceived`, `newApplication`, `newVerification`, `newFeedback`

**Implementation Details:**
- Notifications sent via `sendNotification()` in API route handlers
- Wrapped in try-catch to not block main operation if notification fails
- Logged for debugging: `console.error()` on failure
- Web push requires VAPID_PUBLIC_KEY and VAPID_PRIVATE_KEY env vars
- PushSubscription table stores endpoint + encryption keys per user

### UI Components
- **shadcn/ui** (Radix-based): Button, Card, Input, Form, Select, Tabs, Dialog, Dropdown, etc.
- **Lucide Icons**: Single-import pattern (e.g., `import { Users, Briefcase } from 'lucide-react'`)
- **Toast Notifications**: Use `useToast()` hook (custom, not external library)
- **Tailwind v4**: Class-based (clsx for conditionals), no CSS files needed for components

### Error Handling
- API routes catch errors and return `{ error: string, status: number }`
- Zod validation errors caught as `error.name === 'ZodError'` → 400 response
- Frontend uses `useToast()` to display errors: `toast.error('message')`

### Database Queries
- Use `prisma` singleton from `lib/prisma.ts` (Prisma Postgres adapter + connection pool)
- Always `include` related models to avoid N+1 (e.g., `include: { developer: { select: { id, name } } }`)
- Use `select` for fields to minimize payload
- Add `@@index` on frequently queried fields (e.g., `developerId`, `status`)

### Page & Hook Structure
- **Layout pages** (`layout.tsx`): Server-side, set metadata, provide layout structure
- **Client pages** (`page.tsx`): Marked `'use client'`, use hooks, handle state
- **Custom Hooks**: `use-auth.ts`, `use-applications.ts`, `use-jobs.ts` (all marked `'use client'`)
  - Return loading state before rendering
  - Example: `const { user, loading } = useAuth(); if (loading) return <Spinner />`

### Constants & Config
- `lib/constants.ts` — Platform config (fees 15%, min testers 10, min payment $5, etc.)
- Environment variables accessed directly (process.env); Prisma uses DATABASE_URL, JWT_SECRET, STRIPE_SECRET_KEY, etc.
- No .env.example documented; infer from usage

## Development Workflow

### Build & Dev
```bash
npm run dev                    # Next.js dev server (includes Prisma generate)
npm run build                  # Requires `prisma generate` first
npm run prisma:generate        # Regenerate Prisma client
npm run prisma:push            # Sync schema to DB (dev only)
npm run prisma:studio          # GUI for DB inspection
npm run prisma:migrate         # Create/apply migrations
```

**Note:** `package.json` `postinstall` script runs `prisma generate` automatically.

### Linting
```bash
npm run lint                   # ESLint (eslint-config-next)
```

### Cron Jobs
- Defined in `vercel.json` (Vercel Crons)
- **Auto-completion**: `/api/cron/auto-complete-tests` runs at 2 AM UTC daily
  - Auto-completes applications after testing period expires
  - Moves Application status TESTING → COMPLETED
  - Ensures Payment records are PROCESSING (ready for payout)
- **Payout processing**: `/api/cron/process-payouts` runs at 3 AM UTC daily
  - Transfers funds from escrow to testers' Stripe Connect accounts
  - Updates Payment status PROCESSING → COMPLETED
- Both crons check `CRON_SECRET` header for authentication

### Testing & Debugging
- No Jest/Vitest configured (implied from workspace)
- Use `prisma studio` for DB inspection
- Use `console.error()` for error logging (visible in server logs)

## Common Tasks

### Adding a New API Endpoint
1. Create `app/api/resource/route.ts`
2. Extract user: `const currentUser = await getCurrentUser()`
3. Validate input: `const validated = schema.parse(body)`
4. Query Prisma with includes: `prisma.model.findMany({ include: { ... } })`
5. Return `NextResponse.json({ success: true, data })`

### Adding a New Dashboard Page
1. Create `app/(dashboard)/dashboard/feature/page.tsx` marked `'use client'`
2. Use `const { user } = useAuth()` for role checks
3. Import UI components from `components/ui/`
4. Use Tailwind for styling (no CSS files)
5. Call API routes via `fetch('/api/...')` with error handling

### Modifying Prisma Schema
1. Edit `prisma/schema.prisma`
2. Run `npm run prisma:migrate -- --name description_of_change`
3. New migration created in `prisma/migrations/`
4. `prisma generate` runs automatically; Prisma client regenerated

### Integrating with Stripe
- Import `{ stripe }` from `lib/stripe.ts`
- Create PaymentIntent with metadata: `await stripe.paymentIntents.create({ amount, currency: 'usd', metadata: { jobId } })`
- Handle Stripe webhook at `app/api/webhooks/stripe/route.ts`
- Test with Stripe CLI forwarding

## File Structure Reference

```
lib/                      # Utilities & services
  auth.ts                 # JWT & cookie handling
  validators.ts           # Zod schemas
  stripe.ts               # Stripe API wrapper
  prisma.ts               # Prisma singleton
  fraud-detection.ts      # Fraud scoring
  notifications.ts        # Push notifications
  email.ts                # Email sending (Resend)
  constants.ts            # Platform config

components/
  ui/                     # shadcn/ui components (button, card, form, etc.)
  dashboard/              # Dashboard layouts (sidebar, nav)
  auth/                   # Auth forms
  applications/           # Job application UI
  jobs/                   # Job listing & creation UI

hooks/
  use-auth.ts             # Auth context hook (session + role checks)
  use-jobs.ts             # Job CRUD hook
  use-applications.ts     # Application CRUD hook

app/
  api/                    # REST endpoints organized by domain
    auth/                 # /api/auth/login, /signup, /logout, /session
    jobs/                 # /api/jobs (CRUD)
    applications/         # /api/applications (list, apply)
    payments/             # /api/payments (Stripe integration)
    admin/                # /api/admin/* (stats, users, jobs, etc.)
    webhooks/stripe/      # /api/webhooks/stripe
  (auth)/layout.tsx       # Centered auth layout
  (dashboard)/layout.tsx  # Sidebar dashboard layout (protected by useAuth)
```

## Notes for AI Agents

- **Assumptions**: Always use client-side hooks (`useAuth()`) for protected routes; no middleware intercepts
- **Routing**: Use Next.js App Router conventions; group routes in parentheses for layout sharing
- **Types**: Prefer type inference from Zod (`z.infer<typeof schema>`) over manual type definitions
- **Errors**: Return consistent error objects with `status` codes; frontend expects `{ error: string }`
- **Security**: JWT in httpOnly cookies (not localStorage); role-based checks on every API route
- **Performance**: Always `include` Prisma relations; avoid N+1 queries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evansmunsha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
