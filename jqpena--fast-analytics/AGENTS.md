# Copilot Instructions for analytics-retrieval

This document provides guidance for AI coding agents working on the `analytics-retrieval` codebase. It outlines the architecture, workflows, and conventions specific to this project to ensure productive contributions.

## Project Overview

The `analytics-retrieval` project is an Analytics API built using FastAPI as the backend framework and TimescaleDB for database management. The project is structured to support modular development and maintainability.

### Key Components

1. **Backend Framework**:
   - The backend is implemented using FastAPI, a modern Python web framework.
   - Key entry points for the API are located in `src/routes/`.

2. **Database**:
   - TimescaleDB is used for time-series data storage and management.
   - Database-related logic is modularized under `src/db/`.

3. **Schemas and Models**:
   - Data schemas are defined in `src/schemas/`.
   - Database models are located in `src/models/`.

4. **Utilities**:
   - Shared utility functions are in `src/utils.py`.

### Data Flow

1. API requests are routed through `src/routes/`.
2. Business logic interacts with database models in `src/models/`.
3. Database queries are executed using helper functions in `src/db/`.
4. Responses are serialized using schemas in `src/schemas/`.

## Developer Workflows

### Building the Project
- Use the `Dockerfile` to build the project container.
- Run `scripts/setup.sh` for initial setup.

### Running the Application
- Use the `scripts/entrypoint.sh` script to start the application.

### Testing
- Test files are located under `tests/`.
- Run tests using `pytest`.

### Database Migrations
- SQL migration scripts are in `db/migrate.sql`.
- Apply migrations manually or integrate them into the CI/CD pipeline.

## Project-Specific Conventions

1. **File Organization**:
   - Follow the modular structure under `src/` for scalability.

2. **Logging**:
   - Use the logger defined in `src/entrypoints/logger.py`.

3. **Error Handling**:
   - Centralized error handling is implemented in `src/routes/`.

4. **Code Style**:
   - Adhere to PEP 8 standards.
   - Use `pre-commit` hooks for linting and formatting.

## Integration Points

- **External Dependencies**:
  - FastAPI for the web framework.
  - TimescaleDB for database management.
- **Cross-Component Communication**:
  - Routes interact with models and schemas for data validation and serialization.

## Examples

### Adding a New Route
1. Define the route in `src/routes/`.
2. Implement the business logic in `src/models/` or `src/db/`.
3. Validate request/response data using schemas in `src/schemas/`.

### Writing a Database Query
1. Add the query logic in `src/db/`.
2. Use helper functions in `src/db/utils.py` for common patterns.

---

This document is a living guide. Update it as the project evolves to ensure it remains accurate and helpful.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jqpena) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
