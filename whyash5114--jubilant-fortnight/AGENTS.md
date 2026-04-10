# Copilot Instructions for Tailspin Shelter

## Architecture Overview

This is a **monorepo** dog shelter adoption website with separated frontend and backend:

- **Backend**: Flask API (`/server`) on port 5100 with SQLAlchemy ORM and SQLite database
- **Frontend**: Astro SSR (`/client`) on port 4321 with Svelte components and Tailwind CSS v4
- **API Proxy**: Astro middleware (`client/src/middleware.ts`) forwards `/api/*` requests to Flask backend

The separation allows independent scaling, but they run together locally via `scripts/start-app.sh`.

## Critical Workflows

### Starting the Application
**Always use**: `./scripts/start-app.sh` (or `.ps1` on Windows) from project root
- Sets up Python venv, installs dependencies, seeds database
- Starts Flask server (port 5100) and Astro dev server (port 4321) concurrently
- Ctrl-C gracefully stops both servers
- Never run `python app.py` or `npm run dev` manually

### Database Seeding
**Use**: `./scripts/seed-database.sh` for idempotent database population from CSV files
- Reads from `server/models/*.csv` (breeds.csv, dogs.csv)
- Safe to run multiple times (won't duplicate data)

### Testing
- **Backend unit tests**: `cd server && python -m unittest test_app.py` (uses mocks, no real DB)
- **Frontend E2E tests**: `cd client && npm run test:e2e` (requires both servers running)
- **E2E test modes**: `test:e2e:ui` (interactive), `test:e2e:debug` (step through), `test:e2e:headed` (see browser)

## Python Backend Conventions

### Flask Routes & Type Hints
- **All routes** use Python 3.10+ type hints: `def get_dogs() -> Response:`
- Return types: `Response` (single) or `tuple[Response, int]` (with status code)
- Use SQLAlchemy `.query()` pattern, not raw SQL

### Models Architecture (`server/models/`)
- **BaseModel** (`base.py`): Abstract base with shared validation methods (e.g., `validate_string_length`)
- **Dog model** (`dog.py`): Uses `AdoptionStatus` enum, has `@validates` decorators for validation
- **Breed model** (`breed.py`): Has bidirectional relationship with Dog via `dogs` backref
- Models live in `server/models/__init__.py` which exports `db` (SQLAlchemy instance) and `init_db()`

### Database Pattern
```python
# Always join Dog with Breed to get breed name (it's a foreign key)
query = db.session.query(Dog.id, Dog.name, Breed.name.label('breed'))
    .join(Breed, Dog.breed_id == Breed.id)
```

### API Response Pattern
- Convert SQLAlchemy results to dicts: `[{'id': dog.id, 'name': dog.name, ...} for dog in query.all()]`
- Enum values: serialize as `.name` (e.g., `status.name` returns `"AVAILABLE"`)
- 404s: return `jsonify({"error": "..."}), 404` tuple

## Frontend (Astro + Svelte)

### Astro Configuration
- **SSR mode**: `output: 'server'` in `astro.config.mjs` (NOT static site generation)
- **Adapter**: `@astrojs/node` standalone mode
- **Svelte client directive**: `<DogList client:only="svelte" />` for interactive components

### Component Patterns
- **TypeScript interfaces**: Define inline per component (no shared types file)
  ```typescript
  interface Dog {
    id: number;
    name: string;
    breed: string;
  }
  ```
- **Props**: Use `export let` syntax in Svelte 5 (e.g., `export let dogs: Dog[] = []`)
- **API calls**: Fetch from `/api/dogs` (relative URL), middleware proxies to Flask

### Styling Convention
- **Tailwind v4**: Uses `@tailwindcss/vite` plugin (NOT PostCSS)
- **Color scheme**: Dark theme with slate-800 backgrounds, blue/purple accents
- **Hover effects**: Translate-y, shadow, and border color transitions
- **Loading states**: Skeleton loaders with `animate-pulse` on slate-700 backgrounds

### Svelte State Management
- Use `onMount()` for data fetching
- Track `loading`, `error`, and data states separately
- Use `{#if loading}...{:else if error}...{:else}` control flow

## Testing Patterns

### Backend Unit Tests
- Use `unittest.mock.patch` for `db.session.query`
- Helper methods: `_create_mock_dog()`, `_setup_query_mock()` for reusable mocks
- Test structure: Arrange-Act-Assert with clear comments

### Playwright E2E Tests
- Use `page.getByRole()` for semantic element selection (not CSS selectors)
- Test loading states with try-catch for race conditions
- Mock API responses with `page.route('/api/dogs', route => route.fulfill(...))`

## File Navigation Shortcuts

- New backend endpoint: Add to `server/app.py` with type hints and proper joins
- New frontend page: Create in `client/src/pages/*.astro` (dynamic routes: `[id].astro`)
- New Svelte component: `client/src/components/*.svelte` with inline TypeScript
- Add model validation: Use `@validates` decorator in model file, call `validate_string_length()`

## Contribution Conventions

### Commit Messages
- **Use Conventional Commits**: `type(scope): description`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Examples:
  - `feat(api): add filter endpoint for dog breeds`
  - `fix(ui): resolve loading state race condition`
  - `docs(readme): update setup instructions`

### Naming Conventions
- **Python**: `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_CASE` for constants
- **TypeScript/JavaScript**: `camelCase` for functions/variables, `PascalCase` for components/classes
- **Files**: Follow language conventions (`dog_model.py`, `DogList.svelte`, `api-utils.ts`)

## Common Pitfalls

1. **Port conflicts**: Flask uses 5100 (not 5000) to avoid macOS AirPlay conflicts
2. **Enum serialization**: Always use `.name` not `.value` when returning enums in JSON
3. **Astro prerendering**: Dynamic routes need `export const prerender = false;`
4. **Virtual env**: Scripts auto-create venv—don't manually activate or you'll break isolation
5. **API middleware**: Returns 502 if Flask server isn't running; check terminal for errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WhyAsh5114)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/WhyAsh5114)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
