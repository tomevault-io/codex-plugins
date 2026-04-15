# Copilot Instructions - Cave Tech Labs Website

## Project Overview

A Django website for **The Cave Tech** (Oslo maker space) showcasing members and their projects. Uses TDD workflow with auto-commit on passing tests.

**Key Files**: `cavetechapp/models.py`, `cavetechapp/views.py`, `tests/conftest.py`, `DESIGN.md`

---

## Architecture & Data Flow

### Core Models
- **Person**: Members with `name`, `title`, `bio`, `email`, `image`. Has reverse relation `projects`.
- **Project**: Work created by members with `title`, `slug` (auto-generated), `description`, `category`, `creator` (FK to Person), `featured`, `image`.

**Important Pattern**: Project slugs are auto-generated via `save()` using `slugify(title)` - never set manually.

### Views (Class-Based)
Five simple CBVs in `views.py`:
- `IndexView`: Shows 6 featured projects + all people
- `PeopleListView`: Lists all people
- `PersonDetailView`: Shows person profile + their projects (queryset: `person.projects.all()`)
- `ProjectsListView`: Lists projects with optional category filter via `?category=` GET param
- `ProjectDetailView`: Shows project details + 3 related projects (same category, exclude current)

**URL Pattern**: App uses `app_name='cavetechapp'` with paths like `people/<int:pk>/`, `projects/<slug:slug>/`.

### Database
SQLite at `/tmp/db.sqlite3` (ephemeral in Docker). Migrations in `cavetechapp/migrations/`.

---

## Development Workflows

### Local Setup
```bash
# Containers + migrations + admin user
bash setup.sh

# Site at http://localhost:8000
# Admin at http://localhost:8000/admin/
```

### Running Tests (Critical)
```bash
# Full test suite with auto-commit on ALL PASS
bash run_tests.sh
```

**Auto-commit Behavior**: If ALL tests pass, script auto-commits with message like `✓ All 50 tests passing - auto-commit [50/50 tests]`. **No commit if any test fails.**

### TDD Cycle
1. Write failing test in `tests/test_*.py` (RED)
2. Run `bash run_tests.sh` (confirms failure)
3. Implement feature in models/views (GREEN)
4. If model changed, run: `docker-compose exec -T web python manage.py makemigrations && docker-compose exec -T web python manage.py migrate`
5. Run `bash run_tests.sh` again (should pass and auto-commit)

### Testing Setup
- Uses **pytest** (not Django's unittest) with `pytest-django` plugin
- Fixtures in `conftest.py`: `sample_person`, `sample_project`, `featured_project`
- Fixtures use `db` parameter for database access
- Test files: `tests/test_models.py`, `tests/test_views.py`, `tests/test_admin.py`

### Debugging
**VS Code**: Press **F5** to debug. Debugger attaches to port 5678 in running container.

---

## Project-Specific Conventions

### Model Field Patterns
- `created_at`/`updated_at` on every model (auto-managed with `auto_now_add=True`, `auto_now=True`)
- `class Meta: ordering` defines default sort order (Person by name, Project by `-created_at`)
- Foreign keys with `null=True, blank=True` use `SET_NULL` to preserve history

### Admin Configuration
Admin classes use fieldsets for organization:
- "Basic Information" section for core fields
- "Content" section for rich content (bio/description, images)
- "Metadata" section (collapsed) for timestamps

**Project admin**: Uses `prepopulated_fields = {'slug': ('title',)}` for admin slug generation.

### Categories
Project categories are defined as `CATEGORY_CHOICES` tuple in `Project` model:
- `'3d_printing'`, `'woodworking'`, `'metalworking'`, `'electronics'`, `'software'`, `'art'`, `'robotics'`, `'other'`

When filtering in views, category comes from GET params (`request.GET.get('category')`).

### Template Structure
- `templates/base.html`: Main layout
- `templates/cavetechapp/`: App-specific templates matching view names
- Context passed from views to templates (no extra processing)

---

## Deployment & Docker

### Containers
- Service: `cavetechlabs-web` (Python 3.11, Django 4.2)
- Volume mounts: Current dir at `/app`, `__pycache__` ignored, db at `/app/db`
- Ports: 8000 (web), 5678 (debugger)
- Entrypoint: `entrypoint.sh` runs migrations and starts Django

### Credentials
- Admin credentials stored in `admin_credentials.json` (in `.gitignore`)
- Setup script creates user if missing
- Update via: Edit `admin_credentials.json`, then run `bash update_credentials.sh`

### Key Dependencies
- Django 4.2.8, Pillow (images), pytest-django, debugpy (VS Code debugging)

---

## Common Implementation Tasks

**Adding a model field**: Edit `models.py`, run migrations, update admin.py, write tests, then `bash run_tests.sh`.

**Adding a view**: Create CBV in `views.py`, add URL in `urls.py`, create template, write tests.

**Category filtering**: Use `request.GET.get('category')` and `.filter(category=category)` (see `ProjectsListView`).

**Slug generation**: Never set manually - `Project.save()` auto-generates from title.

**Admin customization**: Use fieldsets to organize fields, `readonly_fields` for timestamps, `list_display` for list view columns.

---

## When Something Breaks

- **Database issues**: SQLite is ephemeral; migrations auto-run via entrypoint
- **Test failures**: Check fixture setup in `conftest.py`; review test file patterns
- **Admin pages blank**: Ensure model has `__str__()` method
- **URLs not found**: Verify `app_name`, path syntax, and view import in `urls.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/The-Cave-Tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
