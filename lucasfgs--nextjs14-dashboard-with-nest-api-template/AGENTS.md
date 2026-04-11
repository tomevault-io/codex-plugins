# AI Agent Instructions for Next.js Dashboard Template

## Architecture Overview

This is a **Next.js 14+ frontend** with App Router that connects to a separate NestJS backend API. The app uses cookie-based JWT authentication with automatic token refresh, role-based access control, and real-time WebSocket integration.

### Key Architecture Patterns

- **Authentication Flow**: JWT tokens stored in cookies, middleware handles auth on `/dashboard/*` routes, automatic refresh via Axios interceptors
- **Permission System**: Granular permissions (create/read/update/delete) per resource (users/roles/settings), enforced via `useCanAccess` hook
- **API Integration**: Single Axios instance (`src/configs/api.ts`) with request queuing during token refresh
- **State Management**: React Query for server state, React Context for user/permissions

## Critical Development Workflows

### Authentication Development

- User context comes from middleware header `x-authenticated-user` → `getAuthenticatedUser()` → dashboard layout
- Permission checks: Use `useCanAccess(EPermission.USERS, EPermissionType.READ)` pattern
- API calls automatically handle 401s and token refresh - never manually manage tokens

### Data Table Pattern

```typescript
// Standard pattern for paginated tables
const { data } = useGetAllUsers(paginationParams);
const { sorting, pagination, columnFilters, ... } = useDataTable();
```

### Service Layer Convention

All API services follow this structure:

```
src/services/api/{resource}/
  ├── use-get-all-{resource}.ts   # List with pagination
  ├── use-get-{resource}.ts       # Single item
  ├── use-add-{resource}.ts       # Create
  ├── use-update-{resource}.ts    # Update
  └── use-delete-{resource}.ts    # Delete
```

## Project-Specific Conventions

### File Organization

- **Routes**: Use App Router with route groups `(auth)` for auth pages
- **Components**:
  - `components/ui/` - Radix UI + Tailwind components
  - `components/custom/` - Project-specific components (data-table, step-menu)
  - `components/pages/{route}/` - Page-specific components
- **Services**: Domain-based (`auth/`, `users/`, `roles/`) with consistent naming

### API Integration Patterns

- Import `api` from `@/configs/api` (never use fetch directly)
- Use React Query with consistent query key patterns: `["resource", "action", params]`
- Pagination utilities: `createPaginationUrl()`, `createPaginationQueryKey()`
- Response types: All APIs return `ApiResponse<T>` wrapper

### Permission & Route Protection

- Dashboard routes automatically protected by middleware
- Component-level permission checks: `const canEdit = useCanAccess(EPermission.USERS, EPermissionType.UPDATE)`
- Auth context provides `user` and `permissions` throughout dashboard

### WebSocket Integration

- Use `useSocket()` hook from `@/services/socket/use-socket`
- Automatically connects when user is authenticated
- Pattern: `listenToEvent(eventName, callback)` and `sendEvent(eventName, data)`

## Development Commands & Environment

```bash
yarn dev          # Development with Turbopack
yarn build        # Production build
yarn lint         # ESLint check
```

### Required Environment Variables

```env
NEXT_PUBLIC_API_URL="http://localhost:4000"  # Backend API URL
JWT_SECRET="your-jwt-secret"                  # For middleware token verification
APP_PREFIX=""                                 # Optional path prefix
```

## Integration Points

### Backend Communication

- **Base URL**: `process.env.NEXT_PUBLIC_API_URL`
- **Credentials**: `withCredentials: true` for cookie-based auth
- **Error Handling**: Axios interceptors handle 401s, queue requests during refresh
- **WebSocket**: Connects to same backend, auth via existing cookies

### External Services

- **Payments**: Stripe integration via `use-create-checkout-session`
- **Real-time**: Socket.io client for live updates
- **UI**: Radix UI primitives with Tailwind styling

## Key Files for Understanding Patterns

- `src/configs/api.ts` - Core API client with refresh logic
- `src/middlewares/authentication.ts` - Route protection and token refresh
- `src/components/providers/authentication.tsx` - User context and permissions
- `src/utils/isAllowed.ts` - Permission checking logic
- `src/components/custom/data-table/` - Reusable table component pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucasfgs)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lucasfgs)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
