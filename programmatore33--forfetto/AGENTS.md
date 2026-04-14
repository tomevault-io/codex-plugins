# Forfetto AI Coding Instructions

## Project Overview

Forfetto is an Italian flat-tax regime (regime forfettario) income tracking application built with Laravel + Vue 3 + Inertia.js. It helps freelancers and sole proprietors track invoices, expenses, and calculate real net income after taxes and contributions.

## Architecture Patterns

### Multi-Tenant User Isolation with Demo System

All models use `HasUserScope` trait (`app/Traits/HasUserScope.php`) for automatic user scoping:

- **Regular users**: Global scope filters by `user_id` automatically
- **Demo users**: Additional filtering by `session_id` for complete isolation
- Auto-assigns `user_id` and `session_id` (for demo users) on creation
- Use `withoutUserScope()`, `forUser($userId)`, or `forDemoSession($sessionId)` for admin access

#### Demo System (`docs/sistema-demo.md`)

- **Demo users** (`is_demo = true`) get isolated sessions with temporary data
- **UserSession model** manages 24-hour demo sessions with UUID identifiers
- **DemoSession middleware** handles session creation, data population, and cleanup
- **Automatic cleanup** via `demo:cleanup` command (scheduled hourly)
- **DemoDataSeeder** creates realistic sample data for each session

#### Demo Usage

```php
// Create demo user
$demoUser = User::factory()->demo()->create();

// DEMO CREDENTIALS - Intentionally public for demo purposes
// Access: demo@forfetto.it / demo123
// System automatically creates isolated session with sample data
```

### Data Layer Pattern

- **Models**: Standard Eloquent with relationships (`app/Models/`)
- **DTOs**: Spatie Laravel Data objects for type-safe data transfer (`app/Dtos/`)
- **Services**: Business logic and query building (`app/Services/`)
- **Example**: `CustomerIndexService` extends base `IndexService` with pagination, search, and filtering

### Frontend Architecture

- **Inertia.js** for SPA-like experience with server-side routing
- **Vue 3 Composition API** with TypeScript
- **Shadcn/UI + Tailwind CSS** for UI components
- **PrimeVue** for complex components (data tables, calendars)

## Development Workflows

### Quick Setup

```bash
composer run setup    # Full project setup
composer run dev       # Start dev server with queue, logs, and Vite
composer run test      # Run PHPUnit tests
```

### Demo System Management

```bash
# View demo sessions (dry run)
php artisan demo:cleanup --dry-run

# Clean expired sessions
php artisan demo:cleanup --force

# Clean specific session
php artisan demo:cleanup --session-id=uuid --force
```

### Code Quality (Automated via Husky)

- **PHP**: Laravel Pint (PSR-12 formatting)
- **JS/TS/Vue**: ESLint + Prettier
- **Pre-commit hooks** run automatically (`docs/husky-lint.md`)

### Docker Development

```bash
./vendor/bin/sail up    # Start Laravel Sail environment
./vendor/bin/sail down  # Stop containers
```

## Code Conventions

### Backend (Laravel)

- **Laravel 12.35.1**: Uses application structure with `bootstrap/app.php`
- **Comments**: Always in English, PHPDoc for public/protected methods
- **Migrations**: Minimal comments unless critical
- **Seeder data**: Use Italian language for realistic data
- **Enums**: Use for constants like `TaxRateEnum`, `PaymentMethodEnum`
- **Scheduling**: Use `routes/console.php` for task scheduling (Laravel 12 style)

### Frontend (Vue 3)

- **Components**: PascalCase, place in appropriate subdirectories
- **Composables**: `use*` pattern for reusable logic (`useTableConfigs`, `useTwoFactorAuth`)
- **Routes**: Use Laravel Wayfinder for type-safe routing
- **Tables**: Use `DataTableWithPagination` component with configuration objects

### File Organization

- **Docs**: Create `.md` files in Italian in `docs/` folder
- **Components**: Shadcn/UI in `components/ui/`, custom in `components/`
- **Types**: Domain-specific types in `types/` directory

## Key Integration Points

### Authentication & Authorization

- **Laravel Fortify** for auth features (2FA, password reset)
- **Inertia middleware** shares auth state globally
- Custom auth views in `resources/js/pages/auth/`

### Database Relationships

- `User` → `Customer` → `Invoice` (one-to-many chains)
- `User` → `Expense` with `ExpenseCategory`
- ATECO codes linked to invoices for tax calculations

### Italian Business Logic

- **Tax calculations**: 5% (first 5 years) vs 15% (standard) rates
- **Invoice fields**: VAT number, tax code, SDI code, PEC for Italian requirements
- **Withholding tax**: 20% deduction calculations when applicable

## Domain-Specific Patterns

### Demo System Patterns

When working with demo functionality:

```php
// Check if user is demo
if (Auth::user()->isDemoUser()) {
    // Demo-specific logic
}

// Access demo session data
$session = Auth::user()->getActiveDemoSession();

// Query demo-only records for cleanup
$demoRecords = Invoice::demoOnly()->get();

// Query specific demo session
$sessionInvoices = Invoice::forDemoSession($sessionId, $userId);
```

### Table Management

Use `useTableConfigs` composable for consistent table behavior:

```vue
const config = useCustomerTableConfig(); const routes =
useRouteHelper('customers');
```

### Data Transfer

Always use DTOs for API responses:

```php
return CustomerDto::collect($paginator->items());
```

### Form Handling

Combine Inertia forms with VeeValidate for client-side validation:

```vue
import { Form } from '@inertiajs/vue3';
```

### Color System

Custom CSS variables for income/expense colors:

- Income: `--color-forfetto-income: #4caf50` (green)
- Expense: `--color-forfetto-expense: #ff5722` (red)
- Accent: `--color-forfetto-accent: #cfd8dc` (cool gray)

## Testing Guidelines

- **Feature tests**: Focus on user workflows and business logic
- **Unit tests**: Critical calculations (tax rates, net income)
- **Demo tests**: Test session isolation, cleanup, and data population
- Use factories for test data generation (`database/factories/`)

### Demo System Testing

```php
// Test demo user creation
$demoUser = User::factory()->demo()->create();

// Test session isolation
$session1 = $demoUser->createDemoSession();
$session2 = $demoUser->createDemoSession();

// Verify data isolation between sessions
```

Remember: This is an MVP focused on Italian business requirements. Keep features simple and user-friendly for small business owners who want to avoid accounting complexity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/programmatore33) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
