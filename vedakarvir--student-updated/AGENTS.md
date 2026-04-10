# Copilot Instructions for Student Management System

## Project Overview
This is a Laravel 12 student management system using AdminLTE 3 dashboard. It manages students, courses, venues, academic histories, and payment records with role-based authentication.

**Tech Stack:**
- **Framework:** Laravel 12 with Laravel Breeze authentication
- **Admin UI:** jeroennoten/laravel-adminlte (^3.15)
- **Frontend:** Vite + SCSS + Bootstrap 5
- **Database:** Migrations-based (SQLite for testing, likely MySQL/PostgreSQL for production)
- **Image Processing:** intervention/image (3.0)

## Database & Model Architecture

### Entity Relationships
- **Students** (`student_list` table) → has many **AcademicHistories** → links to **Courses**
- **Students** → has many **PaymentHistories**
- **Courses** (`course_list` table) → belongs to one **Venue**
- **Venues** (`venue_list` table) → stores event/course venue information
- **Users** handle authentication via Laravel Breeze

### Key Models (Located: `app/Models/`)
- `Student.php` - Custom table: `student_list` with 30+ fields (personal, contact, address, interview info)
- `AcademicHistory.php` - Links students to courses with year/status tracking
- `Course.php` - Custom table: `course_list` (note: `$timestamps = false`)
- `Venue.php` - Custom table: `venue_list` (venue_name, address fields, postcode)
- `PaymentHistory.php` - Financial tracking
- `SystemInfo.php` - Admin settings stored in DB (logo, name - see AppServiceProvider)
- `User.php` - Standard Laravel authentication model
- `Department.php` - *Disabled for now* (can be re-enabled in the future)

### Important Pattern: Soft Delete via Flag
Models use `delete_flag` (boolean, default 0) instead of Eloquent soft deletes. Always query with:
```php
Student::where('delete_flag', 0)->find($id)
// NOT: Student::find($id)
```

## Controller & Routing Structure

### Route Organization
All admin routes protected by `auth` middleware with `/admin` prefix and `admin.` name prefix:
```
routes/web.php:
- /admin/dashboard → DashboardController
- /admin/students → StudentController (custom updateStatus action)
- /admin/users → UserController
- /admin/courses → CourseController
- /admin/venues → VenueController
- /admin/academic-history → AcademicHistoryController
- /admin/payment-history → PaymentHistoryController
- /admin/system-info → SystemInfoController
- /admin/departments → Disabled (can be re-enabled)
```

### Controller Conventions
Controllers located in `app/Http/Controllers/Admin/` follow RESTful patterns:
- Models queried with `delete_flag = 0` filter
- Delete operations set `delete_flag = 1` (not actual deletion)
- Redirects use named routes: `admin.students.index`
- Flash messages: `with('success', 'Message')` for feedback

## Key Configuration & Bootstrap

### AppServiceProvider (`app/Providers/AppServiceProvider.php`)
- Fetches `SystemInfo` table on boot → dynamically sets AdminLTE config
- Shares `$systemInfo` array to all views (used for logo, name)
- Configures AdminLTE title prefix/postfix from DB

### AdminLTE Configuration
- Primary config in `config/adminlte.php` (247+ lines)
- Dynamic branding via SystemInfo model (logo_img, short_name)
- Components available: Card, Alert, Callout, SmallBox, ProfileWidget, Datatable

## Frontend & Build

### Vite Setup
- Input files: `resources/sass/app.scss` + `resources/js/app.js`
- Outputs to `public/build/` (manifest.json + assets/)
- Commands: `npm run dev` (watch) and `npm run build` (production)
- Dependencies: Bootstrap 5, Tailwind 4, SASS, jQuery, Popper.js

### View Structure
Admin views in `resources/views/admin/`:
- `dashboard.blade.php` - Mavenues/`, `in admin panel
- `students/`, `courses/`, `users/`, `system_info/` folders with CRUD views
- `departments/` folder - *Disabled views* (can be re-enabled)
- Partials in `resources/views/partials/`
- Shared layout via `resources/views/layouts/`

### AdminLTE Components (Blade)
When building views, leverage vendor components:
- `<x-adminlte-card>` - Content containers
- `<x-adminlte-datatable>` - Sortable tables with export
- `<x-adminlte-form-input>` - Form fields with validation
- `<x-adminlte-button>` - Styled buttons with icons
- Font Awesome 5 icons: `fas fa-icon-name`

## Testing & Quality
- **PHPUnit** config in `phpunit.xml` (SQLite in-memory DB for tests)
- Test suites: `tests/Unit/` and `tests/Feature/`
- Faker for seeding: `database/factories/UserFactory.php`
- Code style: Laravel Pint formatter available

## Development Workflows

### Database Migrations
- New migrations go in `database/migrations/` with timestamp prefix
- Naming: `YYYY_MM_DD_HHMMSS_action_table.php`
- Use `$table->boolean('delete_flag')->default(0)` for soft delete pattern
- Example: [2025_06_03_155620_create_student_list_table.php](database/migrations/2025_06_03_155620_create_student_list_table.php)

### Common Commands
```bash
php artisan migrate           # Run pending migrations
php artisan tinker            # Interactive shell (laravel/tinker)
npm run dev                   # Watch Vite builds
composer test                 # Run PHPUnit
php artisan pint              # Format code with Laravel Pint
```

### Adding CRUD Features
1. Create Model in `app/Models/` with `$table`, `$fillable`, relationships
2. Create migration if new table (with `delete_flag`)
3. Create Controller in `app/Http/Controllers/Admin/` extending `Controller`
4. Register routes in `routes/web.php` with `Route::resource()`
5. Create Blade views using AdminLTE components (no custom CSS usually needed)

## Critical Patterns to Follow

1. **Always exclude soft-deleted records:** Use `where('delete_flag', 0)` in queries
2. **Pagination:** Use `paginate(10)` for list views (see StudentController::index)
3. **Relationships:** Load with `with()` to avoid N+1: `Student::with(['academicHistories.course'])`
4. **Validation:** Not shown in current code—likely needs implementation; follow Laravel Form Request pattern
5. **Timestamps:** Most models have `timestamps()` except Course (check migration for actual behavior)
6. **View sharing:** Use `view()->share()` in AppServiceProvider for global data (e.g., $systemInfo)

## Storage & File Handling
- Images: Stored in `public/storage/` (managed by `storage:link` symlink)
- Upload destination: `public/storage/uploads/` for user-generated content
- `intervention/image` used for processing (see composer.json)

## Security Notes
- All admin routes protected by `auth` middleware
- No explicit authorization checks visible (consider adding Policies/Gates for role-based access)
- CSRF protection via Laravel Breeze (enabled by default)

---

**For questions about specific components or patterns, check the model relationships and migrations to understand data flow.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vedakarvir)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/vedakarvir)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
