# POS Client - Copilot Instructions

## Architecture Overview

This is a React-based **Point-of-Sale (POS) front-end client** built with TypeScript, Vite, Redux Toolkit, and Material-UI. The app supports multiple user roles (admin, customer, employee) with role-based access to menu management, orders, and cart operations.

### Key Architectural Layers

1. **State Management**: Redux Toolkit (RTK) with RTK Query for API communication
   - Auth state: `stores/slices/authSlice.ts` (credentials, tokens, roles)
   - API queries: `services/authApi.ts` pattern (auto-generated hooks like `useLoginMutation`)

2. **Data Flow**: API requests → Redux store → Components consume via hooks
   - All API responses wrapped in `ApiResponse<T>` type with `isSuccess`, `message`, `result` fields
   - Transformations happen in RTK Query `transformResponse` before storing

3. **Styling**: Material-UI (MUI v7) with dynamic theme switching (light/dark/system)
   - Theme config: `components/layouts/theme.ts` (colors, typography, MUI overrides)
   - Color mode context: `contexts/color-mode.tsx` (localStorage-persisted user preference)

4. **Routing**: React Router v7 in `routers/Routers.tsx`
   - Public: `/`, `/menuItem`, `/cart`, `/login`, `/register`
   - Admin: `/manage-menuItem`, `/manage-category`, `/manage-order`

### Type System Organization

- **DTOs**: `@types/dto/*` - server response types (OrderHeader, MenuCategory, etc.)
- **Create/Update DTOs**: `@types/createDto/*`, `@types/UpdateDto/*` - request payloads
- **Enums**: `@types/Enum.ts` - constants like `SD_Roles`, `SD_OrderStatus`
- **API Wrappers**: `@types/Responsts/*` - ApiResponse, LoginResponse, RegisterResponse

## Critical Workflows

### Development

```bash
npm run dev              # Start Vite dev server (HMR enabled)
npm run build           # TypeScript check + Vite production build
npm run lint            # ESLint validation
```

### Authentication Flow

1. User submits login form (Formik) → `useLoginMutation()` calls `auth/login` endpoint
2. Backend returns JWT token + user data in `LoginResponse`
3. Decode token with `jwtDecode()` and dispatch `setCredentials()` action
4. Store token in localStorage via `storage.setJson("token", token)`
5. On app init, `loadAuth()` thunk restores session if token still valid
6. `useAuth()` hook enforces redirect to "/" if token missing (protection for private routes)

### API Integration Pattern

All API endpoints follow this pattern (see `services/authApi.ts`):

```typescript
endpointName: builder.mutation<ResponseType, RequestType>({
  query: (body) => ({
    url: "endpoint/path",
    method: "POST|GET|PUT",
    body,
  }),
  transformResponse: (response: ApiResponse<ResponseType>) => {
    if (response.result) return response.result;
    throw new Error(response.message);  // API error unwrapping
  },
  invalidatesTags: ["TagName"],  // RTK Query cache invalidation
}),
```

## Project-Specific Patterns

### Form Validation
- Use **Formik + Yup** (see `helpers/validationSchema.ts`)
- Define validation schemas in helpers, reuse across components
- Example: `loginValidate` schema used in Login component

### Storage Management
- Custom helper: `helpers/storageHelper.ts` with JSON type safety
- Usage: `await storage.setJson("key", obj)` / `await storage.getJson<Type>("key")`
- Async-ready API (returns promises even though sync internally)

### Role-Based Access
- Roles defined in `@types/Enum.ts`: `admin`, `customer`, `employee` (typo: "exployee")
- Check auth state: `useAppSelector(state => state.auth.role)`
- Admin pages at `/manage-*` routes should validate role before rendering

### Component Organization
- Page-level components in `pages/` (page wrappers)
- Feature components in `components/pages/` (actual implementations)
- Layout components in `components/layouts/` (Navbar, Footer, theme)
- Admin management UIs use filter bars: `MenuFilterBar`, `CategoryFilterBar`, `OrderFilterBar`
  - These accept state (search query, category, status) + callbacks for changes
  - Responsive: stack vertically on mobile, row on desktop via `useMediaQuery`

### Menu & Order Domain Objects

**Menu Structure**:
- `MenuCategory` → `MenuItem` (has options)
- `MenuItem` has `MenuItemOptionGroup` (groups like "Size")
- `MenuItemOptionGroup` has `MenuItemOption` items
- Options decouple pricing/details via `MenuOption` + `MenuOptionDetail`

**Order Structure**:
- `CreateOrder` → contains `orderType`, `customerName`, `orderDetails[]`
- `OrderHeader` (persisted) → references `OrderDetail[]` + `Payment`
- Order statuses: `inProgress`, `ordered`, `cooking`, `ready`, `served`, `paid`, `cancelled`, `closed`

### Cart Operations
- Likely uses `CreateOrder` → `CreateOrderDetail` (menu item selection)
- `AddToCart` DTO probably includes option selections via `CreateOrderDetailOption`

## Redux & Hooks Best Practices

- **Custom hooks**: `useAppDispatch()`, `useAppSelector()` are typed wrappers (see `hooks/useAppHookState.ts`)
- Always use these instead of bare Redux hooks for type safety
- Auth hook pattern: `useAuth()` runs effect to guard routes on token absence

## Key Files to Review When Working On...

| Task | Key Files |
|------|-----------|
| Adding API endpoint | `services/authApi.ts` template; create new service if needed |
| Auth/Login feature | `stores/slices/authSlice.ts`, `services/authApi.ts`, `features/auth/loadAuth.ts` |
| New form | `helpers/validationSchema.ts` (add Yup schema), component with Formik |
| Theme/styling changes | `components/layouts/theme.ts`, `contexts/color-mode.tsx` |
| Admin page/feature | `components/pages/adminManage/*`, follow filter bar + CRUD pattern |
| Type safety | Add to `@types/` appropriate subdirectory (dto, createDto, Responsts, etc.) |

## Conventions & Code Style

- **Enums as const objects**: `export const EnumName = { ...} as const` then `type EnumName = typeof EnumName[keyof typeof EnumName]`
- **Comments in Thai**: Code includes Thai language comments/annotations (intentional for team communication)
- **No explicit errors**: Some files have `@typescript-eslint/no-explicit-any` disables—preserve when adding to existing files
- **Emotion CSS not used directly**: MUI `sx` prop and theme overrides are preferred
- **Async storage helper**: Always `await` storage operations even though implementation is sync

## Environment & Configuration

- **Base API URL**: `baseUrlAPI = "https://localhost:7061/api/"` in `helpers/SD.ts`
- **Build tool**: Vite with React SWC (faster than Babel)
- **TypeScript target**: ES2020 (browser globals available)
- **Font**: Poppins via `@fontsource/poppins` (already included in theme)

---

**Last Updated**: November 2025  
**Stack**: React 19 + TypeScript 5.9 + Vite 7 + RTK 2.11 + MUI 7.3 + React Router 7.9

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thanawat0107) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
