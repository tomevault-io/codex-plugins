# Project Guidelines

## Code Style
- **Language**: Python 3.12+
- **Frameworks**: FastAPI, SQLAlchemy 2.0+ (async via `asyncpg`), Pydantic V2 for data validation.
- **Async**: Use asynchronous operations exclusively, particularly for database sessions and route handlers.
- **Primary Keys**: Use `uuid.UUID` for Primary Keys across all database models by default.

## Architecture
Clean layered architecture following the repository pattern:
- **`src/api/`**: FastAPI route handlers and endpoint definitions. Keep routes thin. Choose appropriate REST patterns (e.g., nested routes `/videos/{id}/chunks` for strict parent-child relationships).
- **`src/schemas/`**: Pydantic V2 models for request/response validation. Always enable `model_config = {"from_attributes": True}` for proper ORM serialization.
- **`src/services/`**: Business logic layer.
- **`src/repositories/`**: Data access layer (repository pattern abstraction). Isolate all SQLAlchemy `select`, `insert`, `update`, `delete` queries here.
- **`src/models/`**: SQLAlchemy 2.0 declarative models (strict use of `Mapped` and `mapped_column`).
- **`src/database/`**: Database configuration (`config.py`), session management (`session.py`), and auto-seeding logic (`seed.py`).
- **`src/utils/`**: Shared utility functions and helpers.

## Build and Developer Environment
- **Package Manager**: `uv`
- **Docker Compose**: The application runs entirely via Docker Compose (`backend`, `db` as Postgres 16, and `adminer`).
- **Run the Stack**: `docker compose up -d`
- **Hot-Reloading (Recommended)**: Use `docker compose watch` to automatically sync local Python changes directly into the container without rebuilding images.
- **Database Initialization**: `Base.metadata.create_all` and CSV seeding are automatically handled during the FastAPI lifespan application startup in `main.py`.

## Conventions
- **Routing**: Delegate business logic strictly to `services/` and database access to `repositories/`. Avoid direct `session.execute` within `api/`.
- **Database Modifiers**: Evaluate when to use `session.commit()` (e.g., in services for multi-step transactions rather than deep inside repositories).
- **Networking**: Default database connections refer to internal Docker networking (`db:5432`), not `localhost:5432`.

*(Note: See `README.md` and `compose.yaml` for full configuration details.)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duylw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
