# ShopTally AI Agent Instructions

## Project Overview
ShopTally is an **offline-first Expo React Native shop management app** with a Node.js backend for AI features. Multi-tenant architecture with role-based access control (owner vs staff).

**Tech Stack**: TypeScript, Expo Router, PowerSync (offline sync), Supabase (PostgreSQL), Drizzle ORM, React Query, Anthropic AI SDK.

---

## Architecture & Data Flow

### Offline-First Sync with PowerSync
- **Primary DB**: PowerSync (local SQLite) syncs with Supabase when online
- **Fallback**: AsyncStorage for Expo Go / web development
- **Key file**: `lib/powersync/system.ts` — handles native availability checks
- **When to use**: Always prefer PowerSync queries when available, but fallback to AsyncStorage gracefully

### Data Entities & Schema
Core entities in `lib/types.ts` and `lib/powersync/schema.ts`:
- **Product**: name, price, costPrice, stock, lowStockThreshold, barcode, imageUri, category, isMarketplace
- **Sale**: items (SaleItem[]), total, amountPaid, paymentMethod ('cash'|'transfer'|'split'|'gateway'), isCredit
- **Order**: items, total, customerName, status ('new'|'accepted'|'preparing'|'ready'|'delivered')
- **Shop**: multi-tenant container with virtual accounts (Moniepoint, OPay, Palmpay)
- **User**: owner or staff with role-based permissions

### Context Providers (Global State)
1. **AuthContext** (`lib/auth-context.tsx`): Session, user profile, permissions, onboarding status
   - Key hook: `useAuth()` → `{ session, user, permissions, signIn, signUp, signOut, ... }`
2. **ShopContext** (`lib/shop-context.tsx`): Products, sales, orders, staff, shop profile, cart
   - Key hook: `useShop()` → `{ products, sales, orders, cart, addProduct, completeSale, ... }`
3. **ToastContext** (`lib/toast-context.tsx`): Toast notifications
   - Key hook: `useToast()` → `{ success, error, warning }`

---

## Critical Development Patterns

### 1. Database → App Type Conversion
PowerSync rows are untyped. **Always use row converter functions**:
```typescript
function rowToProduct(row: Record<string, unknown>): Product {
  return {
    id: row.id as string,
    price: row.price as number,
    stock: row.stock as number,
    lowStockThreshold: (row.low_stock_threshold as number) ?? 5,
    isMarketplace: !!(row.is_marketplace as number), // SQLite stores booleans as 0/1
    // ... other fields with type casts and nullish coalescing
  };
}
```
See [lib/shop-context.tsx](lib/shop-context.tsx#L27-L48) for examples. Always cast and provide defaults.

### 2. Theming & Colors
**Never hardcode colors**. Always use:
```typescript
import { useThemeColors } from '@/constants/colors';

const colors = useThemeColors(colorScheme); // colorScheme from useColorScheme()
// Access: colors.primary, colors.surface, colors.text, colors.danger, etc.
```
See [constants/colors.ts](constants/colors.ts) for full palette (light/dark modes).

### 3. Permission Checks
Owner vs staff access:
```typescript
const isOwner = user?.role === 'owner';
if (!isOwner) return <RestrictedView />;

// For finer control, check `permissions` object
const canDelete = permissions?.canDeleteProducts || isOwner;
```

### 4. Storage & Persistence
Use the abstraction layer in `lib/storage.ts`:
```typescript
import { saveProducts, loadProducts } from '@/lib/storage';
// These auto-detect PowerSync vs AsyncStorage
```

### 5. Row/Column Naming Convention
- **Database columns**: snake_case (`created_at`, `low_stock_threshold`, `is_marketplace`)
- **TypeScript fields**: camelCase (`createdAt`, `lowStockThreshold`, `isMarketplace`)
- Conversion happens in row converter functions

---

## File Organization

```
app/                          # UI screens (Expo Router file-based routing)
├── (tabs)/                   # Bottom tab-navigated screens
│   ├── index.tsx            # Dashboard/home
│   ├── products.tsx         # Product management (see attachment)
│   ├── sales.tsx            # Sales history
│   ├── orders.tsx           # Order management
│   └── reports.tsx          # Analytics
├── auth/                     # Auth flow (before login)
│   ├── login.tsx
│   ├── onboarding.tsx       # Shop setup
│   └── _layout.tsx          # Auth stack layout
├── add-product.tsx          # Create product
├── edit-product.tsx         # Edit product (params: { productId })
├── new-sale.tsx             # Checkout flow
└── _layout.tsx              # Root layout, providers

lib/                          # Core logic & context
├── auth-context.tsx         # Authentication & user state
├── shop-context.tsx         # Products, sales, orders, cart
├── storage.ts               # AsyncStorage/PowerSync abstraction
├── supabase.ts              # Supabase client
├── types.ts                 # TypeScript interfaces
├── powersync/               # Offline sync engine
│   ├── system.ts            # DB initialization & checks
│   ├── schema.ts            # PowerSync schema definition
│   ├── connector.ts         # Supabase sync connector
│   └── migration.ts         # AsyncStorage → PowerSync migration
└── [other: toast, format, i18n, etc]

components/                   # Reusable UI components
├── AIStockModal.tsx         # AI stock advisor modal
├── BarcodeScanner.tsx       # Camera-based barcode scanning
├── LoadingSpinner.tsx
└── [others]

server/                       # Node.js backend (AI endpoints)
├── index.ts                 # Express app setup
├── routes.ts                # API route definitions
├── ai.ts                    # Anthropic AI logic
└── templates/               # Email/landing page HTML

constants/
├── colors.ts                # Theme color definitions
```

---

## Common Tasks & Patterns

### Adding a New Feature Screen
1. Create file in `app/feature-name.tsx` or `app/(tabs)/feature-name.tsx`
2. Use `useShop()`, `useAuth()`, `useToast()` for state
3. Apply theme with `useThemeColors(colorScheme)`
4. Use Expo Router for navigation: `router.push('/path')`
5. Add haptics feedback for interactions: `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)`

Example (see [app/(tabs)/products.tsx](app/(tabs)/products.tsx)):
```typescript
export default function FeatureScreen() {
  const colorScheme = useColorScheme();
  const colors = useThemeColors(colorScheme);
  const { products } = useShop();
  const { user } = useAuth();
  const toast = useToast();
  
  if (!user?.role === 'owner') {
    return <Text>Unauthorized</Text>;
  }
  
  return <View style={{ backgroundColor: colors.background }}>...</View>;
}
```

### Updating a Product (or any entity)
1. Call `shop.updateProduct(id, updates)` from ShopContext
2. This syncs via PowerSync to Supabase
3. Local state updates immediately (optimistic), then syncs
4. Show toast feedback: `toast.success('Product updated')`

### Payment Processing
- Support multiple payment methods: `'cash' | 'transfer' | 'split' | 'gateway'`
- Payment gateways: `'moniepoint' | 'opay' | 'palmpay'`
- Virtual accounts are stored in shop profile with balance tracking
- See `lib/payment-listener.ts` for webhook handling

### AI Features
- Backend routes in `server/routes.ts` and `server/ai.ts`
- Call from frontend via `fetch('/api/ai/insights', { method: 'POST', body: JSON.stringify(...) })`
- Used by: `AIStockModal`, `AIInsightsCard`, product description generation
- API endpoints: `/api/ai/insights`, `/api/ai/stock-advisor`, `/api/ai/chat`, `/api/ai/product-description`

---

## Build & Development Commands

```bash
npm run expo:dev              # Start Expo dev server (hot reload)
npm run server:dev            # Start Node.js backend (watches TSX)
npm run db:push              # Push Drizzle migrations to Supabase
npm run lint                 # Run ESLint
npm run lint:fix             # Auto-fix linting issues
```

For **local database development**, use PowerSync via Expo dev client (native build), or AsyncStorage fallback in web/Expo Go.

---

## Redux Concerns & Gotchas

1. **JSON in SQLite**: JSON objects (items, marketplace_listing, opening_hours) stored as TEXT. Always `JSON.parse()` when reading, `JSON.stringify()` when writing.

2. **Boolean as Integer**: SQLite booleans stored as 0/1. Use `!!(value as number)` to convert.

3. **Nullable Fields**: Use nullish coalescing `??` with safe defaults in row converters.

4. **Type Safety**: All types defined in `lib/types.ts`. Never use `any`; extend types if needed.

5. **Permissions**: Check `user?.role` for coarse access. Check `permissions` object for fine-grained staff capabilities.

6. **Sync Conflicts**: PowerSync handles conflict resolution. App treats local changes as source of truth until sync completes.

7. **Barcode Scanning**: Requires camera permission. `BarcodeScanner` component shows status and errors.

8. **Images**: Use `expo-image` component, provide `contentFit="cover"` and fallback placeholder when `imageUri` is null.

---

## When to Ask for Clarification

- Should this be owner-only or staff-viewable?
- What payment method combinations are allowed?
- Should this field sync across devices or stay local?
- Is this a shop-level or global setting?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joepiuls)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/joepiuls)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
