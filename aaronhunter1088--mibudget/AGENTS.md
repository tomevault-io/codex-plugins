# GitHub Copilot Instructions for miBudget

## Project Overview

miBudget is a personal budgeting web application built with Java Spring Boot that helps users manage their finances by:
- Connecting to multiple banks and investment accounts via the Plaid API
- Tracking and categorizing transactions automatically
- Creating intelligent budgets that learn from user behavior
- Generating monthly financial reports
- Providing a minimalist interface for budget management

## Technology Stack

- **Backend**: Java 18, Spring Boot 2.7.1
- **Web Framework**: Spring MVC with JSP views
- **Security**: Spring Security
- **Database**: MySQL with JPA/Hibernate
- **ORM**: Hibernate 6.1.1.Final
- **Build Tool**: Maven
- **External APIs**: Plaid API for bank integration
- **Logging**: Log4j2
- **Testing**: JUnit Jupiter

## Project Structure

```
src/
├── main/
│   ├── java/com/miBudget/
│   │   ├── MiBudgetApplication.java  # Spring Boot main application
│   │   ├── controllers/              # REST controllers
│   │   ├── core/                     # Core components (Constants, CorsFilter, SecurityConfig)
│   │   ├── daos/                     # Data Access Objects (JPA repositories)
│   │   ├── entities/                 # JPA entities (User, Budget, Transaction, etc.)
│   │   ├── enums/                    # Enumerations
│   │   ├── processors/               # Business logic processors
│   │   ├── services/                 # Service layer
│   │   ├── servlets/                 # Servlet components
│   │   └── utilities/                # Utility classes
│   ├── resources/
│   │   ├── application.properties    # Spring Boot configuration
│   │   ├── hibernate.cfg.xml         # Hibernate configuration
│   │   └── log4j2.properties         # Logging configuration
│   └── webapp/                       # JSP views and static resources
└── test/java/                        # Test files
```

## Coding Standards

### General Guidelines

1. **Java Version**: Use Java 18 features where appropriate
2. **Code Style**: Follow standard Java conventions
3. **Naming**: Use descriptive, camelCase for variables and methods, PascalCase for classes
4. **Comments**: Add JavaDoc for public methods and classes, especially when the purpose isn't immediately obvious
5. **Error Handling**: Use proper exception handling; log errors using Log4j2

### Entity Classes

- Use JPA annotations: `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
- Include `@Transient` for fields that shouldn't be persisted
- Use `@SequenceGenerator` for ID generation with database sequences
- Provide constructors for different use cases (login, registration, etc.)
- Include getter/setter methods (Lombok `@Data` is available but not always used)

Example pattern:
```java
@Entity
@Table(name="table_name")
public class EntityName {
    @Id
    @SequenceGenerator(name="entityname_sequence", sequenceName="entityname_sequence", allocationSize = 1)
    @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="entityname_sequence")
    private Long id;
    
    // fields, constructors, getters/setters
}
```

Note: Sequence names follow the pattern `{tablename}_sequence` (e.g., `users_sequence`, `budgets_sequence`)

### Controllers

- Use `@RestController` for REST endpoints
- Use `@CrossOrigin(origins = "*")` for CORS support
- Inject DAOs via constructor with `@Autowired`
- Use `@RequestMapping` with explicit path and method
- Use Log4j2 for logging (get logger via `LogManager.getLogger(ClassName.class)`)
- Log method entry with `Constants.start` pattern

Example pattern:
```java
@RestController
@CrossOrigin(origins = "*")
public class ControllerName {
    private static final Logger LOGGER = LogManager.getLogger(ControllerName.class);
    
    private final SomeDAO someDAO;
    
    @Autowired
    public ControllerName(SomeDAO someDAO) {
        this.someDAO = someDAO;
    }
    
    @RequestMapping(path="/endpoint", method=RequestMethod.POST)
    public void methodName(HttpServletRequest request, HttpServletResponse response) {
        LOGGER.info(Constants.start);
        LOGGER.info("ControllerName:methodName");
        // implementation
    }
}
```

### DAOs (Data Access Objects)

- Extend `JpaRepository<Entity, Long>` for standard CRUD operations
- Add custom query methods as needed
- Use method naming conventions for query derivation (e.g., `findByCellphone`)

### Services

- Implement service interfaces
- Use `@Service` annotation
- Handle business logic and orchestrate DAO calls

## Database

- Database: MySQL
- Connection configured in `application.properties` and `hibernate.cfg.xml`
- Tables: users, budgets, categories, accounts, transactions, items, rules
- Use Hibernate for ORM
- Sequences for ID generation (e.g., `users_sequence`, `budgets_sequence`)

## Building and Testing

### Build Commands

```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Create WAR file
mvn package

# Create executable JAR with dependencies
mvn clean compile assembly:single
```

### Test Guidelines

- Place tests in `src/test/java/com/miBudget/`
- Use JUnit Jupiter (JUnit 5)
- Test files should end with `Test.java`
- Test utility classes and processors
- Current test coverage includes: TransactionsProcessor, HibernateConnection, DateAndTimeUtility, EmailUtility, Log4j2

## External Integrations

### Plaid API

- Used for secure bank account integration
- Library: `plaid-java` version 3.0.5
- Handle bank connections, transaction retrieval, and account information

## Logging

- Use Log4j2 for all logging
- Configuration in `src/main/resources/log4j2.properties`
- Get logger: `LogManager.getLogger(ClassName.class)`
- Use appropriate log levels: `LOGGER.info()`, `LOGGER.error()`, etc.
- Log method entry/exit and important operations

## Security Considerations

- Spring Security is configured (but currently excluded in main app for development)
- User passwords should be properly hashed (ensure bcrypt or similar is used)
- Session management via HttpSession
- Validate all user inputs
- Never log sensitive information (passwords, API keys)

## Dependencies Management

- All dependencies managed via Maven in `pom.xml`
- Check for version updates carefully
- Maintain compatibility with Spring Boot 2.7.1
- Test thoroughly when updating dependencies

## Common Patterns

1. **Session Management**: Store user object in HttpSession after login
2. **JSON Responses**: Use JSONObject and JSONArray for API responses
3. **Date Handling**: Use DateAndTimeUtility for date operations
4. **Constants**: Use Constants class for common values
5. **Enums**: Use enums for fixed sets of values (e.g., AppType: FREE vs PAID)

## When Working on Code

1. **Adding New Endpoints**: Follow the controller pattern with proper logging and error handling
2. **Adding New Entities**: Include JPA annotations, sequences, getters/setters, and appropriate relationships
3. **Database Changes**: Update Hibernate configuration if schema changes
4. **Adding Tests**: Place in appropriate test package, use JUnit 5
5. **Error Handling**: Always log errors and provide meaningful responses
6. **API Integration**: Follow existing Plaid integration patterns

## Output Directory

- Build artifacts go to `dist/` directory (configured in pom.xml)
- This directory is gitignored

## Git Workflow

- Ignored paths: `.idea/`, `dist/`, `lib/`, `logs/`, `target/`, `.DS_Store`
- Follow clean commit practices
- Test before committing

## Notes for AI Assistants

- This is a personal project demonstrating full-stack Java development skills
- Focus on clean, maintainable code that follows existing patterns
- Prioritize security, especially around user data and bank information
- Maintain consistency with existing code style
- When suggesting changes, consider the Maven build process
- The project uses JSP for views, not a modern frontend framework

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronhunter1088) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
