# PSPG Requisitions - AI Coding Agent Instructions

## Project Overview

**PSPG Requisitions** is a Next.js 14 (App Router) internal platform for managing personnel requisitions for PSP Group. It features role-based access control (RBAC), dynamic form templates, session auditing, and a public-facing jobs portal.

**Tech Stack**: Next.js 14, TypeScript, Tailwind CSS, Supabase (Auth + PostgreSQL + Storage)

## Architecture & Key Concepts

### Role-Based Access Control (RBAC)
- **Four base roles**: `superadmin`, `admin`, `partner`, `candidate`
- Role definitions stored in `roles` table with `permissions` JSONB field
- User roles linked via `profiles.role_id` → `roles.id`
- **Menu filtering**: `app/components/navigation/menuConfig.ts` defines menu structure with role restrictions
- **Optional module permissions**: If `permissions.modules` exists (e.g., `{"dashboard": true}`), sidebar shows only enabled modules
- **Frontend protection**: Use `<RequireRoleClient allow={["admin"]}>` from `app/components/RequireRole.tsx`
- **Backend validation**: ALWAYS validate roles in API routes AND enforce with Supabase RLS policies

### Authentication & Session Management
- **Global auth context**: `app/providers/AuthProvider.tsx` wraps entire app
- **User data loading**: `lib/getFullUserData.ts` combines `auth.getUser()` + `profiles` + `roles` with retry logic
- **Session tracking**: `lib/sessionTracking.ts` captures geolocation, device info, and maintains session history in `profiles.metadata.sessions`
- **Inactivity timeout**: 15 minutes configured in `AuthProvider`
- **Access pattern**: Always use `useAuth()` hook for user/profile data in client components

### Supabase Integration
- **Client setup**: `lib/supabaseClient.ts` exports anon-key client with PKCE flow
- **Service role**: API routes use `SUPABASE_SERVICE_ROLE_KEY` for privileged operations (never expose to client)
- **RLS functions**: Database has `get_user_role(uuid)` and `current_user_role()` for policy evaluation
- **Storage**: `avatars` bucket for user profile images (`user-{userId}/avatar.webp`)

### Dynamic Form System
- **Templates**: Admin-configurable forms stored in `form_templates` → `form_sections` → `form_fields`
- **Active templates**: Each company has ONE active template retrieved via RPC `get_company_active_template(company_id)`
- **Requisition snapshots**: When creating a requisition, template is snapshot into `template_snapshot` JSONB field (preserves form structure even if template changes)
- **Service layer**: `lib/templates.ts` for CRUD operations on templates, `lib/requisitions.ts` for requisition management
- **Field types**: See `lib/types/requisitions.ts` - supports text, number, currency, select, multiselect, date, etc.

### Public vs Authenticated Areas
- **Public routes**: `/` (landing), `/about`, `/jobs`, `/contact` - uses `app/components/public/*` components
- **Authenticated routes**: `/dashboard`, `/admin/*`, `/requisitions`, `/profile` - wrapped in layouts with sidebar
- **Route protection**: Handled in `AuthProvider` - redirects unauthenticated users to `/auth`

## Development Workflows

### Running Locally
```bash
npm install
npm run dev  # Starts on http://localhost:3000
```

### Environment Variables (`.env.local`)
```bash
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...  # Server-only, NEVER expose to client
ADMIN_SECRET=your-secret-here      # For /api/admin endpoints
```

### Database Setup
- Run SQL scripts in `scripts/` for initial setup (avatars storage policies)
- Core schema documented in `docs/README-SUPABASE.md`
- Use Supabase SQL Editor or migration tools for schema changes

### Debugging Auth Issues
1. Check browser console for `getFullUserData` timing logs (dev mode)
2. Verify session exists: `await supabase.auth.getSession()`
3. Check RLS policies if queries return empty despite correct auth
4. Use `lib/getCurrentUserRole.ts` to verify role assignment

## Critical Code Patterns

### Client vs Server Components
- **Add `'use client'` directive** for components using: hooks (`useState`, `useEffect`, `useRouter`), browser APIs, event handlers
- **Server components** (default): Can directly query Supabase, but cannot use client hooks
- **Pattern**: Keep data fetching in server components/API routes, UI interactions in client components

### API Route Security
```typescript
// app/api/example/route.ts
import { createClient } from '@supabase/supabase-js'

const adminClient = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!  // Service role for admin operations
)

export async function POST(req: Request) {
  // Verify admin secret OR validate user token
  const token = req.headers.get('authorization')?.replace('Bearer ', '')
  const { data: { user } } = await adminClient.auth.getUser(token)
  
  // Fetch user role and validate permissions
  const { data: profile } = await adminClient
    .from('profiles')
    .select('*, roles(*)')
    .eq('id', user.id)
    .single()
  
  if (!['admin', 'superadmin'].includes(profile.roles.name)) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
  }
  
  // Proceed with operation...
}
```

### Toast Notifications
```tsx
'use client'
import { useToast } from '@/lib/useToast'

function MyComponent() {
  const { success, error, warning, info } = useToast()
  
  const handleAction = async () => {
    try {
      await doSomething()
      success('¡Operación exitosa!')
    } catch (err) {
      error('Error al procesar la solicitud')
    }
  }
}
```

### Role-Based UI Rendering
```tsx
import { RequireRoleClient } from '@/app/components/RequireRole'

<RequireRoleClient allow={['admin', 'superadmin']}>
  <AdminOnlyFeature />
</RequireRoleClient>
```

### Fetching Active Template
```typescript
import { getCompanyActiveTemplate } from '@/lib/templates'

const template = await getCompanyActiveTemplate(companyId)
if (!template) {
  throw new Error('No active template found for this company')
}
// template.sections[] contains form structure
```

## Custom Tailwind Classes
- **Admin colors**: Use `admin-primary`, `admin-accent`, `admin-success`, `admin-danger` for consistent styling
- **Admin spacing**: `admin-xs` through `admin-2xl` for standardized gaps
- **Admin borders**: `rounded-admin`, `border-admin-border`
- See `tailwind.config.js` for full palette

## Common Gotchas

1. **RLS blocking queries**: If authenticated queries return empty, check RLS policies. Service role bypasses RLS.
2. **Client-side Supabase client**: Always use `lib/supabaseClient.ts` export, never create new clients
3. **Type imports**: Use `import type { ... }` for type-only imports to avoid runtime bundling
4. **Path aliases**: Use `@/` prefix (configured in `tsconfig.json`) - e.g., `@/lib/supabaseClient`
5. **Session expiry**: `getFullUserData` handles expired sessions gracefully - don't log 403 errors as unexpected
6. **Phone input**: Use `PhoneInput` component from `app/components/PhoneInput.tsx` with country code support
7. **Avatar uploads**: Use `AvatarUpload` component - handles cropping, compression, and storage automatically

## Testing & Validation

- **No formal test suite yet** - manually test features in dev environment
- **Check errors**: Use VS Code Problems panel (`get_errors` tool) after edits
- **Validate RLS**: Test with different user roles to ensure proper access control
- **API testing**: Use tools like Postman/curl with actual auth tokens

## Documentation Reference

- **Frontend**: `docs/README-FRONTEND.md` - UI components, navigation, toasts, avatars
- **Backend**: `docs/README-BACKEND.md` - API routes, security patterns
- **Supabase**: `docs/README-SUPABASE.md` - Database schema, RLS policies, storage
- **Structure**: `docs/README-STRUCTURE.md` - Public portal architecture
- **Main README**: `README.md` - Project overview and getting started

## Key Files to Reference

- `app/layout.tsx` - Root layout with AuthProvider and ToastContainer
- `app/providers/AuthProvider.tsx` - Global auth state and inactivity handling
- `app/components/navigation/menuConfig.ts` - Menu structure and role filtering
- `lib/getFullUserData.ts` - User data loading with retry logic
- `lib/supabaseClient.ts` - Supabase client configuration
- `lib/templates.ts` - Template CRUD operations
- `lib/requisitions.ts` - Requisition management
- `app/api/admin/secure/route.ts` - Example of secure API route with role validation

---

**When in doubt**: Check existing implementations in similar components/routes, reference the docs, and ensure both frontend UI restrictions AND backend/RLS validation are in place.

**Keep in mind**: 

- Security is paramount. Always validate user roles and permissions on the server side, regardless of frontend checks.
- Use environment variables for sensitive information (e.g., API keys, database URLs) and never hard-code them.
- Prevent the creation of innecesaries ".md" files, these documents should be generated upon request from the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mroboticsla)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mroboticsla)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
