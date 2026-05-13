# Sobek Pharma - AI Coding Agent Instructions

## Project Overview
Next.js 15 pharmaceutical company website with SQLite/Prisma backend, managing human and veterinary pharmaceutical products. Migrated from Supabase to local SQLite database.

## Architecture

### Tech Stack
- **Frontend**: Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS 4
- **Database**: SQLite with Prisma ORM
- **Animations**: Framer Motion
- **Forms**: React Hook Form + Yup validation
- **UI Feedback**: react-hot-toast
- **Security**: Custom headers in `next.config.ts`, file upload validation

### Key Directory Structure
```
web/
├── prisma/
│   ├── schema.prisma          # Database schema (3 models: HumanProduct, VeterinaryProduct, Category)
│   ├── dev.db                 # SQLite database file
│   └── seed.js                # Seed data (11 categories, sample products)
├── src/
│   ├── app/
│   │   ├── admin/             # Admin panel (products, categories, login)
│   │   ├── api/               # API routes (products, categories, upload)
│   │   └── products/          # Public product pages by category
│   ├── components/            # Header, Footer, Forms
│   └── lib/
│       ├── prisma.ts          # Prisma client + API helpers (humanProductsAPI, veterinaryProductsAPI, categoriesAPI)
│       ├── api.ts             # Client-side fetch helpers
│       └── uploadHelpers.ts   # Image validation & upload utilities
└── public/images/products/    # Uploaded product images
```

## Critical Patterns

### Database Access
- **Never import `@prisma/client` directly in client components**
- Use API routes (`/api/products/human`, `/api/products/veterinary`, `/api/categories`)
- API helpers in `lib/prisma.ts` handle field mapping (camelCase ↔ snake_case)
- Example: Prisma `genericName` maps to API `generic_name`

### Field Name Mapping
Prisma uses camelCase, APIs expose snake_case:
- `genericName` ↔ `generic_name`
- `dosageForm` ↔ `dosage_form`
- `imageUrl` ↔ `image_url`
- `isActive` ↔ `is_active`

Always handle both formats when creating/updating products.

### Product Management Flow
1. Admin adds/edits products via `/admin/products/{human|veterinary}`
2. Forms submit to `/api/products/{human|veterinary}` routes
3. API routes call `lib/prisma.ts` helpers (`humanProductsAPI.create/update/delete`)
4. Public pages fetch via client-side `lib/api.ts` helpers
5. Toast notifications confirm success/errors

### Image Uploads
- Upload to `/api/upload` (POST multipart/form-data)
- Validates: JPG/PNG/WebP, max 5MB
- Saves to `public/images/products/product_{timestamp}.{ext}`
- Returns public URL: `/images/products/{filename}`
- Use `uploadProductImage()` from `lib/uploadHelpers.ts`

### Authentication
- **Temporary localStorage-based auth** (admin@sobekpharma.com / admin123)
- Check `localStorage.getItem('adminLoggedIn')` in admin pages
- **Production**: Plan to implement proper auth (Supabase Auth was removed)

## Development Commands

```powershell
# Working directory
cd C:\Users\MEKAW\Desktop\sopek\web

# Development
npm run dev              # Runs on http://localhost:3002 with --turbopack

# Database
npx prisma studio        # GUI at http://localhost:5555
node prisma/seed.js      # Reset & seed database
npx prisma migrate dev   # Create new migration
npx prisma generate      # Update Prisma client after schema changes

# Build
npm run build --turbopack
```

## Important Conventions

### Component Structure
- Use `"use client"` for interactive components (forms, animations, state)
- Framer Motion for animations: `motion.div` with `initial/animate/transition`
- Forms: React Hook Form + inline validation
- Error handling: `react-hot-toast` for user feedback

### API Routes Pattern
```typescript
// GET with optional query params
const { searchParams } = new URL(request.url)
const category = searchParams.get('category')

// Error responses
return NextResponse.json({ error: 'message' }, { status: 500 })
```

### Category System
- Categories have `type: 'human' | 'veterinary'`
- Human categories: cardiovascular, anti-infectives, endocrinology-diabetes, gastroenterology
- Veterinary categories: companion-animals, livestock, poultry, equine, aquaculture
- URL slugs match category slugs: `/products/human-new/{slug}`

### Styling
- Tailwind CSS 4 with RTL support (`tailwindcss-rtl`)
- Red accent color: `bg-red-600`, `text-red-600`, `border-red-600`
- Admin panel: Dark gradients (`from-gray-900 via-gray-800 to-red-900`)

## Migration Context
- **Migrated from Supabase to SQLite** (see `MIGRATION_TO_SQLITE_COMPLETE.md`)
- Old Supabase references removed from codebase
- Database is fully local (no internet required)
- RLS policies and Supabase SQL files kept for reference only

## Common Tasks

### Adding a New Product Category
1. Add to `prisma/seed.js` categories array
2. Create page: `src/app/products/{human|veterinary}-new/{slug}/page.tsx`
3. Fetch products: `humanProductsAPI.getByCategory('{slug}')`

### Creating Admin CRUD Page
1. Use existing admin pages as templates (`/admin/products/human/page.tsx`)
2. Include: ProductFormModal, upload support, react-hook-form, toast notifications
3. Call appropriate API helpers from `lib/api.ts`

### Debugging Database Issues
- Run `npx prisma studio` to inspect data visually
- Check field mappings in `lib/prisma.ts` API helpers
- Verify API routes in `src/app/api/products/`

## Security Notes
- Content-Security-Policy configured in `next.config.ts`
- File upload validation: type + size checks
- No sensitive data in localStorage (temporary auth only)
- Server actions body size limit: 10MB

---
> Source: [moabdelbaset1/sobek-Eygpt](https://github.com/moabdelbaset1/sobek-Eygpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
