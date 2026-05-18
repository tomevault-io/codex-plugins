This is a **TypeScript-based Node.js** repository designed with **Clean Architecture**, **DDD**, **SOLID principles**, 
and strong testing practices (**TDD** and **BDD**). 
Please follow these guidelines when contributing or generating code using GitHub Copilot or AI coding agents.

## Code Standards

### Required Before Each Commit
- Run `npm run lint` to ensure the code adheres to our Biome configuration.
- Ensure that all `*.spec.ts` files pass via `npm test`.

### Development Flow
- Build: `npm run build`
- Test: `npm run test`

## Repository Structure
- `src/contexts/[context]/domain/`: Contains Entities, Value Objects, Aggregates, and domain interfaces.
- `src/contexts/[context]/application/`: Contains Use Cases (Commands, Queries), DTOs, Policies, and Mappers.
- `src/contexts/[context]/infrastructure/`: Repositories, Adapters, External Services (when needed).
- `src/contexts/shared/`: Cross-cutting concerns such as base classes, value objects, utilities.
- `tests/contexts/[context]/`: Unit, integration and BDD-style tests organized by context (Vitest).

## Security and Best Practices
- Follow OWASP Node.js best practices (`docs/recommendations/rest-security.md` and `docs/recommendations/node-top-10-security.md`).
- Use Helmet, CORS, and rate limiting middleware in all API layers.
- Avoid using `any` unless strictly necessary. Prefer strict typings and utility types.
- Keep secrets, tokens, and config values outside of source files using env variables and secure stores.
- Validate all incoming inputs at the edge (presentation layer) using zod, joi, or a custom validator.

## Testing & Documentation
- Write unit tests for all use cases and domain logic.
- Use **Given-When-Then** pattern in tests.
- Use `supertest` or similar for HTTP integration tests.
- Use `@openapi` annotations or decorators to document endpoints.
- Use helpers/mocks for creating test fixtures in `tests/fixtures` and `tests/mocks`.

## Anti-Patterns to Avoid
- Avoid circular dependencies between layers.
- Avoid leaking domain logic into infrastructure or presentation layers.
- Avoid business rules inside controllers or route handlers.
- Avoid using magic strings/numbers. Use constants or enums.
- Do not return raw database models to the application layer.

## Key Guidelines
1. Use **Dependency Inversion**: Domain and application layers should depend only on abstractions.
2. Use **Vertical Slices** per feature (modular design): One folder per feature/module, with all layers inside.
3. **Use Cases** should contain application logic and communicate only via **DTOs** (no Entities returned).
4. Domain validation returns **Either<Value, Error>** (value object / entity creation and use case results).
5. **Repositories** are interfaces in the domain layer and implemented in infrastructure.
7. Follow **SRP** in all code units: a file should have a clear, single purpose.

## References
- Clean Architecture by Robert C. Martin
- Domain-Driven Design by Eric Evans
- Node.js Security Checklist (OWASP)
- REST API Design Best Practices (OpenAPI)

---
> Source: [carlosazaustre/hotel-booking-platform](https://github.com/carlosazaustre/hotel-booking-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
