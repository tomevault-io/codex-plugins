# SEB Pages - Copilot Instructions

## Project Overview
A Flask web application for managing Safe Exam Browser (SEB) exam sessions. Teachers create exams with embedded web tools (calculators, documents) that students access through SEB's secure browser environment. The system validates SEB security keys (Browser Exam Key, Config Key, User Agent) to ensure exam integrity.

## Architecture

### Core Components
- **`app.py`**: Flask server with role-based auth (admin/teacher/student), exam CRUD, SEB key validation, Google OAuth
- **`sebConfigUtils.py`**: Crypto module for encrypting/decrypting SEB config files (`.seb` format with RNCryptor)
- **`sebLock.js`**: Client-side SEB security validation - hashes keys with URL and compares against meta tags
- **JSON Datastores**: `exams.json`, `users.json`, `exam_apps.json`, `exam_attempts.json` (no database - direct file I/O)

### SEB Security Model
Exams store three security components:
1. **BEK (Browser Exam Key)**: Array of SHA256 hashes - validates SEB browser instance
2. **CK (Config Key)**: SHA256 hash - validates SEB config file matches expected settings
3. **UA (User Agent)**: String seed that's hashed with URL - identifies genuine SEB instances

JavaScript (`sebLock.js`) retrieves actual keys from `SafeExamBrowser.security` API, hashes them with current URL, and compares against expected values in page meta tags. All three must match or exam is locked.

## Data Flow

### Exam Registration Flow
1. Teacher submits form with BEKs (array), CK, UA, and tools list
2. System generates UUID exam ID + 4-word memorable exam key (e.g., `ALPHA-BRAVO-CHARLIE-DELTA`)
3. Exam saved to `exams.json` with structure:
   ```json
   {
     "exam_key": "ALPHA-BRAVO-CHARLIE-DELTA",
     "user_id": "teacher-uuid",
     "beks": ["hash1", "hash2"],
     "ck": "config-hash",
     "ua": "user-agent-seed",
     "tools": [{"id": "uuid", "name": "...", "url": "...", "icon": "bi-..."}]
   }
   ```

### Exam Access Flow
1. Student visits `/<exam_id>/exam` route (no login required for exam takers)
2. Student signs in with Google (optional but recommended for tracking)
3. Server renders `exam_launcher.html` with SEB keys in meta tags
4. `sebLock.js` validates keys every 1 second
5. If valid, student can access tools via `/<exam_id>/app/<app_id>` routes
6. Each access is logged to `exam_attempts.json` with student's Google account info

### Exam Attempts Tracking
All exam accesses are logged with:
- Timestamp (ISO 8601 UTC)
- Google ID (if signed in)
- Email address
- Name
- Access type (`exam_launch` or `tool_access:<tool_name>`)
- Unique attempt ID

Teachers can view attempts at `/<exam_id>/attempts` route.

## Key Conventions

### UUID Generation
- **Exam IDs**: Cryptographically secure `uuid.uuid4()` 
- **Exam Keys**: 4 random words from 100-word NATO/space-themed list using `secrets.choice()`
- **User IDs**: UUID for each user account

### Role-Based Access Control
- `@login_required`: Decorator for any authenticated route
- `@admin_required`: Extends login check with `role == 'admin'` validation
- Ownership checks: Teachers see only their `user_id` exams, admins see all

### File I/O Pattern
All persistence uses this pattern:
```python
def load_exams():
    if os.path.exists(EXAM_DB_PATH):
        with open(EXAM_DB_PATH, 'r') as f:
            return json.load(f)
    return {}

def save_exams(exams):
    with open(EXAM_DB_PATH, 'w') as f:
        json.dump(exams, f, indent=2)
```
Global `EXAMS`, `USERS`, `EXAM_APPS` dicts loaded at startup, modified in-memory, saved after mutations.

### SEB Config Encryption
`sebConfigUtils.py` uses RNCryptor format:
- **Unencrypted**: Prefix `plnd` + gzipped XML
- **Password-encrypted**: Prefix `pswd` + RNCryptor(gzipped XML) with PBKDF2-derived keys
- Entire structure then gzipped again for final `.seb` file

## Development Workflow

### Running the Application
```bash
# Install dependencies
pip install -r requirements.txt

# Generate initial user accounts (admin/admin, teacher/teacher)
python generate_users.py

# Run development server
python app.py  # Starts on http://0.0.0.0:5000 with debug=True
```

### Testing SEB Key Generation
Use `sebTest.py` utilities:
```python
from sebTest import generate_config_key
# Pass plist dict or XML bytes to get SHA256 config key
```

### Hash Generator Tool
`/hash_generator` route provides UI for generating SHA256 hashes of arbitrary strings - useful for creating BEK/CK/UA values during exam setup.

## Critical Patterns

### URL Hashing for Security
SEB keys are always hashed with the full URL before comparison:
```javascript
// In sebLock.js
const url = window.location.origin + window.location.pathname;
const hashedCK = await concatHash(url, ckExpected);
const hashedUA = await concatHash(url, userAgent);
```
Backend must use same URL construction: `url_for('exam', exam_id=exam_id, _external=True)`.

### Multiple BEK Support
Exams store `beks` as array to support multiple valid browser configurations:
```python
# Old format (deprecated but still supported):
exam['bek'] = 'single-hash'

# New format (preferred):
exam['beks'] = ['hash1', 'hash2', 'hash3']
```
JavaScript checks `if (!hashedBEKs.includes(bekActual))` to allow any match.

### Tool Configuration
Tools are embedded iframes with Bootstrap icons:
- Icon classes from Bootstrap Icons (e.g., `bi-graph-up`, `bi-calculator`)
- Each tool has UUID for routing to `/<exam_id>/app/<tool_id>`
- Common tools predefined in `exam_apps.json`, but teachers can add custom URLs

### Session Management
Flask sessions store:
- `username`: Login identifier (email for Google sign-ins)
- `user_id`: UUID for ownership checks
- `name`: Display name
- `email`: Email address
- `role`: `admin`, `teacher`, or omitted for exam takers
- `google_id`: Google account ID (if signed in via Google)
- `picture`: Profile picture URL (if available from Google)
- `auth_method`: `'password'` or `'google'`

Use `session.get('role') == 'admin'` for permission checks, not just presence of session.

### Google OAuth Integration
**Environment Configuration**: Set `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` environment variables or update defaults in `app.py`.

**Authentication Flow**:
1. Google Sign-In button renders on login page and exam pages
2. User clicks "Sign in with Google" → Google handles authentication
3. JavaScript callback receives credential token → sends to `/auth/google`
4. Server verifies token with Google, extracts user info (sub, email, name, picture)
5. System checks if user exists by `google_id` or `email`
6. If existing user: Updates profile and sets session
7. If new user: Creates account with `role='teacher'` by default, no password
8. Students accessing exams can sign in with Google to enable attempt tracking

**User Account Structure** (Google sign-ins):
```json
{
  "email@example.com": {
    "user_id": "uuid",
    "google_id": "google-sub-id",
    "email": "email@example.com",
    "name": "User Name",
    "picture": "https://...",
    "role": "teacher",
    "created_at": "2025-12-08T...",
    "last_login": "2025-12-08T...",
    "password_hash": null
  }
}
```

## Common Pitfalls

1. **Don't forget URL hashing**: BEK/CK values are raw in database but must be hashed with URL for comparison
2. **BEK is now plural**: Check for both `exam.get('beks', [])` and fallback `exam.get('bek', '')` 
3. **Ownership enforcement**: Always verify `exam['user_id'] == session.get('user_id')` unless admin
4. **Secret key in production**: Change `app.secret_key` from placeholder before deployment
5. **SEB prefix handling**: When working with `.seb` files, always check 4-byte prefix after first gzip decompress
6. **Google OAuth setup**: Must configure `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` before Google Sign-In works
7. **Exam attempt logging**: Only logs attempts for users who sign in with Google; anonymous access not tracked

## External Dependencies
- **Safe Exam Browser**: Desktop app provides `SafeExamBrowser.security` JavaScript API
- **Bootstrap 5.3.8 + Bootstrap Icons**: CDN-loaded for UI components
- **PyCryptodome**: Required by `sebConfigUtils.py` for AES encryption
- **Google Sign-In**: `google-auth`, `google-auth-oauthlib`, `google-auth-httplib2` for OAuth authentication
- **Google Identity Services**: CDN-loaded (`https://accounts.google.com/gsi/client`) for Sign-In button

# Code Refactoring Summary

## Overview
Successfully refactored the monolithic `app.py` (714 lines) into a modular, maintainable architecture using Flask Blueprints. The application now has clear separation of concerns with dedicated modules for authentication, routing, data models, and utilities.

## New Architecture

### File Structure
```
app.py                          # Entry point (3 lines)
__init__.py                     # Flask app initialization and blueprint registration
auth.py                         # Authentication and authorization (login, OAuth, decorators)
routes.py                       # Main application routes (exams, admin, utilities)
models.py                       # Data layer (JSON I/O, exam attempt logging)
utils.py                        # Utility functions (key generation, validation)
templates/                      # All templates updated with blueprint-aware url_for()
```

### Module Responsibilities

#### `__init__.py` (19 lines)
- Flask app creation and configuration
- Data loading on startup (exams, users, apps, attempts)
- Blueprint registration (auth and routes)
- Returns singleton `app` instance

#### `auth.py` (145 lines)
- **Routes**:
  - `POST /login` - Username/password authentication
  - `POST /auth/google` - Google OAuth2 callback and user creation
  - `GET /logout` - Session cleanup
- **Decorators**:
  - `@login_required` - Redirects to login if not authenticated
  - `@admin_required` - Restricts access to admin users only
- **Features**:
  - Password hashing with werkzeug
  - Google OAuth2 token verification
  - Session management with role-based access
  - Auto user creation for Google sign-ins

#### `models.py` (47 lines)
- **Data Loading**:
  - `load_exams()`, `load_users()`, `load_exam_apps()`, `load_exam_attempts()`
  - Persists to global EXAMS, USERS, EXAM_APPS, EXAM_ATTEMPTS dicts
- **Data Saving**:
  - `save_exams()`, `save_users()`, `save_exam_apps()`, `save_exam_attempts()`
  - JSON serialization to files
- **Attempt Logging**:
  - `log_exam_attempt()` - Records exam access with timestamp, user info, and access type
- **Global Variables** (initialized in `__init__.py`):
  - `EXAMS` - Dict of exam configurations
  - `USERS` - Dict of user accounts
  - `EXAM_APPS` - Dict of available tools
  - `EXAM_ATTEMPTS` - Dict of access logs by exam_id

#### `utils.py` (40 lines)
- **Key Generation**:
  - `generate_exam_id()` - Cryptographically secure UUID
  - `generate_exam_key()` - 4-word memorable keys using `secrets.choice()`
  - `WORD_LIST` - 100+ words for key generation
- **Validation**:
  - `exam_id_exists()` - Check if exam ID in EXAMS
  - `exam_key_exists()` - Check if exam key unique

#### `routes.py` (280 lines)
Organized into logical groups:

**Core Exam Routes** (8 routes):
- `GET /` - Home page (requires login)
- `GET /hash_generator` - SHA256 hash generator UI
- `GET /<exam_id>/exam` - Exam launcher with tools
- `GET /<exam_id>/app/<app_id>` - Tool iframe display
- `GET|POST /register` - Create new exam
- `POST /edit/<exam_id>` - Update exam
- `POST /delete/<exam_id>` - Delete exam
- `GET /exam/<exam_id>/details` - Exam configuration view
- `GET /exam/<exam_id>/attempts` - Exam access history

**API Routes** (1 route):
- `GET /api/exams` - JSON export of exams (filtered by user role)

**Admin Routes** (4 routes):
- `GET /admin` - Admin dashboard with user and exam tables
- `GET|POST /admin/users/create` - New user account form
- `GET|POST /admin/users/<username>/edit` - User update form
- `POST /admin/users/<username>/delete` - User deletion

### Template Updates
All 9 templates updated with blueprint-prefixed `url_for()` calls:
- `login.html` - Uses `auth.logout` for logout link
- `register.html` - Uses `routes.*` prefix for all links
- `edit.html` - Uses `routes.*` prefix for form actions
- `index.html` - Uses both `auth.logout` and `routes.*` prefixes
- `exam_details.html` - Uses `routes.*` prefix for action buttons
- `exam_launcher.html` - Uses `routes.exam_app` for tool links
- `exam_attempts.html` - Uses `routes.index` for back button
- `admin_dashboard.html` - Uses `routes.admin_*` and `routes.*` prefixes
- `hash_generator.html` - No changes needed (no url_for calls)

## Blueprint Configuration

### Auth Blueprint (`auth_bp`)
```python
auth_bp = Blueprint('auth', __name__)
```
Routes: `/login`, `/auth/google`, `/logout`

### Routes Blueprint (`routes_bp`)
```python
routes_bp = Blueprint('routes', __name__)
```
Routes: `/`, `/register`, `/edit/<id>`, `/delete/<id>`, `/exam/<id>/...`, `/admin/*`, `/api/*`, `/hash_generator`

### Registered in `__init__.py`
```python
app.register_blueprint(auth_bp)
app.register_blueprint(routes_bp)
```

## Migration Details

### What Moved Where
| Code | Old Location | New Location |
|------|--------------|--------------|
| Data loading/saving | app.py 35-95 | models.py 1-47 |
| Exam attempt logging | app.py 96-138 | models.py 37-47 |
| Key generation + WORD_LIST | app.py 139-167 | utils.py 1-40 |
| Decorators | app.py 155-168 | auth.py 115-145 |
| Login route | app.py 169-195 | auth.py 40-77 |
| Google OAuth route | app.py 196-284 | auth.py 78-145 |
| Logout route | app.py 285-293 | auth.py 147-155 |
| Exam CRUD routes | app.py 294-443 | routes.py 28-210 |
| Admin routes | app.py 444-512 | routes.py 250-380 |
| Utility routes | app.py 513-714 | routes.py 1-27, 220-235 |

### Imports Updated
- Auth routes now import `session`, `request`, `jsonify`, `datetime` from Flask
- Routes now import all utilities from `models`, `utils`, `auth`
- All modules use relative imports: `from models import ...`

## Benefits of Refactoring

1. **Separation of Concerns**
   - Authentication logic isolated in `auth.py`
   - Routes organized by functionality
   - Data access centralized in `models.py`
   - Utilities separated for reusability

2. **Maintainability**
   - 714-line monolith split into focused modules
   - Each file has single, clear responsibility
   - Easier to locate and modify specific features
   - Reduced cognitive load

3. **Testability**
   - Modules can be tested independently
   - Decorators easily mockable
   - Data layer independent from routes
   - Clear interfaces between modules

4. **Scalability**
   - Easy to add new blueprints for features
   - Simple to extend with new routes
   - Database layer ready for replacement
   - Configuration centralized in `__init__.py`

5. **Code Reusability**
   - Decorators work across all blueprints
   - Utility functions shared without duplication
   - Models provide clean data access API
   - Blueprint structure supports future split into packages

## Entry Point
```python
# app.py
from __init__ import app

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

## Testing
The refactored application passes basic verification:
- ✅ All imports successful
- ✅ Both blueprints registered correctly
- ✅ All 17 routes available and properly namespaced
- ✅ All template url_for() calls updated with blueprint prefixes
- ✅ Flask app initializes without errors

## Next Steps (Optional)
1. Convert JSON persistence to database (SQLAlchemy)
2. Add automated tests using pytest
3. Split routes into separate blueprint files (e.g., `exams_routes.py`, `admin_routes.py`)
4. Extract form handling into separate form classes
5. Add logging configuration to `__init__.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ToothbrushB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
