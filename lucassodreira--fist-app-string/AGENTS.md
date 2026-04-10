# Spring Boot Copilot Instructions

## Project Overview
This is a minimal Spring Boot 3.5.4 web application (`fist-string-app`) built with Java 21 and Maven. The project uses Spring Boot's auto-configuration patterns and follows standard Maven directory structure.

## Key Technologies & Dependencies
- **Spring Boot 3.5.4** with `spring-boot-starter-web`
- **Java 21** as target version
- **Lombok** for code generation (with proper annotation processor configuration)
- **Spring Boot DevTools** for development hot-reload
- **JUnit 5** via `spring-boot-starter-test`

## Development Workflows

### Building & Running
- Use Maven wrapper: `.\mvnw.cmd` (Windows) or `./mvnw` (Unix/Linux)
- Build: `.\mvnw.cmd clean compile`
- Run: `.\mvnw.cmd spring-boot:run`
- Test: `.\mvnw.cmd test`
- Package: `.\mvnw.cmd package` (creates JAR in `target/`)

### Hot Reload
Spring Boot DevTools is configured for automatic restart on classpath changes. Simply recompile classes and the application will restart automatically.

## Project Structure Conventions

### Package Organization
- Root package: `com.example.fist_string_app`
- Main class: `FistStringAppApplication` (standard Spring Boot naming)
- Follow Spring Boot's component scanning from root package

### Maven Configuration
- Uses Spring Boot parent POM for dependency management
- Lombok annotation processor properly configured in `maven-compiler-plugin`
- Standard Maven directory layout: `src/main/java`, `src/test/java`, `src/main/resources`

### Configuration
- Primary config: `src/main/resources/application.properties`
- Application name: `fist-string-app` (defined in properties)
- Static web assets: `src/main/resources/static/` (currently empty)
- Templates: `src/main/resources/templates/` (currently empty)

## Development Patterns

### Adding New Components
When adding Spring components, place them in or under the `com.example.fist_string_app` package to ensure auto-discovery:
- Controllers: Create `@RestController` or `@Controller` classes
- Services: Use `@Service` annotation
- Repositories: Use `@Repository` annotation
- Configuration: Use `@Configuration` classes

### Lombok Usage
Project is configured for Lombok - use annotations like `@Data`, `@Builder`, `@Slf4j` etc. The annotation processor is properly configured in the build.

### Testing
- Test classes in `src/test/java/com/example/fist_string_app/`
- Use `@SpringBootTest` for integration tests
- Basic context loading test exists in `FistStringAppApplicationTests`

## Environment Notes
- **Windows development**: Use `mvnw.cmd` for Maven commands
- **Git line endings**: Configured in `.gitattributes` (LF for mvnw, CRLF for .cmd files)
- **IDE compatibility**: Standard Maven project structure works with IntelliJ, Eclipse, VS Code

## Next Steps for AI Agents
This is a minimal starter project. When adding features:
1. Create REST controllers in the root package
2. Add service layers for business logic
3. Configure additional Spring Boot starters as needed (JPA, Security, etc.)
4. Update `application.properties` for new configuration requirements
5. Add integration tests using `@SpringBootTest` patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LucasSodreira)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/LucasSodreira)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
