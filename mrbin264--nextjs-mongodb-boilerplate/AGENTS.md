# Mind Voyage Companion - AI Agent Instructions

## 🏗️ Architecture Overview

**Next.js 15 App Router** with MongoDB/Mongoose, TypeScript, and Tailwind CSS. Privacy-first habit tracking app with dark theme UI (`zinc-900` backgrounds).

### Key Architectural Patterns

- **MongoDB Memory Server**: Auto-starts in development (`src/lib/db.ts`). No manual MongoDB setup needed.
- **Dual Database Clients**: Both NextAuth (`src/lib/auth.ts`) and Mongoose (`src/lib/db.ts`) use separate MongoDB connections
- **Layout Composition**: `DashboardLayout` wraps content with sidebar navigation, `AuthLayout` for auth pages
- **Custom Hooks**: `useHabits` hook manages all habit CRUD operations with optimistic updates

## 🎨 UI/Design System

### Dark Theme Standards
- **Primary Background**: `#0A0A0A` (body), `#101010` (sidebar), `#18181B` (cards/zinc-900)
- **Widget Cards**: Use `WidgetCard` component with `zinc-900` background and `border-white/10`
- **Typography**: JSX quote escaping required: `&ldquo;`, `&rdquo;`, `&apos;`

### Layout Patterns
```tsx
// Dashboard pages: Always wrap with DashboardLayout
<DashboardLayout user={user}>
  <YourContent />
</DashboardLayout>

// Grid layouts: Use xl:col-span-X for responsive columns
<div className="grid grid-cols-1 xl:grid-cols-3 gap-6">
  <div className="xl:col-span-3 grid grid-cols-1 lg:grid-cols-5 gap-6">
```

## 🔄 Data Flow Patterns

### Habit Management
- **Hook**: `useHabits()` for all habit operations (CRUD + real-time sync)
- **Types**: Import from `@/types/habit` - extensive type definitions for habits, logs, progress
- **API**: RESTful `/api/habits/*` with automatic progress calculation
- **State Updates**: Always call `fetchHabits()` and `fetchSummary()` after mutations

### Authentication Flow
- **Middleware**: JWT validation in `src/middleware.ts` protects `/dashboard/*` routes
- **User Data**: Extract from cookies server-side, pass to client components as props
- **Auth State**: Use `UserProfileDropdown` component for logout functionality

## 🛠️ Development Workflows

### Essential Commands
```bash
# Development (auto-starts MongoDB Memory Server)
pnpm dev

# Code quality pipeline (runs before commits)
pnpm lint:fix && pnpm format && pnpm type-check

# Database cleanup (if MongoDB Memory Server issues)
pnpm db:cleanup && pnpm dev
```

### Component Creation Patterns
```tsx
// Dashboard components: Always 'use client' with dark theme
'use client'
import { WidgetCard } from '@/components/ui/widget-card'

// Form components: Use react-hook-form + zod validation
const form = useForm<FormData>({ resolver: zodResolver(schema) })

// Modal patterns: Wrap in fixed overlay with zinc-900 backgrounds
<div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
```

## 🧪 Testing & Quality

### Current Setup
- **Unit Tests**: Vitest + React Testing Library (configured but deferred to MVC-002+)
- **E2E Tests**: Playwright configured for `/e2e` directory
- **Linting**: ESLint + Prettier with pre-commit hooks (husky)
- **Type Safety**: Strict TypeScript mode enabled

### Code Standards
- **Import Alias**: Always use `@/` for internal imports
- **Component Exports**: Default export for single components, named for multiple
- **Error Handling**: Use `handleError` pattern in hooks for consistent UX

## 🔍 Common Pitfalls

### MongoDB Connections
- **Development**: Never set `MONGODB_URI` (uses Memory Server automatically)
- **Production**: Memory Server disabled, requires `MONGODB_URI` environment variable
- **Dual Clients**: NextAuth and Mongoose maintain separate connection pools

### React/Next.js Patterns
- **Server Components**: Default behavior, use for data fetching and static content
- **Client Components**: Add `'use client'` for interactivity, hooks, browser APIs
- **Dynamic Imports**: Avoid `next/dynamic` with `{ ssr: false }` in Server Components

### Design Implementation
- **HTML References**: Match components exactly to files in `design/html/dashboard/`
- **Grid Responsiveness**: Always test xl:col-span patterns for desktop layouts
- **Quote Escaping**: Use HTML entities in JSX to pass ESLint validation

## 📁 Key Files Reference

- `src/hooks/useHabits.ts` - Complete habit management logic
- `src/lib/db.ts` - Database connection with Memory Server
- `src/components/layout/DashboardLayout.tsx` - Sidebar + navigation
- `src/types/habit.ts` - Comprehensive type definitions
- `design/html/dashboard/*.html` - UI design specifications to match exactly

---
> Source: [mrbin264/nextjs-mongodb-boilerplate](https://github.com/mrbin264/nextjs-mongodb-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-02 -->
