# Project Guidelines

## Code Style
- Follow existing FastAPI and Pydantic typing style used in app code.
- Keep endpoint handlers asynchronous (`async def`) unless there is a clear CPU-bound reason.
- Preserve request/response model separation in [app/models/todo.py](app/models/todo.py).

## Architecture
- App entrypoint is [app/main.py](app/main.py): creates `FastAPI` app and includes routers.
- API routes live in [app/routers/](app/routers/) and should contain HTTP behavior only.
- Data schemas live in [app/models/](app/models/) using Pydantic models.
- Storage currently uses in-memory globals in [app/storage/todo.py](app/storage/todo.py).
- Keep this boundary: router -> models/storage. Avoid business logic in `main.py`.

## Build and Test
- Run server: `uv run uvicorn app.main:app --reload`
- Python requirement is 3.12+ (see [pyproject.toml](pyproject.toml)).
- If adding tests, prefer `pytest` and document the exact command in [README.md](README.md).

## Conventions
- TODO endpoints are under `/todos` in [app/routers/todo.py](app/routers/todo.py).
- Validate query/body constraints with `Query(...)` and `Field(...)`.
- Use `exclude_unset=True` for partial updates as shown in [app/routers/todo.py](app/routers/todo.py).
- Raise `HTTPException` with explicit status codes for not-found and validation paths.
- Remember storage is process-local and resets on restart; do not assume persistence.

## References
- Start here for run command and quick setup: [README.md](README.md)
- Learning notes are in [notes/day01-fastapi 시작하기.md](notes/day01-fastapi%20시작하기.md) and [notes/day02-의존성(DI).md](notes/day02-%EC%9D%98%EC%A1%B4%EC%84%B1(DI).md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvmbr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dvmbr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
