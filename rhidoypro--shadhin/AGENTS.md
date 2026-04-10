# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Commands

```bash
npm run dev        # Start Next.js dev server
npm run build      # Prisma generate + Next.js build
npm run start      # Start production server
npm run lint       # Run ESLint
npm run email      # Start React Email dev server (preview email templates)
```

There are no automated tests in this project.

## Stack

- **Framework:** Next.js 14 App Router (TypeScript strict)
- **Database:** MongoDB via Prisma ORM (`prisma/schema.prisma`)
- **Auth:** NextAuth.js v5 beta — JWT sessions, Google OAuth + credentials
- **Storage:** Cloudflare R2 — bucket `shadhin` (APAC), public URL: `https://pub-81b012e4d7214ac491438e1df8c5bf00.r2.dev`
- **Cache / Rate limiting:** Upstash Redis (`shadhin-ratelimit`, Singapore) — `lib/cache.ts` + `lib/rate-limit.ts`
- **Email:** Resend — sender `Shadhin.io <help@shadhin.io>`, templates in `emails/`, sent via `lib/mail.ts`
- **UI:** shadcn/ui + Tailwind CSS + Radix UI, dark/light via `next-themes`
- **Payments:** Manual bKash — user submits TrxID, admin approves in dashboard
- **Moderation:** OpenAI free moderation API + keyword fallback (`lib/moderation.ts`)
- **Push:** web-push (VAPID), service worker in `public/`
- **Analytics:** Vercel Analytics + Speed Insights, GA4, Microsoft Clarity, PostHog (`posthog-js`), Meta Pixel — all gated behind cookie consent (`lib/consent.ts`, `components/ConsentGate.tsx`)

> ⚠️ Old AWS S3 (`utopia-web-app.s3.ap-south-1.amazonaws.com`) is gone — that account was deleted. Fallback UI is in place. All new uploads go to Cloudflare R2.

## Architecture Overview

Shadhin.io is a social/community platform for Bangladesh. Repo: `github.com/RhidoyPro/Shadhin` (branch: `master`). Deployed on Vercel Hobby plan.

### Route Structure

- `app/(auth)/` — Public auth pages (login, signup, verify-email, forgot-password)
- `app/(protected)/` — Authenticated routes with shared Navbar layout
  - `feed/` — Main feed (algorithm-ranked events)
  - `events/[stateName]/` — Events filtered by Bangladesh district
  - `live-chat/[stateName]/` — Real-time district-based chat rooms (HTTP polling)
  - `user/[userId]/` — User profiles
  - `leaderboard/` — Points-based leaderboard
  - `bookmarks/` — Saved events
  - `search/` — Search page
  - `settings/` — Password change, account deletion, org badge application
  - `admin/` — Admin dashboard (ADMIN role only)
    - `users/`, `events/`, `broadcast/`, `audit-log/`, `promotions/`, `org-verification/`, `tickets/`
- `app/api/auth/[...nextauth]/` — NextAuth handler
- `app/api/cron/` — Cron job endpoints (called by Vercel scheduler)
- `app/api/webhooks/email/` — Inbound email support handler (Claude Haiku auto-reply)
- `app/privacy/`, `app/terms/` — Legal pages

### Key Patterns

**Server Actions** (`actions/`): All mutations use `"use server"` server actions — never inline API routes for mutations. Always validate with Zod, then `revalidatePath`/`revalidateTag`.

**Data Layer** (`data/`): Read-only DB queries separated from actions. Import `db` from `@/lib/db` (Prisma singleton). Keep queries out of components.

**Caching** (`lib/cache.ts`): Two-level Redis cache for the feed:
- `feed-pool:{stateName}` — top 100 raw events, 5-min TTL, shared across all users in that state
- `viewer-ctx:{userId}` — user's follow list + home state, 30-min TTL
- Invalidate with `invalidateFeedCache(stateName)` on create/edit/delete/like/comment
- Invalidate with `invalidateViewerContext(userId)` on follow/unfollow

**Rate Limiting** (`lib/rate-limit.ts`): Upstash Redis sliding window with in-memory fallback for local dev. All actions are rate-limited. Prisma `DATABASE_URL` errors in local dev are expected — no local DB.

**Auth** (`auth.config.ts` + `auth.ts` + `middleware.ts` + `routes.ts`):
- `auth.config.ts` — Edge-safe config (no bcrypt), used by middleware
- `auth.ts` — Node.js config (with bcrypt), used by server actions
- `middleware.ts` — Route protection using `routes.ts` arrays (public / auth-only / admin-only)
- Session includes `role` and `id` (augmented in `next-auth.d.ts`)

**Feed Algorithm** (`lib/feed-algorithm.ts`): Weighted scoring — engagement (likes×1, comments×2, attendees×3) / hours^1.5 × district bonus × following bonus + promoted boost. Pool of 100, fetchMultiplier=5. Freshness floor: +1 so zero-engagement posts score positively.

**File Uploads**: Cloudflare R2 with pre-signed URLs. Media URLs stored in MongoDB.

**Analytics & Consent** (`utils/analytics.ts`, `lib/consent.ts`): Central `track()` dispatches to GA4 + Clarity + PostHog + Meta Pixel. All non-essential trackers wrapped in `<ConsentGate category="analytics|marketing">` in layout.tsx. Consent stored in localStorage + cookie. `CookieConsent` banner shown on first visit. Behavioral trackers: SessionDepthTracker, DistrictSwitchTracker, useContentViewTracking hook.

**Email** (`emails/`, `lib/mail.ts`): React Email templates via Resend. Batch limit 50 recipients. Templates: Welcome, Nudge, WeeklyDigest, AdminSummary, EventReminder, NewDistrictMember, LeaderboardAlert.

**Email Support Automation** (`app/api/webhooks/email/route.ts`): Inbound emails to help@shadhin.io → classified by Claude Haiku (simple_question / complaint / legal_urgent) → auto-reply or escalate to admin.

**Moderation** (`lib/moderation.ts`): OpenAI moderation API + keyword fallback. Strike system — auto-suspend at 3 strikes. Suspended users redirected to `/suspended`.

**Bot System** (`app/api/cron/bot-posts/`): 18 bots (2 per district + 2 global), `isBot: true` on User. Post Bengali content from RSS headlines. Cross-day dedup, bot-to-bot liking (60% chance), CDATA filter.

**Push Notifications**: PushSubscription model, web-push VAPID, service worker. Triggered on like/comment/event-attend.

**Real-time**: HTTP polling — chat every 4s (`GET /api/chat/messages`), notifications every 10s (`GET /api/notifications/poll`). No Socket.IO (old Render server can be deleted).

### Cron Jobs (Vercel — daily limit on Hobby plan)

All crons are daily-frequency (Hobby plan constraint):
- `snapshot-leaderboard` — Sunday midnight
- `welcome-sequence` — daily 6am UTC
- `weekly-digest` — Monday 3am UTC
- `admin-summary` — daily 2am UTC
- `event-reminders` — daily 8am UTC
- `bot-posts` — daily 9am UTC

### Database Models

`User`, `Account`, `VerificationToken`, `ForgotPasswordCode`, `Event`, `Comment`, `Like`, `EventAttendee`, `Message`, `Notification`, `Follow`, `Report`, `Bookmark`, `PushSubscription`, `AuditLog`, `PromotionRequest`, `OrgVerificationRequest`, `Ticket`

Enums: `UserRole`, `EventType`, `EventStatus`, `RequestStatus`

### Revenue Features (bKash manual verification)

- **Ticketing**: `Ticket` model — `ticketPrice` + `maxAttendees` on Event, 5% platform fee. `actions/ticket.ts`
- **Promoted posts**: `PromotionRequest` — 3d/৳50, 7d/৳100, 14d/৳200 tiers. `actions/promotion.ts`
- **Org badge**: `OrgVerificationRequest` — one-time fee, amber badge on profile. `actions/org-verification.ts`
- bKash number: `process.env.NEXT_PUBLIC_BKASH_NUMBER` (currently placeholder — needs real merchant number)

### Environment Variables

See `.env.example`. Required: `DATABASE_URL`, `AUTH_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `RESEND_API_KEY`, `FRONTEND_URL`, `CLOUDFLARE_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET_NAME`, `R2_PUBLIC_URL`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, `NEXT_PUBLIC_VAPID_PUBLIC_KEY`, `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `OPENAI_API_KEY`, `ADMIN_EMAIL`, `NEXT_PUBLIC_BKASH_NUMBER`, `NEXT_PUBLIC_MEASUREMENT_ID`. Optional: `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`, `NEXT_PUBLIC_CLARITY_ID`, `NEXT_PUBLIC_META_PIXEL_ID`.

### Path Aliases

`@/*` maps to the project root (e.g., `@/lib/db`, `@/components/ui/button`, `@/actions/event`).

### UI Components

shadcn/ui components in `components/ui/`. Add: `npx shadcn@latest add <component>`. Designs come from v0.dev — integrate fully, don't leave components disconnected.

## Coding Rules

- **TypeScript always** — no `.js` files, strict mode on
- **All mutations in `actions/`** — `"use server"`, never inline API routes for mutations
- **Tailwind for all styling** — no inline styles, no CSS modules
- **Zod validation** — all user input validated at action level via `utils/zodSchema.ts`
- **Data reads in `data/`** — keep DB queries out of components and actions
- **Rate limit all actions** — use `lib/rate-limit.ts` with Upstash Redis
- **Never expose secrets** — all credentials server-side only; `NEXT_PUBLIC_*` vars are client-safe
- **Invalidate cache after mutations** — always call `invalidateFeedCache`/`invalidateViewerContext` as appropriate
- **Local DATABASE_URL errors are expected** — no local DB, Prisma errors during local dev are normal

## Pending

- **NEXT_PUBLIC_BKASH_NUMBER** — Add real bKash merchant number to Vercel env
- **Mobile app (Step 23)** — Expo React Native, separate repo — only remaining major milestone
- **User growth** — Platform is feature-complete; needs real BD users (Facebook groups, university communities)
- **Render Socket.IO server** — Can be safely deleted (already migrated to polling)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RhidoyPro)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/RhidoyPro)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
