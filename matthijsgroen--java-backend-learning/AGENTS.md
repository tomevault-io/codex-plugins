# Spring Dashboarding Project - AI Coding Instructions

## Project Overview
This is a Spring Boot 4.0.2 dashboarding application using **Java 25** with PostgreSQL, organized as a modular monolith using **Spring Modulith**. The architecture emphasizes module boundaries through package structure.

## Architecture & Module Structure

### Module Organization
- **Package = Module boundary**: Each package under `nl.kabisa.dashboarding` represents a distinct module
- Current modules:
  - `restservice`: REST API endpoints (example: greeting service)
  - `dashboard`: JPA entity management for dashboards with PostgreSQL/JSONB
  - `widget`: Widget management with encrypted secrets and configuration validation
- **Spring Modulith enforces module boundaries** - modules communicate via well-defined interfaces, not direct package access

### Key Technologies
- Spring Boot 4.0.2 with Spring Modulith 2.0.2
- Java 25 (uses modern features like records)
- PostgreSQL 16 with JSONB support
- JPA/Hibernate with automatic schema updates (Jackson databind for JSON support)
- Springdoc OpenAPI 3.0.0 for auto-generated API documentation
- Maven wrapper for builds (use `./mvnw`, not global Maven)

## Development Workflows

### Database Setup
```bash
docker compose up # Starts PostgreSQL container
```
Database credentials in [application.properties](src/main/resources/application.properties): `dashboarduser/dashboardpass` on `localhost:5432/dashboarddb`

### Running the Application
```bash
./mvnw spring-boot:run
```
Or use VS Code's "Run: DashboardingApplication" terminal configuration.

### Testing
```bash
./mvnw test
```
- Use `@SpringBootTest` with `@AutoConfigureMockMvc` for integration tests
- Examples: 
  - [GreetingControllerTest.java](src/test/java/nl/kabisa/dashboarding/restservice/GreetingControllerTest.java)
  - [DashboardControllerTest.java](src/test/java/nl/kabisa/dashboarding/dashboard/DashboardControllerTest.java)

## Code Conventions & Patterns

### REST Controllers
- Place in module-specific packages (e.g., `restservice`)
- Use `@RestController` with `@GetMapping`/`@PostMapping`
- Example pattern in [GreetingController.java](src/main/java/nl/kabisa/dashboarding/restservice/GreetingController.java)

### DTOs & Value Objects
- **Use Java records for immutable DTOs**: `public record Greeting(long id, String content)`
- See [Greeting.java](src/main/java/nl/kabisa/dashboarding/restservice/Greeting.java) for reference

### JPA Entities
- Standard JPA annotations with lifecycle callbacks (`@PrePersist`, `@PreUpdate`)
- Use `@JdbcTypeCode(SqlTypes.JSON)` with `columnDefinition = "jsonb"` for PostgreSQL JSON columns
- UUID primary keys with `@GeneratedValue(strategy = GenerationType.UUID)`
- Soft deletes via `deletedAt` timestamps
- Example: [Dashboard.java](src/main/java/nl/kabisa/dashboarding/dashboard/Dashboard.java)

### Widget Security & Serialization
- Widget secrets are encrypted/decrypted in entity lifecycle callbacks
- Use `EncryptionUtil` with PBKDF2-derived AES keys
- Reuse a shared `ObjectMapper` (static or injected); do not instantiate per call
- Catch specific exceptions (e.g., `JsonProcessingException`, `GeneralSecurityException`) with descriptive messages

### Repositories
- Extend `JpaRepository<Entity, ID>` interface
- Mark with `@Repository` annotation
- Example: [DashboardRepository.java](src/main/java/nl/kabisa/dashboarding/dashboard/DashboardRepository.java)

## API Documentation

### Swagger/OpenAPI Support
- Auto-generated OpenAPI documentation via Springdoc OpenAPI 3.0.0
- JSON endpoint: `http://localhost:8080/api-docs`
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- Configured in [application.properties](src/main/resources/application.properties)

## Important Notes
- **Always use `./mvnw`** (Maven wrapper) instead of system Maven for consistency
- Database schema auto-updates via `spring.jpa.hibernate.ddl-auto=update` - suitable for learning, not production
- Spring DevTools enabled for automatic reloading during development
- Jackson databind is required for JSON/JSONB mapping with Hibernate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matthijsgroen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/matthijsgroen)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
