
# FastAPI rules

- Use type hints for all function parameters and return values
- Use Pydantic models for request and response validation
- Use appropriate HTTP methods with path operation decorators (@app.get, @app.post, etc.)
- Use dependency injection for shared logic like database connections and authentication
- Use background tasks for non-blocking operations
- Use proper status codes for responses (201 for creation, 404 for not found, etc.)
- Use APIRouter for organizing routes by feature or resource
- Use path parameters, query parameters, and request bodies appropriately

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlbanAndrieu)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/AlbanAndrieu)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
