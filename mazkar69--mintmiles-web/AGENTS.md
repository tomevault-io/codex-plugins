# Value4Car Copilot Instructions

## Project Overview
Next.js 16 car marketplace connecting verified professionals (doctors, judges, teachers) with premium used car buyers. Two-sided platform: public-facing marketplace + admin dashboard.

## Tech Stack & Architecture
- **Framework**: Next.js 16 (App Router) with React 19, TypeScript
- **Styling**: Tailwind CSS v4 with custom OKLCH color system (see `globals.css`)
- **UI Components**: shadcn/ui (New York style) with Radix UI primitives
- **Animations**: Framer Motion for micro-interactions
- **State**: React Context for admin auth, client components for interactivity
- **Package Manager**: pnpm (lock file present)

## Development Commands
```bash
pnpm dev          # Start dev server
pnpm build        # Production build (TypeScript errors ignored in config)
pnpm lint         # ESLint check
```

## Key Architectural Patterns

### Routing Structure
- `/` - Public homepage with hero, services, featured cars
- `/cars` - Car browsing with client-side filtering
- `/cars/[id]` - Individual car details
- `/admin/*` - Protected admin dashboard (separate layout)
- `/admin/login` - Entry point (hardcoded auth: admin@value4car.com/admin123)

### Client vs Server Components
- **Default to client components** for interactivity (use `"use client"`)
- All admin pages are client components (auth context requirement)
- Most car listing/filtering pages are client components (search, filters)
- Root layout is server component for metadata

### Admin Authentication Pattern
Location: `contexts/admin-auth-context.tsx`
```typescript
// Frontend-only auth via localStorage (marked TODO for backend)
// AdminAuthProvider wraps admin routes in admin/layout.tsx
// Protected routes redirect to /admin/login when unauthenticated
```

**Critical**: Admin layout (`app/admin/layout.tsx`) handles:
1. Auth check with loading skeleton
2. Redirect logic for unauthenticated users
3. Sidebar navigation (LayoutDashboard, Car, Users, MessageSquare icons)
4. Theme toggle in sidebar footer

### Data Flow (Current State)
- **Mock data in components** - No backend yet
- Car data: Hardcoded arrays in `app/cars/page.tsx` and `app/admin/cars/page.tsx`
- Admin credentials: Hardcoded in `admin-auth-context.tsx` (TODO noted for backend)

## Component Conventions

### UI Components Import Pattern
```typescript
// Always use @ alias from tsconfig paths
import { Button } from "@/components/ui/button"
import { CarCard } from "@/components/car-card"
import { cn } from "@/lib/utils"  // Utility for className merging
```

### CarCard Interface
All car objects must match:
```typescript
{
  id: string
  brand: string
  model: string
  year: number
  price: number
  image: string
  ownerType: string  // "Doctor", "Judge", "Teacher", etc.
  location: string
  mileage: string
}
```

### Theme Implementation
- Uses next-themes with system preference detection
- Theme toggle: Sun/Moon icons with rotation transitions (see `navbar.tsx`)
- Dark mode class: `.dark` (applied to root element)
- Custom OKLCH color tokens in `:root` and `.dark` (no RGB/HSL)

### Animation Patterns
Framer Motion usage:
```tsx
// Card hover lift
<motion.div whileHover={{ y: -4 }} transition={{ duration: 0.2 }}>

// Initial page load
<motion.div initial={{ opacity: 0, x: -20 }} animate={{ opacity: 1, x: 0 }}>

// Mobile menu toggle
<AnimatePresence>
  {isOpen && <motion.div initial={{ opacity: 0, height: 0 }} ...>}
</AnimatePresence>
```

## Styling Guidelines

### Color System
- Use OKLCH format (not RGB/HSL) for custom colors
- Semantic tokens: `--background`, `--foreground`, `--primary`, `--accent`, `--muted`, etc.
- Access via Tailwind: `bg-background`, `text-foreground`, `border-border`

### Component Styling
- Use `cn()` utility for conditional classes: `cn("base-class", condition && "conditional")`
- Prefix-free Tailwind (no custom prefix in config)
- Responsive: mobile-first, breakpoints `md:`, `lg:`
- Spacing: consistent with Tailwind scale (p-4, gap-6, space-y-4)

## SEO & Metadata
- Comprehensive metadata in `app/layout.tsx` (OpenGraph, Twitter, Schema.org)
- Page-specific metadata via exported `metadata` object
- Schema.org AutoDealer JSON-LD in homepage
- Images: unoptimized in config (see `next.config.mjs`)

## Known Technical Debt
1. **Auth**: Frontend-only localStorage auth (see TODO in `admin-auth-context.tsx`)
2. **Data**: All car listings are mock data in component files
3. **TypeScript**: Build errors ignored (`ignoreBuildErrors: true` in next.config)
4. **Images**: Unoptimized (referencing local assets in `/public`)

## File Organization
- `app/` - All routes and layouts
- `components/` - Shared components (ui/ for shadcn, home/ for sections)
- `contexts/` - React contexts (currently only admin auth)
- `lib/` - Utilities (only `utils.ts` with `cn()`)
- `hooks/` - Copied from shadcn (toast, mobile detection)

## When Adding New Features
1. **New car routes**: Follow `/cars/[id]` pattern, wrap with Navbar + Footer
2. **Admin pages**: Use client components, ensure AdminAuthProvider in layout
3. **UI components**: Run `npx shadcn@latest add [component]` (components.json configured)
4. **Forms**: Use react-hook-form + zod (dependencies installed)
5. **Icons**: Import from lucide-react (e.g., `Search`, `Car`, `MapPin`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mazkar69)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mazkar69)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
