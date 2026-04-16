# Skincare E-Commerce Project Guidelines

## Project Overview

This is a **fullstack e-commerce platform** for skincare products with multilingual support (Arabic/English, RTL/LTR). Two separate apps:
- **Frontend** (`skincare/`): Next.js 14+ with i18next, Radix UI, TailwindCSS
- **Backend** (`backend/`): Express.js with MongoDB, JWT auth, Cloudinary image storage

Frontend communicates with backend via API routes (proxy pattern). Default language is Arabic.

## Code Style & Naming Conventions

### Backend (`backend/src/`)
- **Case**: camelCase for files, variables, functions
- **Async patterns**: Always use `async/await`, never callbacks
- **Validation**: Use Joi schema validation on all controller inputs
- **Errors**: Throw descriptive errors with status codes; middleware catches and responds
- **Models**: Mongoose schemas with bilingual name fields (`name: { en: string, ar: string }`)
- **File structure**: MVC pattern (models → controllers → routes)

### Frontend (`skincare/`)
- **Case**: kebab-case for files (e.g., `product-card.tsx`), camelCase for functions/variables
- **Components**: Use Radix UI components from `ui/` folder; wrap with Tailwind classes
- **i18n**: Use `useTranslations()` hook from i18next for all user-facing text
- **API calls**: Use `fetch()` wrapped in utility functions in `lib/` folder (see `products.ts`)
- **State management**: Context API for cart, providers in root layout
- **"Use client"**: Client components marked explicitly; use server components by default

## Build & Run Commands

### Backend
```bash
cd backend
npm run dev      # Start with nodemon on port 5000
npm run build    # Compile TypeScript to dist/
npm run start    # Run compiled app
npm run seed     # Create first admin user (requires ADMIN_EMAIL, ADMIN_PASSWORD in .env)
```

### Frontend
```bash
cd skincare
npm run dev      # Start dev server on port 3000
npm run build    # Next.js production build
npm run start    # Start production server
npm run lint     # ESLint check
```

### Environment Setup
Both apps require `.env` files:

**backend/.env**
```
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/skin-care
JWT_SECRET=your-secret-key-here
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=securepassword
CLOUDINARY_CLOUD_NAME=your-cloud
CLOUDINARY_API_KEY=your-key
CLOUDINARY_API_SECRET=your-secret
PORT=5000
```

**skincare/.env.local**
```
NEXT_PUBLIC_API_URL=http://localhost:5000
```

## Architecture & Key Patterns

### Data Flow
1. User interacts with Next.js frontend (port 3000)
2. Frontend API routes (`app/api/*/route.ts`) proxy requests to Express backend (port 5000)
3. Backend connects to MongoDB, performs CRUD operations, returns JSON
4. Frontend re-renders with new data

### Authentication
- **Admin only**: JWT tokens generated at login, stored in httpOnly cookies
- **Customers**: No login required; place orders anonymously with email + phone
- **Middleware**: `auth.ts` checks JWT token; used on protected admin routes

### Internationalization (i18n)
- **Routing**: `app/[locale]/` folder structure handles both `/en/` and `/ar/` paths
- **Messages**: Translation files in `skincare/messages/{ar,en}.json`
- **Switching**: Language toggle changes `[locale]` param in URL
- **Current status**: Home, Nav, Checkout fully translated. **TODO**: Footer, Cart, Contact, About, Brand, Product pages have hardcoded English strings

See [INTERNATIONALIZATION_SETUP.md](../skincare/INTERNATIONALIZATION_SETUP.md) for detailed i18n architecture.

### Product Model
- **Restricted brands**: Only "Topicrem" or "Novexpert" allowed (case-sensitive in validation)
- **Bilingual**: Product names/descriptions use `{ en: "", ar: "" }` objects
- **Images**: Stored on Cloudinary via multer + `cloudinary-api` in products route
- **Pricing**: All prices in JOD (Jordanian Dinar)

### Order Flow
- Orders don't use payment gateway; instead, system generates WhatsApp message template
- Customer phone + email captured; admin receives order notification (details in `orderController.ts`)

## Common Patterns & File Examples

### Backend Controller with Validation
See `backend/src/controllers/productController.ts`:
- Query input validated with Joi
- Async/await error handling
- Response with `200/201/400/500` status codes

### Frontend Component with i18n
See `skincare/components/site-header.tsx`:
- `useTranslations()` for text
- `useRouter()` + `locale` param for language switching
- Client-side state (cart count) from Context

### API Route (Frontend Proxy)
See `skincare/app/api/products/route.ts`:
- Proxy pattern: accepts request, forwards to `NEXT_PUBLIC_API_URL`
- Error handling with try/catch
- JSON response

## Common Pitfalls & Solutions

### Database
- **Seeding fails**: Ensure MongoDB URI is correct and ADMIN_EMAIL/PASSWORD set
- **No 000 imports**: Models must be imported in server.ts before routes

### Authentication
- **Token expired**: Check JWT_SECRET matches between frontend cookies and backend validation
- **Cookies not set**: Backend must send `Set-Cookie` header; ensure frontend doesn't override with manual token

### Internationalization
- **Missing translation**: Check `skincare/messages/{locale}.json` has the key. Use `fallbackLng: 'en'` if missing
- **Hardcoded strings**: Search codebase for non-i18n text on TODO pages (see INTERNATIONALIZATION_SETUP.md for task list)

### Frontend Build
- **Next.js compilation error**: Often a TypeScript issue; check `next build` output
- **Radix UI missing**: Ensure import from `@/components/ui/` and component is exported

### Backend TypeScript
- **Cannot find module**: Check import path is relative with `./` prefix
- **Express types missing**: Use `@types/express` in devDependencies

### API Communication
- **CORS errors**: Backend likely needs CORS middleware (already in `server.ts`)
- **401 Unauthorized**: Check JWT token is being sent in `Authorization` header on protected routes

## Where to Start

1. **Read architecture**: See `PROJECT_ARCHITECTURE.md` for full system design and data models (generated in root)
2. **Run locally**:
   - Set up `.env` files with MongoDB URI and credentials
   - `cd backend && npm install && npm run seed` (creates admin)
   - `cd ../skincare && npm install && npm run dev` (start frontend)
   - Visit `http://localhost:3000/en` to test
3. **Common tasks**:
   - **Add new product field**: Edit `backend/src/models/Product.ts`, add Joi validation in controller
   - **Add new page with i18n**: Create `skincare/app/[locale]/newpage/page.tsx`, add translation keys to `messages/{ar,en}.json`
   - **Add admin route**: Create controller in `backend/src/controllers/`, add route in `backend/src/routes/`, protect with auth middleware

## Next Steps for AI

When assisting with this codebase:
1. **Backend changes**: Always update Mongoose schema + Joi validation together
2. **Frontend changes**: Check i18n status before adding text; use `useTranslations()` for UI
3. **API additions**: Ensure both proxy route AND backend route are created
4. **Testing**: Use `npm run dev` to test locally before suggesting deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rmast2006-gif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
