# Ocean View Resort Backend

## Architecture Overview
- **Framework**: Spring Boot 4.0.1 with Java 17
- **Architecture**: MVC pattern with layered services (Controller → Service → Repository → Model)
- **Database**: MySQL with JPA/Hibernate (DDL auto-update enabled)
- **Frontend**: Static HTML/JS served from `src/main/resources/static/`
- **Package**: `com.oceanviewresort` with subpackages: `controller`, `service`, `repository`, `model`, `config`, `security`

## Key Components
- **Authentication**: Basic login/register via `AuthController` and `AuthService`; uses custom `PasswordEncoder` (implemented with BCrypt)
- **Reservations**: CRUD operations via `ReservationController`; bill calculation in `ReservationService` based on room type and nights
- **Models**: `User` (username/password), `Reservation` (guest details, dates, roomType)
- **Security**: Spring Security configured in `SecurityConfig` to permit API endpoints; JWT utility class exists but is empty

## Conventions & Patterns
- **Entities**: JPA-annotated with `@Entity`, `@Table`, `@Id`, `@GeneratedValue`; manual getters/setters
- **Repositories**: Extend `JpaRepository`; custom methods like `findByUsername` in `UserRepository`
- **Services**: `@Service` with constructor injection; business logic like bill calculation using `ChronoUnit.DAYS.between`
- **Controllers**: `@RestController` with `@RequestMapping`; use `@RequestBody` for POST, `@PathVariable` for GET
- **Bill Rates**: Hardcoded in `ReservationService.calculateBill()` - Single: 8000/night, Double: 12000/night, Suite: 20000/night
- **Error Handling**: Basic try-catch in services; controllers return plain strings for auth responses
- **Frontend Integration**: JS `fetch()` calls backend APIs; redirects on success (e.g., login to dashboard.html)

## Database Configuration
- **Connection**: `jdbc:h2:mem:oceanview` (username: sa, password: password) - in-memory H2 database for demo
- **Properties**: `spring.jpa.show-sql=true`, `hibernate.format_sql=true` for debugging
- **H2 Console**: Enabled at `http://localhost:8081/h2-console` (JDBC URL: jdbc:h2:mem:oceanview)

## Development Workflow
- **Build**: Use Maven wrapper `./mvnw clean compile` or `./mvnw install`
- **Run**: `./mvnw spring-boot:run` (starts on port 8081)
- **Access**: Open `http://localhost:8081/` in browser for the login page
- **Prerequisites**: None - uses in-memory H2 database
- **Testing**: Unit tests in `src/test/java/` using Spring Boot test starters
- **Debugging**: Check SQL logs in console; frontend errors in browser dev tools

## Important Notes
- Auth endpoints return plain text responses ("Login Successful", "Invalid Credentials")
- Reservation dates use `LocalDate`; bill calculation assumes check-out > check-in
- Custom `PasswordEncoder` throws `UnsupportedOperationException` - implement with BCrypt or similar
- No CORS or security filters configured; all endpoints currently open
- Static files served directly; no templating engine used</content>
<parameter name="filePath">c:\Users\USER\Downloads\Backend (1)\.github\copilot-instructions.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pavanirathnatunga99-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
