# Copilot Instructions for Manage-Learners

## Project Overview
This is a simple product catalog management system (despite the repo name "Manage-Learners"). Built with vanilla PHP, jQuery, and SQLite, it follows a classic AJAX-driven architecture without a framework.

## Architecture & Data Flow

**Frontend → Backend Flow:**
- `index.php` (catalog view) ← → `fetch_products.php` (read)
- `admin.php` (add form) → `add_product.php` (create)
- jQuery in `app.js` → `delete_product.php` (delete)

**Database:** SQLite (`mini_shop.db`) with two tables:
- `products`: id, name, price, category_id
- `categories`: id, name (hardcoded IDs: 1=Electronics, 2=Books, 3=Clothing)

All PHP endpoints connect via `php/db.php` which creates a PDO connection to `mini_shop.db` in parent directory.

## Key Patterns & Conventions

### Database Access Pattern
All PHP endpoints follow this structure:
```php
include "db.php";  // Shared PDO connection in $conn variable
// Use prepared statements exclusively
$stmt = $conn->prepare("SELECT...");
$stmt->execute([intval($id)]);  // Always cast integers
```

### Frontend-Backend Communication
- **jQuery AJAX**: All data operations use `$.get()` or `$.post()` - NO fetch API
- **Response format**: Plain HTML strings (not JSON) for product lists
- **Error handling**: Backend uses `die()` with error messages, frontend uses `alert()`

### Inline Event Handlers
Forms use inline jQuery submit handlers directly in HTML files (see `admin.php`), not in `app.js`. The `app.js` file handles only catalog page interactions.

### CSS Structure
Single monolithic `style.css` with:
- Purple gradient theme (`#667eea` to `#764ba2`) for primary actions
- CSS Grid for product layout with auto-fill responsive columns
- No CSS frameworks or preprocessors

## Development Workflow

**Local Server:** Run `php -S 0.0.0.0:8000 -t /workspaces/Manage-Learners` from project root

**Database Access:** Use `sqlite3 mini_shop.db` from project root to inspect data

**No Build Step:** Direct file editing - refresh browser to see changes

## Critical Constraints

1. **Categories are hardcoded** in three places: `index.php` dropdown, `admin.php` dropdown, and product fetch logic. Adding categories requires updating all three locations manually.

2. **Edit functionality is stubbed**: Delete button works, but edit button only shows an alert placeholder.

3. **No input validation beyond browser `required` attributes** - backend accepts all POST data directly.

4. **Database path quirk**: `db.php` uses `__DIR__ . "/../mini_shop.db"` to reference parent directory from `php/` folder.

5. **Response format inconsistency**: Product listing returns HTML strings, but add/delete return plain text messages.

## When Adding Features

- **New endpoints**: Always include `db.php`, use prepared statements, return simple text/HTML
- **New UI elements**: Add jQuery event handlers using `$(document).on()` for dynamic content
- **Styling**: Extend the purple gradient theme and maintain responsive grid patterns
- **Security**: Implement proper input validation and escape output (currently lacking)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rohanpawar99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
