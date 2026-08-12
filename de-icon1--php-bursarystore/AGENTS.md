# Bursary Inventory — Copilot Instructions (concise)

This repo is a monolithic PHP + MySQL web app (OOU Bursary Store inventory). The goal of these instructions is to get an AI coding agent immediately productive with minimal assumptions.

- Project entry: `index.php` is the login + role dispatch entrypoint. Role strings include `admin`, `vc`, `storekeeper`.
- DB connection: `assets/inc/config.php` exposes a global `$mysqli`. Use it — do NOT create ad-hoc new DB connectors.
- Sessions: code relies on `$_SESSION['user_id']`, `$_SESSION['username']`, `$_SESSION['role']`, `$_SESSION['full_name']`. Protect pages with `require_once 'assets/inc/checklogin.php'`.

- Conventions to follow:
  - Use prepared statements via `$mysqli->prepare()`, bind params, call `$stmt->execute()` and `$stmt->close()`.
  - Call `log_action($mysqli, $user_id, $message, $meta)` from `assets/inc/functions.php` for auditable changes.
  - Layout: include `assets/inc/head.php` first, then `nav.php`/`sidebar*.php`, then page body, then `assets/inc/footer.php`.
  - UI: use SweetAlert via `assets/js/swal.js` for notifications, and existing JS initializers under `assets/js/pages/` for component behavior.

- Typical request flow (pattern to mirror): page (GET) → form POST to same or helper endpoint → server validates → prepared statement → `log_action()` → redirect/render.

- Common files to check first:
  - `index.php` — login + redirects
  - `assets/inc/config.php` — DB and environment toggles
  - `assets/inc/functions.php` — helpers including `log_action()`
  - `assets/inc/checklogin.php` — session enforcement
  - `assets/inc/head.php` — common head, active-sidebar JS, SWAL hooks
  - `sql/migrations/` — DB schema/migrations

- Local dev / run:
  - XAMPP-style: start Apache + MySQL, open `http://localhost/bursary/index.php`.
  - Default DB name used in config: `bursary`. Local dev commonly uses `root` without a password.

- Debugging & safety:
  - Temporarily enable `display_errors` and `error_reporting(E_ALL)` in `assets/inc/config.php` only for local debugging; do not commit credential changes.
  - Sanitize and validate uploaded CSVs in `upload_stock.php` before DB writes.

- Tasks/PR checklist for agents:
  1. Reuse the global `$mysqli` and prepared-statement pattern.
  2. Preserve session variable names and `role` values.
  3. Update or add pages following the include/layout pattern (`head.php` → `nav/sidebar` → body → `footer.php`).
  4. Add `log_action()` calls for state-changing operations.
  5. Avoid editing `assets/inc/config.php` unless changing only non-secret flags; never commit production credentials.

- Helpful snippets (copy/paste):

Prepared statement
```php
$stmt = $mysqli->prepare("INSERT INTO items (name, qty) VALUES (?, ?)");
$stmt->bind_param('si', $name, $qty);
$stmt->execute();
$stmt->close();
```

Protect page
```php
require_once 'assets/inc/checklogin.php';
// now $_SESSION enforced
```

If you need additional details (example queries, specific page scaffolds, or test data), tell me which area to expand and I will update this file with a short example block.
```instructions
<!-- Project-specific Copilot instructions for AI coding agents -->
# Bursary Inventory — Copilot Instructions

This repository is a PHP + MySQL web app (OOU Bursary Store inventory). Use these concise rules to make changes safely and productively.

- Project entry & auth:
  - The login entry is [index.php](index.php). Authentication uses `password_hash()`/`password_verify()` and role strings (e.g. `admin`, `vc`, `storekeeper`). See the login flow in [index.php](index.php).

- Database & DB access:
  - DB connection lives in [assets/inc/config.php](assets/inc/config.php). The app uses a global `$mysqli` instance.
  - Common tables: `users` (fields: `user_id`, `username`, `password`, `role`, `full_name`) and `logs` (used by `log_action()`). See [assets/inc/functions.php](assets/inc/functions.php).
  - Prefer prepared statements (`$mysqli->prepare()`) — the project already uses them in `index.php`. When adding queries, follow the same style and call `->close()` on statements.

- Includes & layout conventions:
  - Shared partials live under `assets/inc/` (e.g. [head.php](assets/inc/head.php), `nav.php`, `sidebar.php`, `footer.php`). New pages follow the pattern: include `head.php`, then `nav.php`/`sidebar.php`, then page content, then `footer.php`.
  - Static assets served from `assets/` (CSS, JS, images). Use relative paths like `assets/js/...` as existing pages do.

- Session & access control:
  - Sessions are used heavily — check `$_SESSION['user_id']` / `$_SESSION['role']`. Some pages use `assets/inc/checklogin.php` to enforce login. When adding protected pages, include the existing check login pattern.

- UI & alerts:
  - UI messages use SweetAlert (`assets/js/swal.js`) and small JS injections in `head.php` / `index.php`. Mirror that approach for consistent UX.

- Testing / running locally:
  - This is an XAMPP-style app: start Apache + MySQL and open `http://localhost/bursary/index.php`.
  - DB name used by default: `bursary` (see `assets/inc/config.php`). Expect `root` user w/o password in local dev.

- Safe modification checklist for PRs:
  - Update `assets/inc/config.php` only if changing DB credentials; avoid committing production secrets.
  - Use prepared statements and bind parameters for all new DB work.
  - Preserve session variable names (`user_id`, `username`, `role`, `full_name`).
  - When changing navigation or menus, update the appropriate `assets/inc/sidebar*.php` file and ensure active link JS in `assets/inc/head.php` still functions.

- Helpful references (start here):
  - `index.php` — login + role redirects
  - `assets/inc/config.php` — DB connection
  - `assets/inc/functions.php` — helpers (e.g. `log_action()`)
  - `assets/inc/head.php` — common head, active-sidebar JS, SWAL hooks
  - `assets/inc/checklogin.php` — session enforcement

If anything above is unclear or you want more examples from any page, ask and I'll add focused examples (SQL schema snippets, common query patterns, or typical page scaffolding).

```

## Quick Architecture & Data Flow

- **Monolith PHP app (no separate API):** single Apache/PHP app serving pages and handling forms. UI pages call server-side PHP endpoints which perform DB operations then redirect or render views.
- **Primary data layer:** MySQL via a single global `$mysqli` in [assets/inc/config.php](assets/inc/config.php). SQL schema and migrations live in [sql/migrations/](sql/migrations/).
- **Typical request flow:** user → form page (e.g. [create_request.php](create_request.php)) → POST to same or helper endpoint (e.g. [fulfill_request.php](fulfill_request.php)) → server validates, runs prepared statement, calls `log_action()` in [assets/inc/functions.php](assets/inc/functions.php), then redirects back to UI.
- **Common features:** stock upload/receive (`upload_stock.php`, `receive_stock.php`, `stock_receive.php`), requests (`create_request.php`, `fulfill_request.php`), reporting (`inventory_report.php`, `inventory_charts.php`).

## Developer Workflows & Local Run

- Local dev is XAMPP-style: start Apache + MySQL, then open `http://localhost/bursary/index.php`.
- DB name: default expected in config is `bursary` (see [assets/inc/config.php](assets/inc/config.php)). Local dev commonly uses `root` without password.
- Helpful tools in `tools/`: `create_admin.php` (create local admin), `seed_stationery.php` (seed test items), `count_braces.php` (utility). Use these scripts via browser or CLI PHP when convenient.

## Conventions & Patterns (project-specific)

- **DB access:** always prefer prepared statements using `$mysqli->prepare()` and call `$stmt->close()` after execution — follow style in [index.php](index.php).
- **Includes/layout:** pages include `assets/inc/head.php` first, then `nav.php`/`sidebar*.php`, then page content, and finally `footer.php` (see many top-level pages for examples).
- **Session & roles:** use `$_SESSION['user_id']`, `$_SESSION['username']`, `$_SESSION['role']`, `$_SESSION['full_name']`. Use `assets/inc/checklogin.php` to protect pages.
- **UI/UX hooks:** use SweetAlert via [assets/js/swal.js](assets/js/swal.js). `assets/inc/head.php` contains common JS hooks and active-sidebar logic.
- **JS modules & assets:** frontend uses Bootstrap, DataTables, Footable, Flatpickr, chart libs (Chart.js, c3). Look under `assets/js/pages/` and `libs/` for specific page initializers.

## Integration Points & External Dependencies

- No external microservices — integrations are: browser ↔ PHP ↔ MySQL. Static assets under `assets/` and `libs/` (e.g., datatables, bootstrap-table).
- File uploads and CSV imports handled by `upload_stock.php` — validate file inputs and sanitize before DB writes.

## Debugging & Safe Edits

- To debug SQL or PHP issues, enable errors in `assets/inc/config.php` locally (temporarily set `display_errors` and `error_reporting(E_ALL)`). Commit no secrets.
- When changing `assets/inc/config.php` be careful not to commit production credentials.
- Use `log_action()` in [assets/inc/functions.php](assets/inc/functions.php) to add traceable audit entries when altering behavior.

## Examples (copy/paste patterns)

- Prepared statement pattern:

```php
$stmt = $mysqli->prepare("INSERT INTO items (name, qty) VALUES (?, ?)");
$stmt->bind_param('si', $name, $qty);
$stmt->execute();
$stmt->close();
```

- Protect page pattern (top of page):

```php
require_once 'assets/inc/checklogin.php';
// now $_SESSION is available and enforced
```

- Log action example:

```php
require_once 'assets/inc/functions.php';
log_action($mysqli, $user_id, 'Updated stock', 'SKU-123');
```

## Where to Look First (high value files)

- `index.php` — login + role redirects
- `assets/inc/config.php` — DB and env toggles
- `assets/inc/functions.php` — helpers including `log_action()`
- `assets/inc/head.php` — shared head, active-sidebar JS, SWAL hooks
- `assets/inc/checklogin.php` — session enforcement
- `sql/migrations/000_create_users_and_logs.sql` — initial DB schema

If any section is unclear or you want me to expand examples or add quick code fixes/tests, tell me which area to focus on.
<!-- Project-specific Copilot instructions for AI coding agents -->
# Bursary Inventory — Copilot Instructions

This repository is a PHP + MySQL web app (OOU Bursary Store inventory). Use these concise rules to make changes safely and productively.

- Project entry & auth:
  - The login entry is [index.php](index.php). Authentication uses `password_hash()`/`password_verify()` and role strings (e.g. `admin`, `vc`, `storekeeper`). See the login flow in [index.php](index.php).

- Database & DB access:
  - DB connection lives in [assets/inc/config.php](assets/inc/config.php). The app uses a global `$mysqli` instance.
  - Common tables: `users` (fields: `user_id`, `username`, `password`, `role`, `full_name`) and `logs` (used by `log_action()`). See [assets/inc/functions.php](assets/inc/functions.php).
  - Prefer prepared statements (`$mysqli->prepare()`) — the project already uses them in `index.php`. When adding queries, follow the same style and call `->close()` on statements.

- Includes & layout conventions:
  - Shared partials live under `assets/inc/` (e.g. [head.php](assets/inc/head.php), `nav.php`, `sidebar.php`, `footer.php`). New pages follow the pattern: include `head.php`, then `nav.php`/`sidebar.php`, then page content, then `footer.php`.
  - Static assets served from `assets/` (CSS, JS, images). Use relative paths like `assets/js/...` as existing pages do.

- Session & access control:
  - Sessions are used heavily — check `$_SESSION['user_id']` / `$_SESSION['role']`. Some pages use `assets/inc/checklogin.php` to enforce login. When adding protected pages, include the existing check login pattern.

- UI & alerts:
  - UI messages use SweetAlert (`assets/js/swal.js`) and small JS injections in `head.php` / `index.php`. Mirror that approach for consistent UX.

- Testing / running locally:
  - This is an XAMPP-style app: run Apache + MySQL and open `http://localhost/bursary/index.php`.
  - DB name used by default: `bursary` (see `assets/inc/config.php`). Expect `root` user w/o password in local dev.

- Safe modification checklist for PRs:
  - Update `assets/inc/config.php` only if changing DB credentials; avoid committing production secrets.
  - Use prepared statements and bind parameters for all new DB work.
  - Preserve session variable names (`user_id`, `username`, `role`, `full_name`).
  - When changing navigation or menus, update the appropriate `assets/inc/sidebar*.php` file and ensure active link JS in `assets/inc/head.php` still functions.

- Helpful references (start here):
  - `index.php` — login + role redirects
  - `assets/inc/config.php` — DB connection
  - `assets/inc/functions.php` — helpers (e.g. `log_action()`)
  - `assets/inc/head.php` — common head, active-sidebar JS, SWAL hooks
  - `assets/inc/checklogin.php` — session enforcement

If anything above is unclear or you want more examples from any page, ask and I'll add focused examples (SQL schema snippets, common query patterns, or typical page scaffolding).

---
> Source: [De-Icon1/php_bursarystore-](https://github.com/De-Icon1/php_bursarystore-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
