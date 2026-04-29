<!-- Copilot / AI agent instructions for the OOU Directorate of Works Inventory project -->

# Quick Orientation

- This is a PHP web app served under XAMPP/Apache (project root expected in htdocs). Start Apache + MySQL, then open http://localhost/works to run the app.
- Database connection is via `mysqli` in [assets/inc/config.php](assets/inc/config.php#L1). Use that file for DB credentials and global `$mysqli` usage.

# Key patterns & conventions

- Centralized DB: `assets/inc/config.php` creates a global `$mysqli` used throughout — functions often call `global $mysqli` (see [assets/inc/functions.php](assets/inc/functions.php#L1)).
- Authentication: `index.php` and other login pages use `session_start()` and store `$_SESSION` keys: `user_id`, `username`, `full_name`, `role`. Use these exact keys for role checks and redirects.
- Passwords: stored using PHP password hashing functions. Authentication checks use `password_verify()` (see [index.php](index.php#L1)). When creating/updating users use `password_hash()`.
- Prepared statements: database queries use `prepare()` + `bind_param()` across the codebase. Follow that pattern to avoid SQL injection.
- Role-based redirects: the app routes by `role` strings (examples: `admin`, `director`, `supervisor`, `storekeeper`, `transport`, `vc`, `electrical`). Respect these values when adding new roles.
- Logging: a helper `log_action($user_id, $action)` in `assets/inc/functions.php` inserts into `logs` table; it passes client IP as `mac` column. Prefer using this helper for audit entries.

# Files & directories to inspect when changing behavior

- Configuration & helpers: [assets/inc/config.php](assets/inc/config.php#L1), [assets/inc/functions.php](assets/inc/functions.php#L1)
- Authentication entrypoint: [index.php](index.php#L1) (login flow, session population, role redirects)
- UI assets: `assets/css/`, `assets/js/` (notably `assets/js/swal.js` used for toast/alerts)
- Dashboards & role pages: `admin_dashboard.php`, `transport_dashboard.php`, `electrical_dashboard.php`, `vc_dashboard.php` — follow their existing patterns for authorization and include ordering.

# Practical guidance for AI edits

- When adding DB queries, use prepared statements and `global $mysqli` (match existing bind types). Example pattern in `index.php`: prepare → bind_param → execute → store_result → bind_result → fetch.
- For auth changes, update session keys consistently. Any code that checks role should compare to the string values used in `index.php` redirects.
- When adding new pages, include `assets/inc/config.php` and `assets/inc/functions.php` at top and call `session_start()` if session access is required.
- For user-facing feedback, reuse `assets/js/swal.js` patterns (server prints JS that calls `swal()` when `$err` or `$success` are set in pages like `index.php`).

# Development & debugging notes

- Local run: Place the repository in XAMPP `htdocs`, start Apache & MySQL, import DB schema if needed. The project expects a `users` table, `logs` table and inventory tables (search SQL files like `create_stock_costs_table.sql`).
- DB migrations: check SQL files in repo root for schema snippets to import into the DB.
- Common errors: missing `include` paths are relative to project root; use same relative includes seen in existing files (e.g., `include('assets/inc/config.php')`).

# Examples (copy-n-paste)

- Read DB config and use `$mysqli`:

  <?php
  include('assets/inc/config.php');
  global $mysqli;
  $stmt = $mysqli->prepare("SELECT id FROM table WHERE col = ?");

- Use password hashing:

  $hash = password_hash(
    'secret', PASSWORD_DEFAULT
  );

# What NOT to change without review

- The session key names (`user_id`, `username`, `full_name`, `role`) and the role strings used for redirects.
- The DB connection pattern in `assets/inc/config.php` — other files rely on that global variable.

# If anything is unclear

Please point to the file(s) or a specific flow (login, stock receive, reports) you want expanded and I will augment these instructions with code examples and tests.

---
> Source: [De-Icon1/php_inventoryworks](https://github.com/De-Icon1/php_inventoryworks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
