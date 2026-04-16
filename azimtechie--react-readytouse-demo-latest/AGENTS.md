# Copilot Instructions for AKKGlobal

## Project Overview

React 19 + TypeScript + Vite application using Ant Design v5 and Tailwind CSS v4. Email management system with authentication, protected routes, and dynamic theming via CSS variables.

## Architecture & Key Patterns

### Theme System (CSS Variables → Ant Design)

- **Never hardcode colors** - all colors are CSS variables in `src/index.css` under `@theme` directive
- Theme dynamically reads CSS vars at runtime via `getCssVariable()` in `src/theme.ts`
- Ant Design theme tokens map to CSS variables: `--color-primary`, `--color-sidebar-bg`, etc.
- To add new colors: (1) define in `index.css`, (2) export in `theme.ts` color object, (3) map to Ant Design tokens

### Path Aliases & Import Conventions

- `@/` resolves to `src/` (configured in `vite.config.ts`)
- Always use `@/` imports: `import { useAuth } from '@/contexts/auth'`
- **Use clean folder imports** - leverage `index.ts` barrel files for organized exports
  - ✅ `import { FormInput, FormSelect } from '@/components/common'`
  - ✅ `import { useAuth } from '@/contexts/auth'`
  - ❌ `import FormInput from '@/components/common/FormInput'`
  - ❌ `import { useAuth } from '@/contexts/auth/useAuth'`

### Routing & Code Splitting

- Routes defined in `src/routes.tsx` using React Router v7's `createBrowserRouter`
- **All pages are lazy-loaded** using `lazy()` + `withSuspense()` HOC (see `src/utils/withSuspense.tsx`)
- Pattern: `const Page = withSuspense(lazy(() => import('./pages/...')));`
- Protected routes wrap children with `<ProtectedRoute>` component
- Nested routes use `<Outlet />` from React Router

### Authentication Flow

- Context: `src/contexts/auth/AuthContext.tsx` + `AuthProvider.tsx`
- Auth state stored in `localStorage` keys: `AkkGlobal_user`, `AkkGlobal_token`
- Mock login: `azimtechie@gmail.com` / `admin123` (see `AuthProvider.tsx`)
- `useAuth()` hook provides: `{ user, isAuthenticated, isLoading, login, logout, updateUser }`
- `ProtectedRoute` redirects to `/login` if unauthenticated, shows loading spinner during auth check

### Form Components Pattern

- Reusable form components in `src/components/common/`
- All form components use Ant Design's `Form.Item` wrapper
- Auto-generate placeholders: `generatePlaceholder(label, 'Enter' | 'Select')` from `utils/common.ts`
- Standard props: `label`, `name`, `rules`, `placeholder`, `disabled`, `size` (default: `'middle'`)
- Form wrapper: `FormWrapper` provides consistent layout with auto-generated "Add/Edit" title, back button, and save/cancel actions

Example:

```tsx
<FormWrapper title="Project" onSubmit={handleSubmit} isEditMode={!!id}>
  <FormInput label="Name" name="name" rules={[validationRules.required('Name is required')]} />
  <FormSelect label="Status" name="status" options={statusOptions} />
</FormWrapper>
```

### Sidebar Menu Pattern

- Complex implementation in `src/components/layout/AppSidebar.tsx`
- Uses `MenuItemWithSuffix` for items with action buttons (e.g., plus icon to add new)
- Collapsed state shows custom `CollapsedMenuItem` with popup labels
- Constants outside component prevent re-renders: `MENU_DATA`, `ROOT_SUBMENU_KEYS`
- Menu navigation via `useNavigate()`, selected keys from `useLocation().pathname`
- Submenu auto-opens based on current path in `useEffect`

### Layout Structure

- `AppLayout` uses Ant Design's `Layout` with fixed sidebar, header, and scrollable content
- Outlet renders child routes inside `<Content>` with padding and background

### File Organization

- **Group exports via `index.ts` barrel files** (see `src/components/common/index.ts`) - this enables clean folder-level imports
- Always create `index.ts` when adding multiple files to a folder for cleaner imports
- Separate interfaces/types in context files (e.g., `AuthContext.tsx` has types, `AuthProvider.tsx` has implementation)
- Hooks in `src/hooks/`, utilities in `src/utils/`, constants in `src/constants/`
- Export types when shared: `export type { SelectOption } from './FormSelect'`

### File Organization

- Group exports via `index.ts` barrel files (see `src/components/common/index.ts`)
- Separate interfaces/types in context files (e.g., `AuthContext.tsx` has types, `AuthProvider.tsx` has implementation)
- Hooks in `src/hooks/`, utilities in `src/utils/`, constants in `src/constants/`

### Styling

- Tailwind classes for layout/spacing: `flex`, `items-center`, `gap-4`, `px-6`, `py-4`
- Ant Design inline styles for component-specific theming (using `theme.useToken()`)
- Never use inline hex colors - reference CSS variables or theme tokens
- Example: `style={{ background: colorBgContainer, borderRadius: borderRadiusLG }}`

### TypeScript

- Interfaces over types for component props
- Use `React.FC<PropsInterface>` for functional components
- Export types/interfaces when shared: `export type { SelectOption } from './FormSelect'`
- Prefer `unknown` over `any` in utility functions

## Development Workflow

### Commands

- **Dev server**: `npm run dev` (Vite HMR on port 5173)
- **Build**: `npm run build` (TypeScript check + Vite build)
- **Lint**: `npm run lint` (ESLint with flat config)
- **Preview**: `npm run preview` (preview production build)

### React Compiler

- Enabled via `babel-plugin-react-compiler` in `vite.config.ts`
- Automatically optimizes components - avoid manual `useMemo`/`useCallback` unless needed for deps

### Environment Variables

- API URL: `import.meta.env.VITE_API_BASE_URL` (default: `http://localhost:3001/api`)
- Access via `import.meta.env` (Vite convention)

## Common Utilities

### From `utils/common.ts`:

- `generatePlaceholder(label, 'Enter' | 'Select')` - auto-generate form placeholders
- `validationRules` - reusable Ant Design validation rules (`.required()`, `.email()`, `.minLength()`, etc.)
- `cn(...classes)` - combine classNames, filters falsy values
- `formatDate()`, `formatCurrency()` - formatting helpers
- `debounce()`, `throttle()` - performance optimizers

### From `utils/css.ts`:

- `getCssVariable('--var-name')` - read CSS variables at runtime
- `setCssVariable()`, `removeCssVariable()` - dynamic theme manipulation

## Adding New Features

### New Protected Route:

1. Create lazy-loaded component: `const NewPage = withSuspense(lazy(() => import('./pages/NewPage')))`
2. Add to routes in `routes.tsx` under `ProtectedRoute` children
3. Add menu item in `AppSidebar.tsx` MENU_DATA or items array
4. Use `usePageTitle('Page Title')` hook in component

### New Form:

1. Use `FormWrapper` component with `title`, `onSubmit`, `isEditMode`
2. Compose form fields using `FormInput`, `FormSelect`, `FormTextArea`, etc.
3. Apply `validationRules` from `utils/common.ts`
4. Handle submission with loading state passed to `FormWrapper`

### New Theme Color:

1. Define CSS variable in `src/index.css` under `@theme` block
2. Export in `src/theme.ts` `color` object via `getCssVariable()`
3. Map to Ant Design token in `theme.token` or `theme.components`

## Code Quality

- ESLint flat config with TypeScript, React Hooks, and React Refresh rules
- React 19 features enabled (React Compiler, new hooks)
- Strict TypeScript with project references (`tsconfig.json`, `tsconfig.app.json`, `tsconfig.node.json`)

## Anti-Patterns to Avoid

- ❌ Hardcoding colors instead of CSS variables
- ❌ Importing pages without lazy loading
- ❌ Creating protected routes without `ProtectedRoute` wrapper
- ❌ Bypassing `FormWrapper` for consistency
- ❌ Using relative imports when `@/` alias is available
- ❌ Importing directly from file paths instead of using folder-level `index.ts` exports
- ❌ Forgetting `size="middle"` on form components (project standard)
- ❌ Using relative imports when `@/` alias is available
- ❌ Forgetting `size="middle"` on form components (project standard)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/azimtechie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
