# GitHub Copilot Instructions

## Project Overview

**Invest72** is a Spring Boot application for investment return calculation and financial product management. The domain is Korean finance — users create and manage investment products (정기예금, 정기적금) and calculate returns with various tax scenarios.

---

## Tech Stack

| Category | Technology |
|----------|-----------|
| Language | Java 17 |
| Framework | Spring Boot 3.5.11 |
| Database | PostgreSQL (H2 for tests) |
| ORM | Spring Data JPA / Hibernate |
| Authentication | OAuth2 / OIDC |
| Caching | Caffeine Cache |
| Validation | Jakarta Bean Validation + custom validators |
| Build | Gradle |
| Utilities | Lombok, Jackson |

---

## Architecture

The project follows a **Hexagonal (Ports & Adapters)** architecture with a clear layered structure per bounded context.

### Package Structure

```
co.invest72/
├── investment/          # Investment calculation engine
│   ├── config/
│   ├── application/     # Use cases, DTOs, factories
│   ├── domain/          # Core domain logic (investments, interest, tax, amount)
│   └── presentation/    # REST controllers, request/response objects
│
├── financial_product/   # Financial product CRUD
│   ├── domain/          # Entities (FinancialProduct, DepositProduct, SavingsProduct, CashProduct)
│   ├── application/     # Services
│   ├── infrastructure/  # JPA repository implementations
│   └── presentation/    # REST controllers, DTOs
│
├── user/                # User management
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
│
├── security/            # OAuth2 config, PrincipalUser, success handler
└── common/              # Cross-cutting: cache config, exception handling, validation, time
```

### Layers

1. **Presentation** — REST controllers (`*RestController`), request/response DTOs
2. **Application** — Services, use-case DTOs, factories
3. **Domain** — Entities, value objects, repository interfaces, enums, strategy/policy classes
4. **Infrastructure** — JPA repository implementations, adapters

---

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Controller | `*RestController` | `FinancialProductRestController` |
| Service | `*Service` | `FinancialProductService` |
| Repository interface | `*Repository` | `FinancialProductRepository` |
| JPA repository | `Jpa*Repository` | `JpaFinancialProductRepository` |
| Repository adapter | `*RepositoryImpl` | `FinancialProductRepositoryImpl` |
| Application DTO | `*Dto` | `FinancialProductDto` |
| Request object | `*Request` | `CreateFinancialProductRequest` |
| Response object | `*Response` | `FinancialProductResponse` |
| Value Object | Descriptive noun | `ProductAmount`, `AnnualInterestRate` |

---

## Code Conventions

### Lombok Usage

- Use `@RequiredArgsConstructor` for constructor-based dependency injection (not `@Autowired`).
- Use `@NoArgsConstructor(access = AccessLevel.PROTECTED)` on JPA entities.
- Use `@SuperBuilder` for entities that participate in inheritance hierarchies.
- Use `@Getter` (not `@Data`) on entities and value objects.

### Dependency Injection

Always inject via constructor (Lombok `@RequiredArgsConstructor`), never via field injection.

### JPA Entities

- Mark abstract parent entities with `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` and `@DiscriminatorColumn`.
- Embed value objects with `@Embedded` / `@Embeddable`.
- Never use `public` setters on entities; use domain methods to mutate state.

### Validation

- Use Jakarta Bean Validation (`@NotNull`, `@NotBlank`, `@Min`, `@Max`) on request objects.
- Apply `@Valid` in controller method parameters.
- Define custom validators with `@Constraint` annotation in `common/validation/`.
- Existing custom annotations: `@FinancialAmount`, `@FinancialRate`, `@FinancialMonths`, `@PaymentDay`, `@EnumValid`.

### Exception Handling

- All exceptions are handled centrally in `GlobalExceptionHandler` (`@RestControllerAdvice` in `common/exception/`).
- Throw `IllegalArgumentException` for business rule violations.
- Return `ErrorResponse` with field-level details for validation errors.
- Do **not** add `try/catch` blocks in controllers or services for standard validation/business errors.

### Security

- Access the authenticated user via `@AuthenticationPrincipal PrincipalUser principalUser` in controllers.
- Public API paths are configured in `OAuth2LoginSecurityConfig`; update that class when adding new public endpoints.
- CSRF is disabled; do not re-enable it.

### Caching

- Use `@Cacheable` / `@CacheEvict` with cache names defined in `CacheConfig`.
- Cache names: `userMe`, `productSummary`, `productDetail`, `productCalculate`.
- Always evict related caches when mutating data.

### Time

- Use `LocalDateProvider` (abstraction in `common/time/`) instead of calling `LocalDate.now()` directly. This enables deterministic testing.

---

## Domain Enums

| Enum | Values |
|------|--------|
| `InvestmentType` | `DEPOSIT`, `SAVINGS`, `CASH` |
| `InterestType` | `SIMPLE`, `COMPOUND` |
| `TaxType` | `NON_TAX`, `STANDARD`, `BENEFIT` |

---

## Testing

- **Framework**: JUnit 5 (Jupiter) + AssertJ
- **Test database**: H2 in-memory
- **Test location**: mirrors source structure under `src/test/java/`
- Use `@DisplayName` to describe test intent in Korean or English.
- Use the Builder pattern for test object construction.
- Unit test domain logic independently from Spring context where possible.
- Use `@BeforeEach` for shared setup.

---

## Secrets & Configuration

- Secrets live in the `invest72-config` Git submodule as `application-secret.yaml`.
- Run `./gradlew copySecret` to copy secrets into `src/main/resources/` before building locally.
- **Never commit** `application-secret.yaml` or any credentials to this repository.
- Environment variables expected in production: `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`.

---

## Docker

- `docker-compose.local.yaml` — local development
- `docker-compose.yaml` — default
- `docker-compose.production.yaml` — production

Use `docker-compose -f docker-compose.local.yaml up` for local development with PostgreSQL.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yonghwankim-dev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/yonghwankim-dev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
