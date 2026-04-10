# The Robot Overlord Project Guidelines

## Project Rules

- **RULE #1: NO DOCSTRINGS** - **DO NOT** add docstrings unless specifically requested.
- **RULE #2: ONE ROUTE PER FILE** - Each API endpoint must be in its own file. For example, `list_posts`, `create_post`, `get_post`, etc. should each be in separate files named accordingly.
- **RULE #3: DESCRIPTIVE ROUTE FILENAMES** - Route filenames must match the function name they contain (e.g., `list_posts.py`, `create_post.py`, `get_post.py`) to ensure clarity and discoverability.
- **RULE #4: TRAILING SLASHES IN ROUTES** - All route paths must end with a trailing slash (`/`). For example, use `/users/profile/me/` instead of `/users/profile/me`.
- **RULE #5: DATABASE FUNCTIONS RETURN SCHEMAS** - All database functions that return data should return Pydantic schema objects, not ORM models.
- **RULE #6: DATABASE FUNCTION NAMING** - Database function names should be descriptive and follow consistent patterns:
  - `get_*_by_id`: Retrieve a single item by ID
  - `list_*`: Return a paginated list of items
  - `create_*`: Create a new item
  - `update_*`: Update an existing item
  - `delete_*`: Delete an item
- **RULE #7: ONE FUNCTION PER FILE** - Each database function must be in its own file. For example, `get_user_by_id`, `list_users`, `create_user`, etc. should each be in separate files named accordingly.
- **RULE #8: DESCRIPTIVE FUNCTION FILENAMES** - Database function filenames must match the function name they contain (e.g., `get_user_by_id.py`, `list_users.py`, `create_user.py`) to ensure clarity and discoverability.
- **RULE #9: NO METHODS IN MODEL FILES** - Database models should not contain methods. All database operations should be implemented in the appropriate files in the `db_functions` hierarchy.
- **RULE #10: MODEL-SPECIFIC DB FUNCTIONS** - Each db_function should only directly operate on its own model. Any manipulations of other models should be done by calling those models' respective db_functions.

## Project Vision
- **Concept**: AI-moderated debate platform with satirical Soviet propaganda aesthetic
- **Core Flow**: Users submit posts → AI analysis → APPROVE (post appears) or REJECT (tombstone counter)
- **Theme**: Authoritarian robot overlord ("CITIZEN, YOUR LOGIC REQUIRES CALIBRATION")
- **Tech Stack**: FastAPI+Tortoise ORM backend with integrated frontend rendering, PostgreSQL, OpenAI/Anthropic APIs
- **Key Features**: User accounts with approval/rejection counters, threaded debates, AI moderation pipeline, Soviet-themed UI

## Project Structure
- `backend/`: FastAPI backend (Python)
  - `migrations/`: Tortoise ORM/aerich migrations
  - `src/backend/`: Source code
    - `app.py`: FastAPI application setup
    - `converters/`: Functions to convert between database models and Pydantic schemas
      - `post_to_schema.py`: Converts Post model to PostResponse schema
      - `tag_to_schema.py`: Converts Tag model to TagResponse schema
      - `topic_to_schema.py`: Converts Topic model to TopicResponse schema
      - `user_to_schema.py`: Converts User model to UserSchema
      - `user_session_to_schema.py`: Converts UserSession model to UserSessionSchema
    - `db/`: Database configuration
      - `config.py`: Tortoise ORM/aerich configuration
      - `base.py`: Base model for all database models
      - `models/`: Database models, 1 model per file
    - `db_functions/`: Database access functions organized by entity
      - `users/`: User-related database functions
      - `posts/`: Post-related database functions
      - `topics/`: Topic-related database functions
      - `tags/`: Tag-related database functions
      - `user_sessions/`: User session-related database functions
      - `user_events/`: User event-related database functions
      - `topic_tags/`: Topic-tag relationship database functions
    - `routes/`: API endpoints by feature, one package per feature
    - `schemas/`: Pydantic schemas organized by entity
      - `user.py`: User-related schemas
      - `post.py`: Post-related schemas
      - `topic.py`: Topic-related schemas
      - etc.
    - `tasks/`: Background tasks
    - `utils/`: Shared utilities
  - `tests/`: Test files (mirrors source code directory structure)
  - `pyproject.toml`: Project configuration

- `scripts/`: Shell scripts for workflows
- `plans/`: Markdown files for LLM consumption (technical design documentation)
- `plans/checklists/`: Markdown files for LLM consumption (checklists)
- `justfile`: Command runner
- `LLM_RULES.md`: Central location for AI assistant guidelines
- `.github/workflows/`: CI/CD configuration

### Organization by Technical Concern vs. Feature

#### Routes organized by feature
The backend routes follow a feature-based modular structure, for example:
```
src/backend/
├── app.py             # FastAPI application setup
└── routes/            # API endpoints by feature
    └── health/        # Example feature module
        ├── __init__.py     # Router setup
        ├── check.py        # REST endpoint
        ├── heartbeat.py    # WebSocket endpoint
        └── utils.py        # Feature utilities
```

#### Schemas organized by technical concern
Pydantic schemas are organized by entity type in a centralized `schemas` directory:
```
src/backend/
├── schemas/           # Centralized schemas by entity
    ├── user.py        # User-related schemas
    ├── post.py        # Post-related schemas
    ├── topic.py       # Topic-related schemas
    └── tag.py         # Tag-related schemas
```

**Key principles**:
- Routes organized by feature with separation of concerns (endpoints, utilities)
- Schemas organized by entity type to reduce duplication and improve maintainability
- Database models and functions organized by technical concern
- **Converters** handle the transformation between database models and Pydantic schemas
  - Each converter function is responsible for a specific model-to-schema conversion
  - Converters are used by database functions to return schema objects instead of ORM models
  - This keeps the conversion logic centralized and consistent across the application

### Converters Pattern

- **Purpose**: Converters provide a clean separation between database models and API schemas, ensuring consistent data transformation across the application.

- **Implementation**:
  - Each converter is an async function in its own file under `src/backend/converters/`
  - Converters transform ORM models to Pydantic schemas (e.g., `user_to_schema`, `post_to_schema`)
  - All database functions should use converters to return schema objects instead of raw ORM models
  - Converters handle nested relationships and complex transformations

- **Benefits**:
  - Prevents circular dependencies between models and schemas
  - Centralizes transformation logic for better maintainability
  - Allows for validation and data enrichment during transformation
  - Keeps ORM models focused on database interactions

### Database Functions Pattern

- Database functions should handle all database interactions, including validation and error handling
- Business logic should be kept minimal in database functions
- Each entity's functions are grouped in their own directory under `db_functions/`

## Development
- **Prerequisites**: `git`, `just`, `node`, `uv`, `python` (3.12.10)
- **Key Commands**:
  - `just setup`: Install dependencies
  - `just pre-commit`: Format, lint, test
  - `just uvicorn`: Serve backend in development mode
  - `just gunicorn`: Serve backend in production mode
  - `just ruff-format`: Format backend code
  - `just ruff-check`: Lint backend code
  - `just mypy`: Type check backend (reference implementation)
  - `just pyright`: Type check backend (Microsoft implementation)
  - `just pytest`: Test backend (runs all tests with coverage)

  - `just update-llm-rules`: Update all AI assistant rule files from LLM_RULES.md

- **Database Migration Commands**:
  - `just db-migration-fresh-start`: Reset database, clear migrations, and initialize from scratch (for development)
  - `just aerich-downgrade`: Revert the most recent migration
  - `just aerich-history`: Show migration history
  - `just aerich-migrate`: Create a new migration after model changes
  - `just aerich-upgrade`: Apply pending migrations to the database

- **Running Specific Tests**:
  - From project root: `cd backend && uv run pytest tests/path/to/test_file.py`
  - If already in backend dir: `uv run pytest tests/path/to/test_file.py`
  - Run specific test function: `uv run pytest tests/path/to/test_file.py::test_function_name`

## Python Coding Style

**General principle**: When in doubt, follow PEP 8.

### Code Organization
- **Imports**: Group by source (stdlib → third-party → project), alphabetize, one per line, absolute imports ONLY. NEVER use relative imports. Here is an example:
```python
# Standard library imports
import asyncio
from datetime import datetime

# Third-party imports
from fastapi import APIRouter

# Project-specific imports
from backend.utils.datetime import now
```

- **Formatting**: 88 char line limit, 4 spaces indent, double quotes, two blank lines before top-level definitions
- **Functions**: Type annotations required, trailing commas in multi-line params
```python
async def example(
    param: str,
) -> ReturnType:
    return result
```

### Documentation & Testing
- Avoid writing any docstrings at all.
- **NEVER** add a docstring to the top of a python file unless specifically requested.
- Tests follow Arrange-Act-Assert pattern with 100% coverage required
- Code quality enforced via Ruff (rules: E, F, I, UP, N, B, A, C4, RET, SIM, ERA), Mypy, Pyright, and pre-commit hooks

## Testing

- **Test Coverage**: Aim for high test coverage, especially for repositories and routes
- **Test Organization**: Tests should mirror the structure of the source code
- **Test Naming**: Test functions should be named descriptively, e.g., `test_list_tags_with_search`
- **Checklist Workflow**: When implementing tests using checklists, follow this pattern:
  1. Focus on one test file at a time
  2. Perfect the implementation before moving to the next item
  3. Update the checklist to mark the completed item
  4. Commit the changes with a descriptive message
  5. Repeat for each checklist item
- **Mock Objects**: When testing routes, use proper schema objects for mocks, not raw ORM models
- **Async Testing**: Use `pytest.mark.asyncio` for testing async functions
- **Test Isolation**: Each test should be independent and not rely on the state of other tests

### End-to-End (E2E) Testing

- **Location**: All E2E tests are in the `backend/e2e_tests` directory
- **Purpose**: Verify that the entire application works correctly from the user's perspective
- **Server Error Detection**: E2E tests automatically fail if server errors are detected
  - Captures ERROR level logs during test execution
  - Detects 500 status codes in responses
  - Provides detailed error information for debugging
  - Supports whitelisting expected error patterns
  - Allows ignoring server errors for specific tests with `@pytest.mark.ignore_server_errors`
- **Test Database**: Uses in-memory SQLite database for faster test execution and isolation
- **Key Fixtures**:
  - `server`: Starts the API server for testing
  - `setup_test_db`: Sets up a fresh test database for each test
  - `api_client`: Creates an HTTP client for making API requests
  - `test_user`: Creates a test user and returns user data
  - `authenticated_client`: Creates a client with valid authentication token
- **Test Flow Pattern**: Tests should build complete workflows (e.g., auth flow, post creation/management)
- **Running E2E Tests**: From project root: `cd backend && uv run pytest e2e_tests/`

### Error Handling
- Prefer using exceptions for error handling
- NEVER overload return types (e.g., don't use `Optional[<type>]` with `None` to signal errors)
- Raise specific exceptions rather than generic ones
- Handle exceptions at appropriate levels of abstraction

## Error Handling in Routes

- Use FastAPI's `HTTPException` for client-facing errors
- Include descriptive error messages that align with the project's theme
- Use appropriate HTTP status codes:
  - 400: Bad Request (client error)
  - 401: Unauthorized (authentication required)
  - 403: Forbidden (insufficient permissions)
  - 404: Not Found (resource doesn't exist)
  - 500: Internal Server Error (unexpected server error)
- Log detailed error information for debugging but return sanitized messages to clients

## Schema Design

- **Base and Response Schemas**: Use inheritance to create base schemas (for input) and response schemas (for output)
- **Validation**: Use Pydantic validators for complex validation logic
- **Documentation**: Use Field descriptions for OpenAPI documentation
- **Consistency**: Use similar field names across related schemas
- **List Schemas**: For paginated responses, use consistent list schemas (e.g., `TagList`, `PostList`) with `items` and `count` fields

## Async Programming
- **All API endpoints MUST be async** (`async def`)
- Use async libraries for I/O operations (`asyncio`, async DB clients, `httpx`)
- Avoid blocking operations in async functions
```python
# CORRECT - Always use async for endpoints
async def check() -> HealthCheckResponse:
    """Simple health check endpoint."""
    return build_health_check_response()
```

## CI/CD
- **CI/CD**: GitHub Actions for code quality on push/PR

## MOST IMPORTANTLY: Docstring etiquette

- Docstrings should only be added to functions; not modules!
- If a module needs documentation that would be a sign that it **needs to be broken up into smaller modules**.
- If you put a docstring on a module we will have to keep it up to date which is a waste of time and often is forgotten which leads to confusion.
- This is an application, not a library for external consumption, being built largely by you, the AI assistant.
- Even in functions, docstrings should only exist if they explain something that cannot be explained by the function definition, parameters, and return type.

BAD EXAMPLE (Docstring adds no value):
```python
def check() -> HealthCheckResponse:
    """Simple health check endpoint."""
    return build_health_check_response()
```

GOOD EXAMPLE:
```python
def check() -> HealthCheckResponse:
    return build_health_check_response()
```

BAD EXAMPLE (Docstring adds only liability):
```python
def echo_if_hello(well_named_param: str) -> Optional[str]:
    """
    Simple function that returns a string if the param is "hello".

    Args:
        well_named_param: The param to check.

    Returns:
        Optional[str]: The string "hello" if the param is "hello", otherwise None.
    """
    if well_named_param == "hello":
        return "hello"
    return None
```

GOOD EXAMPLE (Docstring adds value) located in our app's `backend.utils.datetime` module:
```python
from datetime import UTC
from datetime import datetime


def now_utc() -> datetime:
    """
    ALWAYS use this and NEVER use datetime.now() or datetime.utcnow() directly,
    as they are not timezone aware.
    """
    return datetime.now(tz=UTC)

```

## Criteria for Keeping a Docstring

A docstring should **ONLY** be kept if it meets any of the following criteria:
- It explains complex logic that cannot be understood from the function signature alone
- It documents non-obvious side effects or behaviors
- It provides critical context that would be lost if removed
- It cannot be replaced by better naming or type annotations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshSzep)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/joshSzep)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
