# GitHub Copilot Instructions for Personal Finance Dashboard

## Project Architecture

This is a **monorepo** with two main packages:
- **`backend/`**: Node.js + TypeScript + Express API (not yet implemented)
- **`frontend/`**: Next.js 16 (App Router) + TypeScript + shadcn/ui

**CRITICAL**: Frontend currently uses **Next.js**, NOT Vite as stated in `project.md`. The project guide is aspirational - always verify actual implementation before making assumptions.

### Data Flow Architecture
1. CSV files from Money Manager app are uploaded via frontend
2. Backend will parse CSV → normalize to unified Transaction schema → store in SQLite via Prisma
3. Frontend fetches transactions via REST API and displays analytics/charts

Current state: Frontend is standalone with mock data. Backend scaffold exists but has no implementation yet.

## Technology Decisions (Actual vs. Planned)

| Component | Planned (project.md) | Actual Implementation |
|-----------|---------------------|----------------------|
| Frontend Framework | React + Vite | **Next.js 16 (App Router)** |
| UI Library | TailwindCSS | **TailwindCSS + shadcn/ui (New York style)** |
| Charts | Recharts | **Recharts** ✓ |
| Backend | Express + TypeScript | **Not yet implemented** |
| Database | SQLite + Prisma | **Prisma schema defined, not connected** |
| CSV Parsing | csv-parse (backend) | **Frontend parseCSV function** (temporary) |

## File Structure Conventions

### Frontend (`frontend/`)
- **App Router**: All pages in `app/` directory
- **Components**: Separate files per component in `components/`, UI primitives in `components/ui/`
- **Types**: Shared types in `types/` (e.g., `types/transaction.ts`)
- **Utils**: Helper functions in `lib/` (e.g., `lib/utils.ts` for cn(), `lib/mock-data.ts`)
- **Path aliases**: Use `@/` prefix (e.g., `@/components`, `@/types`, `@/lib`)

### Backend (`backend/`)
- **Prisma**: Schema in `prisma/schema.prisma`, generates client to `generated/prisma/`
- **Config**: Custom Prisma config in `prisma.config.ts`
- **Structure**: No source code yet - needs routes, controllers, services architecture

## Critical Data Schema

### Transaction Interface
```typescript
// Frontend: types/transaction.ts
interface Transaction {
  id: string              // Frontend uses string
  date: string            // ISO 8601: YYYY-MM-DD
  description: string
  amount: number          // Negative = expense, positive = income
  category: string
  type: "income" | "expense"
}

// Backend (planned): Will include additional fields
// - account: string | null
// - note: string | null
// - createdAt: timestamp
// - type: "income" | "expense" | "transfer"
```

**Important**: Frontend `Transaction` is simplified. Backend schema in `project.md` is more complete.

## Development Workflows

### Frontend Development
```powershell
cd frontend
pnpm dev           # Start Next.js dev server (port 3000)
pnpm build         # Production build
pnpm lint          # ESLint check
```

### Backend Development (when implemented)
```powershell
cd backend
npm run dev        # ts-node server (planned)
npx prisma migrate dev --name <description>  # Create migration
npx prisma studio  # Open Prisma Studio GUI
```

### Prisma-Specific Commands
- **Prisma Client generation**: Outputs to `backend/generated/prisma/` (custom config)
- **Database**: SQLite file will be at `backend/prisma/dev.db` (via DATABASE_URL env var)

## UI Component Conventions

### shadcn/ui Usage
- All UI components are in `components/ui/` and follow shadcn New York style
- Import using path alias: `@/components/ui/card`, `@/components/ui/button`, etc.
- Icons: Use **lucide-react** (`lucide-react` package)
- Styling: Use Tailwind utility classes with `cn()` helper from `lib/utils.ts`

### Component Structure Pattern
```typescript
// Feature components export named functions
export function DashboardStats({ transactions }: Props) { }

// Use "use client" directive for interactive components
"use client"
import { useState } from "react"
```

### Chart Implementation
- **Library**: Recharts
- **Pattern**: See `components/dashboard-charts.tsx` for examples
- **Category colors**: Defined in `CATEGORY_COLORS` constant with specific hex values
- **Responsive**: Wrap charts in `<ResponsiveContainer>` with height in pixels

## CSV Parsing Strategy

**Current**: Basic `parseCSV()` function in `frontend/app/page.tsx` (lines 118-140)
- Splits by newlines and commas (naive approach)
- Maps CSV headers to transaction fields
- Generates client-side IDs using `Date.now()`

**Planned**: Backend will handle CSV parsing with proper library (csv-parse or papaparse) and validation with Zod.

## State Management

**Current approach**: React `useState` at page component level
- `transactions`: All loaded transactions
- `filteredTransactions`: Subset based on active filters
- No global state management library (Redux, Zustand, etc.)

Filter logic in `handleFilterChange()` supports:
- Category multi-select
- Date range (from/to)
- Month-specific filtering
- Text search (description + category)

## Integration Points (Planned)

### Backend API Endpoints (To Be Implemented)
```
POST   /api/upload          - Upload CSV file
GET    /api/transactions    - List with filters (category, date range, search)
GET    /api/analytics       - Summary stats (monthly totals, category breakdowns)
```

### Frontend → Backend Communication
- Use Axios for HTTP requests (specified in project.md, not yet installed)
- Base URL should be configurable via env var for Docker deployment

## Docker Deployment (Not Yet Configured)

Project is designed for Docker Compose deployment on Linux home server:
- Separate containers for frontend and backend
- SQLite database persisted via volume mount
- No Docker files exist yet - needs creation

## Known Issues & Workarounds

1. **TypeScript errors ignored**: `next.config.mjs` has `ignoreBuildErrors: true` - fix type issues rather than suppressing
2. **Images unoptimized**: `unoptimized: true` in Next config - acceptable for self-hosted
3. **parseCSV is naive**: Doesn't handle quoted commas or escaped characters - will be replaced by backend implementation
4. **Mock data on load**: `generateMockTransactions()` runs on component mount - remove when backend is ready

## Code Style & Conventions

- **TypeScript**: Use `interface` for object shapes, `type` for unions/primitives
- **Async/await**: Prefer over promises chains
- **Named exports**: Use for components (not default exports)
- **File naming**: kebab-case for files (`dashboard-stats.tsx`), PascalCase for components (`DashboardStats`)
- **Component props**: Define inline interface or separate `Props` type above component

## Testing Strategy (Future)

Project.md mentions optional Jest/Vitest but no tests exist yet. When implementing:
- Backend: Unit tests for CSV parsing, transaction validation
- Frontend: Component tests for key dashboard features
- E2E: Not specified, likely not needed for personal project

## References

- **Project Guide**: `project.md` contains architecture decisions and roadmap (but verify against actual code)
- **shadcn/ui Docs**: https://ui.shadcn.com
- **Next.js App Router**: https://nextjs.org/docs/app
- **Prisma Docs**: https://www.prisma.io/docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slate-works) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
