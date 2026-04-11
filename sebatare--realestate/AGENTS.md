# Real Estate Platform - AI Coding Agent Instructions

## Architecture Overview

This is a **full-stack real estate management platform** with a clear separation of concerns:

- **Frontend**: Next.js 15 (App Router) with TypeScript, Redux Toolkit + RTK Query for state management, AWS Amplify for authentication
- **Backend**: Express.js with Prisma ORM (PostgreSQL with PostGIS for geospatial queries), JWT-based authorization
- **Core Entities**: Properties, Tenants, Managers, Leases, Applications, Payments

### Data Flow

1. Frontend authenticates via AWS Amplify (custom:role claim determines user type: "manager" or "tenant")
2. RTK Query caches data and generates Bearer tokens from Amplify session for backend calls
3. Express middleware validates JWT token and enforces role-based access control
4. Prisma manages all database operations with type safety

## Key Patterns & Conventions

### Backend API Structure

- **Routes**: Each entity has a dedicated route file (`src/routes/*Routes.ts`) that imports controllers
- **Controllers**: Pure business logic in `src/controllers/*Controllers.ts` (no middleware concerns)
- **Auth Middleware**: Role-based access controlled via `authMiddleware(["manager"])` on routes (see [propertyRoutes.ts](server/src/routes/propertyRoutes.ts#L21))
- **File Uploads**: Use `multer` with memory storage for photos (see [propertyRoutes.ts](server/src/routes/propertyRoutes.ts#L10-L12)), photos uploaded to AWS S3
- **Prisma Schema**: Enums for controlled values (Amenity, Highlight, PropertyType, ApplicationStatus, PaymentStatus)

### Frontend State Management

- **Redux Store**: Combine `globalReducer` with RTK Query API slice ([redux.tsx](client/src/state/redux.tsx))
- **RTK Query**: Define endpoints with automatic cache invalidation via `tagTypes` ([api.ts](client/src/state/api.ts#L25-L31))
- **Auth Token**: Injected in request headers via `prepareHeaders` hook in RTK Query base query
- **User Context**: Resolved on app init; if user doesn't exist, create via `createNewUserInDatabase()` helper
- **Zod Schemas**: Form validation schemas live in [lib/schemas.ts](client/src/lib/schemas.ts) (defines PropertyFormData, ApplicationFormData, SettingsFormData types)

### Authentication & Authorization

- **Frontend Auth**: AWS Amplify manages login/session; custom:role claim parsed in [api.ts](client/src/state/api.ts#L44)
- **Backend Auth**: JWT decoded without verification in [authMiddleware.ts](server/src/middleware/authMiddleware.ts#L27) (token already verified by Amplify)
- **Route Protection**: Apply `authMiddleware(["manager"])` to restrict endpoints; returns 403 if role not in allowedRoles array
- **User Endpoints**: Routes like `/managers/{userId}` and `/tenants/{userId}` for role-specific user data

### UI Component Patterns

- **Radix UI Components**: Use shadcn/ui versions from [components/ui/](client/src/components/ui/) (already installed: button, input, select, dialog, tabs, table, etc.)
- **Layout Routes**: Use `(dashboard)` and `(nondashboard)` route groups; protect dashboard routes with auth in layout
- **Forms**: Use `react-hook-form` + Zod validation from [lib/schemas.ts](client/src/lib/schemas.ts)
- **Notifications**: Toast notifications via `sonner` (Toaster component auto-mounted in [layout.tsx](client/src/app/layout.tsx#L21))
- **Loading States**: Use `<Loading />` skeleton components during data fetches

## Developer Workflows

### Running the Project

```bash
# Backend setup
cd server
npm install
npm run prisma:generate  # Generates Prisma types to client/src/types/prismaTypes.d.ts
npm run seed            # Seeds database with test data
npm run dev             # Runs tsc in watch + nodemon (auto-reload on changes)

# Frontend setup (in separate terminal)
cd client
npm install
npm run dev             # Next.js dev server on http://localhost:3000
```

**Important**: Always run `npm run prisma:generate` after Prisma schema changes to sync types with frontend.

### Build & Deployment

- **Backend**: `npm run build` → outputs `dist/` → `npm start` runs compiled JS
- **Frontend**: `npm run build` → static Next.js build → `npm run start` runs production server
- **Environment Variables**: `.env.local` for both (API_BASE_URL, DATABASE_URL, JWT_SECRET, AWS credentials)

### Testing & Validation

- No test files found; suggest using Jest/Vitest for new tests
- Lint with `npm run lint` in client (Next.js ESLint)
- Type check: `npx tsc --noEmit` in both directories

## File Structure Decision Patterns

### Adding a New Entity Feature

1. **Database**: Add model to [server/prisma/schema.prisma](server/prisma/schema.prisma), run migrations
2. **Backend Routes**: Create `src/routes/newEntityRoutes.ts`, import in [src/index.ts](server/src/index.ts#L28)
3. **Backend Controllers**: Create `src/controllers/newEntityControllers.ts` with CRUD functions
4. **Frontend API**: Add RTK Query endpoints to [client/src/state/api.ts](client/src/state/api.ts), include proper tagTypes for caching
5. **Frontend Forms**: Add Zod schema to [lib/schemas.ts](client/src/lib/schemas.ts), create form component in `src/components/`
6. **Route Pages**: Add pages in `src/app/(dashboard)/newentity/` with layout nesting

### Database Schema Rules

- Use PostgreSQL with PostGIS enabled (`extensions = [postgis]` in schema)
- Enums for fixed value sets (always define in schema, not as strings)
- `@default(autoincrement())` for primary keys, `@default(uuid())` for UUIDs
- Foreign keys implicit via Prisma relations (e.g., `property Property @relation(...)`)

### Common Import Paths

- Type definitions: `import { Property, Tenant } from "@/types/prismaTypes"`
- Utils & helpers: `import { cleanParams, withToast } from "@/lib/utils"`
- UI components: `import { Button } from "@/components/ui/button"`
- Redux hooks: `import { useAppDispatch, useAppSelector } from "@/state/redux"`

## Critical External Dependencies

- **AWS Amplify**: Handles auth & session management; uses idToken for backend authorization
- **Redis/Cache**: Not yet used; consider for caching frequently queried data
- **AWS S3**: Used via @aws-sdk/client-s3 for photo uploads (multer hands off to S3 integration in controllers)
- **Mapbox**: Integrated for map display (`mapbox-gl` dependency); geospatial queries via PostGIS

## Common Gotchas & Notes

- **JWT Verification**: Backend does NOT verify JWT signature (assumes Amplify token is valid); adjust if adding external integrations
- **File Uploads**: Photos in memory until S3 upload in controller; limit file size to prevent OOM
- **Type Sync**: Frontend types auto-generated from Prisma schema; always regenerate after schema changes
- **Role Case Sensitivity**: Auth middleware normalizes to lowercase; ensure frontend stores roles consistently
- **CORS**: Enabled globally in Express; restrict in production via environment-specific CORS config

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebatare)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sebatare)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
