# SocioPulse / MedicoPulse — Copilot Instructions

> **Updated:** 2026-02-07 | **Version:** 2.1.0

## 1. Project Overview

Multi-brand B2B platform connecting healthcare/social **establishments** (Clients) with **freelancers** (Talents). Single codebase serves two brands via `NEXT_PUBLIC_APP_MODE` env var.

| Brand | Mode | Theme | Target |
|-------|------|-------|--------|
| **SocioPulse** | `SOCIAL` | Electric Teal / Deep Indigo | Social workers, educators (MECS, IME, ITEP, Crèches, CCAS) |
| **MedicoPulse** | `MEDICAL` | Neon Rose / Electric Violet | Healthcare pros (EHPAD, SSIAD, Cliniques, Hôpitaux, HAD) |

### Forbidden Terms
- ❌ "Sanctuary", "Oracle", "SoulMirror" — Legacy astrological features
- ❌ "Eduat'heure" — Use **"Catalogue"**
- ❌ "Les Extras" — Use **"SocioPulse"**

### Monorepo Structure
```
├── app/                    # Next.js 14 App Router (~65+ pages)
│   ├── (auth)/            # Login, register, forgot-password, onboarding
│   ├── (platform)/        # Protected routes (dashboard, bookings, messages, etc.)
│   └── (admin)/           # Admin backoffice (users, contracts, moderation)
├── apps/api/              # NestJS 10 Backend (18 feature modules + 7 common)
├── components/            # React components (~230+ files)
├── lib/                   # Business logic & configuration (19+ files)
│   ├── hooks/             # Custom React hooks (6 files)
│   ├── stores/            # Zustand stores (authStore.ts)
│   ├── constants/         # Static data (jobTags.ts)
│   └── pricing/           # Smart Rate Engine (calculator.ts)
├── packages/shared-types/ # Shared TypeScript DTOs
├── prisma/schema.prisma   # Database schema (1,762 lines, 32 models, 34 enums)
├── seeds/                 # JSON seed data (7 talent category files)
└── tests/                 # Playwright E2E
```

### Tech Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | Next.js 14, React 18, TypeScript 5, Tailwind 3.4, Framer Motion 12, Radix UI, Zustand 5 |
| **Backend** | NestJS 10.3, Prisma ~5.22 (pinned — v7 breaks), PostgreSQL 15, Socket.IO 4.8, LiveKit 2.x |
| **Payments** | Stripe Connect (marketplace mode) + Pulse Economy (virtual currency, 1 Pulse = 5€ HT) |
| **Auth** | JWT (jose), Cookie-based (SameSite=lax, 7 days), Role guards |
| **Testing** | Playwright (E2E smoke tests) |
| **DevOps** | Docker multi-stage (Node 20 Alpine), npm workspaces, Coolify-compatible healthchecks |

---

## 2. Critical Workflows

```bash
# Development (run both in parallel)
npm run dev              # Frontend :3000
npm run api:dev          # Backend :4000

# Database (ALWAYS run db:generate after schema changes!)
npm run db:generate      # ⚠️ REQUIRED after schema.prisma edits
npm run db:push          # Push to dev DB
npm run db:migrate       # Create migration
npm run db:seed          # Seed test data
npm run db:studio        # Prisma Studio GUI
npm run db:seed:sociopulse  # Generate + import SocioPulse seed data

# Testing
npm run test:smoke       # Playwright smoke tests

# Docker build (brand-specific images)
docker build --build-arg NEXT_PUBLIC_APP_MODE=SOCIAL -t sociopulse .
docker build --build-arg NEXT_PUBLIC_APP_MODE=MEDICAL -t medicopulse .
docker build -f Dockerfile.api -t sociopulse-api .
```

---

## 3. Brand Polymorphism (CRITICAL PATTERN)

### Never Hardcode Brand Logic
```typescript
// ✅ CORRECT — Use helpers from lib/brand.ts & lib/domain-config.ts
import { isMedical, isSocial, currentBrand } from '@/lib/brand';
import { getTerm, isFeatureEnabled, getDashboardLayout } from '@/lib/domain-config';

const title = getTerm('mission');     // "Vacation" (Medical) | "Mission" (Social)
const label = getTerm('talent');      // "Soignant" | "Intervenant"
if (isFeatureEnabled('enableWorkshops')) { /* Social only */ }

// ❌ WRONG — Hardcoded brand checks
if (mode === 'MEDICAL') { ... }
```

### Feature Flags (lib/domain-config.ts)
| Flag | Social | Medical | Description |
|------|--------|---------|-------------|
| `enableWorkshops` | ✅ | ❌ | SocioLive workshops/ateliers |
| `enableShiftView` | ❌ | ✅ | Calendar-dense shift view |
| `enablePortfolio` | ✅ | ❌ | Talent portfolio showcase |
| `enableJobTicker` | ❌ | ✅ | Fast urgent shifts ticker |
| `enableCriticalUrgency` | ❌ | ✅ | CRITICAL urgency level |
| `enableTeamVivier` | ✅ | ✅ | TalentPool team management |
| `enableProjects` | ✅ | ❌ | Project-based mission grouping |

### Theming via CSS Variables
```css
/* globals.css (1,380 lines) — :root = Social, [data-theme="medical"] = Medical */
/* Social: Electric Teal */
:root { --primary-h: 174; --primary-s: 84%; --primary-l: 45%; }
/* Medical: Neon Rose — also overrides border-radius to sharp/clinical */
[data-theme="medical"] { --primary-h: 340; --primary-s: 85%; --primary-l: 55%; }
```
```tsx
// ✅ Use CSS variables or Tailwind semantic classes
className="bg-[hsl(var(--primary))]"
className="text-primary-600"    // Tailwind via CSS vars (50-900 scale)
className="shadow-theme-glow"   // Brand-aware glow

// ❌ NEVER use direct colors — breaks cross-brand
className="bg-teal-500"     // Social only!
className="text-rose-600"   // Medical only!

// Static cross-brand colors (always safe):
className="bg-brand-500"    // Indigo — shared accent
className="bg-live-500"     // Teal — SocioLive
className="bg-alert-500"    // Rose — alerts/urgency
```

### Dashboard Resolution
`DashboardResolver` dynamically loads brand+role dashboards:
| Brand | Client Layout | Talent Layout |
|-------|--------------|---------------|
| Social | `project-hub` (card-based) | `portfolio-feed` (skills + feed) |
| Medical | `shift-planner` (calendar-dense) | `job-ticker` (fast list) |

Widget system in `lib/dashboard-config.ts` — 8 client widgets + 8 talent widgets, each with `featureFlag` for brand filtering.

---

## 4. NestJS API Patterns (`apps/api/src/`)

### Endpoint Security Stack
```typescript
import { JwtAuthGuard, RolesGuard } from '@/common/guards';
import { Roles } from '@/common/decorators/roles.decorator';
import { CurrentUser, CurrentUserPayload } from '@/common/decorators/current-user.decorator';

@UseGuards(JwtAuthGuard, RolesGuard)  // Order matters!
@Roles('CLIENT')  // or 'TALENT', 'ADMIN'
@Post('missions')
async create(@CurrentUser() user: CurrentUserPayload) {
  // user.id, user.email, user.role, user.status available
}
```

### Guards
| Guard | Purpose |
|-------|---------|
| `JwtAuthGuard` | Validates Bearer token |
| `RolesGuard` | Enforces `@Roles()` decorator |
| `MissionAccessGuard` | Mission-scoped data access |
| `SeedInterceptionGuard` | Intercepts actions on seed/demo data |
| `PulseGuard` | Checks Pulse wallet balance before actions |

### API Architecture
- **Global prefix:** `/api/v1`
- **Swagger docs:** `http://localhost:4000/docs`
- **Rate limiting:** 3 tiers (10/1s, 50/10s, 200/60s)
- **CORS:** Multi-domain (localhost, sociopulse.fr, medicopulse.fr, sslip.io)
- **Static assets:** `/uploads/` served from cwd

### 18 Feature Modules
`auth`, `admin`, `matching-engine`, `video-booking`, `wall-feed`, `contracts`, `messages`, `payments`, `health`, `growth`, `mission-hub`, `notifications` (WebSocket), `availability`, `finance`, `talents`, `quotes`, `dashboard`, `pulse`

### 7 Common Modules
`guards/`, `decorators/`, `filters/`, `prisma/`, `uploads/`, `mailer/`, `services/geocoding`

---

## 5. Frontend Architecture

### Auth Flow
- JWT stored in `accessToken` cookie (SameSite=lax, 7 days)
- `lib/auth.ts`: `auth.login()`, `auth.register()`, `auth.getToken()`, `auth.logout()`, `auth.fetchMe()`
- `lib/useAuth.ts`: React hook wrapping Zustand `authStore` with hydration handling
- `lib/stores/authStore.ts`: Zustand + persist middleware (284 lines), stores `AuthUser` with profile/establishment data
- `middleware.ts`: Route protection — role-based redirects, admin subdomain (`dash.sociopulse.com`)

### API Calls
```typescript
import { getApiUrl } from '@/lib/config';
import { auth } from '@/lib/auth';

const res = await fetch(`${getApiUrl()}/endpoint`, {
  headers: { Authorization: `Bearer ${auth.getToken()}` }
});
// getApiUrl() handles: browser, SSR (Docker internal), and production URLs
```

### Key Data Hooks (`lib/hooks/`)
| Hook | Purpose |
|------|---------|
| `useDashboardData` | Generic dashboard data fetch with caching & prefetch |
| `useClientStats` | Client stats, upcoming missions, team stability, talent spotlights |
| `useTalentStats` | Talent stats, profile, SOS missions, applications, bookings |
| `usePulseWallet` | Pulse virtual currency wallet |
| `useServices` | Services CRUD |

### Component Organization (~230+ files)
| Folder | Key Components |
|--------|---------------|
| `components/dashboard/` | `DashboardResolver`, brand-specific dashboards, 19 widgets (client/talent/shared) |
| `components/wall/` | `WallResolver`, `WallFeedClient`, 20+ cards (missions, SOS, talents, services) |
| `components/landing/` | 23+ sections (hero, bento grids, testimonials, trust, SEO footer) |
| `components/profile/` | 12 components (full public profile system) |
| `components/feed/` | Social feed (FeedList, FeedItem, UserIdentityCard) |
| `components/pulse/` | 8 components (wallet widget, pack grid, transaction history) |
| `components/finance/` | Wallet, invoices, transactions, budget overview |
| `components/mission/` | Chat live, urgency badges, SOS cards |
| `components/onboarding/` | Wizard, document upload, job tag picker |
| `components/visio/` | LiveKit video room, control bar, pre-join |
| `components/ui/` | 14 Radix/shadcn primitives (button, card, input, badge, etc.) |
| `components/vitrine/` | Marketplace resolver (client/talent viewpoints) |
| `components/vivier/` | Talent CRM card, import modal |
| `components/providers/` | `SocketProvider`, `ThemeProvider` |

### Route Groups
| Group | Routes | Layout |
|-------|--------|--------|
| `(platform)` | ~65+ pages | `DesktopTopNav` + `MobileBottomNav` + `PlatformFooter` |
| `(auth)` | 4 pages | Gradient background with mesh blobs |
| `(admin)` | 6+ pages | Admin layout |
| `/t/[slug]` | Public SEO | Standalone talent profile |

### Dashboard Sub-Routes
- **Client:** 18+ sub-routes (`/dashboard/client/*`) — missions, bookings, planning, vivier, workshops, finance, SOS
- **Talent:** 16+ sub-routes (`/dashboard/talent/*`) — missions, applications, services, sociolive, reputation, finance

---

## 6. Economic Model

### Pricing Plans (`lib/pricing-config.ts`)
```typescript
BETA       // 2% technical fees (current launch period)
STANDARD   // 15% commission (private sector)
ENTERPRISE // 0% commission (SaaS subscription)

IS_LAUNCH_PERIOD: true  // Set false to enable monetization
```

### Smart Rate Engine (`lib/pricing/calculator.ts`)
Breaks shifts into normal/night/Sunday/holiday hours per French conventions collectives:
```typescript
SURCHARGES: { NIGHT: +25% (21h-06h), SUNDAY: +50%, HOLIDAY: +100% }
// Cumulative: Sunday night = +75%, Holiday night = +125%
```

### Pulse Economy (Virtual Currency)
- **1 Pulse = 5€ HT** — separate from EUR wallet (Stripe Connect)
- FIFO consumption: free Pulses burned before paid
- 3 packs: Discovery (20P/100€), Establishment (100P/500€), Grand Compte (500P/2,500€)
- Used for: mission publishing, contract validation, SocioLive sessions, profile boost, matching shortlist

---

## 7. Database Quick Reference (32 models)

### Core Entities
| Model | Purpose | Key Fields |
|-------|---------|------------|
| `User` | Account (CLIENT/TALENT/ADMIN) | `role`, `status`, `clientType`, `allowedBrands[]`, `walletBalance`, `stripeAccountId` |
| `Profile` | Talent professional profile | `jobId`, `complianceStatus`, `slug`, `profileScore`, `hourlyRate` |
| `Establishment` | Client organization | `pricingPlan`, `paymentMode`, `creditLimit`, `siret` |
| `Service` | Catalogue offering | `type` (WORKSHOP/COACHING_VIDEO), `deliveryType`, `pricingOptions` |
| `ReliefMission` | SOS urgent mission | `jobId`, `urgencyLevel`, `missionType`, `specialtiesTags[]` |
| `Booking` | Reservation (polymorphic) | `appliedPlan`, `commissionRate`, all surcharge snapshots |
| `Contract` | E-signature | `type`, `status`, dual signatures (talent + client) |
| `Quote` | Commercial proposal | `reference`, `lines[]`, `status` (DRAFT→ACCEPTED) |

### Supporting Models
`TalentPool`, `TalentPoolMember`, `AvailabilitySlot`, `MissionApplication`, `Post`, `Review`, `Transaction`, `Notification`, `Message`, `UserDocument`, `AdminNote`, `ExternalNews`, `AuditLog`, `Dispute`, `Resource`, `AnalyticsEvent`

### Mission Hub Models
`MissionMessage`, `MissionInstruction`, `MissionReport`, `MissionTimelineEvent`

### Pulse Economy Models
`PulseWallet`, `PulseTransaction`

---

## 8. Common Pitfalls

| Issue | Cause | Fix |
|-------|-------|-----|
| "Property X does not exist on type" | Schema changed without regen | Run `npm run db:generate` |
| Wrong terminology displayed | Hardcoded strings | Use `getTerm('key')` from domain-config |
| Theme colors wrong on one brand | Direct Tailwind colors | Use CSS variables `var(--primary)` |
| Border radius wrong on Medical | Using `rounded-full` | Use `rounded-[var(--radius-*)]` |
| API 404 in production | Wrong base URL | Use `getApiUrl()` helper, check `NEXT_PUBLIC_API_URL` |
| Guards not working | Wrong decorator order | `@UseGuards(JwtAuthGuard, RolesGuard)` before `@Roles()` |
| Commission rate wrong | Using old function | Use `calculateFeePercentage(plan)` from pricing |
| Prisma v7 breaks Docker | Upgrading Prisma past 5.x | Keep `~5.22.0` pinned in both package.json files |
| Hydration mismatch | Auth state not ready | Use `useAuth()` hook with 50ms hydration delay |
| Seed data interactions | Users clicking demo data | `SeedInterceptionGuard` prevents real actions on `isSeed: true` |
| Rate limiting errors | Exceeding 10/1s or 50/10s | Check ThrottlerModule config, 3-tier (10/1s, 50/10s, 200/60s) |

---

## 9. Key Files Reference

| File | Purpose |
|------|---------|
| `lib/brand.ts` | Brand config, `isMedical()`, `isSocial()`, `currentBrand`, `filterByBrand()` |
| `lib/domain-config.ts` | Feature flags, terminology, compliance rules, dashboard layouts |
| `lib/config.ts` | API URL resolution (`getApiUrl()`) — handles browser/SSR/Docker |
| `lib/auth.ts` | Auth module — `login()`, `register()`, `getToken()`, `fetchMe()` |
| `lib/useAuth.ts` | React hook wrapping Zustand authStore with hydration |
| `lib/stores/authStore.ts` | Zustand + persist — `AuthUser` type with full profile/establishment |
| `lib/dashboard-config.ts` | Widget registry (16 widgets), layout config, animations |
| `lib/nav-config.ts` | Navigation items for all roles (primary, secondary, quickActions) |
| `lib/pricing-config.ts` | Economic model — plans, commissions, cancellation rules |
| `lib/pricing/calculator.ts` | Smart Rate Engine — shift breakdown with legal surcharges |
| `lib/sos-config.ts` | 50+ job definitions for both brands (1,071 lines) |
| `lib/wall-config.ts` | Polymorphic feed config by role (missions, SOS, ateliers) |
| `lib/vitrine-config.ts` | Marketplace homepage config (client vs talent viewpoints) |
| `app/globals.css` | Theme CSS vars (1,380 lines) — `:root` + `[data-theme="medical"]` |
| `prisma/schema.prisma` | Source of truth — 32 models, 34 enums (1,762 lines) |
| `middleware.ts` | Route protection, role redirects, admin subdomain |
| `apps/api/src/main.ts` | NestJS bootstrap — CORS, Swagger, validation, healthcheck |
| `tailwind.config.ts` | Custom theme — polymorphic colors, aurora glows, glass shadows |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tachfineamnay)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tachfineamnay)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
