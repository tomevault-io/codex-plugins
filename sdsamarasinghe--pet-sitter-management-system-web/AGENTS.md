# Copilot Instructions - Pet Sitter Management System Web

## Project Overview

This is the **frontend web application** for the Whiskarz Pet Sitter Management System, built with **Next.js 14** (App Router) and **TypeScript**. It provides a modern, responsive interface for pet owners, sitters, and admins to manage pet care services.

**Production URL:** https://www.whiskarz.com  
**Port:** 3000  
**API Backend:** Pet-Sitter-Management-System-API (port 8000)  
**Deployment:** Docker on Azure VM via GitHub Actions

---

## Tech Stack

- **Framework:** Next.js 14 (App Router, React Server Components)
- **Language:** TypeScript 5
- **Styling:** TailwindCSS 3.4 + CSS Variables (HSL)
- **UI Components:** shadcn/ui (Radix UI primitives)
- **HTTP Client:** Axios
- **Icons:** Lucide React
- **Date Handling:** date-fns + date-fns-tz
- **Images:** Cloudinary (CDN + optimization)
- **Font:** Inter (Google Fonts)
- **Containerization:** Docker (Node.js 22 Alpine)

---

## Architecture & Conventions

### Directory Structure
```
app/                    # Next.js App Router pages
  ├── layout.tsx        # Root layout (NavigationProvider + MainLayout)
  ├── page.tsx          # Landing page
  ├── login/            # Auth pages
  ├── signup/
  ├── dashboard/        # Admin/sitter dashboard
  ├── bookings/         # Booking management
  ├── pets/             # Pet management
  ├── profile/          # User profile
  ├── services/         # Service pages (dog, cat, bird, etc.)
  ├── scheduling/       # Availability/scheduling
  ├── messages/         # Messaging
  ├── invoices/         # Invoice management
  ├── reports/          # Reports
  └── api/              # API routes (proxy/server actions)
components/
  ├── ui/               # shadcn/ui components (Button, Card, Dialog, etc.)
  ├── layout/           # MainLayout, Footer, Sidebar
  ├── providers/        # Context providers (NavigationProvider)
  └── pets/             # Feature-specific components
lib/
  ├── api.ts            # Axios instance + interceptors
  ├── auth.ts           # Auth utilities (token management)
  ├── utils.ts          # General utilities (cn helper)
  ├── cloudinary.ts     # Image upload utilities
  └── design-tokens.ts  # Design system tokens
hooks/
  └── use-toast.ts      # Toast notification hook
config/
  └── environments.yml  # Environment configuration
```

### Routing
- Uses Next.js 14 App Router (file-based routing)
- Client components marked with `"use client"` directive
- Dynamic routes: `[id]` folders (e.g., `bookings/[id]`)
- API routes in `app/api/` for server-side operations

### Authentication
- JWT stored in `localStorage`
- Axios request interceptor auto-attaches Bearer token
- 401 response interceptor redirects to `/login`
- Auth utilities in `lib/auth.ts`:
  - `getToken()`, `setToken()`, `removeToken()`
  - `getUserFromToken()` - decodes JWT payload
  - `isAuthenticated()` - checks token validity + expiration
  - `getUserRole()` - extracts role from token

### API Client (`lib/api.ts`)
- Axios instance with configurable `NEXT_PUBLIC_API_URL`
- Request interceptor: JWT token attachment
- Response interceptor: 401 handling with auto-redirect
- Exported auth API methods: `login`, `forgotPassword`, `resetPassword`

---

## Styling System

### Theme Colors (Navy Blue Professional Theme)
- **Primary:** Deep Navy Blue (`#1A2A6C`) - `navy-blue-900`
- **Secondary:** Medium Navy (`#0F3460`) - `medium-navy-900`
- **Accent:** Sky Blue (`#00AEEF`) - `sky-blue-500`
- **Background:** Neutral (`#F5F7FA`) - `neutral-bg-50`
- **Text Primary:** Dark (`#2C3E50`) - `text-primary-800`
- **Text Secondary:** Gray (`#7F8C8D`) - `text-secondary-500`

### CSS Variables (HSL-based for shadcn/ui)
```css
--background, --foreground
--primary, --primary-foreground
--secondary, --secondary-foreground
--muted, --muted-foreground
--accent, --accent-foreground
--destructive, --destructive-foreground
--border, --input, --ring, --radius
```

### Custom Animations
- `fade-in`, `fade-in-up`, `fade-in-down`, `fade-in-left`, `fade-in-right`
- `scale-in`, `slide-up`, `slide-down`
- `bounce-gentle`, `pulse-soft`, `shimmer`

### Component Styling
- Use `cn()` utility from `lib/utils.ts` for conditional classes
- shadcn/ui components with Radix UI primitives
- TailwindCSS utility-first approach
- Responsive design with mobile-first breakpoints
- Dark mode support via `class` strategy

---

## Key Pages & Features

| Page | Route | Purpose |
|------|-------|---------|
| Landing | `/` | Public homepage with service showcase |
| Login | `/login` | User authentication |
| Signup | `/signup` | New user registration |
| Dashboard | `/dashboard` | Admin/sitter management hub |
| Bookings | `/bookings` | Booking list and management |
| Pets | `/pets` | Pet profiles CRUD |
| Profile | `/profile` | User profile management |
| Services | `/services/*` | Service detail pages |
| Scheduling | `/scheduling` | Sitter availability |
| Messages | `/messages` | Internal messaging |
| Invoices | `/invoices` | Invoice management |
| Reports | `/reports` | System reports |
| Find Care | `/find-care` | Find available sitters |
| Service Inquiry | `/service-inquiry` | New service request form |

---

## Environment Variables

```env
# Required
NEXT_PUBLIC_API_URL=http://localhost:8000   # Backend API URL

# Optional (for server-side features)
DATABASE_URL=
JWT_SECRET=
EMAIL_SERVICE_API_KEY=
```

---

## Code Style Guidelines

- Use TypeScript strict mode
- Client components must have `"use client"` at top
- Import paths use `@/` alias (e.g., `@/components/ui/button`)
- Use shadcn/ui components for UI elements
- Use `cn()` for conditional class merging
- API calls via the `api` axios instance from `lib/api.ts`
- State management via React context (NavigationProvider)
- Responsive design: mobile-first with Tailwind breakpoints
- Image optimization via Next.js `<Image>` with Cloudinary domain

---

## Component Patterns

### shadcn/ui Usage
```tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Dialog, DialogContent, DialogTrigger } from "@/components/ui/dialog"
```

### API Calls
```tsx
import api from "@/lib/api"

const fetchData = async () => {
  const response = await api.get('/endpoint');
  return response.data;
};
```

### Auth Check
```tsx
import { isAuthenticated, getUserRole } from "@/lib/auth"

if (!isAuthenticated()) {
  router.push('/login');
}
```

---

## Build & Development

```bash
# Development
npm run dev              # Start dev server
npm run dev:local        # Dev with local env
npm run dev:production   # Dev with prod env

# Production
npm run build            # Build for production
npm run start            # Start production server

# Docker
docker-compose up --build
docker-compose down

# Environment
npm run env:dev          # Switch to dev env
npm run env:prod         # Switch to prod env
```

---
> Source: [SDSamarasinghe/Pet-Sitter-Management-System-Web](https://github.com/SDSamarasinghe/Pet-Sitter-Management-System-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
