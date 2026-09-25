## Copilot Instructions for AI Coding Agents

This guide provides essential knowledge for AI agents working in the capstone-project-portal codebase. Follow these conventions and workflows to be immediately productive.

### 1. Project Architecture
- **Frameworks:** Laravel (PHP backend), Vite (frontend build), Inertia.js (SPA bridge), TypeScript/React (frontend).
- **Backend:**
  - Controllers: `app/Http/Controllers/`
  - Models: `app/Models/`
  - Requests: `app/Http/Requests/`
  - Middleware: `app/Http/Middleware/`
  - Service Providers: `app/Providers/`
- **Frontend:**
  - Main entry: `resources/js/app.tsx`
  - Components: `resources/js/components/`
  - Pages: `resources/js/pages/`
  - Layouts: `resources/js/layouts/`
- **Routing:**
  - Web routes: `routes/web.php`
  - Auth routes: `routes/auth.php`
  - API endpoints are typically handled via controllers and web routes.

### 2. Developer Workflows
- **Frontend Dependencies:**
  - `npm install && npm run build`
- **Run Backend:**
  - `composer run serve` (Laravel dev server)
- **Database:**
  - MariaDB setup as per `.env` file.
  - Migrations: `php artisan migrate`
- **Testing:**
  - PHP: `./vendor/bin/pest` (Feature/Unit tests in `tests/`)
  - JS/TS: No explicit test setup found; add tests in `resources/js/` if needed.

### 3. Project-Specific Conventions
- **TypeScript/React:**
  - Use functional components and hooks (`resources/js/components/`, `resources/js/hooks/`).
  - Shared types: `resources/js/types/`
- **Laravel:**
  - Requests validation via FormRequest classes in `app/Http/Requests/`
  - Service Providers register custom services and Fortify authentication.
- **Inertia.js:**
  - Backend returns Inertia responses from controllers for SPA navigation.

### 4. Integration Points & Dependencies
- **Authentication:** Laravel Fortify (`config/fortify.php`, `app/Providers/FortifyServiceProvider.php`)
- **Frontend/Backend Bridge:** Inertia.js (see controller responses and frontend pages)
- **Build Tools:** Vite (`vite.config.ts`), TypeScript (`tsconfig.json`)
- **Composer/NPM:** PHP dependencies in `composer.json`, JS/TS in `package.json`

### 5. Patterns & Examples
- **User Model:** `app/Models/User.php` (reference for Eloquent usage)
- **Dashboard Feature Test:** `tests/Feature/DashboardTest.php` (testing conventions)
- **Main Layout:** `resources/js/layouts/` (SPA layout patterns)

### 6. Tips for AI Agents
- Always use the context7 MCP server for up-to-date documentation.
- Always update both backend and frontend when adding new features that cross boundaries.
- Reference migrations and seeders for database changes.
- Use Pest for PHP tests; follow existing test structure.
- Prefer TypeScript for new frontend code.

---
For questions or missing conventions, ask for clarification or review related files before making assumptions.

---
> Source: [adulbrich/capstone-project-portal](https://github.com/adulbrich/capstone-project-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-25 -->
