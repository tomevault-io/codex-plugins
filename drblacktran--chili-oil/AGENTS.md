# Benjamin's Chili Oil - AI Coding Agent Instructions

## Project Context

This is a **distribution management system** for Benjamin's Chili Oil, tracking inventory across 10+ retail stores in Melbourne. The system consists of a **completed frontend** (Astro + React PWA) with a **planned backend** (architecture TBD).

**Key Architecture Principle**: Database uses **application-layer constraints** (no DB-level constraints) for business logic flexibility. All validation happens in code.

## Essential File Structure

```
business.config.local.ts    # Gitignored business data (pricing, stores, contacts)
DATABASE_SCHEMA_V2.md       # Complete PostgreSQL schema (11 tables, triggers, functions)
frontend/
  src/
    components/             # React functional components (strict TypeScript)
    pages/                  # Astro file-based routing (SSR + client islands)
    types/                  # Type definitions (authoritative types)
      inventory.ts          # Inventory, product, location types
      product.ts            # Product catalog types
      dashboard.ts          # Dashboard widget types
      hub.ts                # Hub expansion & region types (Phase 2)
    utils/                  # Business logic utilities
      mockData.ts           # Mock data generators (until backend ready)
      melbourneRegions.ts   # 7 default Melbourne regions configuration
      hubEconomics.ts       # Hub economic viability calculator
```

## Critical Workflows

### Development Commands
```bash
cd frontend
npm run dev          # Dev server on :4321
npm run typecheck    # MUST pass (0 errors) before commits
npm run build        # Production build
```

### Pre-commit Flow
- Git hooks auto-run `npm run typecheck` before every commit
- Setup: `./scripts/setup-hooks.sh` (required for new contributors)
- Commits blocked if TypeScript errors exist
- CI/CD validates on push via GitHub Actions

## Code Patterns (Non-negotiable)

### Component Structure
**Every React component must**:
1. Use TypeScript with `interface ComponentNameProps`
2. Be functional (no class components)
3. Include JSDoc header comment explaining purpose
4. Import types from `../types/` (never inline types)

**Example** (`AlertCard.tsx`):
```tsx
/**
 * AlertCard Component
 * Displays low stock alerts with severity levels
 */
import type { LowStockAlert } from '../types/dashboard';

interface AlertCardProps {
  alert: LowStockAlert;
  onAction?: () => void;
}

export default function AlertCard({ alert, onAction }: AlertCardProps) {
  // Implementation...
}
```

### Styling Conventions
- **TailwindCSS 4.1** via Vite plugin (no PostCSS config needed)
- **No custom CSS classes** - use Tailwind utilities exclusively
- Color system: `text-red-600` for primary brand, `text-gray-*` for neutral tones
- Status colors: `red` (critical), `yellow` (warning), `blue` (info), `green` (healthy)

### Type System (Matches DATABASE_SCHEMA_V2.md)
- **Source of truth**: `frontend/src/types/inventory.ts`
- All types mirror PostgreSQL schema exactly (e.g., `stock_status: 'healthy' | 'low' | 'critical' | 'overstocked'`)
- Use `InventoryItem`, `ProductWithVariants`, `StoreLocation`, `StockMovement`, `AlertQueueItem` interfaces
- Calculation fields (e.g., `stock_value`, `profit_per_unit`) are **always computed**, never hardcoded

## Data Flow Architecture

### Current (Mock Data Phase)
```
Astro Page → imports mockData → passes to React component (client:load) → renders
```
**Example**: `pages/index.astro` imports `mockDashboardStats` from `utils/mockData.ts`

### Future (Backend Integration)
```
Astro Page (SSR) → fetch('/api/...') → pass to component → client-side updates
```

### PWA Strategy
- Service worker caches API responses (NetworkFirst, 1hr TTL) and images (CacheFirst, 30d TTL)
- Offline indicator: `<OfflineIndicator />` component in `MainLayout.astro`
- No IndexedDB yet - current focus is online-first with graceful degradation

## Business Logic Patterns

### Restock Calculation (21-day cycle)
```typescript
// Based on DATABASE_SCHEMA_V2.md trigger logic
next_restock_date = last_restock_date + restock_cycle_days
needs_restock = (current_stock < minimum_stock) OR (days_until_restock <= 3)
```

### Profit Formula (R × (1 - C) - U)
```typescript
// From products table schema
profit_per_unit = retail_price * (1 - commission_rate / 100) - unit_cost
// Example: $12.80 × (1 - 0.30) - $4.50 = $4.46
```

### Stock Status Rules
```typescript
// From inventory.stock_status enum
if (current_stock === 0) return 'critical';
if (current_stock < minimum_stock) return 'low';
if (current_stock > maximum_stock) return 'overstocked';
return 'healthy';
```

### Hub Economics Calculation (Phase 2)
```typescript
// From utils/hubEconomics.ts
const economics = calculateHubEconomics(storeCount, commissionRate, storageFee, setupCost);
// Returns: monthly_savings, break_even_months, roi_percentage, is_economical

// Viability criteria:
// - Minimum 3 stores in region
// - Minimum $100/month savings
// - Maximum 24 months to break even
```

### Region Auto-Assignment (Phase 2)
```typescript
// From utils/melbourneRegions.ts
const region = getRegionByPostcode('3000'); // Returns 'CBD & Inner City'
// Auto-assigns stores to 1 of 7 default Melbourne regions based on postcode
```

## Integration Points

### Business Configuration
- **Never commit** `business.config.local.ts` (contains real pricing/contacts)
- Use `business.config.example.ts` as template
- Access via `import { BUSINESS_CONFIG } from '../business.config.local'`

### PWA Configuration
- Manifest in `astro.config.mjs` under `AstroPWA()` integration
- Icons: Use SVG (`/pwa-icon.svg`) for dev; PNG + screenshots required for production
- Workbox caching: Defined per-route pattern (API vs static assets)

### API Endpoints (Planned)
When backend is implemented, follow REST conventions:
- `GET /api/inventory` → list inventory with filters
- `POST /api/stock-movements` → create transfer
- `PATCH /api/inventory/:id` → update stock level
- `POST /api/alerts/:id/approve` → approve alert for sending

## Common Pitfalls

### TypeScript Errors
**Issue**: `Property 'X' does not exist on type 'Y'`
- **Cause**: Type definitions in `src/types/` out of sync with mock data
- **Fix**: Update types to match `DATABASE_SCHEMA_V2.md` schema

### Astro Islands
**Issue**: Component not interactive
- **Cause**: Missing `client:load` directive in `.astro` file
- **Fix**: `<Component client:load />` for any component with hooks/events

### Mock Data Realism
**Issue**: Mock data doesn't reflect real business constraints
- **Cause**: Generated data violates schema rules (e.g., stock_status mismatch)
- **Fix**: Check `utils/mockInventoryData.ts` against trigger logic in `DATABASE_SCHEMA_V2.md`

## Documentation References

- **Database Schema**: `DATABASE_SCHEMA_V2.md` (726+ lines - authoritative schema with Phase 2 hub tables)
- **Frontend Guide**: `frontend/AGENT.md` (1,900+ lines - comprehensive dev guide with hub expansion)
- **CI/CD**: `CI_SETUP.md` (git hooks, GitHub Actions workflow)
- **Main README**: Project overview, tech stack, deployment
- **Hub Economics**: `frontend/src/utils/hubEconomics.ts` (calculator functions and viability criteria)
- **Melbourne Regions**: `frontend/src/utils/melbourneRegions.ts` (7 default regions with postcodes)

## Development Philosophy

1. **Type Safety First**: 0 TypeScript errors is non-negotiable
2. **Schema Alignment**: All types mirror `DATABASE_SCHEMA_V2.md` exactly
3. **Component Isolation**: Each component is self-contained with clear props interface
4. **Mock Realism**: Mock data must follow real business rules (restock cycles, profit formulas)
5. **Offline-Ready**: All features must gracefully handle offline state via PWA
6. **Economic Viability**: Hub expansion decisions driven by data (calculate before building)
7. **Hybrid Approach**: Provide sensible defaults, allow customization (regions, hub types)

## When Making Changes

**Adding a new component**:
1. Create type interface (or import from `src/types/`)
2. Add JSDoc comment
3. Use TailwindCSS utilities only
4. Add to relevant Astro page with `client:load`

**Modifying data structures**:
1. Check `DATABASE_SCHEMA_V2.md` first
2. Update type in `src/types/`
3. Update mock data generators in `utils/`
4. Run `npm run typecheck`

**Adding a route**:
1. Create `.astro` file in `src/pages/`
2. File path = URL path (e.g., `alerts/pending.astro` → `/alerts/pending`)
3. Wrap in `<MainLayout>` with `title` and `currentPage` props

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drblacktran)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/drblacktran)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
