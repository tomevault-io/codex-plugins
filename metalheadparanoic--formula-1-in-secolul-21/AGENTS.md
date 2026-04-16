# Copilot Instructions - F1 Century 21 Web Application

## Project Overview
A dockerized PHP/MySQL web application showcasing Formula 1 history, drivers, teams, and circuits. Runs on Apache (port 8888) with MySQL database. Authentication-based access to exclusive Paddock Club content.

## Architecture Patterns

### Database Structure
- **Relational design** with cascading deletes: `history_seasons` (parent) → `history_races`, `history_standings` (children)
- **Users table**: Stores login credentials with hashed passwords (`password_hash()`, `PASSWORD_DEFAULT`)
- Connection via `db.php` using PDO with exception handling; credentials defined in `docker-compose.yml`
- Example: `FOREIGN KEY (year) REFERENCES history_seasons(year) ON DELETE CASCADE`

### Session & Authentication
- Sessions start with `<?php session_start(); ?>` at top of every page
- User context stored in `$_SESSION['user_id']` and `$_SESSION['username']`
- Admin-only pages check: `if ($_SESSION['username'] !== 'admin') { header("Location: index.php"); }`
- Default admin credentials: username=`admin`, password=`f12025` (set in `setup_db.php`)

### Page Template Pattern
All pages follow: `session_start()` → HTML head with stylesheets → `include 'header.php'` → `<main>` content
- Header shows conditional nav (login/logout buttons based on session state)
- Stylesheets: `stil_general.css` (global), then page-specific CSS (`style_piloti.css`, `style_calendar.css`, etc.)
- Conditional premium features visible only when logged in (e.g., Paddock Club link in header)

## Key Conventions

1. **Romanian Content**: All UI text is in Romanian; maintain language consistency in new content
2. **Color Scheme**: Team colors used for styling cards (e.g., `.team-mclaren`, `.team-ferrari`). F1 red (#ff0000), gold (#FFD700)
3. **Data Structure**: 2025 season as current; circuit/driver names match real F1 grid
4. **Security**: Use prepared statements (`$pdo->prepare()`), `htmlspecialchars()` for output, never echo user input directly
5. **Image Format**: AVIF format for circuit maps; JPG/WebP for drivers (in `imagini/` folder)

## Developer Workflows

### Local Development
```bash
docker-compose up -d  # Start Apache + MySQL
# Access: http://localhost:8888
# Run setup: http://localhost:8888/setup_db.php (once, creates tables & admin)
```

### Database Management
- `setup_db.php`: Initialize database structure and admin user
- `test_db.php`: Debug database connections
- MySQL accessible on port 3307 (local) with user `roacer`/password `roacer9053`

### Common Task Patterns
- **Add new page**: Create `new_page.php`, include header/footer, add session_start()
- **Add database field**: Alter table in `setup_db.php`, test with `test_db.php`
- **Style new content**: Create `style_newpage.css`, link in page head

## File Organization
- `studenti/`: All application files (Apache serves this directory)
- `db.php`: Single DB connection point; all queries use this `$pdo` object
- `header.php` / `footer.php`: Shared navigation and layout components
- `admin_users.php`: Example of admin-gated page pattern
- `imagini/`: All images (circuits, drivers, team logos)

## Integration Points
- No external APIs; all data static in database
- No frontend build tools; vanilla CSS and JavaScript (`script.js`)
- Docker: Compose orchestrates Apache ↔ MySQL communication via service name `mysql` (not localhost)

## Critical Files to Understand
- [db.php](studenti/db.php) - Connection configuration; start here for database issues
- [setup_db.php](studenti/setup_db.php) - Schema documentation with FK relationships
- [header.php](studenti/header.php) - Auth-aware navigation; shows conditional UI based on session
- [admin_users.php](studenti/admin_users.php) - Example of security checks and admin-only pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/metalheadparanoic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
