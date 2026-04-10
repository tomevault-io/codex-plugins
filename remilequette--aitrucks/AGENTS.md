# GitHub Copilot Instructions for AItrucks

## Project Overview
AItrucks is a delivery planning and fleet management application built with React, TypeScript, Vite, and Supabase. It helps manage deliveries, vehicles, trips, and route optimization.

## Documentation

Comprehensive documentation is available in the `docs/` folder. **Always update documentation when making significant changes:**

- **`docs/ARCHITECTURE.md`** - System design, component structure, tech stack
- **`docs/DATABASE.md`** - Database schema, relationships, PostGIS usage, RLS policies  
- **`docs/BUSINESS-LOGIC.md`** - Business rules, workflows, status transitions, validation rules
- **`docs/API.md`** - API endpoints, request/response formats, authentication patterns
- **`README.md`** - Quick start guide and project overview
- **`.github/copilot-instructions.md`** (this file) - Coding standards

### When to Update Documentation

Update documentation when:
- ✅ Adding new tables or modifying database schema → Update `DATABASE.md`
- ✅ Changing business rules (e.g., status transitions, validation) → Update `BUSINESS-LOGIC.md`
- ✅ Adding new API endpoints or modifying existing ones → Update `API.md`
- ✅ Introducing new architectural patterns → Update `ARCHITECTURE.md`
- ✅ Adding new features or major changes → Update `README.md`
- ✅ Changing coding conventions → Update this file

## Architecture Principles

### Component Organization
- **`src/components/`** - Reusable, business-agnostic UI components (Table, Map, Button, Modal)
  - Must NOT contain business logic or domain-specific code
  - Should be generic and reusable across any project
  - Export via barrel files (index.ts)
  
- **`src/layout/`** - Application-specific layout components (AppLayout)
  - Can contain business domain knowledge (navigation, routes)
  - Tied to AItrucks application structure
  
- **`src/pages/`** - Business domain pages (Deliveries, Vehicles, Trips, Dashboard)
  - Contains business logic and domain-specific features
  - Uses components from `src/components/` and `src/layout/`

### Code Style

#### TypeScript
- Use explicit types for all props interfaces
- Prefer `interface` over `type` for object shapes
- Use `React.FC` or explicit function declarations with typed props
- Enable strict mode checks

#### React Patterns
- Functional components only
- Use hooks (useState, useEffect, useMemo, useCallback)
- Prefer composition over inheritance
- Keep components focused and single-responsibility

#### Naming Conventions
- Components: PascalCase (e.g., `DeliveryTable`, `AppLayout`)
- Files: Match component name (e.g., `DeliveryTable.tsx`)
- Utilities: camelCase (e.g., `createBadgeColumn`, `parseLocation`)
- CSS classes: kebab-case with BEM-like structure (e.g., `data-table__header--sortable`)

### Reusable Components

#### Table Component
- Use `@tanstack/react-table` for table logic
- Export column helper functions from `columnHelpers.tsx`
- Support sorting, filtering, row selection
- Accept generic type parameter: `Table<TData>`

#### Map Component
- Use `react-leaflet` for mapping
- Parse PostGIS POINT format: `"POINT(lng lat)"`
- Support markers, routes, auto-fit bounds
- Provide helper functions for creating markers from domain data

### State Management
- Use React Context for global state (Auth, Theme)
- Use Supabase real-time subscriptions where needed
- Keep local state in components when possible
- Use `useMemo` for expensive computations

### API & Data
- All API calls go through `src/services/api.ts`
- Use Supabase client from `src/config/supabase.ts`
- Handle errors gracefully with try-catch
- Show user-friendly error messages
- Use TypeScript types for API responses

### Styling
- Use CSS modules or component-specific CSS files
- Keep global styles minimal (in `index.css`)
- Follow existing color scheme and spacing
- Prefer CSS Grid and Flexbox for layouts
- Mobile-responsive where applicable

### Testing
- Write tests using Vitest and React Testing Library
- Test files: `*.test.tsx` alongside component
- Focus on user behavior, not implementation
- Mock Supabase calls in tests
- Use Playwright for E2E tests

### Security
- Never commit `.env` files
- Use Row Level Security (RLS) policies in Supabase
- Validate user permissions with `hasRole()` helper
- Sanitize user inputs
- Use HTTPS for all external requests

### Performance
- Memoize expensive computations with `useMemo`
- Memoize callbacks with `useCallback`
- Lazy load routes and heavy components
- Optimize re-renders with React.memo where needed
- Use pagination for large datasets

### Code Quality
- No console.logs in production code (use proper logging)
- Handle loading and error states
- Provide empty states for lists/tables
- Add TypeScript JSDoc comments for complex functions
- Keep functions small and focused

### Domain-Specific Rules

#### Deliveries
- Status: `pending`, `assigned`, `in_transit`, `delivered`, `failed`
- Always validate `delivery_location` exists before displaying on map
- Weight in kg, Volume in m³

#### Vehicles
- Status: `available`, `in_use`, `maintenance`, `inactive`
- Track capacity (weight and volume)
- Store `current_location` as PostGIS POINT

#### Trips
- Status: `planned`, `in_progress`, `completed`, `cancelled`
- Use `trip_deliveries` junction table with `sequence_order`
- Calculate totals (weight, volume, distance) from assigned deliveries
- Support drag-and-drop reordering of deliveries in trips

#### Users & Auth
- Roles: `driver`, `dispatcher`, `trip_planner`, `admin`
- Use Supabase Auth for authentication
- Store sessions in localStorage
- Implement role-based access control (RBAC)

## Common Patterns

### Creating a new reusable component
1. Create folder in `src/components/ComponentName/`
2. Add `ComponentName.tsx`, `types.ts`, `ComponentName.css`
3. Add helper functions if needed
4. Export via `index.ts`
5. Write tests in `ComponentName.test.tsx`

### Adding a new page
1. Create `src/pages/PageName.tsx`
2. Add route in `App.tsx`
3. Add navigation link in `AppLayout.tsx` (if needed)
4. Use existing components from `src/components/`
5. Add role-based access control if needed

### Working with Supabase
```typescript
// Always handle errors
try {
  const { data, error } = await supabase.from('table').select('*');
  if (error) throw error;
  return data;
} catch (error) {
  console.error('Error:', error);
  alert('User-friendly message');
}
```

## Don't
- ❌ Put business logic in reusable components
- ❌ Hardcode domain entities in `src/components/`
- ❌ Use `any` type without good reason
- ❌ Commit sensitive data or API keys
- ❌ Skip error handling
- ❌ Create deeply nested component hierarchies
- ❌ Use inline styles for complex styling
- ❌ Fetch data in render functions

## Do
- ✅ Keep components focused and reusable
- ✅ Use TypeScript strictly
- ✅ Handle loading and error states
- ✅ Write tests for critical functionality
- ✅ Follow the established patterns
- ✅ Document complex logic
- ✅ Validate user inputs
- ✅ Use semantic HTML
- ✅ **Update relevant documentation files when making significant changes**
- ✅ **Keep business rules documented in BUSINESS-LOGIC.md**
- ✅ **Document database changes in DATABASE.md**
- ✅ **Update API.md when adding/modifying endpoints**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RemiLequette)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/RemiLequette)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
