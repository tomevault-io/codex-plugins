# SportMaps Copilot Instructions

AI coding agents should use this guide to become immediately productive in the SportMaps codebase.

## Architecture Overview

**SportMaps** is a comprehensive sports management platform built with React 18, TypeScript, Tailwind CSS, and Supabase. The architecture uses a component-driven, role-based design with clear separation of concerns.

### Key Data Flow (Real Implementation)

```
AuthContext (user, profile, session)
    ↓
useDashboardStats(role) → fetch real data from Supabase
    ↓
useDashboardConfig(role, stats) → transform to UI config
    ↓
DashboardPage composes: StatCard, ActivityList, QuickActions, NotificationList
    ↓
PermissionGate + usePermissions validate access layer
    ↓
Final role-adaptive UI rendered
```

### 7 Core Roles with Feature Flags & Data Visibility

Each role has:
1. **Permissions** - Granular actions (`calendar:create`, `teams:delete`, etc.)
2. **Feature Flags** - UI feature toggles (e.g., `canViewFinances`)
## SportMaps — Copilot Instructions (concise)

Purpose: Give AI coding agents the exact, repo-specific knowledge needed to be productive quickly.

Quick start
- Install: `npm install`
- Dev server: `npm run dev` (Vite)
- Build: `npm run build`
- Lint: `npm run lint`
- Type check: `tsc --noEmit`

Architecture (short)
- Stack: React 18 + TypeScript + Tailwind + Supabase (RLS) + Vite
- Key pattern: `AuthContext` → `useDashboardStats` → `useDashboardConfig` → presentational components

Key files & where to look
- Auth & profile: `src/contexts/AuthContext.tsx`
- Dashboard config: `src/hooks/useDashboardConfig.ts`
- Data fetching: `src/hooks/useDashboardStats.ts`
- Permissions matrix: `src/lib/permissions.ts`
- Permission hook: `src/hooks/usePermissions.ts`
- UI guard: `src/components/PermissionGate.tsx`
- Navigation: `src/config/navigation.ts`
- Design tokens: `src/index.css` (HSL vars — must use tokens)
- Demo creds: `src/lib/demo-credentials.ts`

Core conventions (examples)
- Components: PascalCase `.tsx` (e.g., `StatCard.tsx` in `src/components/dashboard/`)
- Hooks: `useCamelCase` (e.g., `usePermissions.ts`)
- Fetch at page level: use `useDashboardStats` in page components and pass data down to pure components
- Permission pattern: update `lib/permissions.ts` → use `usePermissions()` → wrap UI with `PermissionGate`

Styling & tokens
- Use CSS tokens in `src/index.css` (HSL format). Example: `className="bg-primary text-primary-foreground"`
- Avoid hardcoded colors or inline RGB/HEX.

Supabase notes
- Use the centralized `supabase` client (under `src/integrations/supabase/`).
- Backend enforces RLS; front-end uses `AuthContext` + `usePermissions` for UI gating only.

Developer tips & gotchas
- Do not hardcode role strings — use `usePermissions().hasRole()` or `PermissionGate`.
- Do not fetch inside small components — fetch at page/hook level and pass props.
- Dashboard pattern is authoritative: always prefer `useDashboardConfig(role, stats)` for role-specific UI.

If you edit this file
- Keep it concise. Include code references (file paths) and concrete examples.
- Preserve the Dashboard → Permissions → Tokens patterns above.

Questions or missing info?
- Tell me which section you want expanded (auth, permissions, CI, or local debug steps).

Last updated: November 2025

---
> Source: [brylop/sportmaps-demo](https://github.com/brylop/sportmaps-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
