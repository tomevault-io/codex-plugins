- Minimize @app.on_event("startup") and @app.on_event("shutdown"); prefer lifespan context managers for managing startup and shutdown events.
- Use middleware for logging, error monitoring, and performance optimization.
- Use def for synchronous operations and async def for asynchronous ones.
- Utilize async/await for non-blocking operations
- Use path operations decorators (@app.get, @app.post, etc.)
- Implement proper error handling with HTTPException
- Use FastAPI's built-in OpenAPI and JSON Schema support
- Use lazy loading techniques for large datasets and API responses.
- Follow RESTful API design principles.
- Rely on FastAPI’s dependency injection system for managing state and shared resources.
- Use SQLAlchemy 2.0 for ORM features, if applicable.
- Ensure CORS is properly configured for local development.
- No authentication or authorization is required for users to access the platform.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dfrojas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dfrojas)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
