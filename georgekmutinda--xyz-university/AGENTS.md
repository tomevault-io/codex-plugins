# XYZ University API - Copilot Instructions

## Architecture Overview

**Clean Architecture with Event-Driven Design**  
This .NET 8 API implements layered architecture: Domain → Infrastructure → Application → API.

**Key Components:**
- **Domain**: Entity definitions (Student, Payment, Admin, Course, Client)
- **Infrastructure**: EF Core DbContext, repositories, RabbitMQ publisher, security (JWT), caching
- **Application**: Services, DTOs, mappers, interfaces; business logic lives here
- **API**: Controllers handle HTTP requests; dependency injection wires everything in `Program.cs`

**Critical Data Flows:**
1. **Student Validation**: `StudentController` → `StudentService` → `StudentRepository` → PostgreSQL + Redis cache
2. **Payment Processing**: `PaymentsController` → `PaymentService` → publishes to RabbitMQ → webhook simulator (3s delay) → `BankWebhookController`
3. **JWT Authentication**: `AuthController` issues tokens; `TokenExpiryMiddleware` validates on each request

## Layering & Naming Conventions

- **Interfaces**: In `Application/Interfaces/` (e.g., `IStudentService`, `IPaymentRepository`)
- **Services**: In `Application/Services/` (implement business logic, use repositories and other services)
- **DTOs**: In `Application/DTOs/` (request/response contracts; use DTOs, not domain entities, in API responses)
- **Repositories**: In `Infrastructure/Repository/` (handle all database access; depend on entities, not DTOs)
- **Mappers**: In `Application/Mappers/` (use extension methods to convert Entity ↔ DTO)
- **Entities**: In `Domain/Entities/` (primary key: `AdmissionNumber` for Student; relationships defined in `AppDbContext.OnModelCreating()`)

## Infrastructure & External Systems

**Database**: PostgreSQL via Entity Framework Core 8.0  
- Primary key for Student is `AdmissionNumber`, not auto-increment
- Migrations in `Migrations/` folder; run `dotnet ef database update` or use `Update-Database.ps1`

**Caching**: Redis with `IDistributedCache`  
- StudentService caches validation results with prefix `XYZUniversity_`
- Configuration: `Redis:Connection` in appsettings.json (default: `localhost:6379`)

**Message Broker**: RabbitMQ via `RabbitMQ.Client`  
- Services publish events using `IRabbitMqPublisher` interface
- Asynchronous processing decouples side-effects (auditing, notifications) from request lifecycle
- Configuration: `RabbitMQ:Host` in appsettings.json (default: `localhost` port 5672)
- Queue declaration is durable; messages use `Persistent = true`

**Logging**: Serilog with console + file sinks  
- Configured in `Program.cs` with bootstrap logger
- Use `ILogger<T>` injected into services (e.g., `PaymentService` logs payment events)

**Security**: JWT Bearer tokens  
- Settings in `JwtSettings` class, configuration from `appsettings.json` section `JwtSettings`
- Token generation in `TokenService`, validation in `TokenValidator`
- `[Authorize]` attribute on protected endpoints; `TokenExpiryMiddleware` enforces expiry

## Critical Patterns

**Error Handling**: Use `ApiErrorDto` for all error responses (StatusCode, Message, ErrorType, Details, Timestamp)  
Example from StudentController:
```csharp
var error = new ApiErrorDto 
{ 
    StatusCode = 404, 
    Message = "Student not found.", 
    ErrorType = "NotFoundError",
    Details = $"No student found with admission number: {trimmedAdmissionNumber}"
};
return BadRequest(error);
```

**Dependency Injection**: All services registered in `Program.cs` (Services section). Always inject interfaces, never concrete classes.  
Example:
```csharp
public StudentService(IStudentRepository repo, IDistributedCache cache, IRabbitMqPublisher publisher)
```

**Async/Await**: All I/O operations are async (`async Task<T>`, `await`). Background operations use `Task.Run()` for fire-and-forget (e.g., payment simulator in `PaymentService`).

**Payment Simulator**: `PaymentService.AddPaymentAsync()` spawns a background task that:
1. Delays 3 seconds
2. Calls `BankWebhookController` endpoint with simulated payment confirmation
3. Logs success/failure; catches exceptions to prevent webhook failures from crashing

## Build & Database Setup

**Build**: `dotnet build` from `XYZUniversityAPI/` directory  
**Run**: `dotnet run` from `XYZUniversityAPI/` directory  
**Database Migration**: `dotnet ef database update` or PowerShell script `Update-Database.ps1`  
**Swagger**: Available at `https://localhost:5001/swagger` (or configured port)

**External Services Required**:
- PostgreSQL running (connection in appsettings.json)
- Redis running (default `localhost:6379`)
- RabbitMQ running (default `localhost:5672`, credentials: guest/guest)

## Common Tasks

**Adding a New API Endpoint**:
1. Create DTO in `Application/DTOs/`
2. Add method to service interface in `Application/Interfaces/`
3. Implement in `Application/Services/`
4. Add controller action in `API/Controllers/`
5. Return `ApiErrorDto` on errors, mapped DTOs on success

**Adding a Database Entity**:
1. Create entity class in `Domain/Entities/`
2. Add `DbSet<Entity>` in `AppDbContext`
3. Configure relationships in `AppDbContext.OnModelCreating()`
4. Create repository interface + implementation
5. Register in `Program.cs` dependency injection

**Publishing Async Events**:
```csharp
await _rabbitMqPublisher.PublishAsync("queue-name", eventObject);
```
Background consumers subscribe to same queue outside this API.

**Caching Results**:
```csharp
var cached = await _cache.GetStringAsync(cacheKey);
if (cached != null) return JsonSerializer.Deserialize<T>(cached);
await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(data));
```

## Key Files to Review

- [Program.cs](../XYZUniversityAPI/Program.cs) - DI registration, middleware, authentication setup
- [AppDbContext.cs](../XYZUniversityAPI/Infrastrucure/Data/AppDbContext.cs) - Entity relationships & primary keys
- [StudentService.cs](../XYZUniversityAPI/Application/Services/StudentService.cs) - Caching + RabbitMQ publishing pattern
- [PaymentService.cs](../XYZUniversityAPI/Application/Services/PaymentService.cs) - Background task simulator, external HTTP calls
- [StudentController.cs](../XYZUniversityAPI/API/Controllers/StudentController.cs) - Error handling + validation pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/georgekmutinda)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/georgekmutinda)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
