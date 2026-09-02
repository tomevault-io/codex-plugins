# AMG Workspace - Copilot Instructions

## Project Overview

A coworking space booking system built with **Next.js 16**, **Prisma 7**, **NextAuth v5**, and **shadcn/ui**. Supports space bookings, subscriptions (memberships), add-ons, shop, and visitor management.

## Architecture

### Data Flow Pattern

1. **Server Actions** (`actions/*.ts`) - All data mutations use `'use server'` directive
2. **Prisma Client** via `@/lib/db` - Single PostgreSQL connection with adapter pattern
3. **Client State** - Zustand stores (`store/`) for booking flow persistence

### Key Directories

-   `actions/` - Server actions organized by domain (bookings, subscriptions, payments, etc.)
-   `app/admin/` - Admin panel (staff, front desk, admins)
-   `app/dashboard/` - Client portal
-   `lib/permissions.ts` - RBAC with role hierarchy (CLIENT → SUPER_ADMIN)

### Two Space Types

-   **SUBSCRIPTION** spaces (shared desks, private offices) → Creates `Membership`
-   **BOOKING** spaces (board room, training room) → Creates `Booking`

## Code Conventions

### Server Actions Pattern

```typescript
// actions/example.ts
'use server';
import { prisma } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export interface ExampleResult {
	success: boolean;
	message: string;
	data?: SomeType;
	error?: string;
}

export async function doSomething(): Promise<ExampleResult> {
	try {
		// ... logic
		revalidatePath('/relevant-path');
		return { success: true, message: 'Done', data };
	} catch (error) {
		return {
			success: false,
			message: 'Failed',
			error: error instanceof Error ? error.message : 'Unknown error',
		};
	}
}
```

### Currency Handling

All prices stored in **kobo** (smallest unit). Display in Naira using `formatCurrency()` from `lib/utils/`.

### Authentication

-   `getCurrentUser()` from `actions/auth.ts` - Get session user in server actions
-   `auth()` from `@/auth` - Direct NextAuth access
-   Middleware protects routes; admin routes require `ADMIN_ROLES` check

### Role-Based Access

Six roles with hierarchy in `lib/permissions.ts`:

```
CLIENT < FRONT_DESK_ASSISTANT < FRONT_DESK < STAFF < ADMIN < SUPER_ADMIN
```

Use `hasPermission(role, 'permission:action')` for access control.

### UI Components

Use shadcn/ui from `components/ui/`. Import pattern:

```typescript
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardContent } from '@/components/ui/card';
```

## Database Commands

```bash
pnpm db:studio    # Open Prisma Studio
pnpm db:seed      # Seed database
pnpm db:reset     # Reset and reseed
npx prisma migrate dev --name <name>  # Create migration
```

## Key Patterns

### Booking Flow (Zustand Store)

```typescript
import { useBookingStore } from '@/store/booking-store';
const { bookingData, setBookingData, clearData } = useBookingStore();
```

### Email Templates

Create email content in `lib/email-templates.ts`, send via `sendEmail()` from `lib/email.ts`.

### Plan Perks & Add-ons

-   `PlanPerk` - Benefits included with pricing plans (e.g., "1hr Board Room/week")
-   `Addon` - Purchasable extras with types: SUBSCRIPTION, BOOKING, SHOP, UNIVERSAL

## Critical Files

-   [prisma/schema.prisma](prisma/schema.prisma) - Complete data model (1000+ lines)
-   [middleware.ts](middleware.ts) - Route protection logic
-   [lib/permissions.ts](lib/permissions.ts) - RBAC definitions
-   [actions/index.ts](actions/index.ts) - All server actions exported

## Development Notes

-   Next.js App Router with TypeScript strict mode
-   Form validation with Zod + react-hook-form
-   Toast notifications via Sonner
-   Vercel deployment with special Prisma handling (see `vercel-build.sh`)

---
> Source: [iisce/amg](https://github.com/iisce/amg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-02 -->
