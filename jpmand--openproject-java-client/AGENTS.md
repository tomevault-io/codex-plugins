# Copilot Instructions for OpenProject Java Client

## Project Overview

This is a Java HTTP API Client for OpenProject API v3, supporting HAL+JSON format. The library provides:
- Core HAL+JSON model classes (`HalResource`, `Link`, `PagedCollection`)
- Pluggable HTTP client architecture with OkHttp as the default
- Multiple authentication methods (API Key, Basic Auth, OAuth2, OIDC JWT)
- Service interfaces for Work Packages, Users, Time Entries, and Notifications
- Query builder for filters and sorts

## Technology Stack

- **Language**: Java 17
- **Build Tool**: Maven 3.x
- **Testing**: JUnit 5
- **HTTP Client**: Retrofit 3.0.0 with OkHttp 5.3.0
- **JSON Processing**: Jackson 2.18.3
- **CI/CD**: GitHub Actions (Maven workflow)

## Project Structure

```
openproject-java-client/
├── src/
│   ├── main/java/com/github/jpmand/openproject/client/
│   │   ├── api/                    # Client and service interfaces
│   │   │   ├── OpenProjectClient.java
│   │   │   └── services/           # Service implementations
│   │   ├── auth/                   # Authentication providers
│   │   ├── core/                   # Core HAL+JSON model classes
│   │   │   └── model/
│   │   └── http/                   # HTTP client factories
│   └── test/java/                  # Test classes mirroring main structure
├── pom.xml                         # Maven configuration
├── README.md                       # Main documentation
└── AUTHENTICATION.md               # Authentication examples
```

## Build and Test Commands

### Build the project
```bash
mvn clean install
```

### Run tests
```bash
mvn test
```

### Run specific test
```bash
mvn test -Dtest=ClassName
```

### Package without tests
```bash
mvn package -DskipTests
```

## Code Style and Conventions

### General Java Conventions
- Use Java 17 features where appropriate
- Follow standard Java naming conventions:
  - Classes: PascalCase
  - Methods/variables: camelCase
  - Constants: UPPER_SNAKE_CASE
- Use meaningful variable and method names
- Keep methods focused and concise

### Code Organization
- Place classes in appropriate packages based on their purpose
- Service interfaces go in `api/services/`
- Authentication providers go in `auth/`
- Core model classes go in `core/model/`
- HTTP-related utilities go in `http/`

### Testing
- Write JUnit 5 tests for new functionality
- Place tests in the same package structure as main code under `src/test/java/`
- Test class names should end with `Test` (e.g., `OpenProjectClientTest`)
- Use descriptive test method names that explain what is being tested
- Use MockWebServer for integration tests when testing HTTP interactions
- Ensure all tests pass before committing changes

### Documentation
- Add Javadoc comments for public classes and methods
- Document authentication methods in AUTHENTICATION.md if adding new auth types
- Update README.md if adding new major features
- Update CHANGELOG.md following [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format for all changes

## Dependencies

### Adding Dependencies
- All dependencies are managed in `pom.xml`
- Check for existing dependencies before adding new ones
- Use provided scope for compile-only dependencies (e.g., Jackson annotations)
- Keep dependency versions consistent (use properties when possible)
- Retrofit and Jackson are core dependencies; avoid alternatives

### Security
- Do not commit credentials, API keys, or tokens
- Authentication examples should use placeholders
- Review dependency vulnerabilities before adding new libraries

## Common Tasks

### Adding a New Service
1. Create service interface in `api/services/`
2. Define methods following the existing pattern
3. Add service getter in `OpenProjectClient`
4. Write tests in corresponding test package
5. Document usage in README.md if it's a major feature

### Adding a New Authentication Method
1. Implement `AuthProvider` interface in `auth/` package
2. Add tests in `auth/` test package
3. Document usage in AUTHENTICATION.md with examples
4. Update README.md to mention the new auth method

### Working with HAL+JSON
- Use Jackson annotations for JSON mapping
- Extend `HalResource` for resource classes
- Use `PagedCollectionResource` for paginated responses
- Follow the existing pattern for `_links` and `_embedded` handling

## GitHub Actions CI/CD

The repository uses GitHub Actions for continuous integration:
- Workflow file: `.github/workflows/maven.yml`
- Runs on: pushes to `main`, pull requests, manual dispatch
- Java version: 17 (Temurin distribution)
- Maven caching is enabled
- Build command: `mvn -B package --file pom.xml`

## Important Notes

- This library is in active development (version 0.1.0-SNAPSHOT)
- Maintain backward compatibility when possible
- Focus on minimal, surgical changes
- Always run tests before pushing changes
- The project targets Java 17 as the minimum version
- Source encoding is UTF-8

---
> Source: [JpMand/openproject-java-client](https://github.com/JpMand/openproject-java-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-20 -->
