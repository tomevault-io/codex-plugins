# FastAPI Base Project

FastAPI Base is a production-ready backend template built with FastAPI, SQLAlchemy, Alembic, Redis caching, Celery, and PostgreSQL. The project uses Docker Compose for local development and provides comprehensive tooling for code quality, testing, and database migrations.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Installation

- Install `uv` package manager: `pip install uv`
- Install `pre-commit`: `pip install pre-commit`
- Ensure Docker and Docker Compose are available
- Copy `.env.example` to `.env` and configure environment variables

### Bootstrap and Setup

1. **Install dependencies** (takes 10-30 seconds):

   ```bash
   cd fastapi-base/
   uv sync
   ```

2. **Set up pre-commit hooks**:

   ```bash
   uvx pre-commit install
   ```

3. **Configure environment**:

   ```bash
   cp .env.example .env
   # Edit .env with appropriate database and Redis URLs
   ```

### Docker-Based Development

**IMPORTANT**: Docker builds may fail in sandboxed environments due to TLS certificate issues. If you encounter `invalid peer certificate: UnknownIssuer` errors, this is a known limitation and the instructions will note alternative approaches.

1. **Build and run with Docker Compose** (takes 2-5 minutes if successful):

   ```bash
   make build    # NEVER CANCEL: May take up to 5 minutes in normal environments
   ```

2. **Start services**:

   ```bash
   make up       # Starts all services (FastAPI, PostgreSQL, Redis, Celery, Beat)
   ```

3. **Access the application**:
   - API: `http://localhost:8666/v1/ping`
   - Documentation: `http://localhost:8666/docs`
   - OpenAPI: `http://localhost:8666/v1/`

### Native Development (Alternative to Docker)

If Docker builds fail due to network restrictions:

1. **Install dependencies**:

   ```bash
   cd fastapi-base/
   uv sync
   ```

2. **Run linting and formatting** (takes 5-10 seconds each):

   ```bash
   uv run ruff check src/        # Linting - very fast
   uv run black --check .        # Code formatting check
   uv run isort --check-only .   # Import sorting check
   uv run mypy src/              # Type checking - takes 5-10 seconds
   ```

3. **Fix linting/formatting issues**:

   ```bash
   uv run black .                # Apply black formatting
   uv run isort .                # Fix import sorting
   ```

4. **Run tests** (takes 5-10 seconds for unit tests):

   ```bash
   # Note: Full tests require running PostgreSQL and Redis
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/ -v
   ```

## Development Workflow

### Making Code Changes

1. **Always start with code quality checks**:

   ```bash
   cd fastapi-base/
   uv run ruff check src/
   uv run mypy src/
   ```

2. **Make your changes** in the appropriate directory:
   - API routes: `src/api/v1/`
   - Business logic: `src/core/`
   - Database models: `src/models/`
   - Repository patterns: `src/repositories/`
   - Pydantic schemas: `src/schemas/`

3. **Apply formatting** after changes:

   ```bash
   uv run black .
   uv run isort .
   ```

4. **Validate changes**:

   ```bash
   uv run ruff check src/
   uv run mypy src/
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/test_routes.py::test_health -v
   ```

### Adding New API Endpoints

1. **Create new route** in `src/api/v1/`
2. **Add corresponding tests** in `tests/`
3. **Update schemas** in `src/schemas/` if needed
4. **Add database models** in `src/models/` if needed
5. **Follow the health endpoint pattern** for caching and error handling

### Database Schema Changes

1. **Modify models** in `src/models/`
2. **Generate migration**:

   ```bash
   # With Docker (if available):
   make alembic-make-migrations "description of change"

   # Alternative: modify Alembic directly
   # Note: Requires running PostgreSQL instance
   ```

### Understanding the Architecture

- **FastAPI Application**: Entry point in `src/main.py`
- **API Versioning**: Routes organized under `src/api/v1/`
- **Dependency Injection**: Common dependencies in `src/api/deps.py`
- **Configuration Management**: `src/core/config.py` with Pydantic settings
- **Database Layer**: SQLAlchemy models and repository pattern
- **Caching**: Redis-based caching with FastAPI-Cache
- **Background Tasks**: Celery worker configuration
- **Testing**: Pytest with async support and fixtures

### Available Make Commands

**Docker-based commands** (may fail in sandboxed environments):

- `make build`: Build and start all services
- `make up`: Start existing services
- `make down`: Stop all services
- `make bash`: Connect to the FastAPI container

**Database commands** (require running PostgreSQL):

- `make alembic-init`: Initialize first migration
- `make alembic-migrate`: Apply migrations
- `make alembic-make-migrations`: Create new migration
- `make alembic-reset`: Reset database
- `make init-db`: Initialize database with sample data

**Code quality commands** (require running Docker container):

- `make lint`: Run ruff linting
- `make black`: Apply black formatting
- `make isort`: Apply import sorting
- `make mypy`: Run type checking
- `make test`: Run full test suite
- `make all`: Run all code quality checks

**Note**: Make commands that depend on Docker containers will fail if Docker build failed. Use `uv run` alternatives instead.

## Validation

### Manual Testing Steps

- **Always run through this complete validation after making changes**:

1. **Code Quality Checks** (NEVER CANCEL - each takes 5-20 seconds):

   ```bash
   uv run ruff check src/        # Should pass with no errors
   uv run black --check .        # Should show "would be left unchanged"
   uv run isort --check-only .   # Should pass or show skipped files
   uv run mypy src/              # May show type errors (16 known issues)
   ```

2. **Basic Functionality Test**:

   ```bash
   # Test that the FastAPI app can import successfully
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run python -c "from src.main import app; print('FastAPI app imports successfully')"
   ```

3. **Test Suite** (with mock database - takes 1-2 seconds):

   ```bash
   # The health endpoint test should pass without database
   POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/test_routes.py::test_health -v
   # Expected output: "1 passed in 0.11s"
   ```

### CI/CD Validation

- **Always run before committing**:

  ```bash
  # If pre-commit works in your environment:
  uvx pre-commit run --all-files

  # Otherwise, run manually:
  uv run ruff check src/
  uv run black --check .
  uv run isort --check-only .
  ```

## Common Issues and Solutions

### Docker Build Failures

- **Symptom**: `invalid peer certificate: UnknownIssuer` during Docker build
- **Solution**: Use native development approach with `uv` instead of Docker
- **Root Cause**: TLS certificate issues in sandboxed environments

### Database Connection Errors

- **Symptom**: `OSError: Multiple exceptions: [Errno 111] Connect call failed`
- **Solution**: Ensure PostgreSQL is running or use mock environment variables for testing
- **For Testing**: Set `POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test"`

### Configuration Errors

- **Symptom**: `ValidationError: POSTGRES_URL Value error, invalid literal for int()`
- **Solution**: Ensure `.env` file has complete database URL or set environment variables
- **Fix**: Set `POSTGRES_URL=postgresql+asyncpg://test:test@localhost:5432/test` in `.env`

### Cache Initialization Errors

- **Symptom**: `AssertionError: You must call init first!` from FastAPI cache
- **Solution**: The application requires Redis for full functionality
- **For Testing**: Use Docker Compose or ensure Redis is running locally

## Time Expectations (MEASURED)

- **NEVER CANCEL**: All operations listed below must be allowed to complete
- `uv sync`: 10-30 seconds (downloads Python interpreter if needed, measured: 30+ seconds first time)
- `uv run ruff check`: < 1 second (measured: 0.089s)
- `uv run black --check`: < 1 second (measured: 0.485s)
- `uv run isort --check-only`: < 1 second (measured: 0.196s)
- `uv run mypy src/`: 5-10 seconds (measured: 7.378s)
- `uv run pytest tests/test_routes.py::test_health`: < 2 seconds (measured: 1.07s)
- `make build` (Docker): 2-5 minutes in normal environments (may fail in sandboxed environments)
- `make up` (Docker): 1-2 minutes after successful build
- **Complete validation sequence**: < 15 seconds total

## Project Structure

### Key Directories

- `src/`: Main application code
  - `api/`: API routes and dependencies
  - `core/`: Configuration and backend utilities
  - `db/`: Database session management
  - `models/`: SQLAlchemy models
  - `repositories/`: Data access layer
  - `schemas/`: Pydantic schemas
- `tests/`: Test files
- `ops/`: Deployment and operations files
- `docs/`: Documentation (currently TODO)

### Key Files

- `pyproject.toml`: Project dependencies and configuration
- `docker-compose.yml`: Local development services
- `Makefile`: Common development commands
- `.env.example`: Environment variable template
- `alembic.ini`: Database migration configuration

## Dependencies and Tools

### Core Dependencies

- **FastAPI**: Web framework
- **SQLAlchemy**: ORM
- **Alembic**: Database migrations
- **Redis**: Caching
- **Celery**: Task queue
- **PostgreSQL**: Database (asyncpg driver)

### Development Tools

- **uv**: Package manager (faster than pip)
- **ruff**: Linting (replaces flake8, isort partially)
- **black**: Code formatting
- **isort**: Import sorting
- **mypy**: Type checking
- **pytest**: Testing framework
- **pre-commit**: Git hooks

## Known Issues

### Type Checking

- MyPy currently reports 16 errors in 6 files
- These are mostly related to third-party library type annotations
- The application runs successfully despite these type errors

### Pre-commit Hooks

- May fail in sandboxed/restricted network environments
- Alternative: Run linting tools manually before committing

### Docker in Restricted Environments

- TLS certificate validation may fail
- Alternative: Use native development with `uv`

## Testing Instructions Validation

To validate these instructions work correctly, run this complete test sequence:

```bash
# 1. Setup (should take 10-30 seconds)
cd fastapi-base/
uv sync

# 2. Copy environment template
cp ../env .env

# 3. Test linting (should take < 10 seconds total)
uv run ruff check src/        # Should pass
uv run black --check .        # Should show files unchanged
uv run isort --check-only .   # Should pass or show skipped
uv run mypy src/              # May show 16 known type errors

# 4. Test app import (should take < 1 second)
POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run python -c "from src.main import app; print('SUCCESS: FastAPI app imports')"

# 5. Test health endpoint (should take < 2 seconds)
POSTGRES_URL="postgresql+asyncpg://test:test@localhost:5432/test" REDIS_URL="redis://redis:6379" uv run pytest tests/test_routes.py::test_health -v

# Expected final output: "1 passed in 0.11s"
```

**If all tests pass, the development environment is ready for use.**

---
> Source: [GabrielVGS/fastapi-base](https://github.com/GabrielVGS/fastapi-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
