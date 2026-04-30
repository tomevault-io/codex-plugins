# Copilot Instructions for IESA Platform

## Project Overview

IESA is a departmental web platform built with Next.js 16 (App Router) and a FastAPI backend. It features JWT authentication, a dashboard with protected routes, and a **vibrant multi-color design system** using Tailwind CSS v4.

**Institution:** University of Ibadan (UI), Nigeria

---

## 🎨 DESIGN SYSTEM v3: Vibrant Academic

### Design Philosophy

Bold, vibrant, multi-color design inspired by modern card-based editorial layouts. Colorful bento grids, thick-bordered cards, playful but professional.

**Core Aesthetic Principles:**

- **Multi-Color Vibrancy** — Full palette: lime, lavender, coral, teal, sunny yellow
- **Thick & Chunky** — 4-8px borders, large radius (16–24px), bold button shapes
- **Hard Shadows** — Pure black/navy shadows (5-10px offset), never lime shadows
- **Bento Everything** — Asymmetric card grids with colored blocks, varied sizes
- **Typography-Led** — TBJ Endgraph black weight headlines, brush highlight accents
- **Playful-Professional** — Diamond sparkle decorators, fun but polished
- **Single Theme** — Light mode only, no dark mode support

---

## 🎨 TAILWIND CSS v4 THEME SYSTEM

### Theme Architecture

```css
/* 1. STATIC COLORS - in @theme, support opacity modifiers */
@theme {
  --color-lime: oklch(88% 0.2 128);
  --color-navy: oklch(15% 0.02 280);
}
/* Usage: bg-lime, text-navy, bg-lime/50, text-navy/60 */

/* 2. DYNAMIC COLORS - CSS variables that change with theme */
:root { --surface: #FAFAFE; }
.dark { --surface: oklch(15% 0.02 280); }

/* 3. DYNAMIC UTILITIES - Map CSS vars to Tailwind via @theme inline */
@theme inline { --color-surface: var(--surface); }
/* Usage: bg-surface (auto-switches with dark mode) */
```

### Color Usage Rules

**For static colors (don't change with theme):**
```jsx
// ✅ CORRECT - Uses @theme colors, supports opacity
<div className="bg-lime text-navy">
<div className="bg-lime/50 text-navy/70">
<div className="bg-lavender text-navy border-lime/30">

// ❌ WRONG - Raw hex or old color names
<div className="bg-[#C8F31D]">
<div className="bg-green-accent">
```

**Shadow & Border Rules:**
```jsx
// ✅ CORRECT - Pure black/navy shadows, navy borders
<div className="bg-snow border-[4px] border-navy shadow-[8px_8px_0_0_#000]">
<button className="bg-lime border-[4px] border-navy shadow-[5px_5px_0_0_#0F0F2D]">

// ❌ WRONG - Never use lime shadows or lime borders with black shadows
<div className="shadow-[8px_8px_0_0_#C8F31D]"> // Never lime shadow
<div className="border-lime shadow-[8px_8px_0_0_#000]"> // Conflicts
```

### Available Static Colors

| Color | Value | Usage |
|-------|-------|-------|
| `lime` | `oklch(88% 0.2 128)` | Primary accent, CTAs, active states |
| `lime-light` | `oklch(95% 0.08 128)` | Lime tint backgrounds |
| `lime-dark` | `oklch(75% 0.2 128)` | Hover/pressed states |
| `lavender` | `oklch(72% 0.12 295)` | Secondary: info, tags, decorative |
| `lavender-light` | `oklch(92% 0.05 295)` | Lavender tint backgrounds |
| `coral` | `oklch(68% 0.18 25)` | Alerts, deadlines, important |
| `coral-light` | `oklch(92% 0.06 25)` | Coral tint backgrounds |
| `teal` | `oklch(78% 0.14 175)` | Success, completion, positive |
| `teal-light` | `oklch(94% 0.05 175)` | Teal tint backgrounds |
| `sunny` | `oklch(85% 0.14 90)` | Warnings, stars, ratings |
| `sunny-light` | `oklch(96% 0.05 90)` | Yellow tint backgrounds |
| `navy` | `oklch(15% 0.02 280)` | Dark text, dark backgrounds |
| `navy-light` | `oklch(22% 0.02 280)` | Elevated dark surfaces |
| `navy-muted` | `oklch(35% 0.01 280)` | Secondary dark text |
| `slate` | `oklch(55% 0.01 280)` | Muted text, placeholders |
| `cloud` | `oklch(92% 0.005 280)` | Light borders, dividers |
| `ghost` | `oklch(97% 0.003 280)` | Off-white background |
| `snow` | `#ffffff` | White card surfaces |

### Shadow System Rules

**CRITICAL:** Follow these shadow rules strictly:

1. **Light backgrounds** (snow, ghost, lime-light, etc.) → Use pure black shadow: `shadow-[Xpx_Ypx_0_0_#000]`
2. **Lime buttons/elements** → Use navy shadow: `shadow-[Xpx_Ypx_0_0_#0F0F2D]`
3. **Dark backgrounds** (navy, navy-light) → Use lime shadow: `shadow-[Xpx_Ypx_0_0_#C8F31D]`
4. **NEVER use lime shadow on light backgrounds**
5. **Badges** (small labels like "Est. 2018") → No shadow at all

### Border System Rules

1. **Pair navy borders with black/navy shadows** → `border-navy shadow-[X_X_0_0_#000]`
2. **Never use lime borders when shadow is black** → This creates visual conflict
3. **Standard thickness:** 4-8px borders → `border-[4px]` or `border-[6px]`

---

## Typography System

**Font Families:**
- **Display Font:** `TBJ Endgraph` — Headlines, titles (weight: 900/black)
- **Body Font:** `TBJ Endgraph` — All text content (weights: 100, 300, 400, 500, 700)

**Font Weights:**
```css
.font-display    /* Black: 900 - Use for all headlines */
.font-bold      /* Bold: 700 - Use for emphasis */
.font-medium    /* Medium: 500 - Use for subheadings */
.font-normal    /* Regular: 400 - Use for body text */
.font-light     /* Light: 300 - Use sparingly */
.font-thin      /* Thin: 100 - Use for subtle text */
```

**Type Scale:**
```css
.text-hero       /* Hero: clamp(3.5rem, 10vw, 8rem) */
.text-display-xl /* Section heads: clamp(3rem, 8vw, 5rem) */
.text-display-lg /* Page titles: clamp(2rem, 5vw, 3.5rem) */
.text-display-md /* Card titles: clamp(1.5rem, 3vw, 2.5rem) */
.text-display-sm /* Subheads: clamp(1.25rem, 2vw, 1.75rem) */
.text-label      /* Labels: 0.75rem, tracking 0.08em, uppercase */
.text-label-sm   /* Small labels: 0.625rem, tracking 0.12em */
.text-body       /* Body: tight tracking */
```

## Brush Highlight System

**Purpose:** Add colorful brush-stroke highlights under key words in headlines.

**Base Usage:**
```jsx
<h1><span className="brush-highlight">Industrial Engineering</span></h1>
// Default: lime background, works on white/snow backgrounds
```

**Context-Aware Variants:**
```jsx
// For coral/sunny backgrounds: use sunny yellow brush
<h3 className="bg-coral">
  <span className="brush-highlight brush-coral">Our Mission</span>
</h3>

// For lime backgrounds: use coral brush
<h3 className="bg-lime">
  <span className="brush-highlight brush-lime">Start Your</span>
</h3>

// For navy backgrounds: use sunny yellow brush
<h3 className="bg-navy">
  <span className="brush-highlight brush-navy">Join Us</span>
</h3>
```

**Technical Notes:**
- Brush uses `::after` pseudo-element with `z-index: -1`
- Parent has `overflow: hidden` to prevent rotated brush from extending beyond bounds
- Variants use chained selectors: `.brush-highlight.brush-coral::after`
- Colors set with `!important` to override base style

## Decorator System

**Diamond Sparkles:**
```jsx
// Use 4-point diamond shapes (NOT 5-point stars)
<svg className="fixed top-16 left-[10%] w-6 h-6 text-lime/20" viewBox="0 0 24 24" fill="currentColor">
  <path d="M12 0l1.5 7.5L21 9l-7.5 1.5L12 18l-1.5-7.5L3 9l7.5-1.5z" />
</svg>
```

**Rules:**
- Sprinkle 6-10 sparkles across page using fixed positioning
- Use low opacity: 12-20% (`text-lime/20`, `text-coral/15`)
- Add `pointer-events-none` and `z-0` to prevent interaction issues
- Position with percentage values for responsive placement: `left-[10%]`, `right-[12%]`
- Extra sparkles near major headlines for emphasis
- Page must have `overflow-x: hidden` on body to prevent decorator overflow

## Student Live Quiz UI Art Direction (Mandatory)

When working on `src/app/(student)/dashboard/iepod/quizzes/page.tsx` live gameplay interfaces, prioritize a **game-show card experience** over generic dashboard blocks.

### Visual Direction

- **Mobile-first arena composition**: center the experience in a phone-like frame (`max-w-xl`, thick border, hard shadow), with optional side stat cards on large screens.
- **Layered card language**: use stacked/offset cards, playful rotations (`rotate-[-1deg]`, `rotate-[1deg]`), and asymmetric grouping.
- **High-contrast rhythm**: alternate `snow`, `lime-light`, `lavender-light`, `sunny-light`, `teal-light` sections for scannability.
- **Do not use flat, monotonous sections**: avoid repetitive same-color blocks with identical spacing.

### Live Phase UX Rules

- **Waiting**: emphasize readiness and connected players with a calm, friendly tone.
- **Question**: make options feel tactile and distinct (A/B/C/D chips + bold card choices).
- **Reveal**: present answer review as an editorial breakdown card with clear correctness cue.
- **Leaderboard**: center rank movement and score momentum with strong hierarchy and row personality.
- **Ended**: show a celebratory wrap-up state, not a plain status line.

### Styling Constraints For Live Quiz

- Keep design system tokens only (`lime`, `navy`, `snow`, etc.); no raw Tailwind gray/blue palettes.
- Continue press feedback classes for interactive controls (`press-*`).
- Keep hard-shadow look and thick borders from the platform style.
- Preserve accessibility: readable contrast, keyboard focus, and concise labels.
- Preserve deterministic phase rendering logic; styling must not alter live quiz sequencing behavior.

### Anti-Patterns To Avoid

- Reusing generic admin-card layouts in student live game screens.
- Overloading one giant card without hierarchy.
- Tiny, low-contrast timers/status text.
- Color noise without role meaning.
- Visual polish that delays or hides phase-critical feedback.

### UX Tone And Layout Direction (Required)

- If an interface feels "a bit too colorful with no direction", move to restrained tones, stronger spacing rhythm, and a clear focal hierarchy.
- Not everything has to be carded; choose layout primitives intentionally instead of forcing card wrappers everywhere.
- Prefer friendly, less technical copy. Use a human, game-like tone where appropriate.
- Reduce opacity aggression and visual noise: softer backgrounds, cleaner alignment, calmer side information placement.

### Execution Quality Gate (Required)

- Never ship half-fixes for quiz/live interfaces. If a requirement includes motion, sequencing, or control logic, complete behavior and visuals together.
- Do not stop at recoloring existing cards when asked for a game-show style redesign; restructure layout hierarchy and phase choreography.
- Before finishing, verify no stale state, dead controls, or contradictory phase labels remain in student/admin quiz screens.
- Do not take "shortcut" passes when asked for a revamp. If user requests parity with a reference interaction, implement the missing interaction (state flow, timing, controls, transitions) instead of surface-only styling.
- For multi-part quiz/live requests, close each requested behavior with explicit implementation and verification before moving on; unfinished items are not acceptable as final output.
- Mandatory completion counter for multi-part revamps: track and report `Completed/Requested` items in each substantial update; do not claim completion when any requested behavior is still partial.

---

## Card System

**Base Card Pattern:**
```jsx
<div className="bg-snow border-[4px] border-navy rounded-3xl p-6 shadow-[8px_8px_0_0_#000]">
  <!-- Card content -->
</div>
```

**Colored Card Variants:**
```jsx
// Lime card
<div className="bg-lime border-[6px] border-navy rounded-3xl p-8 shadow-[10px_10px_0_0_#000] rotate-[-1deg] hover:rotate-0 transition-transform">

// Coral card
<div className="bg-coral border-[4px] border-navy rounded-3xl p-6 shadow-[8px_8px_0_0_#000]">

// Lavender card
<div className="bg-lavender border-[4px] border-navy rounded-3xl p-6 shadow-[8px_8px_0_0_#000] rotate-[1deg]">

// Navy card (dark background)
<div className="bg-navy border-[4px] border-lime rounded-3xl p-6 shadow-[8px_8px_0_0_#C8F31D]">

// Teal card
<div className="bg-teal border-[4px] border-navy rounded-3xl p-6 shadow-[8px_8px_0_0_#000]">

// Sunny (yellow) card
<div className="bg-sunny border-[4px] border-navy rounded-3xl p-6 shadow-[8px_8px_0_0_#000]">
```

**Card Rules:**
- Max 2–3 colored cards per bento grid to avoid visual clutter
- Add slight rotation (`rotate-[-1deg]` or `rotate-[1deg]`) for playfulness
- Use `hover:rotate-0` to straighten on hover
- Border thickness: 4-6px for medium cards, 6-8px for hero cards
- Shadow offset: 8-10px for standard, 5px for small cards
- Navy cards get lime borders + lime shadow (exception to the rule)

---

## Button System

**Primary Pattern (Lime CTA):**
```jsx
<button className="bg-lime border-[4px] border-navy shadow-[5px_5px_0_0_#0F0F2D] 
  px-8 py-4 rounded-2xl font-display text-lg text-navy 
  hover:shadow-[8px_8px_0_0_#0F0F2D] hover:translate-x-[-2px] hover:translate-y-[-2px] 
  transition-all">
  Join IESA
</button>
```

**Secondary Pattern (Navy):**
```jsx
<button className="bg-navy border-[4px] border-lime shadow-[5px_5px_0_0_#000] 
  px-6 py-3 rounded-xl font-display text-base text-lime 
  hover:scale-105 transition-all">
  Learn More
</button>
```

**Outline Pattern:**
```jsx
<button className="bg-transparent border-[3px] border-navy 
  px-6 py-3 rounded-xl font-display text-navy 
  hover:bg-navy hover:text-lime transition-all">
  View All →
</button>
```

**Button Rules:**
- Primary buttons: Lime bg + navy border + navy shadow
- Hover effects: Translate shadow or scale (not both)
- No shadows on small badge-like elements
- Always use thick borders: 3-4px minimum

---

## Bento Grid

```jsx
<div className="bento-grid bento-3 gap-4">
  <div className="card card-lime bento-span-2">Hero Card</div>
  <div className="card card-navy">Stats</div>
  <div className="card">Regular</div>
  <div className="card card-lavender">Info</div>
  <div className="card">Another</div>
</div>
```

---

## Architecture & Key Files

### Frontend
- **Framework:** Next.js 16 (App Router, TypeScript, React 19)
- **Styling:** Tailwind CSS v4 with CSS-first configuration
- **Theme:** Light mode only — `next-themes` with `defaultTheme="light"` — **no dark mode**
- **Auth:** JWT (Argon2id + httpOnly refresh cookies)
- **Data Fetching:** SWR v2.4.0 installed (gradually adopting — most pages still use manual fetch)
- **Real-time:** WebSocket for study group chat, SSE for cache revalidation

### Backend
- **Framework:** FastAPI async
- **Auth:** `verify_token` (JWT payload only) vs `get_current_user` (full user doc from DB)
- **Auth hashing:** Argon2id via `run_in_executor` (async-safe, no thread blocking)
- **JWT keys:** Three separate secrets — `JWT_SECRET_KEY` (access), `JWT_REFRESH_SECRET_KEY` (refresh), `JWT_EMAIL_SECRET_KEY` (email verification + password reset). Never mix them.
- **Database:** MongoDB (Motor async driver) + Pydantic V2
- **Permissions:** `require_permission("scope:action")` dependency in `app/core/permissions.py` — cached in-memory with TTL
- **DB Indexes:** Compound indexes on hot collections (paystackTransactions, bankTransfers, payments) — created at startup via `init_db()`
- **Blocking I/O:** Groq SDK and Cloudinary calls wrapped in `run_in_executor` — never block the event loop
- **External students:** Non-IPE users have `isExternalStudent: True` on their user document. Use `_ipe_gate` / `require_ipe_student` to block them from IPE-only routes.

### Key Files
- `src/app/globals.css` — Design tokens, utility classes, press system
- `src/app/layout.tsx` — Root layout, fonts, providers
- `src/app/providers.tsx` — ThemeProvider, AuthProvider, SessionProvider, PermissionsProvider, ToastProvider
- `src/context/AuthContext.tsx` — JWT auth state, `getAccessToken()`, `userProfile`
- `src/context/SessionContext.tsx` — Active academic session context
- `src/hooks/useData.ts` — SWR-based hooks: `useStudentDashboard`, `useAdminStats`, `prefetchRoute`
- `src/hooks/useGrowthData.ts` — Dual localStorage + API persistence for growth tools
- `src/hooks/useSSE.ts` — Server-Sent Events hook for SWR cache revalidation
- `src/lib/api/` — Typed API client (`api.get()`, `api.post()`, etc.)
- `src/components/ui/` — UI component library (Modal, Toast, Table, Button, etc.)

### Platform Features
- **Student Dashboard:** Announcements, events, timetable, payments, library (resources), profile, press, team pages, applications, receipts, tickets
- **Admin Dashboard:** User management, announcements, events, sessions, timetable, payments, enrollments, resources, audit logs, messages, IEPOD, TIMP
- **Growth Hub:** 8 tools — habits tracker, CGPA calculator, Pomodoro timer, flashcards, journal, planner, courses list, goals (all use `useGrowthData` hook)
- **Study Groups:** Real-time chat (WebSocket), sessions scheduling, resource sharing, pinned notes
- **IESA AI:** Groq-powered assistant (llama-3.3-70b-versatile) with personalised student context
- **Press / Blog:** Student article submission, editorial review, public blog
- **IEPOD:** Departmental orientation programme — registration, phases, quizzes
- **TIMP:** Technical/Industry Mentorship Programme — applications, matching, pairs
- **Resource Library:** Student-submitted study materials with admin approval flow

### Removed Features (do not reference)
- **PWA:** Service worker, manifest.json — fully removed
- **Grades system:** `grade.py` model, grade router, all grade-related UI — fully removed (CGPA calc in Growth Hub is self-contained, not API-backed grades)

---

## Data Fetching Patterns

### Current State
SWR v2.4.0 is installed. Only ~2 pages use it. Most pages use manual `useState` + `useEffect` + `fetch()`.

### Preferred Pattern (new pages)
Use SWR hooks from `src/hooks/useData.ts` or create new ones:
```tsx
import useSWR from "swr";
import { useAuth } from "@/context/AuthContext";
import { getApiUrl } from "@/lib/api";

function useResources(filters: Filters) {
  const { getAccessToken } = useAuth();
  const key = `/api/v1/resources?${new URLSearchParams(filters)}`;
  return useSWR(key, async () => {
    const token = await getAccessToken();
    const res = await fetch(getApiUrl(key), { headers: { Authorization: `Bearer ${token}` } });
    if (!res.ok) throw new Error("Failed to fetch");
    return res.json();
  });
}
```

### Loading State Pattern
Separate initial loading from subsequent data fetches:
- `initialLoading` — true until first successful fetch → shows full-page spinner
- `isFetching` / SWR's `isValidating` — true during re-fetches → shows inline overlay on the data area only
- **Never** show a full-page spinner when filters/search/sort change after initial load

### Backend Auth in Endpoints
- **Read-only public endpoints:** No auth
- **Student endpoints:** `user: dict = Depends(get_current_user)` — returns full user doc
- **Permission-gated endpoints:** `user: dict = Depends(require_permission("scope:action"))`
- **JWT-only endpoints (avoid):** `verify_token` returns only JWT payload — no permissions, no profile data
- **WebSocket auth:** Use query param `?token=...` since browsers can't set WS headers
- **Never use `user_data.get("role")` for admin checks** — the JWT role claim is stale after role changes. Do an inline DB lookup:
  ```python
  db_user = await db["users"].find_one({"_id": ObjectId(user_id)}, {"role": 1})
  current_role = db_user.get("role", "") if db_user else ""
  ```

---

## Backend Security Patterns

### External Student Access Control
IPE-only features (Study Groups, IESA AI, Timetable, Payments, etc.) must block non-IPE students.

**User model field:** `isExternalStudent: bool` — `True` for non-IPE department students.

**Pattern 1 — Router-level gate (most endpoints):**
```python
async def _ipe_gate(user_data: dict = Depends(verify_token)):
    db = get_database()
    u = await db["users"].find_one({"_id": ObjectId(user_data["sub"])}, {"department": 1, "role": 1})
    if u and u.get("role") == "student" and u.get("department") != "Industrial Engineering":
        raise HTTPException(status_code=403, detail="This feature is only available to IPE students")

router = APIRouter(dependencies=[Depends(_ipe_gate)])
```

**Pattern 2 — Shared dependency (via `require_ipe_student`):**
```python
from ..core.security import require_ipe_student
# Used in events, academic_calendar, study_groups, etc.
```

**Frontend:** `src/lib/studentAccess.ts` — checks `userProfile.isExternalStudent` to conditionally hide navigation items and redirect away from IPE-only routes.

### JWT Key Separation
Three signing keys prevent one token type from being used as another:

| Env Var | Scope | TTL |
|---------|-------|-----|
| `JWT_SECRET_KEY` | Access tokens | 15 min |
| `JWT_REFRESH_SECRET_KEY` | Refresh tokens | 7 days |
| `JWT_EMAIL_SECRET_KEY` | Email verification + password reset | 1–24 h |

All three **must** be set in production (the app raises `RuntimeError` at startup if any are missing).
In development, deterministic fallbacks let tokens survive server restarts without invalidation.

### Safe Error Details
Never return `str(exception)` in HTTP error responses — use `safe_detail()` from `app/core/error_handling.py`:
```python
from app.core.error_handling import safe_detail

try:
    ...
except Exception as e:
    raise HTTPException(status_code=500, detail=safe_detail("Operation failed", e))
# → Returns "Operation failed" in production, "Operation failed: <full error>" in development
```

### AI Rate Limiting (Account-Linked)
The IESA AI uses per-user rate limits persisted in MongoDB, so they survive across devices and server restarts.

- **Collection:** `ai_rate_limits` — keyed by `userId + hourWindow` or `userId + dayWindow`
- **Limits:** 20 requests/hour, 60 requests/day (per user, UTC-aligned windows)
- **Implementation:** `app/routers/iesa_ai.py` — checks and increments atomically with `$inc` + `upsert`
- **Never use in-memory rate limiting for AI** — it doesn't persist across deploys

### Account Deletion Cleanup
When deleting a user account, clean all 8 related collections:
- `enrollments`, `notifications`, `refresh_tokens`, `bankTransfers`, `growth_data`, `ai_rate_limits`, `roles`
- For `paystackTransactions`: **anonymise** (null out PII fields) rather than delete — needed for financial audit trail
- Remove user from `study_groups` member arrays via `$pull`

---

## Performance Patterns

### Backend Performance
- **Async hashing:** Use `run_in_executor` for Argon2id `hash()` and `verify()` — keeps event loop free
- **Permissions caching:** In-memory LRU with 5-minute TTL in `app/core/permissions.py` — avoids DB hit per request
- **User projection:** Pass `{"projection": {...}}` to MongoDB queries to fetch only needed fields
- **Fire-and-forget writes:** Non-critical updates (e.g. `lastLogin`) use `asyncio.create_task()` — don't block response
- **CORS max_age:** Set to `86400` (24h) so preflight requests cache in browser
- **N+1 prevention:** Use batch `$in` queries instead of per-item DB lookups (e.g. enriching admin transactions)
- **Parallel context:** Use `asyncio.gather()` when fetching multiple independent data sources (e.g. AI context)
- **Blocking SDKs:** Groq and Cloudinary calls must use `loop.run_in_executor(None, fn)` — they are sync libraries

### Frontend Performance
- **Context memoization:** All context providers (`AuthContext`, `SessionContext`, `PermissionsContext`) must `useMemo` their value objects to prevent re-renders
- **Dynamic imports:** Use `next/dynamic` with `ssr: false` for heavy dashboard components (Timetable, StudyGroups, GrowthHub, Admin pages)
- **Server Components:** Public pages (home, about, contact, history, events, blog) should be Server Components where possible — avoid `"use client"` at the top level
- **Image optimization:** Use `next/image` with `priority` on hero images and proper `sizes` attribute for responsive images
- **Bundle splitting:** Keep client components small; extract sub-components into separate files to enable tree-shaking
- **SSE revalidation:** `useSSE` hook listens for server events and calls `mutate()` to revalidate specific SWR keys — avoids polling

---

## Payment System Architecture

### Overview
Three payment-related backend routers:
- `app/routers/payments.py` — CRUD for payment dues (amount, deadline, category, session)
- `app/routers/paystack.py` — Paystack integration (initialize, verify, webhook, transactions)
- `app/routers/bank_transfers.py` — Bank transfer submission/review workflow

### Payment Flow (Paystack)
1. Student clicks "Pay" → frontend calls `POST /api/v1/paystack/initialize` with `paymentId`
2. Backend creates Paystack transaction, stores in `paystackTransactions` collection
3. Backend returns `authorization_url` → frontend does `window.location.href = url` (full-page redirect)
4. After payment, Paystack redirects back with `?reference=XXX` in the callback URL
5. Frontend detects `reference` param → calls `POST /api/v1/paystack/verify/{reference}`
6. Backend verifies with Paystack API, adds student UID to payment's `paidBy` array
7. Frontend refreshes **all** payment data (payments, transactions, bankAccounts, transfers, settings)

### Payment Flow (Bank Transfer)
1. Student fills sender details + receipt image → `POST /api/v1/bank-transfers/`
2. Transfer stored with `status: "pending"` in `bankTransfers` collection
3. Admin reviews in Bank Transfers tab → `PATCH /api/v1/bank-transfers/{id}/review`
4. On approval: backend adds student UID to payment's `paidBy` array

### Key Frontend Files
- `src/lib/api/payments.ts` — Payment API functions + types
- `src/lib/api/bank-transfers.ts` — Bank transfer API + `BankTransfer`/`BankAccount` types + `TRANSFER_STATUS_STYLES`
- `src/app/(student)/dashboard/payments/page.tsx` — Student payment page (Paystack + bank transfer)
- `src/app/(admin)/admin/payments/page.tsx` — Admin payment management (4 tabs: Dues, Transactions, Bank Accounts, Transfers)
- `src/app/(student)/dashboard/events/page.tsx` — Event payment (Paystack for paid events)

### BFCache Handling
When students return from Paystack via browser back (cancel), the page restores from bfcache with stale state. Use `pageshow` event to reset payment-in-progress state:
```tsx
useEffect(() => {
  const handler = (e: PageTransitionEvent) => {
    if (e.persisted) setProcessingId(null);
  };
  window.addEventListener("pageshow", handler);
  return () => window.removeEventListener("pageshow", handler);
}, []);
```

### Admin Transactions Enrichment
The `GET /api/v1/paystack/transactions` endpoint enriches data for admin users:
- Batch-fetches payment docs to add `paymentCategory` and `paymentTitle`
- Parses `studentName`/`studentEmail` into structured `user` object `{firstName, lastName, email}`
- Student view returns raw documents (no enrichment needed)

### Paid Students Endpoint
`GET /api/v1/payments/{payment_id}/paid-students` — requires `payment:view_all` permission:
- Batch-fetches users (from `paidBy` array), Paystack transactions, and bank transfers
- Returns array with `{uid, firstName, lastName, email, matricNumber, level, paidAt, method, reference}`

---

## Press Effect System (Interactive Feedback)

**ALL interactive elements** (buttons, links, clickable cards) MUST use the global press effect classes instead of inline shadow hover patterns.

### How it works
Press classes set a hard shadow at rest, partially sink on hover, and fully press on active (click). They combine a **size** class with a **color** class.

**Size classes:** `press-1` through `press-10` (shadow offset in px)
**Color classes:** `press-black` (#000), `press-navy` (#0F0F2D), `press-lime` (#C8F31D)

### Usage
```jsx
// ✅ CORRECT — Use press classes
<button className="bg-lime border-[4px] border-navy press-5 press-navy px-8 py-4 rounded-2xl">
<button className="bg-navy border-[3px] border-lime press-3 press-lime px-5 py-2 rounded-xl">
<a className="bg-snow border-[3px] border-navy press-3 press-black px-4 py-2 rounded-xl">

// ❌ WRONG — Never use inline shadow hover patterns
<button className="shadow-[5px_5px_0_0_#0F0F2D] hover:shadow-[8px_8px_0_0_#0F0F2D] hover:translate-x-[-2px]">
<button className="hover:scale-105 transition-transform">  // For buttons — use press instead
```

### Color Pairing Rules
| Element Background | Press Color | Example |
|-------------------|-------------|---------|
| Light (snow, ghost, lime, coral, etc.) | `press-navy` or `press-black` | `bg-lime press-5 press-navy` |
| Dark (navy, navy-light) | `press-lime` | `bg-navy press-5 press-lime` |
| Any with `border-navy` | `press-navy` or `press-black` | `border-navy press-4 press-navy` |
| Any with `border-lime` | `press-lime` | `border-lime press-4 press-lime` |

### Exceptions (OK without press classes)
- **Card system classes** (`.card`, `.card-lime`, etc.) — have built-in hover/active
- **Button system classes** (`.btn-primary`, `.btn-secondary`, etc.) — have built-in hover/active
- **Image/icon zoom effects** — `group-hover:scale-105` on images/icons within cards is fine
- **Footer social links** — small icon links can use subtle scale
- **Static shadows** — non-interactive elements (cards, badges) with shadow but no hover are fine

---

## Hero Section Pattern

Public page hero sections MUST be full-screen on tablet+ screens:

```jsx
<section className="pt-16 pb-16 relative overflow-hidden md:min-h-[calc(100vh-5rem)] flex flex-col justify-center">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    {/* Hero content */}
  </div>
</section>
```

The `5rem` accounts for the header height. This applies to all public pages: home, about, contact, events, blog, history, iepod.

---

## Level System

Student level is auto-calculated from the session they were admitted. Never ask users to select their level manually, and never ask for a raw year — always ask for the admitted session.

**Formula:** `level = Math.max(100, Math.min(500, (currentSecondYear - admittedSecondYear) * 100 + 100))` + "L" suffix

Where:
- `currentSecondYear` = second year of the active session (e.g., **2026** from `"2025/2026"`)
- `admittedSecondYear` = second year of the student's admitted session (e.g., **2023** from `"2022/2023"`)
- `admissionYear` stored on the user = `admittedSecondYear` (the second year of the admitted session)

Example: active session `2025/2026`, admitted in `2022/2023` → `(2026 − 2023) × 100 + 100 = 400L`

**Registration flow:**
1. User selects the **session they were admitted** from a dropdown (e.g. `2022/2023`)
2. System derives `admissionYear` = second year of that session (e.g. `2023`)
3. System calculates level automatically using the formula above
4. Shows confirmation: "Admitted 2022/2023, current session 2025/2026 → your level is 400L"
5. User must click "Confirm" before form can be submitted

**Backend:** `currentLevel` and `admissionYear` (= second year of admitted session) stored on user document. IEPOD registration auto-populates level from user profile.

---

## Accessibility Rules

1. **Never remove focus outlines from buttons** — the global CSS removes outlines from form inputs only; buttons retain `:focus-visible` ring
2. **Decorative SVGs** must have `aria-hidden="true"`
3. **Mobile menu toggle** must have `aria-expanded={isMenuOpen}`
4. **Modals** must use the shared `<Modal>` component (has `role="dialog"`, `aria-modal`, Escape handling, focus restoration)
5. **Use `text-slate` or `text-navy-muted`** for secondary text instead of `text-navy/60` (opacity may fail contrast)
6. **All `<main>` tags** on pages must have `id="main-content"` for skip-link

---

## Quick Reference

| Purpose | Pattern |
|---------|--------|
| Page background | `bg-snow` (white) or `bg-ghost` (off-white) |
| Card | `bg-snow border-[4px] border-navy rounded-3xl shadow-[8px_8px_0_0_#000]` |
| Colored card | `bg-{color} border-[4px] border-navy rounded-3xl shadow-[8px_8px_0_0_#000]` |
| Navy card | `bg-navy border-[4px] border-lime rounded-3xl shadow-[8px_8px_0_0_#C8F31D]` |
| Display heading | `font-display font-black text-4xl text-navy` |
| Headline with brush | `<span className="brush-highlight">Key Word</span>` |
| Label/badge | `text-label uppercase tracking-wider text-xs` (no shadow) |
| Primary button | `bg-lime border-[4px] border-navy press-5 press-navy` |
| Secondary button | `bg-navy border-[4px] border-lime press-5 press-lime` |
| Outline button | `bg-transparent border-[3px] border-navy hover:bg-navy hover:text-lime` |
| Small action button | `bg-teal border-[2px] border-navy press-2 press-navy` |
| Hero section | `md:min-h-[calc(100vh-5rem)] flex flex-col justify-center` |
| Sparkle decorator | `<svg className="fixed top-16 left-[10%] w-6 h-6 text-lime/20" aria-hidden="true" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0l1.5 7.5L21 9l-7.5 1.5L12 18l-1.5-7.5L3 9l7.5-1.5z"/></svg>` |
| Container | `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12` |
| Secondary text | `text-slate` or `text-navy-muted` (NOT `text-navy/60`) |
| White text | `text-snow` (NOT `text-white`) |
| White background | `bg-snow` (NOT `bg-white`) |
| Light hover bg | `hover:bg-cloud` (NOT `hover:bg-ghost-light`) |

### Don'ts
- ❌ Old tokens: `bg-bg-primary`, `text-text-primary`, `green-accent`, `cream`, `charcoal`
- ❌ Raw Tailwind: `bg-gray-100`, `text-blue-500`, `bg-white`, `text-white`, `bg-black`
- ❌ Non-existent tokens: `bg-ghost-light`, `bg-ghost-dark`, `border-ghost/20-dark`
- ❌ Emojis in UI (use SVG icons instead)
- ❌ Lime shadows on light backgrounds
- ❌ Lime borders paired with black shadows
- ❌ Navy bg with black shadow (must use lime shadow `#C8F31D`)
- ❌ Navy bg with non-lime border (must use `border-lime`)
- ❌ Inline shadow hover patterns (`hover:shadow-[...]`) — use press classes
- ❌ `hover:scale-105` on buttons — use press classes
- ❌ 5-point star decorators (use 4-point diamonds)
- ❌ Soft/blurred shadows (use hard offset shadows only)
- ❌ Dark mode classes or logic (`dark:`, `enableSystem`)
- ❌ Forgetting `overflow: hidden` on brush highlight parents
- ❌ Forgetting `overflow-x: hidden` on body when using fixed decorators
- ❌ Forgetting `aria-hidden="true"` on decorative SVGs
- ❌ Forgetting `id="main-content"` on `<main>` tags

### Do's
- ✅ `font-display font-black` for all headlines
- ✅ TBJ Endgraph font for everything (display and body)
- ✅ Brush highlights on key headline words
- ✅ Context-aware brush colors (`.brush-coral`, `.brush-lime`, `.brush-navy`)
- ✅ Diamond sparkle decorators with low opacity + `aria-hidden="true"`
- ✅ Mix 2-3 colored cards per grid
- ✅ 4-8px thick borders on cards and buttons
- ✅ Pure black or navy hard shadows (5-10px offset) on light backgrounds
- ✅ Lime hard shadows on navy backgrounds
- ✅ SVG icons from libraries (no emojis)
- ✅ Press effect classes for all interactive hover/active feedback
- ✅ Full-screen hero sections on tablet+ with `md:min-h-[calc(100vh-5rem)]`
- ✅ `text-snow` instead of `text-white`, `bg-snow` instead of `bg-white`
- ✅ `hover:bg-cloud` for light hover backgrounds
- ✅ Overflow management on body and containers

---
> Source: [devqing00/iesa](https://github.com/devqing00/iesa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
