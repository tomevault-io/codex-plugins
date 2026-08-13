# Copilot Instructions for Balance Cloud App

## Project Overview
**Balance Cloud App** is a Laravel 12 + Vue.js 3 financial web application for personal checkbook management. Users can track accounts, transactions, and financial data through both web interface and admin dashboard. The application uses Laravel Passport for authentication, PrimeVue for UI components, and modern tooling including Vite and TailwindCSS.

**Repository Size**: ~50MB (excluding node_modules/vendor)  
**Tech Stack**: PHP 8.2+, Laravel 12, Vue.js 3, MySQL, Node.js, Vite, TailwindCSS 4, PrimeVue 4  
**Primary Purpose**: Personal finance management with multi-user support and admin interface

## Build & Development Commands

### Environment Setup (CRITICAL - Do This First)
```bash
# Always install dependencies before any build/test operations
composer install
npm install

# Copy environment file (required for all operations)
cp .env.example .env

# Configure .env with database credentials and generate app key
php artisan key:generate

# Setup database (MySQL required)
php artisan migrate
php artisan passport:install
```

### Build Commands (Validated Working)
```bash
# Development build with hot reload
npm run dev                    # Starts Vite dev server
php artisan serve             # Starts PHP server on :8000

# Production build (always works)
npm run build                 # Takes ~4-5 seconds, generates optimized assets
npm run prod                  # Alias for build

# Combined development server
npm run serve                 # Runs both PHP server and Vite dev concurrently
```

### Testing (Known Issues & Workarounds)
```bash
# WORKING: Run Laravel tests
php artisan test              # Standard Laravel test runner
php artisan test --stop-on-failure

# ISSUE: Some tests fail with 418 status (SSL requirement)
# WORKAROUND: Tests require proper .env.testing configuration
# Ensure APP_URL=http://localhost:8000 in .env.testing

# Test database setup required:
php artisan migrate:fresh --env=testing
```

### Build Validation Steps
1. **Always run `composer install && npm install` first**
2. **Build assets**: `npm run build` (must complete without errors)
3. **Test server**: `php artisan serve` (should start on :8000)  
4. **Clear caches**: `php artisan optimize:clear` (if encountering issues)

### Common Build Errors & Solutions
- **"Mix manifest not found"**: Run `npm run build` first
- **Tests failing with 418**: Check .env.testing SSL configuration  
- **Composer deprecation warnings**: Safe to ignore (PHP 8.4 compatibility issue)
- **Vue/Vite errors**: Ensure Node.js 16+ and clean `npm install`

## Architecture & Project Layout

### Directory Structure
```
app/
├── Http/Controllers/Api/Admin/     # Admin API controllers
├── Http/Controllers/Api/           # User API controllers  
├── Models/                         # Eloquent models
├── Helpers/                        # Custom helpers (WithHelper, SearchHelper, PaginationHelper)
└── Services/                       # Business logic services

resources/
├── js/admin/                       # Vue.js admin interface
│   ├── views/                      # Vue components
│   ├── router.js                   # Vue router config
│   └── api.js                      # API client
├── css/                           # Stylesheets
└── views/                         # Blade templates

routes/
├── api.php                        # User API routes  
├── admin_api.php                  # Admin API routes
└── web.php                        # Web routes

config/                            # Laravel configuration
tests/Feature/                     # Feature tests
tests/Unit/                        # Unit tests
```

### Key Configuration Files
- **vite.config.js**: Asset bundling, includes image copying
- **.eslintrc.js**: Vue.js code standards, Airbnb base + Vue 3 rules  
- **tailwind.config.js**: CSS framework configuration
- **phpunit.xml**: Test configuration
- **composer.json**: PHP dependencies & scripts
- **package.json**: Node.js dependencies & build scripts

### Database Models & Relationships
**Core Models**: User, Account, Transaction, TransactionName, AccountType, CurrentBalance  
**Admin Models**: ContactUs, WhatsNew, UserProfile  
**Auth**: Laravel Passport OAuth clients & tokens

**Key Relationships**:
- Users have many Accounts, Transactions, TransactionNames
- Accounts belong to Users and AccountTypes  
- Transactions belong to Users and reference TransactionNames
- CurrentBalance tracks real-time account totals

### API Architecture
**User API** (`/api/`): User-owned resource management  
**Admin API** (`/api/admin/v1/`): Cross-user administration, analytics, dashboard

**Authentication**: Laravel Passport with both Bearer tokens and cookie-based auth  
**Middleware**: `auth:api`, `scope:dashboard`, `role:admin`, `log_ip`

### Frontend Architecture  
**Vue.js 3 SPA** with Composition API  
**PrimeVue 4 Components**: DataTable, Form, Card, Toast, etc.  
**Router**: Vue Router 4 with lazy loading  
**State**: Vuex 4 for auth state management  
**Styling**: TailwindCSS 4 + PrimeVue themes

## Development Guidelines

### Backend (Laravel) - CRITICAL PATTERNS

#### Helper Classes (Use These!)
```php
// Use WithHelper for relationship loading
WithHelper::build(request()->with, User::class)->find($id);

// Use SearchHelper for filtering
$query = SearchHelper::applySearch(User::class, $request, ['name', 'email']);

// Use PaginationHelper for consistent pagination  
PaginationHelper::searchAndPaginate(User::class, $request, $options);
```

#### Controller Patterns (Follow Exactly)
```php
public function show(User $user) {
    return WithHelper::build(request()->with, User::class)->find($user->id);
}

public function update(Request $request, User $user) {
    $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users,email,' . $user->id
    ]);
    
    $user->update($request->only(['name', 'email']));
    return $user->fresh();
}
```

#### Validation Rules (User Ownership Critical)
```php
// For user-owned resources, always scope uniqueness
'name' => Rule::unique('transaction_names', 'name')
    ->where('user_id', $model->user_id)
    ->ignore($model->id)
```

#### Route Organization
- Admin routes: `routes/admin_api.php` with middleware `['auth:api', 'scope:dashboard', 'role:admin']`
- User routes: `routes/api.php` with middleware `['auth:api']`
- Use resource routes with appropriate restrictions: `->only(['show', 'update'])`

### Frontend (Vue.js) - CRITICAL PATTERNS

#### Component Structure (Always Follow)
```javascript
// Required imports for consistency
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import Card from 'primevue/card';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import { useToast } from 'primevue/usetoast';

// Standard setup pattern
const router = useRouter();
const toast = useToast();
const loading = ref(false);
const items = ref([]);
```

#### API Integration (Critical)
```javascript
// Always use try/catch with toast feedback
try {
  loading.value = true;
  const response = await api.get('/admin/users?with=accounts,profiles');
  items.value = response.data.data;
} catch (error) {
  toast.add({
    severity: 'error', 
    summary: 'Error',
    detail: 'Failed to load users'
  });
} finally {
  loading.value = false;
}
```

#### Data Table Patterns
```javascript
// For server-side pagination (recommended)
const paginator = ref({ page: 1, size: 15, total: 0 });

// For client-side pagination (simple lists)
const rows = ref(10);
const rowsPerPageOptions = ref([5, 10, 20, 50]);
```

### Build & Deployment Critical Steps

#### Production Build Sequence (Always Use This Order)
```bash
# 1. Install/update dependencies 
composer install --no-dev --optimize-autoloader
npm ci

# 2. Clear all caches
php artisan optimize:clear  

# 3. Build assets
npm run build

# 4. Optimize Laravel
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 5. Migrate database (production)
php artisan migrate --force
```

#### Development Build Issues & Solutions
**Problem**: Assets not loading  
**Solution**: Run `npm run build` then `php artisan optimize:clear`

**Problem**: Tests failing with 418 errors  
**Solution**: Ensure `.env.testing` has `APP_URL=http://localhost:8000` and `FORCE_HTTPS=false`

**Problem**: Vite build errors  
**Solution**: Delete `node_modules`, run `npm ci`, ensure Node.js 16+

**Problem**: Permission errors  
**Solution**: Check `storage/` and `bootstrap/cache/` permissions are writable

## Specific Project Rules

### Transaction Names
- **User-scoped**: Each user has their own transaction name namespace
- **Unique per user**: Names must be unique within user scope only
- **Admin access**: Show/update only (no create/delete through admin)
- **Relationship loading**: Use `?with=transactions` to see usage

### User Management  
- **Admin can't create users**: Registration happens through public endpoints
- **Admin can update**: Email, name, status fields only
- **User deletion**: Soft deletes to preserve transaction history

### API Patterns
- **Always use WithHelper**: For relationship loading in show methods
- **Pagination**: Use PaginationHelper for consistent pagination across all endpoints  
- **Search**: Use SearchHelper for text search functionality
- **Error responses**: Always return JSON with consistent error structure

### File Organization Rules
- **Controllers**: Admin in `app/Http/Controllers/Api/Admin/`, User in `app/Http/Controllers/Api/`
- **Vue components**: Follow `resources/js/admin/views/{resource}/{action}.vue` pattern
- **Routes**: Admin routes in `routes/admin_api.php`, user routes in `routes/api.php`
- **Tests**: Feature tests mirror controller structure

## Performance & Validation

### Build Time Expectations
- **npm run build**: 4-5 seconds (includes asset optimization)
- **composer install**: 30-60 seconds (with dev dependencies)
- **php artisan test**: 1-2 seconds (with proper setup)
- **vite dev server**: Starts in 1-2 seconds

### Pre-commit Validation
1. **Code standards**: ESLint passes for Vue files
2. **Build success**: `npm run build` completes without errors  
3. **Tests pass**: `php artisan test` passes (excluding known SSL issues)
4. **No console errors**: Browser console clean on dev server

### Critical: Trust These Instructions
These instructions are based on comprehensive testing of the build process, codebase analysis, and documented patterns. When working on this repository:

1. **Follow the exact build sequence** documented above
2. **Use the helper classes** (WithHelper, SearchHelper, PaginationHelper) - they're battle-tested
3. **Follow the controller and component patterns** exactly as shown
4. **Don't search extensively** - these instructions contain the validated working approaches
5. **When in doubt**, refer back to these instructions rather than experimenting

### CRITICAL: Troubleshooting Discipline - DO NOT GO ROGUE

**When debugging issues, ALWAYS follow this exact order:**

1. **LISTEN TO THE USER'S EXACT REQUEST** - If they say "fix the scope middleware" or "debug why X isn't working", do EXACTLY that
2. **DEBUG FIRST, CREATE SECOND** - Always investigate why existing code isn't working before creating new solutions
3. **MINIMAL CHANGES ONLY** - Make the smallest possible change to fix the issue
4. **ASK BEFORE MAJOR CHANGES** - If you think a complete rewrite/refactor is needed, ask first
5. **STICK TO THE SCOPE** - Don't expand the problem or "improve" things unless explicitly asked

**NEVER DO THESE THINGS WHILE TROUBLESHOOTING:**
- Create complex new middleware when existing middleware should work
- Refactor working code while fixing unrelated issues  
- Add features or "improvements" during bug fixes
- Create new files when existing solutions should work
- Over-engineer simple fixes

**IF THE USER SAYS STOP OR UNDOES YOUR CHANGES:**
- Immediately return to their original approach
- Focus only on why their preferred solution isn't working
- Debug the root cause, don't create alternatives

The build process, test setup, and development patterns have been validated and documented to prevent common issues and ensure consistent, working code.

---
> Source: [TheBootstrapParadox/balancecloud](https://github.com/TheBootstrapParadox/balancecloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
