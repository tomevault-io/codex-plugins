# Copilot Instructions for CocktailBerry-Payment

## Project Overview

CocktailBerry-Payment is a payment and balance management service for the [CocktailBerry](https://github.com/AndreWohnsland/CocktailBerry) cocktail machine project. It provides NFC-based payment options, user account management, and transaction handling for service personnel.

## Tech Stack

- **Backend**: FastAPI (Python web framework for building APIs)
- **Frontend/Admin UI**: NiceGUI (Python-based web app framework)
- **Server**: Uvicorn (ASGI server)
- **Database**: SQLite with SQLAlchemy ORM and Alembic for migrations
- **Python Version**: >= 3.13
- **Package Manager**: uv (modern Python package manager)

## Code Style & Conventions

### Python Standards
- Follow PEP 8 (ruff) with a line length of 120 characters
- Use type annotations for all function parameters and return values
- Use pathlib for file path operations (enforced via ruff's PTH rules)
- Prefer list/dict comprehensions over map/filter

### Naming Conventions
- Use snake_case for functions and variables
- Use PascalCase for classes
- Use UPPER_CASE for constants
- Prefix private methods/attributes with underscore

### Imports
- Group imports: standard library, third-party, local
- Use isort for import ordering
- Use conventional import aliases (e.g., `import numpy as np`)

### Documentation
- Docstrings are optional but encouraged for complex functions
- Use Google-style docstrings when documenting

## Project Structure (Planned)

INFO: Currently here is streamlit as the frontend, but we will probably move to NiceGUI later. Adjust instructions accordingly.

```
cocktailberry-payment/
├── src/
│   ├── backend/           # FastAPI application code
│   │   ├── api/           # FastAPI routes and endpoints
│   │   ├── core/          # Core/internal (config, settings)
│   │   ├── models/        # Pydantic models and database schemas
│   │   ├── services/      # Business logic layer
│   │   ├── db/            # Database connections and queries
│   │   └── utils/         # Helper functions and utilities
│   └── frontend/          # Streamlit admin interface
│       ├── view/          # basic ui components + pages
│       ├── data/          # if needed: data handling functions
├── tests/                 # pytest test files
├── main.py                # Application entry point
└── pyproject.toml         # Project configuration
```

## FastAPI Guidelines

- Use Pydantic models for request/response validation
- Implement proper HTTP status codes
- Use dependency injection for shared resources
- Group related endpoints with APIRouter
- Follow RESTful naming conventions for endpoints
- Use async/await for I/O-bound operations

## Frontend Guidelines

- Keep the UI simple and intuitive for service personnel
- Use build in css classes where possible
- Use session state for maintaining user data across reruns
- Organize pages using NiceGUI's multipage app structure
- Validate user inputs before processing

## Testing

- Use pytest for unit and integration tests
- Aim for good test coverage (pytest-cov)
- Test API endpoints with FastAPI's TestClient
- Mock external dependencies in tests

## Development Workflow

1. Install dependencies: `uv sync`
2. Install pre-commit hooks: `uv run pre-commit install --install-hooks`
3. Run the application: `uv run main.py`
4. Run tests: `uv run pytest`
5. Type checking: `uv run mypy .`
6. Linting: `uv run ruff check .`

## Key Features to Implement

- User account creation and management via NFC
- Balance top-up and deduction
- Transaction history and logging
- Service personnel authentication
- Integration with CocktailBerry main application
- Real-time balance updates

## When Helping with This Project

1. **Always use type hints** for function signatures
2. **Prefer async functions** for API endpoints
3. **Use Pydantic models** for data validation
4. **Follow the existing ruff configuration** for linting rules
5. **Write code compatible with Python 3.13+**
6. **Use uv commands** instead of pip/poetry for package management
7. **Keep the code simple** - this is for service personnel, not developers

---
> Source: [AndreWohnsland/CocktailBerry-Payment](https://github.com/AndreWohnsland/CocktailBerry-Payment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
