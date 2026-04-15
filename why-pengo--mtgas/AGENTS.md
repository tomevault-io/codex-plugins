# MTG Arena Statistics Tracker - Copilot Instructions

## Project Overview

Django web application for tracking and analyzing Magic: The Gathering Arena game statistics. Parses MTGA's Player.log file to extract match data, stores it in SQLite/PostgreSQL, and provides a dashboard with D3.js visualizations.

**Tech Stack**: Django 6.0+, Python 3.10+, SQLite (default), D3.js, vanilla CSS/JS

## Build & Test Commands

### Setup
```bash
make setup              # Install dev dependencies + run migrations (installs into .venv)
make download-cards     # Download Scryfall bulk data (one-time, ~350MB)
```

### Development
```bash
make run                        # Start Django dev server on http://127.0.0.1:8000
.venv/bin/python manage.py shell  # Django shell
make import-log LOG=/path/to/Player.log  # Import MTGA log file (CLI)
# Or use web UI: http://127.0.0.1:8000/import/
# Card data management: http://127.0.0.1:8000/card-data/
```

### Testing
```bash
.venv/bin/pytest                          # Run all tests
.venv/bin/pytest tests/test_parser.py     # Run specific test file
.venv/bin/pytest tests/test_models.py::TestMatchModel  # Run specific test class
.venv/bin/pytest -k "test_parse_match"    # Run tests matching pattern
.venv/bin/pytest --cov=stats --cov=src --cov=cards  # Run with coverage
```

### Code Quality
```bash
make format      # Format with black (line-length=100) and isort
make lint        # Run flake8 (max-line-length=100)
make lint-css    # Run stylelint on CSS files
make check       # Run format-check + lint + lint-css
make ci          # Run check + test (use before commits)
# Note: make targets invoke .venv/bin/* tools directly; no need to activate venv
```

## Project Structure

### Three-Layer Architecture

1. **`src/`** - Core business logic (parser, services)
   - `parser/log_parser.py` - Parses MTGA Player.log JSON events
   - `services/scryfall.py` - Downloads/caches Scryfall bulk data
   - `services/import_service.py` - Orchestrates import workflow (used by management command and `stats/views/imports.py`)
   - `services/play_advisor.py` - Per-match strategic play analysis (per-turn suggestions)
   - `exceptions.py` - Custom exception types

2. **`stats/`** - Django app (models, views, templates)
   - `models.py` - ORM models (Match, Deck, DeckSnapshot, Card, GameAction, etc.)
   - `views/` - Django views split by domain (dashboard, decks, matches, imports)
   - `management/commands/` - CLI commands (import_log, download_cards)
   - `templates/` - Django templates
   - `static/` - CSS (`css/style.css`) and JS (`js/app.js`, `js/charts.js`)

3. **`cards/`** - Django app for physical (paper) card inventory
   - `models.py` - `PaperCard` model: stores card metadata fetched from Scryfall named-card API
   - `views.py` - `card_index`, `add_paper_card` (fuzzy name lookup → Scryfall), `paper_card_detail`
   - `templatetags/cards_extras.py` - `mana_icons` and `cmc_value` template filters
   - `urls.py` - Mounted at `/cards/` (app_name = `cards`); routes: `/`, `/add/`, `/paper/<pk>/`
   - `templates/cards/` - `index.html` (sortable/searchable table), `add_paper_card.html`, `paper_card_detail.html`

4. **`mtgas_project/`** - Django project configuration
   - `settings.py` - Django settings (INSTALLED_APPS, DATABASE, etc.)
   - `urls.py` - Root URL configuration
   - `wsgi.py` - WSGI entry point

### Key Files
- `manage.py` - Django CLI entry point
- `pyproject.toml` - Dependencies, black/isort/pytest config
- `Makefile` - Development task automation
- `data/mtga_stats.db` - SQLite database (gitignored)
- `data/cache/` - Scryfall card cache (gitignored)

## Core Concepts

### Log Parsing Flow
1. **Read**: `log_parser.py` reads Player.log line-by-line
2. **Extract**: Finds JSON events (prefixed with `[UnityCrossThreadLogger]` or timestamps)
3. **Parse**: Extracts key event types:
   - `matchGameRoomStateChangedEvent` - Match metadata (players, result, format)
   - `EventSetDeckV2` - Deck composition
   - `greToClientEvent` - In-game actions (plays, attacks, life changes)
4. **Aggregate**: Groups events by `match_id` into `MatchData` objects
5. **Import**: `import_service.py` saves to database via Django ORM

### Match Deduplication
- **Primary Key**: `match_id` (UUID from Arena logs)
- Import checks `Match.objects.filter(match_id=...).exists()` before creating
- Use `--force` flag to re-import existing matches

### Scryfall Integration
- Uses Scryfall's bulk data to map Arena's `grpId` to card names
- **Image caching**: `download_card_image()` and `get_cached_image_path()` methods
  - Downloads card images from Scryfall on demand
  - Caches locally in `data/cache/card_images/{grp_id}.jpg`
  - Batch download available in deck gallery view

### Database Schema Key Relationships
```
Match (1) ←──→ (1) DeckSnapshot ←──→ (*) DeckCard (*) ←──→ (1) Card
  ↓                    ↓
Deck (identity)    is_sideboard flag
  ↓ (1:*)
GameAction
  ↓ (1:*)
LifeChange, ZoneTransfer
```

- Each `Match` has one `DeckSnapshot` (exact cards played that match)
- `Deck` is an identity anchor; `DeckSnapshot` stores the actual card list per match
- `DeckCard` has `is_sideboard` flag; `snapshot_id` FK (not `deck_id`)
- `GameAction` stores in-game events (indexed by match_id)
- `LifeChange` and `ZoneTransfer` track detailed game state

## Coding Conventions

### CSS Conventions
- **All styles in `stats/static/css/style.css`** — no `<style>` blocks in templates, no `style="..."` inline attributes
- Run `make lint-css` (stylelint) to validate; use modern `rgb()` notation: `rgb(0 0 0 / 50%)` not `rgba(0,0,0,0.5)`
- **Allowed exceptions** (unavoidable):
  - Django template variable widths: `style="width: {{ value }}%"` on progress bars
  - JS-controlled initial state: `style="display:none"` on elements toggled by JavaScript

### Python Code Style
- **Line length**: 100 characters (black and flake8 configured)
- **Import order**: `isort` with profile="black", custom sections for Django
  - Order: FUTURE → STDLIB → DJANGO → THIRDPARTY → FIRSTPARTY → LOCALFOLDER
- **Type hints**: Used throughout codebase (Django 6.0+ compatible)
  - Views: `HttpRequest`, `HttpResponse`, `JsonResponse` return types
  - Models: Method return types (`str`, `int`, `float`, `str | None`)
  - Services: Full typing with `Optional`, `Dict`, `List`, `Set`, `Any`
  - Use `from __future__ import annotations` in models for forward references
- **Docstrings**: Required for modules, classes; optional for obvious methods
- **Django naming**: Models use singular (Match, Deck), tables use plural (`db_table = "matches"`)

### Django Patterns
- **Version**: Django 6.0+ - use latest features freely (no backward compatibility needed)
- **Migrations**: Never edit existing migrations; create new ones with `make makemigrations`
- **Model queries**: Prefer `select_related()` and `prefetch_related()` to avoid N+1
- **Admin**: All models registered in `stats/admin.py` with list_display and search_fields
- **Management commands**: Extend `BaseCommand`, use `self.stdout.write()` for output

### Error Handling
- Parser continues on malformed JSON (logs errors, doesn't crash)
- Missing card data (grpId not in Scryfall) uses fallback: `f"Unknown ({grp_id})"`
- Import service wraps in transaction: all-or-nothing per match
- Use custom exceptions from `src/exceptions.py` (e.g., `InvalidLogFormatError`)

### Testing Conventions
- **Fixtures**: `conftest.py` provides Django test client and sample data
- **Test data**: Use factories or `tests/fixtures/` for sample log files
- **Naming**: `test_<function>_<scenario>` (e.g., `test_parse_match_with_missing_player`)
- **Database**: Django's test runner uses in-memory SQLite (fast, isolated)

## Common Tasks

### Adding a New Model
1. Add model to `stats/models.py`
2. Run `python manage.py makemigrations`
3. Review migration file
4. Run `python manage.py migrate`
5. Register in `stats/admin.py` if needed
6. Add tests in `tests/test_models.py`

### Adding a New Chart
1. Add data aggregation in `stats/views/dashboard.py` (return JSON or context)
2. Create D3.js function in `stats/static/js/charts.js`
3. Call function from template (e.g., `stats/templates/dashboard.html`)
4. Style in `stats/static/css/style.css`

### Extending Log Parser
1. Identify new event type in Player.log (use grep or search tools)
2. Add parsing logic to `log_parser.py`'s event type handlers
3. Update `MatchData` or create new dataclass if needed
4. Add test case with sample event JSON in `tests/test_parser.py`
5. Update `import_service.py` to persist new data

### Adding a CLI Command
1. Create `stats/management/commands/<command_name>.py`
2. Extend `BaseCommand`, implement `add_arguments()` and `handle()`
3. Use `self.stdout.write(self.style.SUCCESS('...'))` for output
4. Add to Makefile if used frequently
5. Consider if functionality should also be exposed via web UI (see `import_log` view for example)

## Installed Skills — Project Configuration

### `web-design-reviewer`
- Dev server: `make run` → **`http://127.0.0.1:8000`**
- All styles live in `stats/static/css/style.css` — never add `<style>` blocks in templates or `style="..."` inline attributes
- CSS color notation: `rgb(0 0 0 / 50%)` — never `rgba(0,0,0,0.5)`
- After any CSS fix: run `make lint-css` and resolve all errors
- Allowed inline-style exceptions (do not flag or remove):
  - Django template variable widths: `style="width: {{ value }}%"` on progress bars
  - JS-controlled state: `style="display:none"` on elements toggled by JavaScript
- Templates: `stats/templates/`, `cards/templates/`; JS: `stats/static/js/`

### `polyglot-test-agent`
- Run tests with `.venv/bin/pytest` (not bare `pytest`)
- Coverage targets: `--cov=stats --cov=src --cov=cards`
- Test naming: `test_<function>_<scenario>`
- Fixtures in `conftest.py`; test database is in-memory SQLite

### `webapp-testing`
- Dev server: `make run` → `http://127.0.0.1:8000`
- Key routes: `/` (dashboard), `/import/`, `/card-data/`, `/cards/` (paper card inventory), `/matches/`, `/decks/`

## Development Workflow

1. **Branch**: Every piece of work must happen on a feature branch (never commit directly to `main`). The branch must be linked to a GitHub issue — create one first if none exists. Name branches after the issue: `issue-{number}-{short-description}` (e.g., `issue-23-semantic-versioning`).
2. **GitHub Issue**: All work must be tracked by a GitHub issue. Before starting any task, confirm an issue exists (or create one). Reference the issue number in the branch name and in commit messages / PR descriptions.
3. **Format**: Run `make format` before committing
4. **Test**: Run `make ci` to check format, lint, and tests
5. **Docs**: After all changes, review and update **every** markdown doc that is affected:
   - `README.md`, `CONTRIBUTING.md`, `QUICKSTART.md` — top-level user-facing docs
   - `docs/` — technical reference docs (`DATABASE_SCHEMA.md`, `DEVELOPMENT.md`, `LOG_PARSING.md`, `LOGGING.md`, `MATCH_REPLAY.md`)
   - **Rule**: If your commit adds, removes, or changes a feature, model, route, command, or configuration value described in any of these files, update that file in the same commit. Do not leave docs stale.
6. **Commit**: Use conventional commit prefixes:
   - `feat:` - New features
   - `fix:` - Bug fixes
   - `refactor:` - Code refactoring
   - `test:` - Test additions/changes
   - `docs:` - Documentation updates

## Important Notes

- **Paper Cards inventory** (`cards/` app): Catalogue physical MTG cards at `/cards/`
  - Add a card by name at `/cards/add/` — uses Scryfall fuzzy named-card API to fetch metadata
  - Card list (`/cards/`) is sortable by column and searchable; rows show mana icons via `mana_icons` template filter
  - `PaperCard` is independent of the Arena `cards` table — tracks any physical card by name
  - No Celery, no image upload, no phash — fully synchronous Scryfall lookups
- **Import logging**: Comprehensive logging added to import process for debugging
  - Logger name: `stats.views`
  - INFO level: Match-level progress, import summaries (including deck name/format updates)
  - DEBUG level: Detailed per-operation logging (deck creation, card lookups, bulk creates)
  - ERROR level: Full exception tracebacks with context data
  - Fixed bug: LifeChange model uses `change_amount` field (was incorrectly `change`)
- **Deck versioning and analysis history**: Deck name/format are kept in sync on every import
  - `_import_match` updates `Deck.name` and `Deck.format` whenever Arena reports a change (not just on creation)
  - `_analyze_snapshot(snapshot)` in `stats/views/decks.py` computes analysis for any `DeckSnapshot`; used in both `deck_detail` and `deck_history` views
  - The deck history page (`/deck/<id>/history/`) shows a collapsible analysis section for every version, so users can track how suggestions changed as the deck evolved
- **Card image caching**: Deck gallery view downloads and caches Scryfall card images locally
  - Images stored in `data/cache/card_images/`
  - One-click batch download for all cards in a deck
  - Progress indicator shows cache status
  - Images served via Django static files
- **Card data management**: Web UI at `/card-data/` shows download status and triggers Scryfall bulk data download
  - Displays index status, card count, file size, last download date
  - One-click download (or force re-download) with progress feedback
  - Shows local database card count vs. Scryfall index
- **Web-based imports**: Log imports available via web UI at `/import/` (uses temporary file upload)
  - Handles file uploads, shows progress messages, redirects to import history
  - Helper functions in `views.py` mirror CLI command logic
- **Timezone handling**: Timestamps stored in UTC, displayed in `America/New_York` (Eastern Time)
  - Change `TIME_ZONE` in `settings.py` for different local timezone
  - Parser stores all timestamps with `timezone.utc`
  - Django's `USE_TZ = True` enables timezone-aware datetimes
- **Log file size**: Player.log can be 100MB+; parser uses generators for memory efficiency
- **Match ID stability**: Arena reuses match_ids occasionally; check timestamps to detect
- **Scryfall updates**: Re-run `download_cards` periodically for new card sets
- **SQLite limitations**: Consider PostgreSQL for production (add `psycopg2-binary` from `[postgres]` extra)
- **CSS linting**: Requires `npm install` for stylelint; Python linting works without Node.js
- **Migration conflicts**: Always pull latest before `makemigrations`

## Troubleshooting

- **Import fails silently**: Check `match_id` isn't already in database
- **Card names show as "Unknown"**: Run `make download-cards`
- **Parser crashes**: Validate log format with `log_parser.validate_log_file()`
- **Test failures**: Ensure migrations are up to date (`make migrate`)
- **Stylelint errors**: Run `npm install` first, or skip CSS checks during development

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/why-pengo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
