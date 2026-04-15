# SE357-FE: MusicStream Frontend - AI Coding Agent Guide

## Project Overview
React 19 + TypeScript + Vite music streaming frontend with dual-layout architecture (User + Admin). Backend proxied at `/api` to `https://music-share-system.onrender.com`.

**Tech Stack**: React 19.2, TypeScript 5.9, Vite 7.2, React Router 7.9, Recharts 3.5, Lucide-React

## Architecture Patterns

### File-Based Routing (Next.js-Inspired)
Routes defined in `src/main.tsx` but follow directory structure pattern:
- `src/app/*/page.tsx` → Route pages (e.g., `albums/page.tsx` → `/home/albums`)
- `src/app/*/layout.tsx` → Layout wrappers with `<Outlet />`
- Three layout zones: `/auth` (no sidebar), `/home` (user layout), `/admin` (admin layout)
- **Critical**: All user-facing routes under `/home` base path, not root `/`

**Navigation Example**:
```tsx
// ❌ Wrong - navigates to /albums
navigate('/albums')

// ✅ Correct - navigates to /home/albums  
navigate('/home/albums')
```

### Dual Layout System
1. **User Layout** (`src/app/layout.tsx`):
   - Structure: Sidebar + Header + Content + MusicPlayer
   - Spotify-inspired glassmorphism design (purple/blue gradients)
   - Base route: `/home/*`

2. **Admin Layout** (`src/app/admin/components/AdminLayout.tsx`):
   - Structure: AdminSidebar + Content + AdminFooter
   - Dark theme with teal accents (`--admin-bg`, `--admin-primary`)
   - Base route: `/admin/*`
   - Uses Recharts for analytics dashboards

### API Client Pattern
Centralized request handler in `src/lib/apiClient.ts`:
- **Token Management**: `tokenManager.getToken()` / `setToken()` / `clearAll()`
- **Auto-Auth Headers**: Adds `Bearer ${token}` unless `skipAuth: true`
- **Typed Responses**: `ApiResponse<T>` wrapper with `{ data, message, success }`
- **Error Handling**: Throws `ApiError` with statusCode + message

**Service Layer Pattern**:
```typescript
// src/services/authService.ts or adminService.ts
import { apiClient } from '../lib/apiClient';
import { ENDPOINTS } from '../config/api';

export const exampleService = {
  async getData(): Promise<DataType> {
    return apiClient.get<DataType>(ENDPOINTS.example.path, { 
      params: { page: 1 }
    });
  }
};
```

### API Endpoint Configuration
All endpoints centralized in `src/config/api.ts`:
- `API_BASE_URL = '/api'` (proxied via Vite to backend)
- `ENDPOINTS` object with nested structure (auth, music, user, admin)
- Dynamic path functions: `ENDPOINTS.admin.userById(id)` → `/admin/users/${id}`

**Vite Proxy Config**: Dev server rewrites `/api/*` → `https://music-share-system.onrender.com/*` (strips `/api` prefix)

### Import Path Conventions
Relative imports from `src/app/*` use `../` levels:
- `src/app/page.tsx` → `'../components/Header'`
- `src/app/albums/page.tsx` → `'../../lib/icons'`
- `src/app/admin/reports/page.tsx` → `'../components/AdminFooter'`

**No path aliases** - use relative paths consistently.

## Component Patterns

### Icons
Two icon libraries coexist:
1. **Custom SVGs** in `src/lib/icons.tsx` (Home, Play, Search, etc.)
2. **Lucide-React** for admin UI (Bell, Moon, Info)

Import from appropriate source based on context:
```tsx
import { Play, Search } from '../lib/icons';        // User UI
import { Bell, Moon, Info } from 'lucide-react';    // Admin UI
```

### State Management
- **No global state library** - use React Context for shared state
- **PlayerProvider** (`src/providers/PlayerProvider.tsx`): Global music player state
  - Context: `{ currentTrack, isPlaying, volume, currentTime, queue }`
  - Methods: `play(track)`, `pause()`, `resume()`, `setVolume()`, `seek()`, `addToQueue()`
- **Local State**: `useState` for page-specific data
- **Custom Hooks**: `useLocalStorage`, `useMediaQuery` in `src/hooks/index.ts`

### Type System
Core types in `src/types/index.ts` and `src/types/admin.ts`:
- `Track`, `Playlist`, `User`, `PlayerState`
- `ArtistApplication`, `Report`, `AdminUser` (admin domain)
- Services return typed `ApiResponse<T>` from apiClient

## Styling Architecture

### CSS File Organization
Page-specific stylesheets in `src/styles/`:
- `spotify-theme.css` - User UI theme (CSS variables, gradients)
- `admin-theme.css` - Admin UI theme (dark colors, teal accents)
- `*-page.css` - Page-specific styles (e.g., `albums-page.css`)
- `*-layout.css` - Layout styles (e.g., `app-layout.css`, `auth-layout.css`)

**Import Pattern**: Component imports its own CSS:
```tsx
// src/components/Header.tsx
import '../styles/header.css';
```

### Design System
**User Theme** (Spotify-inspired):
- Glassmorphism: `backdrop-filter: blur()`, `background: rgba()`
- Gradients: `linear-gradient(135deg, #667eea, #764ba2)`
- Purple/blue accent colors
- Border-radius: `12px` for cards

**Admin Theme**:
- Dark backgrounds: `var(--admin-bg)` (#0f0f0f range)
- Primary accent: `var(--admin-primary)` (teal/cyan)
- Clean, minimal card-based layout

## Development Workflow

### Commands
```bash
npm run dev      # Start Vite dev server (http://localhost:5173)
npm run build    # TypeScript check + Vite build
npm run lint     # ESLint check
npm run preview  # Preview production build
```

### Backend Integration
- **Dev Mode**: Vite proxies `/api` to production backend (see `vite.config.ts`)
- **Auth Flow**: Register → OTP Verify → Login → Token stored in localStorage
- **Token Storage Keys**: `accessToken`, `refreshToken` (managed by `tokenManager`)

### Adding New Routes
1. Create `src/app/{route}/page.tsx`
2. Add route to `src/main.tsx` router config under appropriate parent (`/home` or `/admin`)
3. Add navigation link to `Sidebar.tsx` or `AdminSidebar.tsx`
4. Create matching stylesheet in `src/styles/{route}-page.css`

### Creating Services
1. Define types in `src/types/` or `src/types/admin.ts`
2. Add endpoints to `src/config/api.ts` ENDPOINTS object
3. Create service in `src/services/{domain}Service.ts` using `apiClient`
4. Export typed functions returning `Promise<T>`

## Common Pitfalls

1. **Route Navigation**: Always use `/home/` prefix for user routes (not `/`)
2. **Auth Headers**: apiClient auto-adds auth; don't manually set `Authorization`
3. **API Base URL**: Use `/api` not full backend URL (proxy handles rewriting)
4. **Import Paths**: No `@/` or `~/` aliases; use relative `../` paths
5. **Token Keys**: Must match `accessToken`/`refreshToken` (not `token`/`auth_token`)
6. **Layout Context**: User pages render inside `<Layout>`, Admin inside `<AdminLayout>`
7. **Dual Icon Sources**: Check if component uses custom icons or Lucide-React

## Key Files Reference

- **Router Config**: [src/main.tsx](../src/main.tsx#L38-L162)
- **API Client**: [src/lib/apiClient.ts](../src/lib/apiClient.ts)
- **Endpoint Config**: [src/config/api.ts](../src/config/api.ts)
- **User Layout**: [src/app/layout.tsx](../src/app/layout.tsx)
- **Admin Layout**: [src/app/admin/components/AdminLayout.tsx](../src/app/admin/components/AdminLayout.tsx)
- **Type Definitions**: [src/types/index.ts](../src/types/index.ts), [src/types/admin.ts](../src/types/admin.ts)
- **Auth Service**: [src/services/authService.ts](../src/services/authService.ts)
- **Admin Service**: [src/services/adminService.ts](../src/services/adminService.ts)
- **Proxy Config**: [vite.config.ts](../vite.config.ts#L8-L26)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hoangfff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
