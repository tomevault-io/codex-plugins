# Copilot instructions (checkiso-saas)

## Project snapshot
- Framework: CodeIgniter 4 (`codeigniter4/framework`), PHP 8.2 (see `composer.json`).
- Web entrypoint is `public/index.php`; Docker nginx serves `/var/www/html/public` (see `docker/nginx/default.conf`).
- Current routing is minimal (`/` -> `Home::index`) in `app/Config/Routes.php`.

## Local dev workflows
- Install deps: `composer install`.
- Start dev server: `php spark serve` → `http://localhost:8080/`.
- Configure environment via `.env` (copy from `env`). `app.baseURL` defaults to `http://localhost:8080/`.
- MySQL (local): database `pers_checkiso`, user `pers_checkiso`, host `127.0.0.1:3306`.

## Database & domain model (migrations)
- Migrations live in `app/Database/Migrations/` and are enabled in `app/Config/Migrations.php`.
- Domain schema already defined (Feb 24, 2026 migrations):
  - SaaS tenancy: `tenants`, `companies`, `org_units` (tree via `parent_id`).
  - Identity: `users`, and `memberships` (user↔tenant, optionally company/org unit).
  - RBAC: `roles` (tenant|platform), `permissions`, `role_permissions`, `membership_roles`.
  - Standards catalog: `standards`, `standard_versions` (global, versioned).
  - ISO hierarchy: `domains` → `clauses` → `controls` (global, linked to `standard_versions`).
- Migration conventions used here (follow existing files): `BIGINT UNSIGNED` ids, InnoDB engine, explicit indexes, and foreign keys with cascading rules; most tenant-scoped tables include `created_at/updated_at/deleted_at`.
- Apply migrations: `php spark migrate`.

## Sessions
- App config currently uses file sessions (`Config\Session::$driver = FileHandler`).
- A DB session table migration exists (`ci_sessions`). If switching to DB sessions, use `CodeIgniter\Session\Handlers\DatabaseHandler` and set `Config\Session::$savePath = 'ci_sessions'`.

## Adding new HTTP features
- Put new controllers under `app/Controllers/Web/` (web, session-based) or `app/Controllers/Api/` (future API). Extend `app/Controllers/BaseController.php`.
- Register routes explicitly in `app/Config/Routes.php`. Protected routes go inside the `auth`-filtered group:
  ```php
  $routes->group('', ['filter' => 'auth'], static function ($routes): void {
      $routes->get('/my-page', 'Web\MyController::index');
  });
  ```
- CSRF is enforced on all POST requests (via `$methods = ['post' => ['csrf']]`). Always include `<?= csrf_field() ?>` in every HTML form.
- Session keys set at login: `user_id`, `user_email`, `user_name`, `tenant_id`, `membership_id`.
- `form` and `url` helpers are auto-loaded for all controllers via `BaseController::$helpers`.

## Auth & tenant context
- `app/Filters/AuthFilter.php` — checks `session()->get('user_id')`; redirects to `/login` if absent.
- Login flow: `AuthController::loginPost` → verify password → load default membership → set session.
- Register flow: one DB transaction creates `users` + `tenants` + `memberships` rows atomically.
- Tenant context lives in the session (`tenant_id`, `membership_id`). Every tenant-scoped query must filter by `session()->get('tenant_id')`.

## Views & layouts
- Layouts live in `app/Views/layouts/`: `auth.php` (centred, for login/register) and `app.php` (sidebar nav, for authenticated pages).
- Views use CI4 native inheritance: `$this->extend('layouts/app')` + `$this->section('content')` / `$this->endSection()`.

## Tests
- Run: `composer test` or (Windows) `vendor\bin\phpunit`.
- PHPUnit is configured by `phpunit.xml.dist` and boots CI via `vendor/codeigniter4/framework/system/Test/bootstrap.php`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaizaBahaedinne) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
