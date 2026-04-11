# Copilot Agent Instructions

## Project Overview

This is a Laravel 12 application using PHP 8.3, with a focus on native PHP and driver switching for SQL databases. The codebase follows standard Laravel conventions but may include customizations for database drivers and native PHP integration.

## Architecture

- **App Structure**:  
  - `app/Http/Controllers/` — Route handlers and business logic  
  - `app/Models/` — Eloquent models (e.g., `User.php`)  
  - `app/Providers/` — Service providers for bootstrapping and configuration  
  - `routes/web.php` — Web routes  
  - `resources/views/` — Blade templates  
  - `config/` — Application and service configuration  
  - `database/migrations/` — Schema migrations  
  - `database/factories/` — Model factories for testing/seeding  
  - `database/seeders/` — Seed data

- **Native PHP/Driver Switching**:  
  - The project is designed to support switching SQL drivers natively.  
  - Look for custom logic in `app/Providers/` and `config/database.php`.

## Developer Workflows

- **Build & Serve**:  
  - Use `php artisan serve` to run the development server.
- **Database**:  
  - SQLite is used by default (`database/database.sqlite`).  
  - Migrate with `php artisan migrate`.  
  - Seed with `php artisan db:seed`.
- **Testing**:  
  - Pest is used for tests (`tests/Feature/`, `tests/Unit/`).  
  - Run tests: `./vendor/bin/pest`
- **Code Quality**:  
  - PHPStan config: `phpstan.neon`  
  - Run: `./vendor/bin/phpstan analyse`
  - Pint for code style: `pint.json`  
  - Run: `./vendor/bin/pint`

## Patterns & Conventions

- **Controllers**:  
  - Grouped by domain in `app/Http/Controllers/`.
- **Models**:  
  - Eloquent models in `app/Models/`.
- **Service Providers**:  
  - Custom providers may handle driver switching and app bootstrapping.
- **Testing**:  
  - Feature tests simulate HTTP requests; unit tests cover logic.
- **Assets**:  
  - Frontend assets built with Vite (`vite.config.js`).

## Integration Points

- **External Libraries**:  
  - Laravel, Pest, PHPStan, Pint, Vite.
- **Native PHP**:  
  - Custom logic may exist for direct PHP database access or driver switching.

## Key Files

- `config/database.php` — Database configuration and driver logic
- `app/Providers/VoltServiceProvider.php` — Custom service provider (potential driver logic)
- `routes/web.php` — Main route definitions
- `tests/Feature/ExampleTest.php` — Example feature test

## Example: Running All Tests

```bash
./vendor/bin/pest
```

## Example: Migrating Database

```bash
php artisan migrate
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abdallah-Tah)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Abdallah-Tah)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
