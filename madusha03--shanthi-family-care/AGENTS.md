## AI agent instructions for this codebase

This is a PHP/MySQL XAMPP app with two backend roles (Admin, Doctor) and a static public front page. There is no build step; it runs under Apache/PHP. Use the shared MySQLi connection and follow the include/session patterns below.

### Architecture and routing

- Public entry: `index.php` links to role portals under `backend/`.
- Role apps:
  - Admin: `backend/admin/*.php` with shared includes in `backend/admin/assets/inc/`.
  - Doctor: `backend/doc/*.php` with shared includes in `backend/doc/assets/inc/`.
- No framework/router: each PHP file is a page/action. Navigation chrome is composed with `include()` of `head.php`, `nav.php`, `sidebar.php`, `footer.php`.
- Database schema: `Database/hmisphp.sql` (tables prefixed `his_…`, e.g., `his_patients`, `his_docs`, `his_pharmaceuticals`). Many “numbers” (patient, doc, barcode) are alphanumeric and stored as VARCHAR.

### Core app patterns (follow these)

- Always start PHP pages that touch data with:
  - `session_start();`
  - `include('assets/inc/config.php');` which defines `$mysqli = new mysqli(...)`.
  - For authenticated pages: `include('assets/inc/checklogin.php'); check_login();` using the correct session key:
    - Admin session key: `$_SESSION['ad_id']`
    - Doctor session key: `$_SESSION['doc_id']`
- Use prepared statements with `$mysqli->prepare()` and `bind_param()` + `get_result()`.
  - Bind types: use `'s'` for alphanumeric keys like `pat_number`, `doc_number`, `phar_bcode` (they are VARCHAR), and `'i'` for integer primary keys like `pat_id`.
- Passwords follow a legacy “double-hash”: `sha1(md5($plain))` across the codebase (see `backend/admin/index.php`, `his_admin_register.php`, `his_admin_add_employee.php`, doc equivalents). If you add login/registration or reset flows, use the same function unless migrating everywhere.
- Random IDs (numbers) are generated with `substr(str_shuffle('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'),1,5)` and stored as VARCHAR. Preserve this convention for new “number” fields.

### Auth and access control

- Login pages:
  - Admin: `backend/admin/index.php` sets `$_SESSION['ad_id']` on success.
  - Doctor: `backend/doc/index.php` sets `$_SESSION['doc_id']` on success.
- Guard all role pages by including `checklogin.php` and calling `check_login()` before any output; redirects to role `index.php` if not logged in.

### Data access examples (from repo)

- Counting rows: see `backend/admin/his_admin_dashboard.php` (e.g., `SELECT count(*) FROM his_patients WHERE pat_type = 'OutPatient'`). Prefer parameterized variants for new code: `WHERE pat_type = ?` and bind `'s'`.
- Insert employee: `backend/admin/his_admin_add_employee.php` shows collecting `$_POST`, hashing password, `INSERT INTO his_docs (...) VALUES (?,?,?,?,?)` with `'sssss'` bind.
- Detail views commonly pattern: read id/number from `$_GET`, prepare `SELECT … WHERE <col> = ?`, bind, `$res = $stmt->get_result()`, and render.

### Local setup and workflows

- DB: import `Database/hmisphp.sql` into a MySQL/MariaDB server. Defaults in config are `localhost`, user `root`, empty password, db `hmisphp`.
- Runtime: serve the repo from Apache/PHP (e.g., XAMPP). Access via `http://localhost/Shanthi-Family-Care/`.
- To create an admin if none can log in, use `backend/admin/his_admin_register.php` (it uses the same double-hash) or insert via SQL matching the hashing convention.

### Conventions and gotchas specific to this repo

- Shared includes live under `backend/<role>/assets/inc/`; do not create parallel configs elsewhere. Reuse `$mysqli` from `config.php` and do not create additional connections.
- Treat “number” fields as strings in SQL binds; many pages historically used `'i'` by mistake—prefer `'s'` for values like `doc_number`, `pat_number`, `phar_bcode` to avoid leading-zero and alpha loss.
- UI assets are bundled per role under `backend/<role>/assets/`; keep new pages’ CSS/JS consistent by including `head.php` and footers.
- Redirects often compute host/uri from `$_SERVER` in `checklogin.php`. Call `check_login()` before sending any output.

### When adding a new admin/doc page

1. Start with:
   - `session_start(); include('assets/inc/config.php'); include('assets/inc/checklogin.php'); check_login();`
2. Pull IDs from `$_GET` or `$_POST` and validate minimally.
3. Use prepared statements; pick correct bind types (`'s'` for alphanumeric codes, `'i'` for numeric IDs).
4. Compose chrome with `include('assets/inc/head.php'); include('assets/inc/nav.php'); include('assets/inc/sidebar.php'); include('assets/inc/footer.php');`.

Key files to reference: `backend/admin/his_admin_dashboard.php`, `backend/admin/his_admin_add_employee.php`, `backend/admin/index.php`, `backend/admin/assets/inc/{config.php,checklogin.php}`, and doctor equivalents in `backend/doc/`.

---
> Source: [Madusha03/Shanthi-Family-Care](https://github.com/Madusha03/Shanthi-Family-Care) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-22 -->
