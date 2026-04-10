# GitHub Copilot Context - Demo Spring Boot Project

## 📋 Project Overview

**Demo** is a production-grade Spring Boot application demonstrating:
- Concurrent request handling with `CompletableFuture` async patterns
- REST API development with Spring Web MVC
- JPA/Hibernate database integration with Spring Data
- Thread-safe concurrent operations and file handling
- COMPLY API blueprint with asynchronous endpoints
- Custom HTTP server integration (OrderFulfillmentServer)

## 🏗️ Architecture & Tech Stack

### Backend Technologies
- **Java 17+** - Modern Java with latest language features (switch expressions, records, sealed classes)
- **Spring Boot 3.5.6** - Enterprise application framework
- **Spring Data JPA + Hibernate 6.6** - ORM and database abstraction
- **Spring Web** - REST API support
- **MySQL 8.0+ / H2** - Database options (MySQL for production, H2 for testing)
- **Maven 3.6+** - Build automation with central dependency management
- **Lombok 1.18.42** - Boilerplate reduction (annotations like @Getter, @Setter, @RequiredArgsConstructor)

### Key Libraries
- **Jackson** - JSON serialization/deserialization
- **Java WebSocket 1.6.0** - WebSocket support
- **Spring Validation** - Bean validation with JSR-303/380

### Project Version
- **Current**: 0.0.1-SNAPSHOT
- **Group ID**: com.init-spring-bean-mvn
- **Artifact ID**: demo

## 📁 Project Structure

```
demo/
├── src/
│   ├── main/
│   │   ├── java/com/init_spring_bean_mvn/demo/
│   │   │   ├── controller/              # REST API endpoints (CompanyController, ComplyController)
│   │   │   ├── service/                 # Business logic interfaces (CompanyService, AsyncHttpService, ComplyApiService)
│   │   │   ├── service/impl/            # Service implementations
│   │   │   ├── entity/                  # JPA entities (Company)
│   │   │   ├── dto/                     # Data transfer objects (CompanyDTO as record)
│   │   │   ├── repository/              # Spring Data repositories (CompanyRepository)
│   │   │   ├── mapper/                  # Entity <-> DTO mapping (CompanyMapper)
│   │   │   ├── config/                  # Spring configuration (AsyncConfig)
│   │   │   ├── util/                    # Utilities (ConcurrentFileWriter)
│   │   │   ├── networking/              # HTTP client operations
│   │   │   ├── httpserver/              # Custom HTTP server (OrderFulfillmentServer)
│   │   │   └── [other packages]/        # Support packages (bean, emsbackend, etc.)
│   │   └── resources/
│   │       └── application.properties   # Spring Boot configuration
│   └── test/
│       └── java/                        # JUnit tests
└── pom.xml                              # Maven configuration

```

## 🔑 Key Coding Standards & Patterns

### 1. **REST Controller Design**
- **Naming Convention**: Controllers end with `Controller` (e.g., `CompanyController`, `ComplyController`)
- **Path Mapping**: Use `@RequestMapping` at class level for base path
- **HTTP Methods**: Use appropriate annotations (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`)
- **Response Handling**: Return `ResponseEntity<T>` with proper HTTP status codes
- **Async Support**: Methods returning `CompletableFuture<ResponseEntity<T>>` for non-blocking operations
- **Documentation**: Comprehensive JavaDoc for all endpoints

**Example Pattern:**
```java
@RestController
@RequestMapping("companys")
public class CompanyController {
    private final CompanyService _companyService;
    
    @PostMapping
    public ResponseEntity<CompanyDTO> createCompany(@RequestBody CompanyDTO companyDTO) {
        CompanyDTO saved = _companyService.createCompany(companyDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
    
    @PostMapping("/async")
    public CompletableFuture<ResponseEntity<CompanyDTO>> createCompanyAsync(
            @RequestBody CompanyDTO companyDTO) {
        return _companyService.createCompanyAsync(companyDTO)
                .thenApply(saved -> ResponseEntity.status(HttpStatus.CREATED).body(saved))
                .exceptionally(ex -> ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build());
    }
}
```

### 2. **Service Layer Architecture**
- **Interface-First Design**: Always create interface before implementation
- **Single Responsibility**: Each service handles one domain (CompanyService, AsyncHttpService)
- **Naming**: Service interfaces don't use "I" prefix; implementations use "Impl" suffix
- **Dependency Injection**: Use constructor injection (preferred over @Autowired)
- **Async Support**: Use `@Async` annotation on methods returning `CompletableFuture<T>`

**Service Interface Pattern:**
```java
public interface CompanyService {
    CompanyDTO createCompany(CompanyDTO companyDTO);
    CompletableFuture<CompanyDTO> createCompanyAsync(CompanyDTO companyDTO);
    CompletableFuture<List<CompanyDTO>> createCompaniesAsync(List<CompanyDTO> companies);
}
```

**Service Implementation Pattern:**
```java
@Service
public class CompanyServiceImpl implements CompanyService {
    private final CompanyRepository _companyRepository;
    private final CompanyMapper _companyMapper;
    
    public CompanyServiceImpl(CompanyRepository repository, CompanyMapper mapper) {
        this._companyRepository = repository;
        this._companyMapper = mapper;
    }
    
    @Async
    @Override
    public CompletableFuture<CompanyDTO> createCompanyAsync(CompanyDTO companyDTO) {
        try {
            CompanyDTO result = createCompany(companyDTO);
            return CompletableFuture.completedFuture(result);
        } catch (Exception e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

### 3. **Entity & JPA Mapping**
- **Entity Naming**: Use singular form (Company, Order)
- **Table Annotations**: Explicit `@Table(name = "...")` for clarity
- **Primary Keys**: Always use `@Id` with `@GeneratedValue(strategy = GenerationType.IDENTITY)`
- **Columns**: Use `@Column` annotations for database mapping
- **Relationships**: Document foreign key relationships with comments
- **No-Arg Constructor**: Required by JPA/Hibernate

**Entity Pattern:**
```java
@Entity
@Table(name = "companys")
public class Company {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false)
    private String name;
    
    @Column(unique = true)
    private String email;
    
    public Company() {} // Required by JPA
    
    // Getters and setters
}
```

### 4. **Data Transfer Objects (DTOs)**
- **Use Records**: Modern approach with `record` keyword for immutable DTOs
- **Naming Convention**: Append "DTO" suffix (CompanyDTO, OrderDTO)
- **Fields**: Only expose necessary fields (id, name, email for CompanyDTO)

**DTO Pattern:**
```java
public record CompanyDTO(Long id, String name, String email) {
}
```

### 5. **Repository/Persistence Layer**
- **Extend**: `JpaRepository<Entity, PrimaryKeyType>`
- **Naming**: Repository interfaces end with "Repository"
- **Methods**: Leverage Spring Data query methods or `@Query` for complex queries
- **Transactions**: Handled automatically by Spring Data

**Repository Pattern:**
```java
@Repository
public interface CompanyRepository extends JpaRepository<Company, Long> {
    Optional<Company> findByEmail(String email);
    List<Company> findByNameContaining(String name);
}
```

### 6. **Entity-DTO Mapping**
- **Purpose**: Convert between database entities and API DTOs
- **Naming**: Classes end with "Mapper"
- **Implementation**: Simple conversion logic

**Mapper Pattern:**
```java
@Component
public class CompanyMapper {
    public CompanyDTO toDTO(Company entity) {
        return new CompanyDTO(entity.getId(), entity.getName(), entity.getEmail());
    }
    
    public Company toEntity(CompanyDTO dto) {
        Company company = new Company();
        company.setId(dto.id());
        company.setName(dto.name());
        company.setEmail(dto.email());
        return company;
    }
}
```

### 7. **Asynchronous Operations**
- **Framework**: Use `CompletableFuture<T>` from Java 11+
- **Threading**: Use Spring's `@Async` annotation with configured thread pool
- **Composition**: Chain operations with `thenApply()`, `thenAcceptAsync()`, `thenCombine()`
- **Aggregation**: Use `CompletableFuture.allOf()` for batch operations
- **Error Handling**: Use `exceptionally()` or `handle()` for error processing

**Async Patterns:**
```java
// Single async operation
@Async
public CompletableFuture<CompanyDTO> createCompanyAsync(CompanyDTO dto) {
    try {
        CompanyDTO result = createCompany(dto);
        return CompletableFuture.completedFuture(result);
    } catch (Exception e) {
        return CompletableFuture.failedFuture(e);
    }
}

// Batch parallel processing
@Override
public CompletableFuture<List<CompanyDTO>> createCompaniesAsync(List<CompanyDTO> companies) {
    List<CompletableFuture<CompanyDTO>> futures = companies.stream()
            .map(this::createCompanyAsync)
            .toList();
    
    return CompletableFuture.allOf(futures.toArray(new CompletableFuture<?>[0]))
            .thenApply(v -> futures.stream()
                    .map(CompletableFuture::join)
                    .toList())
            .exceptionally(ex -> {
                throw new RuntimeException("Batch processing failed", ex);
            });
}

// Concurrent HTTP requests (Stream-based)
var futures = requestData.entrySet().stream()
    .map(entry -> buildPostRequest(baseUrl, entry.getKey(), entry.getValue()))
    .map(request -> httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString()))
    .map(responseFuture -> responseFuture.thenApply(HttpResponse::body))
    .toList();

return CompletableFuture.allOf(futures.toArray(new CompletableFuture<?>[0]))
    .thenApply(v -> futures.stream()
        .map(CompletableFuture::join)
        .toList());
```

### 8. **Thread Safety**
- **File Operations**: Use `ReentrantLock` for thread-safe file writes
- **Database**: JPA/Hibernate handles thread safety automatically
- **HTTP Client**: Use `HttpClient` (Java 11+) which is thread-safe
- **Collections**: Use concurrent collections when needed

**Thread-Safe File Writing Pattern:**
```java
public class ConcurrentFileWriter {
    private static final ReentrantLock _lock = new ReentrantLock();
    
    public static void writeAsync(Path filePath, String content) throws IOException {
        _lock.lock();
        try {
            Files.writeString(filePath, content + System.lineSeparator(),
                    StandardOpenOption.APPEND);
        } finally {
            _lock.unlock();
        }
    }
}
```

### 9. **Configuration Classes**
- **Async Configuration**: Configure thread pool with `@Configuration` and `@EnableAsync`
- **Bean Declaration**: Use `@Bean` methods for explicit bean configuration
- **Documentation**: Include configuration parameters in JavaDoc

**Async Config Pattern:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("Async-");
        executor.setAwaitTerminationSeconds(60);
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.initialize();
        return executor;
    }
}
```

### 10. **Naming Conventions**
- **Private Fields**: Use underscore prefix (`_companyService`, `_companyRepository`)
- **Classes**: PascalCase (CompanyController, CompanyService)
- **Methods**: camelCase (createCompany, updateCompany)
- **Constants**: UPPER_SNAKE_CASE
- **Packages**: lowercase (controller, service, entity, dto, repository, mapper, util, config)
- **Database Tables**: Plural names (companys, orders)

### 11. **Dependency Injection**
- **Constructor Injection**: Always use constructor injection (preferred)
- **Final Fields**: Mark injected fields as `final`
- **No Field Injection**: Avoid `@Autowired` on fields

**Injection Pattern:**
```java
@Service
public class CompanyServiceImpl implements CompanyService {
    private final CompanyRepository _companyRepository;
    private final CompanyMapper _companyMapper;
    
    // Constructor injection
    public CompanyServiceImpl(CompanyRepository repository, CompanyMapper mapper) {
        this._companyRepository = repository;
        this._companyMapper = mapper;
    }
}
```

### 12. **Documentation & Comments**
- **Class-Level JavaDoc**: Always include for public classes
- **Method-Level JavaDoc**: Include for all public methods
- **Complex Logic**: Add inline comments explaining non-obvious code
- **Database Changes**: Document foreign keys and relationships

**Documentation Pattern:**
```java
/**
 * REST Controller for Company management with COMPLY API integration
 * Supports both synchronous and asynchronous request handling
 *
 * Endpoints receive requests with concurrency and parallelism from Angular frontend
 * All async operations are non-blocking and thread-safe
 */
@RestController
@RequestMapping("companys")
public class CompanyController {
    
    /**
     * Asynchronous company creation for concurrent requests
     * Processes request non-blocking on async executor pool
     * Ideal for high-concurrency scenarios from Angular frontend
     *
     * @param companyDTO Company data to create
     * @return CompletableFuture<ResponseEntity<CompanyDTO>>
     */
    @PostMapping("/async")
    public CompletableFuture<ResponseEntity<CompanyDTO>> createCompanyAsync(
            @RequestBody CompanyDTO companyDTO) {
        // Implementation
    }
}
```

## 🌊 Data Flow Patterns

### Synchronous Flow
```
Client → Controller → Service → Repository → Database → Entity → DTO → ResponseEntity → Client
```

### Asynchronous Flow
```
Client → Controller → Returns CompletableFuture immediately
                  ↓
            Async Executor Pool (background)
                  ↓
            Service → Repository → Database
                  ↓
            CompletableFuture completes with result
                  ↓
            Client receives response (non-blocking)
```

### Batch Parallel Processing
```
Client → Controller → Stream of async operations
                  ↓
        ├─ Service[1] → Database [Thread 1]
        ├─ Service[2] → Database [Thread 2]
        └─ Service[N] → Database [Thread N]
                  ↓
        CompletableFuture.allOf() aggregation
                  ↓
        ResponseEntity with List results
```

## 🚀 Guidelines for Code Generation

### When Creating New Features

1. **REST Endpoints**
   - Always create controller method in appropriate controller
   - Return `ResponseEntity<T>` for sync or `CompletableFuture<ResponseEntity<T>>` for async
   - Use proper HTTP status codes (201 CREATED, 200 OK, 404 NOT_FOUND, 500 INTERNAL_SERVER_ERROR)
   - Add comprehensive JavaDoc comments
   - Include both sync and async versions when appropriate

2. **Service Methods**
   - Create interface first, then implementation
   - Use `@Service` annotation on implementations
   - Use `@Async` for async methods returning `CompletableFuture<T>`
   - Use constructor injection for dependencies
   - Handle exceptions with `CompletableFuture.failedFuture()`

3. **Entity & DTO Creation**
   - Create JPA entity with `@Entity` and `@Table` annotations
   - Always include no-arg constructor
   - Use primary key with `@GeneratedValue`
   - Create corresponding DTO as `record` for immutability
   - Create mapper to convert between entity and DTO

4. **Database Operations**
   - Create repository extending `JpaRepository<Entity, ID>`
   - Use Spring Data query methods or `@Query` annotations
   - Let transactions be handled by Spring automatically

5. **Error Handling**
   - For sync: Use try-catch and throw appropriate exceptions
   - For async: Use `CompletableFuture.failedFuture(exception)`
   - In controllers: Use `.exceptionally()` to return proper error status codes

### When Modifying Existing Code

1. **Maintain Async Support**: If adding to existing service, provide both sync and async versions
2. **Thread Safety**: If adding file I/O, use `ConcurrentFileWriter` or `ReentrantLock`
3. **Documentation**: Always update/add JavaDoc when modifying methods
4. **Consistency**: Follow existing naming conventions and patterns

### Build & Run Commands

```bash
# Build project
mvn clean install

# Run application
mvn spring-boot:run

# Run specific test class
mvn test -Dtest=CompanyControllerTest

# Run all tests
mvn test

# Package as JAR
mvn clean package

# Run packaged JAR
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

## 📊 Key Components to Know

### Core Services
1. **CompanyService** - Company CRUD operations (sync & async)
2. **AsyncHttpService** - Concurrent HTTP operations with `HttpClient`
3. **ComplyApiService** - COMPLY API blueprint operations

### Controllers
1. **CompanyController** - Company REST endpoints (/companys)
2. **ComplyController** - COMPLY API endpoints (/api/comply)

### Database
1. **Company Entity** - Mapped to "companys" table
2. **CompanyRepository** - Spring Data repository

### Configuration
1. **AsyncConfig** - Thread pool configuration for @Async
   - Core size: 10, Max size: 20, Queue: 100

## 🔐 Common Pitfalls to Avoid

1. **Don't use field injection**: Use constructor injection instead
2. **Don't forget no-arg constructor in entities**: Required by JPA/Hibernate
3. **Don't return raw exceptions from async methods**: Use `CompletableFuture.failedFuture()`
4. **Don't forget `@Async` annotation**: Without it, method won't run asynchronously
5. **Don't use ReentrantLock without try-finally**: Always unlock in finally block
6. **Don't forget ResponseEntity wrapping**: Always return `ResponseEntity<T>` from controllers
7. **Don't mix sync and async without clear intent**: Choose appropriate pattern for use case

## 🎯 Best Practices

1. **Use Records for DTOs**: Immutable, concise, modern Java style
2. **Stream API**: Use for transformations and collections processing
3. **CompletableFuture**: Use for async non-blocking operations
4. **Constructor Injection**: Cleaner than field injection, easier to test
5. **Comprehensive JavaDoc**: Document public interfaces and complex logic
6. **Consistent Naming**: Follow underscore prefix for private fields
7. **Batch Operations**: Use parallel streams with `CompletableFuture.allOf()`
8. **Error Handling**: Always handle exceptions in async chains

## 🔗 Integration Points

- **Angular Frontend**: Communicates via REST endpoints (both sync and async)
- **MySQL Database**: Stores company and order data
- **OrderFulfillmentServer**: Legacy HTTP server on port 8080 (for compatibility)
- **External APIs**: Accessed via AsyncHttpService for concurrent requests

## 📝 Application Properties

Key configuration in `application.properties`:
```properties
spring.application.name=demo
spring.datasource.url=jdbc:mysql://localhost:3306/ems
spring.datasource.username=deadpool
spring.datasource.password=God.Karen0922
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
logging.level.org.springframework=DEBUG
```

---

**Project Status**: Production-Ready with full async support and COMPLY API integration  
**Last Updated**: 2026-02-19  
**Maintainer**: Development Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mjh5153)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mjh5153)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
