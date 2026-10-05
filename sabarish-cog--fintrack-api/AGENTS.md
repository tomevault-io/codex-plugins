-  Remeber to Save all the prompts in a separate file for reference.

### 1. Technology Stack Declaration
- Java 21
- Spring Boot 3
- Maven build system
- PostgreSQL database
- Lombok for boilerplate reduction
- JUnit 5 + Mockito for tests
- GitHub Actions for CI/CD

### 2. Architecture Conventions
- Layered architecture: `controller` → `service` → `repository`
- Domain model objects in `model`
- DTOs for external API contracts
- Use `@Service`, `@Repository`, `@RestController`
- Keep controllers thin; business logic belongs in services
- Transaction boundaries managed at service layer
- API versioning via path (`/api/v1/...`)
- Configuration in `application.yaml` and environment variables

### 3. Coding Standards
- Naming
  - Classes: `PascalCase`
  - Methods and variables: `camelCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Packages: `lowercase`
- Type annotations
  - Prefer concrete types where appropriate
  - Use `Optional<T>` for nullable return values
  - Avoid raw types and unchecked casts
- Logging
  - Use `org.slf4j.Logger`
  - Log at appropriate levels: `DEBUG`, `INFO`, `WARN`, `ERROR`
  - Do not log sensitive data
  - Add context in log messages (`transactionId`, `userId`)

### 4. Security Rules
- Validate all input at the edge
- Use parameterized queries / JPA to prevent SQL injection
- Enforce authentication and authorization for protected endpoints
- Sanitize or encode data before output
- Do not store secrets in source control
- Use HTTPS-only communication
- Handle exceptions gracefully without leaking internals

### 5. Testing Expectations
- Unit tests for service and utility logic
- Integration tests for repositories and controller endpoints
- Use test data builders or fixtures for readability
- Achieve meaningful coverage for business-critical behavior
- Run tests in CI on every pull request
- Prefer deterministic, isolated tests over brittle integration-only tests
- Make sure coverage stays above 80%, especially if new code is added, add tests side-by-side.

---
> Source: [Sabarish-Cog/fintrack-api](https://github.com/Sabarish-Cog/fintrack-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-05 -->
