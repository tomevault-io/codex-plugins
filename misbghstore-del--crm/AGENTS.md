<!-- @format -->

# CRM Codebase AI Agent Instructions

## Project Overview

This is a **Next.js 16 Business Development Manager (BDM) CRM** built with TypeScript, Supabase, and Tailwind CSS. It manages customers, visits, tasks, and analytics for sales teams. PWA-enabled for offline functionality.

### Tech Stack

- **Frontend**: Next.js 16 (App Router), React 19, TypeScript
- **Styling**: Tailwind CSS 4, Radix UI, Lucide icons
- **Backend**: Supabase (PostgreSQL + Auth), Server Actions
- **Forms**: React Hook Form + Zod validation
- **Data Fetching**: TanStack React Query, Supabase client
- **Additional**: next-themes (dark mode), PWA support

## Architecture Patterns

### 1. **Data Layer: Server-First with Supabase**

- **Server Client** (`utils/supabase/server.ts`): For Server Components and Server Actions. Handles cookies via Next.js `cookies()`. Use in RSC and actions.
- **Browser Client** (`utils/supabase/client.ts`): For Client Components only. Never import server client in client code.
- **Auth Flow**: Middleware validates user sessions and clears expired auth cookies. Redirects unauthenticated users to `/login`.
- **RLS Policies**: All tables enforce Row-Level Security. Users see only their own data (assigned_to/user_id = auth.uid()).

### 2. **Server Actions Pattern** (Located in `app/actions/`)

Wrap database operations in Server Actions with `"use server"` directive. Key conventions:

- **Return type**: `ActionState = { error?, message?, success? }`
- **Auth check**: Always call `supabase.auth.getUser()` first; return error if null
- **Validation**: Use Zod when needed, extract FormData directly for simple cases
- **Revalidation**: Call `revalidatePath()` after mutations to sync ISR cache
- **File uploads**: Use `supabase.storage.from("bucket-name").upload()` → get public URL with `getPublicUrl()`
- **Related mutations**: Example: `createVisit()` calls `updateCustomerStage()` to trigger cross-entity updates

### 3. **Component Organization**

- **Server Components** (default): Pages, layouts, data-fetching wrappers. Query Supabase directly with server client.
- **Client Components** (`"use client"`):
  - Dialogs, forms, interactive features (`components/customers/add-customer-dialog.tsx`)
  - Use client-side Supabase client for queries
  - Call Server Actions for mutations
  - Use `useFormState` hook with actions for state management
- **UI Library**: Radix UI components in `components/ui/` (button, card, dialog, select, etc.) are pre-built wrappers

### 4. **Page Structure**: App Router with Route Groups

```
app/(main)/         ← Main layout with Header
  dashboard/
  customers/
  tasks/
  visits/
  admin/
  profile/
  kpi/
  planner/
  ...
app/login/          ← Public login
app/auth/           ← OAuth callbacks
```

- Grouped routes `(main)` share a layout with Header. Unauthenticated users redirected by middleware.

## Data Model & Relationships

### Core Tables (Supabase)

1. **profiles**: User metadata (id [FK auth.users], email, full_name, role='admin'|'bdm', region)
   - RLS: Public read, users edit own profile
2. **customers**: Sales leads (id, name, type, stage, contact_person, phone, address, city, meeting_count, assigned_to [FK profiles], created_by)
   - Types: "Prospect Dealer", "Dealer", "Professional", "Site"
   - Stages: "New Lead", "Active", "Closed - Converted", "Closed - Not Converted"
   - RLS: BDMs see only assigned customers
3. **visits**: Customer interactions (id, customer_id, user_id, timestamp, purpose, outcome, notes, location_lat/lng, photo_url)
   - RLS: Users see only their own visits
4. **tasks**: Follow-ups (id, user_id, customer_id, description, due_date, priority, is_completed)
   - RLS: Users see only their own tasks

### Storage Buckets

- `visit-photos`: Visit and customer site photos. Naming: `{user_id}/{timestamp}.{ext}`

## Development Workflow

### Build & Run

```bash
npm run dev           # Next.js dev server (http://localhost:3000)
npm run build         # Production build (uses PWA config)
npm run start         # Start production server
npm run lint          # ESLint check (Next.js core-web-vitals)
```

### Environment Variables

Create `.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

- Check setup: `node scripts/check_env.js`

### Key Developer Patterns

#### Adding a New Feature with Data

1. **Define Server Action** in `app/actions/{feature}.ts`
   - Import `createClient` from server utils
   - Get user via `supabase.auth.getUser()`
   - Perform mutation, return `ActionState`
   - Call `revalidatePath()` for ISR invalidation
2. **Create Form Component** in `components/{feature}/{dialog}.tsx`
   - Mark with `"use client"`
   - Use `useFormState(action)` hook for action integration
   - Validate inputs; handle loading/error states
3. **Fetch Data in Page** (Server Component)
   - Use server client to query Supabase
   - Pass data as props to interactive child components
   - Example: `dashboard/page.tsx` fetches tasks, visits, customers; passes to charts

#### Styling Conventions

- **Tailwind + Radix**: Use Radix UI primitives (`Dialog`, `Select`, `Popover`) wrapped in custom components
- **Theme**: Dark/light mode via `next-themes`; respect CSS variable names (`--background`, `--foreground`)
- **Animations**: Use Tailwind classes (`animate-in`, `fade-in`, `slide-in-from-bottom-4`)
- **Spacing/Layout**: Consistent use of `gap`, `p-`, `mx-auto max-w-7xl` for page containers

#### Common UI Patterns

- **Forms**: React Hook Form + Zod for validation (`components/customers/add-customer-dialog.tsx`)
- **Lists with Filters**: Client-side filtering on server-fetched data; `Select` for dropdowns (`customer-list.tsx`)
- **Dialogs**: Radix `Dialog` + `useFormState` for form handling
- **Analytics**: Recharts for charts (`components/analytics/visits-chart.tsx`)
- **Date Handling**: Use `new Date().toISOString()` for timestamps; format display with `DateDisplay` component

## Important Conventions & Gotchas

1. **Always Use Server Client in Server Components**: Importing browser client in RSC causes hydration issues.
2. **Revalidate After Mutations**: Forgetting `revalidatePath()` leaves ISR cache stale.
3. **RLS Enforcement**: All queries are scoped by auth.uid() at DB layer; don't duplicate auth checks in actions (trust the policy).
4. **File Naming**: Use kebab-case for component/action files (`add-customer-dialog.tsx`, `visits.ts`).
5. **formAction Usage**: In client forms, use `formAction={createCustomer}` to avoid manual submission handling.
6. **Type Safety**: Supabase queries require explicit `.returns<Type[]>()` for TS inference.
7. **Metadata & PWA**: Only set metadata in root `app/layout.tsx`; manifest auto-generated via `app/manifest.ts`.

## Extension Points for AI Agents

### When Adding Pages

- Create directory under `app/(main)/{feature}`
- Add `page.tsx` (Server Component) to fetch data
- Create interactive components in `components/{feature}/` with `"use client"`
- Add Server Action in `app/actions/{feature}.ts`

### When Modifying Supabase Schema

- Add migration in `supabase/migrations/` with date prefix
- Update types in `types/supabase.ts` (auto-generated by Supabase CLI in production)
- Ensure RLS policies cover CRUD operations for roles (admin, bdm)

### Testing Database Connectivity

- Run `node scripts/check_schema.js` to validate table structure
- Run `node scripts/test_connection.js` for auth/connection issues

## Key Files Reference

- **Pages**: `app/(main)/{dashboard,customers,tasks,visits}/page.tsx`
- **Server Actions**: `app/actions/{customers,visits,tasks,analytics,profile,admin}.ts`
- **Components**: `components/{customers,tasks,analytics}/` for feature-specific; `components/ui/` for primitives
- **Utilities**: `utils/supabase/{client,server}.ts`, `lib/utils.ts` (cn(), classname merger)
- **Middleware**: `middleware.ts` (auth validation, session refresh)
- **Database**: `supabase/schema.sql` (RLS policies + table definitions)

---
> Source: [misbghstore-del/CRM](https://github.com/misbghstore-del/CRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
