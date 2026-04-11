# AI Agent Instructions for Sekolah Project

## Project Overview
Laravel 12 + React 18 Islamic school profile website with full CMS. Backend API built with Laravel, frontend SPA with React Router, admin panel with authentication, and MySQL database. Features include: slider management, articles with categories, school profile sections, registration system, and comprehensive admin CRUD operations.

## Architecture & Stack

### Core Technologies
- **Backend**: Laravel 12 with Sanctum for API authentication
- **Frontend**: React 18 + React Router + Axios for API calls
- **Styling**: Tailwind CSS 4.0 with Islamic minimalist design (emerald green theme)
- **Build**: Vite with HMR for React
- **Database**: MySQL with eloquent ORM and relationships
- **Authentication**: Session-based admin authentication with custom middleware

### Database Configuration
- **Primary**: MySQL connection (not SQLite) - check `.env` for `DB_CONNECTION=mysql`
- **Session Storage**: Database-driven (`SESSION_DRIVER=database`)
- **Cache**: Database-backed (`CACHE_STORE=database`)
- **Queue**: Database driver (`QUEUE_CONNECTION=database`)

## Development Workflows

### Essential Commands
```powershell
# Setup from scratch
composer run setup  # Installs dependencies, generates key, migrates DB, builds assets

# Development server with hot reload
composer run dev    # Runs server, queue, logs (pail), and Vite concurrently

# Testing
composer run test   # Clears config and runs PHPUnit tests

# Artisan commands
php artisan migrate
php artisan make:model ModelName -m
php artisan make:controller ControllerName
```

### Asset Building
- **Dev**: `npm run dev` (Vite with HMR)
- **Build**: `npm run build` (production bundle)
- Entry points: `resources/css/app.css`, `resources/js/app.js`
- Configured in `vite.config.js` with Tailwind v4 plugin

### Testing Environment
- Uses in-memory SQLite for tests (see `phpunit.xml`)
- Test environment variables override `.env` settings
- Run tests with `php artisan test` or `vendor/bin/phpunit`

## Project-Specific Conventions

### File Structure
- **Controllers**: `app/Http/Controllers/` - only base `Controller.php` exists
- **Models**: `app/Models/` - only `User.php` exists
- **Migrations**: `database/migrations/` - standard Laravel auth + cache/jobs tables
- **Routes**: `routes/web.php` (single welcome route), `routes/console.php`
- **Views**: `resources/views/` - Blade templates with inline Tailwind CSS

### Code Patterns
- **Namespace**: `App\` for application code, PSR-4 autoloading
- **Service Providers**: Registered in `bootstrap/providers.php` (Laravel 12 convention)
- **Eloquent Models**: Extend `Illuminate\Foundation\Auth\User` for User model
- **View Rendering**: Use `return view('welcome')` in routes/controllers

### Styling Approach
- **Tailwind CSS v4**: Latest version with `@tailwindcss/vite` plugin
- **Dark Mode**: Configured with `prefers-color-scheme` in `welcome.blade.php`
- **No Config File**: Tailwind v4 uses CSS-based configuration, not `tailwind.config.js`
- Use `@vite(['resources/css/app.css', 'resources/js/app.js'])` in Blade templates

## Environment & Dependencies

### Windows/Laragon Specifics
- Project located in `c:\laragon\www\sekolah`
- Use PowerShell commands (`;` for chaining, not `&&`)
- MySQL accessible at `127.0.0.1:3306` via Laragon

### Composer Scripts
Reference `composer.json` for available scripts:
- `setup`: Full project initialization
- `dev`: Multi-process development environment with concurrently
- `test`: Config-cleared testing

## Common Tasks

### Adding New Features
1. Create migration: `php artisan make:migration create_table_name`
2. Create model: `php artisan make:model ModelName -m` (with migration)
3. Create controller: `php artisan make:controller NameController`
4. Add routes in `routes/web.php`
5. Create Blade views in `resources/views/`

### Database Operations
- Always run `php artisan migrate` after creating migrations
- Use database driver for sessions, cache, and queues (already configured)
- Factory pattern available in `database/factories/`

### Frontend Development
- Add Tailwind classes directly to Blade templates
- JavaScript modules in `resources/js/app.js` (uses Axios, configured in `bootstrap.js`)
- Vite auto-refreshes on file changes in dev mode

## Important Notes
- **Admin Login**: Email: admin@madrasah.sch.id, Password: admin123
- **API Endpoints**: All available at `/api/v1/` for public and `/api/admin/` for protected routes
- **File Uploads**: Images stored in `storage/app/public` with symlink at `public/storage`
- **React SPA**: All routes handled by React Router, Laravel serves single blade template
- **Models**: Full CRUD for sliders, articles, categories, about, vision/mission, history, organization, programs, facilities, registrations
- **Database**: Seeded with sample data including admin user and content for all sections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kusumarama)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kusumarama)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
