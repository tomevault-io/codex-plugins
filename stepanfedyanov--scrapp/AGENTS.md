# Scrapp Project Guidelines

## Architecture: Feature-Sliced Design (FSD)

The frontend uses strict Feature-Sliced Design architecture with the following layer hierarchy:

### Layer Structure

1. **app** - Application shell
   - Router setup
   - Global shell layout (App.vue)
   - Public API exports all router functionality

2. **pages** - Page-level components (routes)
   - Thin wrapper components
   - Use only widget/feature public APIs
   - Structure: `pages/{page-name}/ui/{PageName}.vue`
   - Public API: `pages/{page-name}/index.js` exports the page component

3. **widgets** - Complex domain-independent UI components
   - Composed of features and entities
   - Business logic orchestration at widget level
   - Structure: `widgets/{widget-name}/ui/{WidgetName}.vue`
   - Public API: `widgets/{widget-name}/index.js` exports the component

4. **features** - User interaction features or business workflows
   - Example: auth (login/register forms)
   - Structure: `features/{feature-name}/ui/{FeatureName}.vue`
   - Public API: `features/{feature-name}/index.js` exports the component(s)

5. **entities** - Business domain entities
   - Pinia stores for state management
   - Structure: `entities/{entity-name}/model/{store-name}.js`
   - Public API: `entities/{entity-name}/index.js` exports stores
   - Examples: user (auth store), blog, note

6. **shared** - Shared utilities and infrastructure
   - API client (`shared/api/client.js`)
   - Utilities and helpers
   - Public API: `shared/api/index.js` exports `api` client
   - Public API: `shared/{segment}/index.js` exports utilities

### Public API Rules

- Each segment (entities, pages, widgets, features) **must have** an `index.js` file
- Use **only public APIs** when importing between layers - no deep imports
- Deep imports (e.g., `from '~/src/entities/user/model/auth'`) are **forbidden**
- Import pattern: `import { SomeComponent } from '~/src/widgets/some-widget'`

### Component Organization

- UI components go in `ui/` subdirectory
- Model/stores go in `model/` subdirectory
- Each component file is singular and capitalized: `BlogsDashboard.vue`, not `BlogsDashboards.vue`

### Layer Dependencies

|        | Can import from          |
|--------|--------------------------|
| app    | pages, shared            |
| pages  | widgets, features, entities, shared |
| widgets| features, entities, shared |
| features | entities, shared |
| entities | shared           |
| shared | (nothing)        |

### Technology Stack

- **Framework**: Vue 3 (Composition API)
- **Build tool**: Vite
- **UI Library**: PrimeVue
- **State management**: Pinia
- **Routing**: Vue Router
- **HTTP client**: Axios (via `shared/api`)
- **Styling**: SCSS (scoped)

### Naming Conventions

- Components: PascalCase (`BlogsPage.vue`, `NoteEditor.vue`)
- Stores: `use{EntityName}Store` (`useAuthStore`, `useBlogsStore`)
- Files/folders: kebab-case (`blogs-dashboard`, `note-editor`)
- Imports: Use alias `~/src` for absolute paths

### No Legacy Code

Do not leave any legacy files or folders. All old code must be removed when refactoring.

## Docker Compose Setup

The project includes two Docker Compose configurations:

### Production Compose (`docker-compose.yml`)

Runs backend and frontend services in production mode:

```bash
docker-compose up -d
```

Services:
- **backend** - Django API server (port 8000)
- **frontend** - Vue.js app built with Vite (port 5173)

### Development Compose (`docker-compose.dev.yml`)

Runs services with development settings (hot-reload, debug mode):

```bash
docker-compose -f docker-compose.dev.yml up -d
```

Use this for active development with live reloading.

### Common Commands

```bash
docker-compose ps                           # View running services
docker-compose logs -f backend              # Follow backend logs
docker-compose logs -f frontend             # Follow frontend logs
docker-compose down                         # Stop all services
docker-compose down -v                      # Stop and remove volumes
```

## Backend Architecture

Backend is a Django REST Framework (DRF) API server with JWT authentication.

### Project Structure

```
backend/
├── config/              # Project configuration
│   ├── settings.py     # Django settings (DB, AUTH, CORS, etc.)
│   ├── urls.py         # Main URL routes (api/)
│   ├── wsgi.py         # WSGI application
│   └── asgi.py         # ASGI application
├── blog/               # Main Django app
│   ├── models.py       # Data models
│   ├── views.py        # API ViewSets
│   ├── serializers.py  # DRF Serializers
│   ├── permissions.py  # Custom permissions (IsOwner)
│   ├── urls.py         # App URL routes (REST endpoints)
│   ├── signals.py      # Django signals
│   ├── migrations/     # Database migrations
│   └── admin.py        # Django admin configuration
├── manage.py           # Django management script
├── requirements.txt    # Python dependencies
└── Dockerfile          # Container configuration
```

### Technology Stack

- **Framework**: Django 6.0
- **API**: Django REST Framework (DRF) 3.16
- **Authentication**: JWT (djangorestframework-simplejwt)
- **Database**: PostgreSQL (with fallback to SQLite)
- **CORS**: django-cors-headers
- **Server**: Gunicorn

### Database Models

#### User (Django built-in)
- Standard Django User model
- Username, email, password
- Used as owner for blogs, notes, integrations

#### Blog
- `uuid` - Unique identifier (indexed)
- `owner` - Foreign key to User
- `title` - Blog title
- `created_at`, `updated_at` - Timestamps
- Soft delete support via `is_deleted`, `deleted_at`

#### Note
- `uuid` - Unique identifier (indexed)
- `blog` - Foreign key to Blog
- `title`, `body` - Content
- `status` - One of: draft, scheduled, published, archived, deleted
- `scheduled_at`, `published_at`, `archived_at` - Timestamps
- Soft delete support

#### Integration
- `owner` - Foreign key to User
- `name` - Integration name
- `provider` - One of: medium, devto, telegram
- `config` - JSONField for integration settings
- Soft delete support

#### BlogIntegration (junction table)
- `blog` - Foreign key to Blog
- `integration` - Foreign key to Integration
- `enabled` - Boolean flag
- `settings` - JSONField for blog-specific integration settings
- Unique constraint on (blog, integration)

#### NoteIntegration (junction table)
- `note` - Foreign key to Note
- `integration` - Foreign key to Integration
- `enabled` - Boolean flag
- `use_blog_defaults` - Whether to use blog-level settings
- `settings` - JSONField for note-specific integration settings
- Unique constraint on (note, integration)

### API Endpoints

Base URL: `/api/`

#### Authentication
- `POST /api/auth/register/` - Register new user
- `POST /api/auth/token/` - Obtain JWT tokens (username + password)
- `POST /api/auth/refresh/` - Refresh access token using refresh token

#### Blogs
- `GET /api/blogs/` - List user's blogs
- `POST /api/blogs/` - Create new blog
- `GET /api/blogs/{uuid}/` - Get blog details
- `PATCH /api/blogs/{uuid}/` - Update blog
- `DELETE /api/blogs/{uuid}/` - Soft delete blog

#### Notes
- `GET /api/notes/` - List user's notes (filter by `blog_uuid` query param)
- `POST /api/notes/` - Create new note
- `GET /api/notes/{uuid}/` - Get note details
- `PATCH /api/notes/{uuid}/` - Update note
- `DELETE /api/notes/{uuid}/` - Soft delete note
- `POST /api/notes/{uuid}/archive/` - Archive note (custom action)

#### Integrations
- `GET /api/integrations/` - List user's integrations
- `POST /api/integrations/` - Create new integration
- `GET /api/integrations/{id}/` - Get integration details
- `PATCH /api/integrations/{id}/` - Update integration
- `DELETE /api/integrations/{id}/` - Soft delete integration

#### Blog Integrations
- `GET /api/blog-integrations/` - List blog-integration relationships
- `POST /api/blog-integrations/` - Create blog-integration relationship
- `PATCH /api/blog-integrations/{id}/` - Update blog-integration
- `DELETE /api/blog-integrations/{id}/` - Remove blog-integration

#### Note Integrations
- `GET /api/note-integrations/` - List note-integration relationships
- `POST /api/note-integrations/` - Create note-integration relationship
- `PATCH /api/note-integrations/{id}/` - Update note-integration
- `DELETE /api/note-integrations/{id}/` - Remove note-integration

### Authentication & Authorization

**JWT Authentication**
- AccessToken lifetime: 30 minutes
- RefreshToken lifetime: 7 days
- Tokens obtained via `/api/auth/token/` endpoint with username + password

**Permissions**
- `IsOwner` - Custom permission in `permissions.py`
  - For Blog/Integration: checks if `obj.owner == request.user`
  - For Note/BlogIntegration/NoteIntegration: checks if `obj.blog.owner == request.user`
- All endpoints except registration require authentication
- Each user can only access their own resources

### Soft Delete Pattern

Custom `SoftDeleteModel` abstract class implements soft deletion:
- `is_deleted` - Boolean flag (default: False)
- `deleted_at` - DateTime stamp
- `QuerySet.alive()` - Filter non-deleted objects
- `QuerySet.deleted()` - Filter deleted objects
- Models override `delete()` method to mark as deleted instead of removing

### Development Guidelines

- Always check user permissions in `get_queryset()` methods
- Use `perform_create()` to automatically set the owner
- Check ownership in `perform_create()` for foreign key relationships
- Use soft delete instead of hard delete
- Avoid N+1 queries - use `select_related()` and `prefetch_related()`
- All JSONField data in integrations should be validated before saving
- Use custom actions (via `@action` decorator) for non-standard operations

## Frontend Development Guidelines

- Keep pages as thin as possible (just route handlers and layout)
- Move all logic to widgets, features, or entities
- Use Pinia stores for all state management
- Use `shared/api` client for all HTTP requests
- Follow Vue 3 Composition API patterns
- Use scoped SCSS for component styling

Before starting a new task in the above plan, update progress in the plan.
-->
- Work through each checklist item systematically.
- Keep communication concise and focused.
- Follow development best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stepanfedyanov)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/stepanfedyanov)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
