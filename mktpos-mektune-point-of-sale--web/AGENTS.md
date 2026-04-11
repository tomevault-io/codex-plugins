# MktPOS Web Codebase Instructions

## Architecture Overview

**MktPOS** is a SaaS POS (Point-of-Sale) system built on Next.js 16 with multi-tenant support. The codebase follows a clear separation between API integration, schema validation, and UI components.

### Core Tech Stack

- **Framework**: Next.js 16 (App Router, React 19, Server Components)
- **Language**: TypeScript (strict mode)
- **UI**: shadcn/ui (Radix UI + Tailwind CSS v4)
- **State**: Zustand for local state, TanStack React Query for server state
- **Validation**: Zod schemas for runtime validation
- **API Client**: Axios with custom validation wrapper

### Key Architectural Patterns

**API & Schema Layer** (`src/api/`, `src/schema/`)

- Schemas split into two directories:
  - `schema/models/` — Domain models (Customer, Product, Category, etc.)
  - `schema/http/` — Request/response DTOs that transform models for API
- All API calls use `validateResponse()` to parse and validate responses against Zod schemas
- API modules (`src/api/v1/tenant/`) use `v1` instance from `createHttpInstance('v1', '/tenant')`
- Validation errors parsed into field-level messages via `parseValidationError()` for form binding, if no field errors, use `parseError(error, defaultMessage)` for generic messages

**Hooks Pattern** (`src/hooks/`)

- Domain-specific hooks (e.g., `useCategories()`, `useCustomers()`) wrap React Query
- Return `{ data, refetch }` tuple pattern (not full query object)
- Complex data transformation logic lives in hooks (e.g., `useCategoryOptions()` flattens/deduplicates nested categories)

**Component Organization**

- `src/components/ui/` — Shadcn/ui primitives (button, dialog, select, etc.)
- `src/components/` — Feature-specific groups (cashier/, dialogs/, charts/, etc.)
- Dashboard sections at `src/app/dashboard/{section}/` (transactions, customers, categories, etc.)

**Multi-Tenant Context** (`src/contexts/tenant-context.tsx`)

- Detects subdomain at runtime to determine tenant
- `getSubdomainType()` classifies subdomain as 'root' | 'admin' | 'auth' | 'api' | 'tenant'
- Reserved subdomains defined in `lib/subdomain.ts`

**State Management** (`src/store/tenant/`)

- Zustand stores for auth (user, tokens) and loading state
- Auth store holds user data and plan info for access checks

## Developer Workflows

### Build & Run

```bash
npm run dev          # Start Next.js dev server (localhost:3000)
npm run build        # Production build
npm start            # Run production server
```

### Testing & Linting

```bash
npm run test:unit              # Jest (single run, coverage enabled)
npm run test:unit:watch        # Watch mode
npm run lint                   # ESLint (includes TanStack Query rules)
```

Jest configured with coverage (v8 provider, `coverage/` dir). Next.js built-in test setup via `next/jest`.

### Key Configuration Files

- `tsconfig.json` — Path alias `@/*` maps to `src/`; strict mode enabled
- `next.config.ts` — React Compiler enabled; allowed origins for subdomain dev
- `components.json` — shadcn config (baseColor: neutral, new-york style)
- `eslint.config.mjs` — Flat config with Next.js rules + TanStack Query plugin

### Code Quality Standards

**All code changes must comply with the project's ESLint configuration** (`eslint.config.mjs`). This includes:

- TypeScript strict mode enforcement (no implicit `any`)
- Next.js best practices (image optimization, Link usage, etc.)
- TanStack React Query rules (avoid raw query objects, use typed selectors)
- React Hook exhaustive dependency checks
- No unused variables or imports
- Proper error handling patterns

- **Zod usage**: prefer top-level helpers such as `z.email()`, `z.url()`, `z.uuid()`, `z.ulid()` etc. Avoid deprecated chained helpers like `z.string().email()`, `z.string().ulid()` or `z.string().url()` — use the dedicated `z` helpers for correct typing and messages.

Run the following checks before submitting changes:

```bash
npm run lint           # Validate ESLint rules (includes TanStack Query rules)
npm run test:types    # Check TypeScript compilation for type errors
```

ESLint and TypeScript errors must be resolved—do not disable rules or suppress warnings unless explicitly justified and documented in code comments. Configuration violations indicate architectural issues that need addressing, not skipping.

## Essential Patterns & Conventions

### API Call Pattern

```typescript
// src/api/v1/tenant/customer.ts
export function customers(params: AllCustomersParams = {}) {
  return validateResponse(
    AllCustomersResponseSchema,
    v1.get('/customers', {
      params: AllCustomersParamsSchema.parse(params),
    }),
  );
}

// src/hooks/customer.ts
export function useCustomers(params: AllCustomersParams = {}) {
  const query = useQuery({
    queryKey: ['customers', params],
    queryFn: () => customers(params),
  });
  return { data: query.data?.data ?? [], refetch: query.refetch };
}
```

### Form Error Handling

```typescript
// Parse validation errors from API (422 status)
const validationErrors = parseValidationError(error);
// Result: { message, fields: { fieldName: "error message" } }
// Use with react-hook-form: form.setError(field, { message })
```

### Category Trees

`useCategories()` flattens nested categories to prevent duplicates, then rebuilds tree structure. `useCategoryOptions()` further flattens for combobox with full paths. See [category.ts](../src/hooks/category.ts) for deduplication logic.

### Zustand Usage

Use `useShallow()` from `zustand/react/shallow` when selecting multiple properties to avoid unnecessary re-renders. Example in [onboarding/page.tsx](../src/app/auth/onboarding/page.tsx).

### Error Utilities

- `parseValidationError(error)` — Extracts field-level validation errors (422 responses)
- `parseError(error, defaultMessage)` — Falls back to generic error message
- Both handle Axios errors specifically; return user-friendly strings for toast/UI

### CRUD & Mutation Pattern

All CRUD operations follow consistent structure (create, update, delete):

```typescript
// API module - always use PayloadSchema.parse() to validate before send
export function create(payload: CreateCustomerPayload) {
  return validateResponse(
    CreateResponseSchema,
    v1.post('/customers', CreateCustomerPayloadSchema.parse(payload)),
  );
}

export function update(id: string, payload: UpdateCustomerPayload) {
  return validateResponse(
    UpdateResponseSchema,
    v1.put(`/customers/${id}`, UpdateCustomerPayloadSchema.parse(payload)),
  );
}

export function remove(id: string) {
  return unwrap(v1.delete<void>(`/customers/${id}`));
}

// In components: use try/catch, distinguish validation errors from generic ones
try {
  await create(data);
  toast.success('Created successfully');
  refetch();
} catch (error) {
  console.error('Form submission error:', error);

  const errors = parseValidationError(error);

  if (errors) {
    Object.entries(errors.fields).forEach(([field, message]) => {
      setError(field as keyof UseFormType, {
        type: 'manual',
        message,
      });
    });
  } else {
    toast.error(parseError(error, 'Failed to submit form.'));
  }
}
```

### Form Dialog Pattern

All create/edit dialogs wrap `FormDialog` component with mode-based logic:

```typescript
// Separate interfaces for create vs edit modes
interface CreateFormProps {
  mode: 'create';
  initialData?: undefined;
}
interface EditFormProps {
  mode: 'edit';
  itemId: string;
  initialData: Partial<UpdatePayload>;
}

// Switch schema based on mode
const schema = mode === 'create' ? CreatePayloadSchema : UpdatePayloadSchema;

// Reset on close (create) or sync on load (edit)
useEffect(() => {
  if (initialData) {
    if (mode === 'create') reset(initialData);
    else reset(initialData);
  }
}, [initialData, reset, mode]);

// In onSubmit: use type narrowing with `as`
if (mode === 'create') {
  const response = await create(data as CreatePayload);
} else if (mode === 'edit') {
  const response = await update(itemId, data as UpdatePayload);
}
```

### Cashier/Transaction Pattern

Cashier module is special — local state management (cart array) + multi-step dialogs:

- Cart stored in component state: `useState<CartItem[]>([])`
- Calculate totals from cart: `subtotal = cart.reduce((sum, item) => sum + item.subtotal, 0)`
- Multi-dialog flow: Customer Select → Checkout (payment method) → Success Receipt
- Payment methods enum: `'cash' | 'debit_card' | 'credit_card' | 'e_wallet' | 'bank_transfer'`
- See [cashier/page.tsx](../src/app/dashboard/cashier/page.tsx) for full pattern

### Authentication & Token Management

Auth store (`useAuthStore`) holds: `access` (token + expiry), `user` (Tenant), `otp` (verification)

Token refresh handled automatically via `useRefreshToken()` hook:

- Decodes JWT expiry using `jwtDecode()`
- Schedules refresh 5 min before expiry (configurable)
- Redirects to login on 401/403 response
- Single `performRefresh()` async function orchestrates full cycle

```typescript
// Login flow
const response = await login({ email, password, device });
setAccess(response.access);
setUser(response.user);

// Logout
await logout(); // Clears store + redirects to auth.domain.com/login
```

### Subdomain & URL Routing

URL helper class (`lib/classes/url.ts`) provides subdomain manipulation:

- `url.subdomain = 'tenant-slug'` — Sets subdomain safely (validates format)
- `url.baseDomain` — Returns `domain.tld` without subdomain
- `getSubdomainType(subdomain)` — Returns 'root'|'admin'|'auth'|'api'|'tenant'
- `buildTenantUrl(subdomain, path)` — Quick helper to build tenant URLs
- `createRootUrl(path)` / `createSubUrl(subdomain, path)` — Environment-aware URL builders

Multi-tenant routing in [proxy.ts](../src/proxy.ts):

- Root domain (no subdomain): marketing pages + redirects to auth/admin
- Auth subdomain: login, register, onboarding
- Admin subdomain: global admin panel
- Tenant subdomain: dashboard, cashier, reports

Layouts enforce subdomain checks via `useTenant()` + `subdomainType` to redirect or gate access.

### Enums & Constants

Located in `src/lib/enums/` and used throughout:

- `PaymentStatus` — 'pending' | 'completed' | 'failed'
- `TransactionStatus` — 'completed' | 'voided' | 'refunded' | 'partially_refunded'
- `DeviceType` — 'mobile' | 'tablet' | 'desktop'
- Payment methods are typed as union literals in cashier module

## Critical Cross-File Patterns

**Response Structure**: All API responses follow `{ data: T }` wrapper. Hooks extract `query.data?.data ?? []` to avoid null checks in components.

**Query Keys**: Use domain names as first level (`['customers']`, `['products']`), params as second. Enables targeted invalidation in mutations.

**Type Safety**: Schemas are Zod types; infer TypeScript types with `z.infer<typeof Schema>` rather than hand-written interfaces.

**Subdomain Routing**: Use `buildTenantUrl()` for cross-tenant navigation; check `useTenant()` context in layouts before rendering protected sections.

**Payment & Billing**: Payment polling every 3s via `useCallback` + `setInterval` in dialog. Statuses check against `PaymentStatus` enum. Invoices generated via jsPDF utility.

**Device Tracking**: Device UUID stored in cookies via `getOrCreateDeviceUUID()`. Device info collected on login (type, name). Used for multi-device session management.

## Codebase Quick Reference

### File Structure by Feature

- **API**: `src/api/v1/tenant/` — API functions (customers.ts, product.ts, category.ts, auth.ts, payment.ts, device.ts, tenant.ts, transaction.ts, store.ts)
- **Hooks**: `src/hooks/` — Domain hooks + `useRefreshToken()`, `useTenant()`, auth helpers
- **Schemas**: `src/schema/models/` + `src/schema/http/` — Zod type definitions
- **Dialogs**: `src/components/dialogs/` — Form dialogs for CRUD, success dialogs, payment dialogs
- **Cashier**: `src/components/cashier/` — Cart, checkout, product grid, customer card, transactions
- **Dashboard**: `src/app/dashboard/` — Main dashboard, cashier, products, categories, customers, employees, inventory, reports, transactions, settings

### Important Class/Utility Files

- `lib/classes/url.ts` — URL parsing with subdomain support, test coverage in `tests/classes/url.test.ts`
- `lib/classes/error.ts` — Custom error class for exceptions
- `lib/http.ts` — Axios instance creation, error parsing (`parseValidationError`, `parseError`)
- `lib/schema.ts` — `validateResponse()` wrapper around Zod
- `lib/subdomain.ts` — Subdomain validation, type detection, reserved subdomains list
- `lib/device.ts` — Device detection utilities
- `lib/billing.ts` — Invoice PDF generation via jsPDF
- `lib/cookies.ts` — Device UUID management
- `proxy.ts` — Middleware for subdomain-based routing (Next.js middleware)

### Store Architecture

```typescript
// Auth store (persisted to localStorage)
useAuthStore(state) → {
  access: Access | null,        // { token, type, expiry }
  user: Tenant | null,           // User + Store + Plan data
  otp: Otp | null,               // OTP state for email verification
  setAccess, setUser, setOtp
}

// Loading store (transient)
useLoadingStore(state) → { loading: boolean, setLoading }
```

## Common Pitfalls & Solutions

1. **Nested Categories Duplication**: Always use `useCategories()` which deduplicates + rebuilds tree. Don't call API directly in UI.

2. **Form Reset on Close**: Modal dialogs should reset on close for create mode; use conditional reset in `useEffect` on `initialData` changes.

3. **Validation Field Mapping**: When setting form errors from API, loop `Object.entries(errors.fields)` to set each field individually.

4. **Subdomain Navigation**: Never hardcode domain names; use `URL` class or `buildTenantUrl()`. Local dev requires `subdomain.localhost` format.

5. **Auth Redirect Loops**: Ensure `useTenant()` checks happen in layout. Redirect logic must account for `isLoading` state to avoid flashing.

6. **Pagination Keys**: Query keys must include params: `['customers', { search, page }]`. Without it, pagination changes won't trigger new queries.

7. **Payment Polling**: Dialog should stop polling on unmount. Use cleanup function in `useEffect` to clear interval.

## References

- [HTTP client & error parsing](../src/lib/http.ts) — `parseValidationError`, `parseError`, `createHttpInstance`
- [Schema validation](../src/lib/schema.ts) — `validateResponse` wrapper
- [Tenant context](../src/contexts/tenant-context.tsx) — Multi-tenant detection & subdomain handling
- [Category hook](../src/hooks/category.ts) — Complex data transformation example
- [Auth API & token refresh](../src/hooks/auth.ts) — `useRefreshToken()` full implementation
- [Cashier page](../src/app/dashboard/cashier/page.tsx) — Complete transaction + checkout flow
- [Form dialog example](../src/components/dialogs/customer-form-dialog.tsx) — Create/edit pattern
- [URL helper](../src/lib/classes/url.ts) — Subdomain + URL utilities
- [Proxy middleware](../src/proxy.ts) — Subdomain routing rules

## Design Context (IMPECCABLE)

This repository includes a design-context file generated by the `teach-impeccable` workflow. The canonical draft is stored at `.github/IMPECCABLE.md`. A short repository memory summary is saved at `/memories/repo/impeccable-design-context.md` for tooling and future agents to reference.

When updating visual style, tokens, or high-level UX principles, prefer editing `.github/IMPECCABLE.md` so the design intent remains visible to all contributors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MKTPOS-Mektune-Point-of-Sale)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MKTPOS-Mektune-Point-of-Sale)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
