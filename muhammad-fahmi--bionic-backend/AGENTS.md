# Bionic Backend AI Coding Guidelines

## Architecture Overview

This is a **CodeIgniter 4** task management and shift-tracking application with three user roles:
- **Admin**: Dashboard, manage users, locations, items, and actions
- **Operator**: Submit task completions and view assignments
- **Verifikator**: Verification role (work in progress)

**Core Data Model**: Locations contain Items; Items require Actions (checklist); Operators submit TaskSubmissions tracking action completion.

## Project Structure

```
app/
  Controllers/          # Route handlers (Auth, Admin/*, Operator, Verifikator)
  Models/               # CodeIgniter Models extending Model class (m_* = master, r_* = relational)
  Database/Migrations/  # Timestamped migrations (migrations create tables, applied via spark migrate)
  Database/Seeds/       # Data seeding (e.g., ActionSeeder)
  Config/              # Routes.php defines URL groups, Database.php configures connections
  Filters/             # MyCors for CORS handling
  Libraries/           # JwtService for EdDSA JWT handling (encode/decode)
  Commands/            # CLI commands: shift:initialize, status:clean, shift:rotate
  Views/               # Not in backend—frontend is separate
```

## Key Patterns & Conventions

### Models & Database
- **Naming**: Master tables: `m_` prefix (m_users, m_locations, m_items, m_actions); relational: `r_` prefix (r_task_submission, r_task_submission_detail)
- **Model Definition**: Each model declares `$table`, `$primaryKey`, `$allowedFields`, `$returnType = 'array'`
- **Migrations**: Timestamped (e.g., 2026-01-04-112511_Users.php); run via `php spark migrate`
- **Seeds**: Run via `php spark db:seed ActionSeeder`

### Authentication & Authorization
- **JWT Service** (`JwtService`): EdDSA signing; stores JWT and public key in session
- **Session-based**: Tokens stored in `$_SESSION['jwt']` and `$_SESSION['key']`
- **Login Flow**: `Auth::login()` checks session JWT expiration; routes to `/admin`, `/operator`, or `/verifikator` based on role
- **Validation Rules**: Inline in controllers with custom error messages in Indonesian

### Controllers
- **BaseController**: All controllers extend it; initializes `$this->jwt = new JwtService()`
- **Routes**: Grouped by function (auth, admin/manage/*, operator); use RESTful conventions (GET, POST, PUT, DELETE)
- **Admin Routes**: Nested structure for managing users, locations/tasks/items/actions
- **Response Pattern**: Controllers return view() or redirect(); datatable endpoints expected (e.g., `get_datatable_location`)

### CORS & Filters
- **MyCors Filter**: Handles preflight OPTIONS requests; currently allows all origins (commented allowlist for localhost:3000, localhost:5173, production domain)
- **Applied via Config/Routes**: Use `$routes->group(...)` with filters if needed

### CLI Commands
- **shift:initialize**: Set up operator shift assignments (supports `--dry-run`, `--force`)
- **status:clean**: Likely cleanup task
- **shift:rotate**: Rotate shift assignments
- **Pattern**: Extend `BaseCommand`; use CLI::write() for output

## Development Workflows

### Database
```bash
# Create migration: php spark make:migration create_table_name
# Run migrations: php spark migrate
# Rollback: php spark migrate --rollback
# Refresh: php spark migrate:refresh
# Seed: php spark db:seed SeedName
```

### Testing
```bash
# Run PHPUnit: ./vendor/bin/phpunit
# Coverage: ./vendor/bin/phpunit --coverage-html=build/logs/html/
# Config: phpunit.xml.dist (covers app/, logs to build/logs/)
```

### Local Dev
- **Base URL**: http://localhost:8080/ (configured in Config/App.php)
- **Database**: MySQLi driver; configure app/Config/Database.php
- **Public Root**: public/index.php (web root should point here, not project root)

## Important Implementation Details

### JWT Handling
- Uses Firebase\JWT library with EdDSA (EdDSA requires sodium extension)
- Keys generated per session via sodium_crypto_sign_keypair()
- Payload must include `expire_time` (Unix timestamp) for expiration checks
- **Gotcha**: Tokens tied to session; losing session = losing key to decode

### Field Validation & Localization
- Validation rules declared inline in controller methods
- Error messages in Indonesian (e.g., 'harus diisi' = must be filled)
- Use `service('validation')` to access validator

### Data Types
- Models return arrays (not objects) via `$returnType = 'array'`
- Migrations use standard CI4 types (integer, string, datetime, etc.)
- Table prefix empty by default in Config/Database.php

## Common Pitfalls & Solutions

1. **JWT Expiration**: Always check `time() > $jwt['expire_time']` after decode
2. **Role Mapping**: JWT stores 'administrator', 'operator', 'verifikator' but routes use 'admin' directory—map in controller
3. **Protected Fields**: Models have `$protectFields = true`; only fields in `$allowedFields` can be mass-assigned
4. **CORS Origins**: Currently permissive; update MyCors if restricting to frontend domains
5. **Missing Database Config**: Database credentials must be set in app/Config/Database.php (hostname, username, password, database)

## Useful Commands & Tips

- **Spark**: CodeIgniter CLI runner (`php spark <command>`)
- **View Structure**: Backend has no Views; frontend is separate (likely Vue/React at localhost:3000 or :5173)
- **Logging**: Configured in Config/Logger.php; logs go to writable/logs/
- **Debugging**: enable DBDebug = true in Database.php for SQL debug output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammad-fahmi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/muhammad-fahmi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
