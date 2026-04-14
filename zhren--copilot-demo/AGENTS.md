# Copilot Instructions

## Project Summary

This is a Spring Boot REST API application (`copilot-demo`) that exposes two endpoints:
- `GET /api/health` — returns `{"status": "UP"}`
- `GET /api/files/download?filename=<name>` — serves files from a configurable upload directory

## Tech Stack

- **Language:** Java 17
- **Framework:** Spring Boot 3.2.2 (spring-boot-starter-web)
- **Build tool:** Maven 3 (`mvn`)
- **Testing:** JUnit 5, Spring MockMvc (`@WebMvcTest`)

## Project Layout

```
pom.xml                              Maven project descriptor
src/main/java/com/example/demo/
  DemoApplication.java               Spring Boot entry point (@SpringBootApplication)
  HealthController.java              GET /api/health
  FileDownloadController.java        GET /api/files/download
src/main/resources/
  application.properties             app config (port=8080, app.upload-dir)
src/test/java/com/example/demo/
  HealthControllerTest.java          MockMvc tests for HealthController
  FileDownloadControllerTest.java    MockMvc tests for FileDownloadController
```

## Build & Test Commands

```bash
# Compile and package (skipping tests)
mvn package -DskipTests

# Run all tests
mvn test

# Run the application
mvn spring-boot:run
```

The application starts on port **8080** by default.

## Key Conventions

- All REST controllers live in `com.example.demo` and use `@RestController` + `@RequestMapping("/api")`.
- Tests use `@WebMvcTest` (slice tests) with `MockMvc` — do not use `@SpringBootTest` for controller unit tests.
- The upload directory for file downloads is configured via `app.upload-dir` (defaults to `uploads`). Tests override this with `@TestPropertySource`.
- Filenames in `/api/files/download` are validated against `[\\w.\\-]+` to prevent path traversal.
- Always run `mvn test` after making changes to verify nothing is broken.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZHRen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
