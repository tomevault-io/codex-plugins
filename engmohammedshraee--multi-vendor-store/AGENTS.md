# Copilot Instructions for AI Agents

## Project Overview
This is a Laravel-based e-commerce application. The codebase follows Laravel conventions but includes custom business logic for store management, cart operations, and order processing.

## Architecture & Key Components
- **app/Models/**: Eloquent models for core entities (User, Product, Cart, Order, etc.).
- **app/Http/Controllers/**: Handles HTTP requests and business logic.
- **app/Events/** & **app/Listeners/**: Implements event-driven patterns for actions like cart updates and order notifications.
- **app/Notifications/**: Custom notification classes for order events.
- **app/Observers/**: Observers for model lifecycle hooks (e.g., CartObserver, OrderObserver).
- **app/Facades/**: Custom facades (e.g., Cart) for reusable logic.
- **app/Helpers/**: Utility functions (e.g., Currency.php).
- **routes/**: Route definitions, split by context (web, api, dashboard, etc.).
- **resources/views/**: Blade templates for frontend rendering.

## Developer Workflows
- **Build assets**: Use Vite (`npm run dev` or `npm run build`).
- **Run tests**: Use Pest or PHPUnit (`vendor/bin/pest` or `vendor/bin/phpunit`).
- **Database migrations**: `php artisan migrate`.
- **Seed database**: `php artisan db:seed`.
- **Serve app**: `php artisan serve` (for local dev).

## Project-Specific Patterns
- **Event-driven actions**: Cart and order changes trigger events and listeners (see `app/Events/` and `app/Listeners/`).
- **Custom facades**: Business logic is often wrapped in facades (see `app/Facades/Cart.php`).
- **Observers**: Model events (create, update, delete) are handled via observers (see `app/Observers/`).
- **Helpers**: Utility logic is centralized in `app/Helpers/`.
- **Route organization**: Multiple route files for different contexts (web, api, dashboard).

## Integration Points
- **Notifications**: Uses Laravel's notification system for order events.
- **External packages**: Standard Laravel packages plus custom logic; check `composer.json` for dependencies.
- **Frontend**: Uses Vite and Tailwind CSS for asset management and styling.

## Examples
- To add a new cart event, create an event in `app/Events/`, a listener in `app/Listeners/`, and register them in `EventServiceProvider`.
- To extend cart logic, update `app/Facades/Cart.php` and related models/controllers.
- For custom validation, add rules in `app/Rules/` and use them in request classes.

## References
- [Laravel Documentation](https://laravel.com/docs)
- Key files: `app/Models/Cart.php`, `app/Facades/Cart.php`, `app/Events/EmptyCart.php`, `app/Listeners/EmptySpecialCart.php`, `routes/web.php`, `resources/views/`

---
_If any section is unclear or missing, please provide feedback to improve these instructions._

---
> Source: [engMohammedshraee/multi-vendor-store](https://github.com/engMohammedshraee/multi-vendor-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
