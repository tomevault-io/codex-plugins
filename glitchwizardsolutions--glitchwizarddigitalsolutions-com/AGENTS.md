# GlitchWizard Digital Solutions - AI Coding Guide

## Project Context

This is a **legacy modernization and cleanup project** for a production XAMPP-based PHP monolith. The codebase was migrated from production to:
1. **Redesign email infrastructure** - Replace current email sending mechanisms
2. **Clean up unused code** - Identify and document/remove unused files and applications
3. **Create development specs** - Document all components for future development and troubleshooting
4. **Establish version control** - Initialize Git repository at https://github.com/GlitchWizardSolutions/glitchwizarddigitalsolutions.com.git

**AI Agent Role:** Help identify unused code, document existing functionality, plan refactoring, and implement improvements while preserving working production features.

**Version Control Note:** Only the `public/` directory is in Git. The `private/` directory (containing config.php with credentials) is NOT versioned and must be set up separately in each environment.

## Architecture Overview

This is a **legacy XAMPP-based PHP monolith** serving as a multi-tenant business management portal with 12+ separate MySQL databases. The system is structured as a collection of semi-independent subsystems under a shared authentication layer.

**Known Issues:**
- Many files present but not in use or not working correctly
- Email sending mechanisms need redesign
- Lack of comprehensive documentation
- **Path resolution differences between dev and production** - local development paths don't match production paths, causing index.php and other files to fail locally

**Critical Development Constraint:**
The application works correctly in **production** but path resolution fails in **development** due to environment differences. When fixing path issues:
- ✅ USE configuration constants (defined in `private/config.php`)
- ❌ NEVER hardcode paths specific to dev environment
- ✅ ADD new constants to `private/config.php` if needed
- ❌ NEVER use `__DIR__`, `dirname(__FILE__)`, or relative paths without validation
- The goal: Make paths work in BOTH dev and production through configuration, not code changes

### Key Directories
- `private/` - Configuration files with database credentials, paths, and API keys (NEVER commit changes here)
- `public/` - Web-accessible root; contains member login, registration, and public pages
- `public/admin/` - Admin dashboard with subsystem modules (tickets, invoices, budget, projects, newsletter, blog)
- `public/client-dashboard/` - Client-facing portal for authenticated members
- `public/client-invoices/` - Payment processing and invoice viewing
- `AI-DEV/` - **AI workspace for development artifacts** (NOT in production)
  - Use for: test files, experiments, specifications, documentation, analysis
  - All AI-generated non-production files MUST go here
  - This directory does NOT exist in production
  - Files here are committed to Git for collaboration but never deployed

### Database Architecture
The system uses **5 MySQL databases** (all with same credentials):
- `glitchwizarddigi_login_db` (db_name) - Main database with accounts, invoices, clients, domains, tickets, etc.
- `glitchwizarddigi_onthego` (db_name2) - On-the-go task management
- `glitchwizarddigi_budget_2025` (db_name7) - Budget tracking system
- `glitchwizarddigi_error_handling` (db_name9) - Error logging and monitoring
- `glitchwizarddigi_envato_blog_db` (db_name12) - Blog and newsletter system
- See `private/config.php` for complete list

**Connection Pattern:** Each subsystem creates its own PDO connection to specific databases. Main connections initialized in:
- `public/assets/includes/main.php` - Login DB
- `public/admin/assets/includes/main.php` - Admin systems
- `public/client-dashboard/assets/includes/main.php` - Multiple DBs (login, budget, blog)

## Configuration System

### Config File Hierarchy
The system uses a **layered configuration approach**:

1. **Base:** `private/config.php` - Master config with 12 databases, paths, feature flags, payment gateways, SMTP settings
2. **Bootstrap files** (include config + utilities):
   - `public/assets/includes/public-config.php` - Public pages (login, registration)
   - `public/assets/includes/process-config.php` - Form processing (minimal, no page setup)
   - `public/admin/assets/includes/admin_config.php` - Admin pages
   - Each subsystem has its own `admin_config.php` (e.g., `admin/ticket_system/assets/includes/admin_config.php`)

**Critical Pattern:** Always use the appropriate bootstrap:
```php
// For public pages with full UI
include 'assets/includes/public-config.php';

// For AJAX/form handlers (no page wrapper)
include 'assets/includes/process-config.php';

// For admin pages (requires Admin role)
require 'assets/includes/admin_config.php';
```

### Path Constants (from `private/config.php`)

**CRITICAL:** The application has environment-specific path differences. Production works correctly. Development may have path resolution failures. **ALWAYS use these constants - NEVER hardcode paths.**

```php
// Base paths - calculated from config file location
private_path          // Directory containing config.php
project_path          // Parent of private/ directory
public_path           // project_path . '/public/' (or '/public_html/' in production)
hidden_path           // private_path . '/'

// Application paths - all derived from public_path
admin_path            // public_path . 'admin/'
dashboard_path        // public_path . 'client-dashboard/'
admin_includes_path   // admin_path . 'assets/includes/'
includes_path         // dashboard_path . 'assets/includes/'
process_path          // includes_path . 'process/'
communication_path    // dashboard_path . 'communication/'

// Subsystem paths - for specific applications
budget_directory_path      // admin_path . 'budget_system'
invoice_directory_path     // admin_path . 'invoice_system'
project_tickets_directory_path  // admin_path . 'project_system'

// URL paths - for links and redirects (environment-specific)
base_url              // 'https://glitchwizarddigitalsolutions.com/' (production)
                      // 'http://localhost/digitalsolutions-com/public/' (development)
budget_link_path      // base_url . 'admin/budget_system/'
invoice_link_path     // base_url . 'admin/invoice_system/'
```

**Path Resolution Rules:**

1. **Filesystem includes** - Use path constants:
   ```php
   ✅ include admin_includes_path . 'main.php';
   ✅ require private_path . '/config.php';
   ❌ include '../admin/assets/includes/main.php';  // Will break in one environment
   ```

2. **URLs/Redirects** - Use URL constants:
   ```php
   ✅ header('Location: ' . base_url . 'login.php');
   ❌ header('Location: /login.php');  // Fails in dev with subdirectories
   ```

3. **Relative references in config bootstraps**:
   ```php
   ✅ require_once '../private/config.php';  // OK - only in specific bootstrap files
   ✅ require_once '../../private/config.php';  // OK - for nested directories
   ```

4. **Adding new paths** - Update `private/config.php`:
   ```php
   // Add to config.php, don't hardcode in application files
   if(!defined('new_feature_path')) define('new_feature_path', admin_path . 'new_feature/');
   ```

**Important:** All bootstrap config files reference `../private/config.php` or `../../private/config.php` - the private directory is outside the Git repository.

**Production vs Development:**
- Production: Web root is likely `/public_html/` or direct domain root
- Development: Web root is `c:\xampp\htdocs\digitalsolutions-com\public\`
- The `public_path` constant must be set correctly in each environment's `private/config.php`
- **NEVER change code to fix dev paths** - Fix the config constant instead

## Authentication & Session Management

### Login Flow
1. User submits credentials to `authenticate.php`
2. Checks `login_attempts` table (IP-based rate limiting: 1 day lockout after failed attempts)
3. CSRF token validation: `$_POST['token']` must match `$_SESSION['token']`
4. Password verification with `password_verify()`
5. Account status checks:
   - `activation_code` must be 'activated' (email activation system)
   - `approved` must be true if `account_approval` enabled
   - IP mismatch triggers two-factor auth redirect (`twofactor.php`)
6. Session variables set:
   ```php
   $_SESSION['loggedin'] = TRUE;
   $_SESSION['id'], $_SESSION['role'], $_SESSION['access_level']
   $_SESSION['email'], $_SESSION['full_name'], $_SESSION['document_path']
   ```
7. Optional "Remember Me": 60-day cookie with hashed token stored in `accounts.rememberme`

### Role-Based Access
- **Admin** - Full system access (checked in `public/admin/assets/includes/main.php` line 20)
- **Member** - Client dashboard access
- **Manager** - Elevated member privileges
- Roles stored in `accounts.role`, access levels in `accounts.access_level` ('Admin', 'Guest', 'Onboarding', 'Development', 'Production', 'Hosting', 'Services', 'Master', 'Closed', 'Banned')

### Session Checking Pattern
```php
// In public/assets/includes/main.php
check_loggedin($pdo, $redirect_file = site_menu_base . 'index.php');
// Auto-checks rememberme cookie, updates last_seen timestamp
```

## Subsystem Structure

Each admin subsystem follows this pattern:
```
public/admin/{subsystem_name}/
├── assets/
│   └── includes/
│       ├── admin_config.php      // Requires ../../private/config.php
│       └── main.php               // Templates and utilities
├── index.php                      // Dashboard/list view
├── {entity}.php                   // Detail view (e.g., ticket.php, invoice.php)
└── {entity}s.php                  // Plural list (tickets.php, invoices.php)
```

### Major Subsystems
- **ticket_system** - Support tickets (db: login_db, tables: tickets, tickets_comments, tickets_uploads)
- **project_system** - Project-specific tickets (separate tables: project_tickets, project_tickets_comments)
- **invoice_system** - Client billing (db: login_db, uses FPDF, PayPal)
- **budget_system** - Financial tracking (db: db_name7)
- **newsletter_system** - Email campaigns (tables: subscribers, campaigns, newsletters)
- **blog** - Blog and newsletter system (db: db_name12)
- **gws_legal_system** - Legal document management

**Dual Ticketing Systems:** Standard tickets and project tickets are completely separate codebases with parallel table structures. Project system prefixes everything with `project_*`.

## Code Conventions

### File Organization
- **Entry points:** `index.php` in each directory
- **Processing:** Separate `*-process.php` files for form handling
- **Templates:** Functions `template_admin_header()` and `template_admin_footer()` in `main.php`
- **Includes:** Use absolute paths via constants (e.g., `admin_includes_path . 'main.php'`)

### Database Patterns
```php
// Standard PDO prepared statement pattern used throughout
$stmt = $pdo->prepare('SELECT * FROM table WHERE id = ?');
$stmt->execute([$id]);
$result = $stmt->fetch(PDO::FETCH_ASSOC);
// For lists
$results = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

### Naming Conventions
- **Files:** Lowercase with hyphens (`admin-config.php`, `ticket-uploads/`)
- **Databases:** Snake_case with prefix (`glitchwizarddigi_*`)
- **Tables:** Snake_case (`tickets_comments`, `invoice_items`)
- **Constants:** UPPERCASE with underscores (`db_name`, `admin_path`, `SECRET_KEY`)
- **Functions:** Snake_case (`check_loggedin`, `pdo_connect_mysql`)
- **Variables:** Snake_case (`$login_attempts`, `$account`)

### Feature Flags (from config.php)
```php
account_activation          // Require email activation (TRUE)
account_approval           // Admin approval required (FALSE)
auto_login_after_register  // Skip login after registration (FALSE)
mail_enabled               // Enable email notifications (TRUE)
SMTP                       // Use SMTP vs mail() (TRUE)
paypal_enabled             // Accept PayPal (TRUE)
paypal_testmode            // Use sandbox (TRUE)
```

## Payment Integration

### PayPal (Enabled, Sandbox Mode)
- IPN handler: `public/client-invoices/ipn.php?method=paypal`
- Business email: `clarityconnect@glitchwizardsolutions.com`
- Return URL: `https://glitchwizarddigitalsolutions.com/`

### Stripe (Disabled)
- Keys in config.php, IPN endpoint defined but unused

**Payment Flow:** Invoice system in `public/admin/invoice_system/` creates invoices, client views at `client-invoices/`, payment processed via IPN handlers.

## Email System

### SMTP Configuration
```php
smtp_host: mail.glitchwizarddigitalsolutions.com
smtp_port: 465 (SSL)
smtp_user: no_reply@glitchwizarddigitalsolutions.com
```

### Email Templates
Located in `public/`:
- `activation-email-template.php` - Account activation
- `resetpass-email-template.php` - Password reset
- `email_template_twofactor.php` - 2FA codes
- `custom-email-template.php` - General notifications

### Notification Pattern
```php
if (mail_enabled) {
    // PHPMailer or mail() based on SMTP constant
    // From: mail_from or newsletter_mail_from
    // Reply-To: notify_admin_email
}
```

## Common Development Workflows

### Adding a New Feature to Admin
1. Create directory under `public/admin/{feature_name}/`
2. Create `assets/includes/admin_config.php`:
   ```php
   <?php
   session_start();
   require('../../private/config.php');
   include admin_includes_path . 'main.php';
   include admin_includes_path . 'admin_page_setup.php';
   ?>
   ```
3. Add database connection if needed (see `main.php` for PDO patterns)
4. Use `template_admin_header($title, $selected)` and `template_admin_footer()`
5. Add navigation link in `public/admin/assets/includes/main.php` (around line 50+)

### Adding Database Functionality
1. Define new database in `private/config.php` as `db_name{N}`
2. Create connection in appropriate `main.php`:
   ```php
   $pdo_new = new PDO('mysql:host=' . db_host . ';dbname=' . db_name13 . ';charset=' . db_charset, db_user, db_pass);
   $pdo_new->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
   ```
3. Use prepared statements exclusively (never concatenate SQL)

### File Upload Handling
Standard pattern (see ticket systems):
```php
define('uploads_directory', 'ticket-uploads/'); // in config.php
$upload_path = uploads_directory . $filename;
// Check file extension against attachments_allowed constant
// Respect max_allowed_upload_file_size (default 5MB)
```

### AJAX Patterns
```php
// Enable AJAX updates in config
if (ajax_updates) {
    // Refresh every ajax_interval milliseconds (default 10000)
    // Return plain text responses (not JSON)
    // Check for "Success:" or "Error:" prefix in responses
}
```

## Security Considerations

### Sensitive Data
- **Credentials in private/config.php** - Database passwords, API keys, SMTP credentials
- **SECRET_KEY** - Used for hashing (`rockyflower493`)
- **Never expose** private/ directory contents or commit credential changes

### CSRF Protection
All forms require:
```php
$_SESSION['token'] = md5(uniqid(rand(), true)); // Generate
<input type="hidden" name="token" value="<?=$_SESSION['token']?>"> // Use in form
if ($_POST['token'] != $_SESSION['token']) exit('Token expired'); // Validate
```

### IP-Based Rate Limiting
`login_attempts` table tracks failed logins by IP, locks out for 24 hours after limit reached.

### File Upload Validation
```php
$allowed = explode(',', attachments_allowed); // 'png,jpg,jpeg,gif,webp,bmp,doc,docx,xls,xlsx,ppt,pptx,pdf,zip,rar,txt,csv'
// Validate extension before moving uploaded file
```

## Debugging & Logs

### Error Reporting
Set in main.php files:
```php
ini_set('display_errors', 1);
error_reporting(E_ALL);
```

### Error Logging
```php
error_log('Message'); // Writes to error_log file in current directory
```

### Common Issues
- **Session not set:** Check if bootstrap config file is included (public-config.php, admin_config.php)
- **Database connection fails:** Verify database constant (db_name, db_name2, etc.) matches intended database
- **Path issues:** Use constants (admin_path, includes_path) not relative paths
- **Admin access denied:** Verify `accounts.role = 'Admin'` in database

## Testing Approach

This is a **production XAMPP environment** - no test framework exists. Manual testing workflow:
1. Test in browser at `https://glitchwizarddigitalsolutions.com/{path}`
2. Check `error_log` files for PHP errors
3. Use browser DevTools for JavaScript/AJAX debugging
4. Database changes: Use phpMyAdmin or MySQL command line
5. **Always backup database before schema changes**

## Key Integration Points

### Google OAuth
- Client ID/Secret in config.php
- Handler: `public/google-oauth.php`
- Creates account or logs in existing user

### Blog Integration
Three separate blog databases integrated with main login:
```php
$_SESSION['bloggedin'] = TRUE; // Set on main login
// Blog comment system checks this session variable
```

### Client Document System
- Path per user: `accounts.document_path` (e.g., `system-client-access_files/Barbara_Moore/`)
- SECRET_KEY and VERIFY_TOKEN used for access control

### Cron Jobs
- Endpoint: `public/admin/cron.php`
- Secret: `cron_secret` constant ('rockyflower493')
- Processes newsletter campaigns, scheduled tasks
- Rate: `cron_mails_per_request` emails per cycle, sleep `cron_sleep_per_request` seconds

## AI-DEV Directory Usage

**MANDATORY:** All AI-generated files that are NOT production application code MUST be placed in the `AI-DEV/` directory.

### What Goes in AI-DEV/
- ✅ **Specifications** - System documentation, API specs, database schemas
- ✅ **Analysis files** - Code analysis, dependency maps, usage reports
- ✅ **Test scripts** - Testing code, proof-of-concepts, experiments
- ✅ **Planning documents** - Refactoring plans, architecture proposals
- ✅ **Migration scripts** - One-time database migrations, data transforms
- ✅ **Development notes** - Investigation findings, decision logs
- ✅ **Generated documentation** - Auto-generated from code analysis

### What Does NOT Go in AI-DEV/
- ❌ Production application code
- ❌ User data or uploads
- ❌ Configuration files (those go in `private/`)
- ❌ Fixes to existing production files (edit in place)

### AI-DEV/ Structure (Suggested)
```
AI-DEV/
├── specs/              # System specifications and documentation
├── analysis/           # Code analysis and reports
├── tests/              # Test scripts and experiments
├── plans/              # Refactoring and improvement plans
└── migrations/         # One-time database/data scripts
```

### Example Usage
```bash
# ✅ CORRECT - Analysis goes in AI-DEV
AI-DEV/analysis/email-system-audit.md
AI-DEV/specs/invoice-system-spec.md
AI-DEV/tests/test-email-sending.php

# ❌ WRONG - Don't clutter production directories
public/email-analysis.md
public/admin/test-invoice.php
```

## Project-Specific Workflows

### Identifying Unused Code
When analyzing if code is in use:
1. **Check for includes/requires** - Use grep_search to find references
2. **Database table usage** - Search for table names in SQL queries
3. **Navigation links** - Check if feature appears in admin/client menus
4. **File access logs** - Consider checking server logs for accessed files
5. **Config references** - Search for feature flags or constants

### Documenting Existing Systems
When creating specs for a subsystem:
1. **Entry points** - Document all public-facing URLs
2. **Database schema** - List tables, key columns, relationships
3. **Dependencies** - Config constants, external libraries, other subsystems
4. **User flows** - Step-by-step process descriptions
5. **Known issues** - Document bugs or incomplete features

### Email System Redesign Considerations
Current email infrastructure uses:
- PHPMailer (if `SMTP` constant is TRUE)
- Native `mail()` function as fallback
- Multiple email templates in `public/` directory
- SMTP config in `private/config.php`
- Email sending scattered across subsystems

When redesigning:
- Centralize email sending logic
- Create email service abstraction
- Standardize template system
- Improve error handling and logging
- Consider queueing for bulk operations

## CRITICAL RULES - DO NOT VIOLATE

### Path Handling - MOST IMPORTANT
- ❌ **NEVER** hardcode file paths specific to your development environment
- ❌ **NEVER** fix path issues by changing relative paths in code
- ❌ **NEVER** assume directory structure matches your local setup
- ✅ **ALWAYS** use path constants from `private/config.php`
- ✅ **ALWAYS** test that changes work in production (or document environment-specific config needed)
- ✅ **ALWAYS** add new path constants to config instead of hardcoding paths

**Example of what NOT to do:**
```php
❌ include 'C:\xampp\htdocs\digitalsolutions-com\private\config.php';  // Dev-specific
❌ include '/home/user/www/private/config.php';  // Also dev-specific
❌ include dirname(__FILE__) . '/../../../private/config.php';  // Brittle

✅ include private_path . '/additional_config.php';  // Uses constant
```

### File Organization
- ❌ **NEVER** create test files, specs, or experiments in production directories
- ✅ **ALWAYS** place AI-generated development artifacts in `AI-DEV/`
- ✅ **ALWAYS** organize AI-DEV/ with subdirectories (specs/, analysis/, tests/, plans/)

### Security & Configuration
- ❌ **NEVER** modify `private/config.php` database credentials without explicit instruction
- ❌ **NEVER** commit sensitive data or credentials to Git
- ❌ **NEVER** use `eval()` or `exec()` functions
- ❌ **NEVER** create new databases without adding constants to config.php

### Application Integrity
- ❌ **NEVER** break existing authentication checks
- ❌ **NEVER** remove CSRF token validation
- ❌ **NEVER** change role-checking logic in admin main.php
- ❌ **NEVER** modify session variable structure without updating all subsystems
- ❌ **NEVER** delete files without first documenting their purpose and confirming they're unused

### Production Safety
- ❌ **NEVER** assume production environment matches development
- ✅ **ALWAYS** consider: "Will this work in production where paths are different?"
- ✅ **ALWAYS** document environment-specific requirements
- ✅ **ALWAYS** use configuration-based solutions, not code-based workarounds

## Quick Reference

**Base URL:** `https://glitchwizarddigitalsolutions.com/`  
**Admin Login:** `public/admin/index.php` (requires Admin role)  
**Member Login:** `public/index.php` → redirects to `client-dashboard/`  
**SMTP Test:** Check `mail_enabled` constant, verify credentials in config.php  
**Database Prefix:** All tables use `glitchwizarddigi_` prefix in database names  
**File Uploads:** Stored in subdirectories like `ticket-uploads/`, `contact-form-uploads/`

---
> Source: [GlitchWizardSolutions/glitchwizarddigitalsolutions.com](https://github.com/GlitchWizardSolutions/glitchwizarddigitalsolutions.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-10 -->
