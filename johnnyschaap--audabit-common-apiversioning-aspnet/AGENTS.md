# Development Guidelines & Code Patterns - Audabit Common Library

> **Note**: This is a **library-specific** template for Audabit.Common.* NuGet packages. These are infrastructure libraries, NOT Web API projects.

This file serves as a reference for code generation patterns used in Audabit Common libraries. Follow these conventions when generating extension methods, handlers, settings, validators, and other library code.

---

## Core Design Principles

### DRY Principle (Don't Repeat Yourself)
**CRITICAL**: Always avoid code duplication. This is a fundamental requirement for maintainable, testable, and scalable code.

#### When to Apply DRY
- **Method Overloads**: More specific overloads should delegate to more general overloads
  ```csharp
  // ✅ GOOD: Generic overload delegates to specific overload
  public static IServiceCollection AddObservability<TSettings>(
      this IServiceCollection services,
      IConfigurationSection configurationSection,
      Func<TSettings?, string?> serviceNameSelector) where TSettings : class
  {
      ArgumentNullException.ThrowIfNull(services);
      ArgumentNullException.ThrowIfNull(configurationSection);
      ArgumentNullException.ThrowIfNull(serviceNameSelector);

      var settings = configurationSection.Get<TSettings>();
      var serviceName = serviceNameSelector(settings) ?? "UnknownService";

      return services.AddObservability(serviceName); // Delegates to core method
  }

  // ❌ BAD: Duplicate registration logic
  public static IServiceCollection AddObservability<TSettings>(...)
  {
      // ... validation ...
      services.AddSingleton(typeof(IEmitter<>), typeof(Emitter<>)); // DUPLICATE
      LoggingEvent.SetServiceName(serviceName ?? "UnknownService");  // DUPLICATE
      return services;
  }
  ```

- **Configuration Methods**: Extract common configuration, then compose
  ```csharp
  // ✅ GOOD: Clear providers then delegate
  public static IServiceCollection UseJsonConsoleLogging(
      this IServiceCollection services,
      Action<ILoggingBuilder>? configureLogging = null)
  {
      ArgumentNullException.ThrowIfNull(services);
      services.AddLogging(builder => builder.ClearProviders());
      return services.AddJsonConsoleLogging(configureLogging); // Reuses core config
  }

  // ❌ BAD: Duplicate JSON console configuration
  public static IServiceCollection UseJsonConsoleLogging(...)
  {
      services.AddLogging(builder =>
      {
          builder.ClearProviders();
          builder.AddJsonConsole(options => { /* DUPLICATE */ });
          configureLogging?.Invoke(builder);
      });
      return services;
  }
  ```

- **Shared Logic**: Extract to helper methods or base classes
- **Constant Values**: Define once, reference everywhere
- **Validation Rules**: Centralize validation logic (FluentValidation)

#### Benefits of DRY
- **Single Source of Truth**: Changes happen in one place
- **Reduced Bugs**: Fix once, fixes everywhere
- **Easier Testing**: Test core logic once, not repeatedly
- **Better Maintainability**: Less code to understand and maintain
- **Consistency**: Behavior is uniform across all usages

#### DRY Detection Patterns
Watch for these signs of duplication:
- Copying and pasting code blocks
- Similar method implementations with minor variations
- Repeated conditional logic
- Identical configuration patterns
- Same validation rules in multiple places

**Rule**: If you find yourself writing the same code twice, extract it. If you find yourself writing similar code, abstract it.

---

## Global Usings Pattern

### CRITICAL: No Usings.cs File Required

**All projects** in the Audabit solution use **global usings defined in the .csproj file**, NOT in a separate `Usings.cs` file.

**Rule**: Never create a `Usings.cs` file. All global usings are declared in the project file.

**Template**:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="AutoFixture" Version="4.18.1" />
    <PackageReference Include="AutoFixture.Xunit2" Version="4.18.1" />
    <PackageReference Include="NSubstitute" Version="5.3.0" />
    <PackageReference Include="Shouldly" Version="4.3.0" />
    <PackageReference Include="xunit" Version="2.9.2" />
  </ItemGroup>

  <!-- Global usings defined HERE, not in Usings.cs -->
  <ItemGroup>
    <Using Include="AutoFixture" />
    <Using Include="AutoFixture.Xunit2" />
    <Using Include="NSubstitute" />
    <Using Include="Shouldly" />
    <Using Include="Xunit" />
  </ItemGroup>
</Project>
```

**Why**:
- Single source of truth (no duplicate files across projects)
- Clear visibility in project file
- Consistent with modern .NET conventions
- Easier to maintain and understand

**Unit Test Projects**: Include AutoFixture, AutoFixture.Xunit2, NSubstitute, Shouldly, Xunit  
**Integration Test Projects**: Same as unit tests (consistent pattern)  
**Library Projects**: Minimal usings based on actual dependencies

---

## Library Structure

### Typical Audabit.Common.* Package Organization
```
Audabit.Common.{Feature}.AspNet/
├── Audabit.Common.{Feature}.AspNet/       # Main library project
│   ├── Extensions/                        # Service registration extensions
│   ├── Middleware/ (AspNet only)          # ASP.NET Core middleware
│   ├── Handlers/                          # HTTP message handlers (if applicable)
│   ├── Settings/                          # Configuration records
│   ├── Validators/                        # FluentValidation validators
│   ├── Attributes/                        # Custom attributes (if applicable)
│   ├── Events/Emitters/Models/            # Feature-specific types
│   └── {Feature}.csproj
├── Tests/
│   └── Audabit.Common.{Feature}.AspNet.Tests.Unit/
│       ├── Extensions/                    # Extension method tests
│       ├── Handlers/                      # Handler tests
│       ├── Validators/                    # Validator tests
│       ├── TestHelpers/                   # Test utilities
│       └── Usings.cs                      # Global usings for tests
├── .copilot-instructions.md               # This file
├── README.md
├── LICENSE
├── nuget.config
└── azure-pipelines.yml
```

### Namespace Conventions
- **Always** match namespace to file path
- **Pattern**: `Audabit.Common.{Feature}.AspNet.{FolderName}`
- **Examples**:
  - `Audabit.Common.HttpClient.AspNet.Extensions`
  - `Audabit.Common.Security.AspNet.Middleware`
  - `Audabit.Common.Observability.Emitters`
  - `Audabit.Common.HttpClient.AspNet.Validators`

### File Naming Conventions
- **One class per file** (strict requirement)
- **File name must match class name exactly**
- **Extension classes**: `{Feature}Extensions.cs`
- **Middleware classes**: `{Feature}Middleware.cs`
- **Validators**: `{Type}Validator.cs`
- **Settings**: `{Feature}Settings.cs`
- **Events**: `{Feature}{Action}Event.cs`

---

## Code Patterns

### 1. Extension Method Pattern

**Used by**: ALL Audabit.Common.* packages

**File**: `Extensions/{Feature}Extensions.cs`

**Template**:
```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace Audabit.Common.{Feature}.AspNet.Extensions;

/// <summary>
/// Extension methods for configuring {feature description}.
/// </summary>
public static class {Feature}Extensions
{
    /// <summary>
    /// Adds {feature} services to the service collection.
    /// </summary>
    /// <param name="services">The service collection.</param>
    /// <param name="configuration">The configuration section.</param>
    /// <returns>The service collection for fluent chaining.</returns>
    /// <exception cref="ArgumentNullException">Thrown when services or configuration is null.</exception>
    public static IServiceCollection Add{Feature}(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        ArgumentNullException.ThrowIfNull(services);
        ArgumentNullException.ThrowIfNull(configuration);

        // Register services
        services.AddSingleton<I{Feature}Service, {Feature}Service>();
        
        // If settings exist, bind and validate
        services
            .AddOptions<{Feature}Settings>()
            .Bind(configuration)
            .ValidateWithFluentValidation();

        return services;
    }
}
```

**Key Points**:
- Static class with `Extensions` suffix
- Extension methods on `IServiceCollection`, `IApplicationBuilder`, etc.
- Fluent API - always return the builder/collection
- `ArgumentNullException.ThrowIfNull()` for all parameters
- XML documentation with exceptions, examples
- Use method delegation for overloads (DRY)

---

### 2. Settings Pattern

**Used by**: HttpClient.AspNet, Security.AspNet, ServiceInfo.AspNet, Swagger.AspNet

**File**: `Settings/{Feature}Settings.cs`

**Template**:
```csharp
namespace Audabit.Common.{Feature}.AspNet.Settings;

/// <summary>
/// Configuration settings for {feature}.
/// </summary>
public record {Feature}Settings
{
    /// <summary>
    /// Gets or initializes the {property description}.
    /// </summary>
    public string Property { get; init; } = string.Empty;

    /// <summary>
    /// Gets or initializes the optional {property description}.
    /// Null means use default behavior.
    /// </summary>
    public int? OptionalProperty { get; init; }

    /// <summary>
    /// Gets or initializes the nested settings.
    /// </summary>
    public NestedSettings Nested { get; init; } = new();
}

/// <summary>
/// Nested configuration settings.
/// </summary>
public record NestedSettings
{
    public bool Enabled { get; init; } = true;
}
```

**Key Points**:
- Use `record` keyword for immutability
- Use `init` accessors (not `set`)
- Provide sensible defaults
- Nullable types for optional overrides
- Nested records for complex settings
- XML documentation for all properties
- Collection initialization syntax: `= [];` or `= new();`

---

### 3. Validator Pattern

**Used by**: HttpClient.AspNet, Security.AspNet, ServiceInfo.AspNet

**File**: `Validators/{Type}Validator.cs`

**Template**:
```csharp
using FluentValidation;

namespace Audabit.Common.{Feature}.AspNet.Validators;

/// <summary>
/// Validator for <see cref="{Type}"/> configuration.
/// </summary>
public class {Type}Validator : AbstractValidator<{Type}>
{
    /// <summary>
    /// Initializes a new instance of the <see cref="{Type}Validator"/> class.
    /// </summary>
    public {Type}Validator()
    {
        RuleFor(x => x.Property)
            .NotEmpty()
            .WithMessage("{Feature}:{Property} is required in configuration");

        RuleFor(x => x.NumericProperty)
            .GreaterThan(0)
            .WithMessage("{Feature}:{NumericProperty} must be greater than 0");

        // ✅ CRITICAL: Use When() guard for nullable collections/objects
        When(x => x.NullableCollection != null, () =>
        {
            RuleForEach(x => x.NullableCollection!.Values)
                .SetValidator(new NestedValidator());
        });

        // Nested validator for required property
        RuleFor(x => x.RequiredNested)
            .NotNull()
            .SetValidator(new NestedValidator());
    }
}
```

**Key Points**:
- Inherit from `AbstractValidator<T>`
- Include configuration path in error messages
- **CRITICAL**: Use `When()` guards to prevent null reference exceptions on nullable properties
- Use `SetValidator()` for nested validation
- Clear, descriptive error messages
- Validate at application startup with `.ValidateWithFluentValidation()`

**When() Guard Pattern** (Required for nullable collections):
```csharp
// ✅ CORRECT: Prevents NullReferenceException
When(x => x.Clients != null, () =>
{
    RuleForEach(x => x.Clients.Values)
        .SetValidator(new ClientValidator());
});

// ❌ WRONG: Will throw if Clients is null
RuleForEach(x => x.Clients.Values)
    .SetValidator(new ClientValidator());
```

---

### 4. Middleware Pattern

**Used by**: CorrelationId.AspNet, ExceptionHandling.AspNet, Security.AspNet

**File**: `Middleware/{Feature}Middleware.cs`

**Template**:
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

namespace Audabit.Common.{Feature}.AspNet.Middleware;

/// <summary>
/// Middleware for {feature description}.
/// </summary>
public class {Feature}Middleware(RequestDelegate next, ILogger<{Feature}Middleware> logger)
{
    /// <summary>
    /// Invokes the middleware.
    /// </summary>
    /// <param name="context">The HTTP context.</param>
    /// <exception cref="ArgumentNullException">Thrown when context is null.</exception>
    public async Task InvokeAsync(HttpContext context)
    {
        ArgumentNullException.ThrowIfNull(context);

        // Pre-processing logic
        // ...

        await next(context);

        // Post-processing logic
        // ...
    }
}
```

**Extension Method for Middleware**:
```csharp
public static class {Feature}MiddlewareExtensions
{
    /// <summary>
    /// Adds the {feature} middleware to the application pipeline.
    /// </summary>
    public static IApplicationBuilder Use{Feature}Middleware(
        this IApplicationBuilder builder)
    {
        ArgumentNullException.ThrowIfNull(builder);
        return builder.UseMiddleware<{Feature}Middleware>();
    }
}
```

**Key Points**:
- Primary constructor with dependencies
- Async invocation pattern
- Argument validation on `context`
- Pre/post processing around `await next(context)`
- Extension method for easy registration

---

### 5. Handler Pattern (DelegatingHandler)
**Used by**: HttpClient.AspNet

**File**: `Handlers/{Feature}DelegatingHandler.cs`

**Template**:
```csharp
namespace Audabit.Common.{Feature}.AspNet.Handlers;

/// <summary>
/// Delegating handler for {feature description}.
/// </summary>
public class {Feature}DelegatingHandler : DelegatingHandler
{
    /// <summary>
    /// Sends an HTTP request with {feature}.
    /// </summary>
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        // Pre-request logic
        
        var response = await base.SendAsync(request, cancellationToken).ConfigureAwait(false);
        
        // Post-request logic
        
        return response;
    }
}
```

---

### 7. Background Service Pattern

**Used by**: HealthChecks.AspNet

**File**: `Services/{Feature}BackgroundService.cs`

**Template**:
```csharp
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

namespace Audabit.Common.{Feature}.AspNet.Services;

/// <summary>
/// Background service that {description}.
/// </summary>
public sealed class {Feature}BackgroundService(
    IOptions<{Feature}Settings> settings,
    SharedContext sharedContext,
    ILogger<{Feature}BackgroundService> logger) : BackgroundService
{
    private Timer? _timer;
    private long _field; // Use long for Interlocked support

    /// <summary>
    /// Executes the background task.
    /// </summary>
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _timer = new Timer(
            DoWork,
            state: null,
            dueTime: TimeSpan.Zero,
            period: TimeSpan.FromSeconds(interval));

        // Register cancellation to dispose timer
        stoppingToken.Register(() => _timer?.Dispose());

        return Task.CompletedTask;
    }

    /// <summary>
    /// Disposes the timer resource.
    /// </summary>
    public override void Dispose()
    {
        _timer?.Dispose();
        base.Dispose();
    }

    private void DoWork(object? state)
    {
        try
        {
            // Use Interlocked for thread-safe field access
            var previous = Interlocked.Read(ref _field);
            var current = /* calculate new value */;
            Interlocked.Exchange(ref _field, current);

            // Work logic
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error in background service");
        }
    }
}
```

**Key Points**:
- Sealed class (cannot be inherited)
- Primary constructor with ILogger
- Timer-based periodic execution
- Thread-safe operations with Interlocked
- Proper cancellation token registration
- Proper disposal pattern
- Error handling in callback

---

### 8. Shared Context Pattern

**Used by**: HealthChecks.AspNet (CpuMonitorSharedContext)

**File**: `Helpers/{Feature}SharedContext.cs`

**Template**:
```csharp
using System.Collections.Concurrent;

namespace Audabit.Common.{Feature}.AspNet.Helpers;

/// <summary>
/// Shared context for {feature} data across services.
/// </summary>
public sealed class {Feature}SharedContext
{
    /// <summary>
    /// Gets the thread-safe collection of {data}.
    /// </summary>
    public ConcurrentQueue<T> Data { get; } = new();
}
```

**Key Points**:
- Sealed class
- Thread-safe collections (ConcurrentQueue, ConcurrentDictionary)
- Used for sharing state between background services and request handlers
- Registered as singleton in DI container
- Read-only collection properties (get-only)

---

### 9. Event Pattern (Telemetry/Logging)

**Used by**: Observability, HttpClient.AspNet

**File**: `Events/{Feature}{Action}Event.cs` or `Telemetry/{Feature}{Action}Event.cs`

**Template**:
```csharp
using System.Diagnostics.CodeAnalysis;
using Audabit.Common.Observability.Events;

namespace Audabit.Common.{Feature}.AspNet.Events;

/// <summary>
/// Event raised when {description}.
/// </summary>
[ExcludeFromCodeCoverage]
public sealed class {Feature}{Action}Event : LoggingEvent
{
    /// <summary>
    /// Initializes a new instance of the <see cref="{Feature}{Action}Event"/> class.
    /// </summary>
    public {Feature}{Action}Event(string property1, int property2)
        : base(nameof({Feature}{Action}Event))
    {
        Properties.Add(nameof(property1), property1);
        Properties.Add(nameof(property2), property2);
    }
}
```

**Key Points**:
- Inherit from `LoggingEvent`
- Event classes must be `sealed` (final implementation, enables JIT optimization)
- `[ExcludeFromCodeCoverage]` attribute
- Constructor-based property initialization
- Use `nameof()` for type-safe property names
- Clear, descriptive event names

---

## Security Patterns

### Constant-Time Comparison Pattern

**Used by**: Security.AspNet (ApiKeyMiddleware)

**Purpose**: Prevent timing attacks on secret comparison

**Template**:
```csharp
using System.Security.Cryptography;
using System.Text;

/// <summary>
/// Performs constant-time string comparison to prevent timing attacks.
/// </summary>
/// <remarks>
/// <para>
/// Standard string comparison stops immediately when it finds a character mismatch.
/// Attackers can exploit these microsecond timing differences to brute-force
/// secrets character by character.
/// </para>
/// <para>
/// <see cref="CryptographicOperations.FixedTimeEquals"/> always compares ALL bytes
/// regardless of where mismatches occur, eliminating timing side-channels.
/// </para>
/// </remarks>
private static bool ConstantTimeEquals(string a, string b)
{
    if (a == null && b == null) return true;
    if (a == null || b == null) return false;

    var aBytes = Encoding.UTF8.GetBytes(a);
    var bBytes = Encoding.UTF8.GetBytes(b);

    return CryptographicOperations.FixedTimeEquals(aBytes, bBytes);
}
```

**Key Points**:
- **CRITICAL** for comparing API keys, tokens, passwords
- Use `CryptographicOperations.FixedTimeEquals` from System.Security.Cryptography
- Convert strings to byte arrays with UTF8 encoding
- Handle null cases explicitly
- Never use `==` or `string.Equals` for secrets

**When to Use**:
- API key validation
- Token comparison
- Password verification (after hashing)
- Session ID validation
- Any security-sensitive string comparison

---

### Sensitive Data Masking Pattern

**Used by**: Serialization (ConvertorsExtensions), HttpClient.AspNet (Header masking)

**Purpose**: Prevent leaking sensitive data in logs and serialization

**Attribute-Based Template**:
```csharp
[AttributeUsage(AttributeTargets.Property)]
public class SensitiveDataAttribute : Attribute { }

public class UserSettings
{
    public string Username { get; init; } = string.Empty;
    
    [SensitiveData]
    public string ApiKey { get; init; } = string.Empty;
    
    [SensitiveData]
    public string Password { get; init; } = string.Empty;
}
```

**Header Masking Template**:
```csharp
private static bool IsSensitiveHeader(string headerName)
{
    var normalizedName = headerName.ToLowerInvariant();

    // Exact matches
    if (normalizedName is "authorization" or "cookie" or "set-cookie" or "proxy-authorization")
    {
        return true;
    }

    // Pattern matches
    return normalizedName.Contains("token") ||
           normalizedName.Contains("key") ||
           normalizedName.Contains("secret") ||
           normalizedName.Contains("password") ||
           normalizedName.Contains("auth") ||
           normalizedName.Contains("credential");
}
```

**Key Points**:
- Use attributes to mark sensitive properties
- Recursive masking for nested objects
- Pattern-based detection for headers
- Replace with `***` or similar mask
- Apply before logging or serialization

---

### Thread-Safe One-Time Initialization Pattern

**Used by**: Observability

**Purpose**: Ensure a static field is set exactly once in a thread-safe manner

**Template**:
```csharp
using System.Threading;

private static string? _serviceName;

/// <summary>
/// Sets the service name for all logging events. Can only be called once.
/// </summary>
/// <param name="serviceName">The service name to set.</param>
/// <exception cref="ArgumentException">Thrown when serviceName is null or whitespace.</exception>
/// <exception cref="InvalidOperationException">Thrown when service name has already been set.</exception>
public static void SetServiceName(string serviceName)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(serviceName);
    
    if (Interlocked.CompareExchange(ref _serviceName, serviceName, null) != null)
    {
        throw new InvalidOperationException(
            "ServiceName has already been set and cannot be changed.");
    }
}
```

**Key Points**:
- Use `Interlocked.CompareExchange` for atomic compare-and-swap
- Returns original value (null if first call, existing value if already set)
- Thread-safe without locks
- Prevents accidental reconfiguration
- Clear error message when already initialized

**When to Use**:
- One-time configuration settings
- Static field initialization from multiple threads
- Preventing duplicate initialization
- Ensuring immutable configuration after first set

---

### Sensitive Header Detection Pattern

**Used by**: HttpClient.AspNet

**Purpose**: Identify and mask sensitive HTTP headers before logging

**Template**:
```csharp
/// <summary>
/// Determines if an HTTP header contains sensitive information that should be masked.
/// </summary>
/// <param name="headerName">The name of the header to check.</param>
/// <returns>True if the header is sensitive, false otherwise.</returns>
private static bool IsSensitiveHeader(string headerName)
{
    var normalizedName = headerName.ToLowerInvariant();

    // Exact matches for known sensitive headers
    if (normalizedName is "authorization" or "cookie" or "set-cookie" or "proxy-authorization")
    {
        return true;
    }

    // Pattern-based detection for custom headers
    return normalizedName.Contains("token") ||
           normalizedName.Contains("key") ||
           normalizedName.Contains("secret") ||
           normalizedName.Contains("password") ||
           normalizedName.Contains("auth") ||
           normalizedName.Contains("credential");
}
```

**Key Points**:
- Case-insensitive comparison (ToLowerInvariant)
- Exact match for standard headers (authorization, cookie, etc.)
- Pattern matching for custom headers (X-API-Key, Bearer-Token, etc.)
- Comprehensive coverage of sensitive terms
- Returns true for any potentially sensitive header

**When to Use**:
- HTTP request/response logging
- Telemetry and observability
- Debugging output
- Anywhere headers are serialized or displayed

---

### Cached Reflection Results Pattern

**Used by**: HttpClient.AspNet

**Purpose**: Cache reflection results to avoid repeated expensive operations

**Template**:
```csharp
public class HttpLoggingDelegatingHandler<T>(ILogger<HttpLoggingDelegatingHandler<T>> logger) 
    : DelegatingHandler
{
    // Cache type name to avoid repeated reflection on every request
    private readonly string _clientName = typeof(T).Name;
    
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        // Use cached _clientName instead of typeof(T).Name
        logger.LogInformation("HTTP request for client {ClientName}", _clientName);
        
        var response = await base.SendAsync(request, cancellationToken).ConfigureAwait(false);
        
        return response;
    }
}
```

**Key Points**:
- Cache in readonly field (initialized once)
- Primary constructor parameter becomes field automatically
- Avoid `typeof(T).Name` in hot paths
- Significant performance improvement for high-frequency operations
- No thread safety needed (readonly, set once during construction)

**When to Use**:
- Generic handlers/middleware with type parameters
- Reflection results used repeatedly
- Hot paths (invoked frequently)
- Performance-critical code

---

### Tag-Based Health Check Filtering Pattern

**Used by**: HealthChecks.AspNet

**Purpose**: Register health checks with tags and expose filtered endpoints

**Template**:
```csharp
// Registration with tags
services.AddHealthChecks()
    .AddCheck<StartupHealthCheck>("startup", tags: ["startup"])
    .AddCheck<ReadinessHealthCheck>("ready", tags: ["ready"])
    .AddCheck<LivenessHealthCheck>("live", tags: ["live"])
    .AddCheck<CpuHealthCheck>("cpu", tags: ["cpu", "resource"]);

// Endpoint mapping with tag filtering
app.MapHealthChecks("/health/startup", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("startup")
});

app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready")
});

app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("live")
});

app.MapHealthChecks("/health/cpu", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("cpu")
});

// Composite endpoint (all checks)
app.MapHealthChecks("/health", new HealthCheckOptions
{
    Predicate = _ => true  // All checks
});
```

**Key Points**:
- Use tags to categorize health checks
- Single health check can have multiple tags
- Predicate filters which checks run on each endpoint
- Separate endpoints for startup, readiness, liveness probes
- Kubernetes-compatible endpoint patterns
- Collection expressions `["tag1", "tag2"]` for tag arrays

**When to Use**:
- Kubernetes health probes (startup, readiness, liveness)
- Different health checks for different monitoring systems
- Selective health check execution
- Resource-intensive health checks (CPU, memory)

---

### Recursive JSON Masking Pattern

**Used by**: Serialization

**Purpose**: Recursively mask sensitive properties in JSON during serialization

**Template**:
```csharp
using System.Text.Json;

/// <summary>
/// Attribute to mark properties containing sensitive data that should be masked.
/// </summary>
[AttributeUsage(AttributeTargets.Property)]
public sealed class SensitiveDataAttribute : Attribute { }

/// <summary>
/// Settings class with sensitive data marked.
/// </summary>
public record ApiSettings
{
    public string ServiceName { get; init; } = string.Empty;
    
    [SensitiveData]
    public string ApiKey { get; init; } = string.Empty;
    
    public NestedSettings Nested { get; init; } = new();
}

public record NestedSettings
{
    [SensitiveData]
    public string Secret { get; init; } = string.Empty;
}

/// <summary>
/// Recursively masks properties marked with [SensitiveData] in JSON.
/// </summary>
private static void MaskSensitiveProperties(JsonElement element, JsonSerializerOptions options)
{
    if (element.ValueKind == JsonValueKind.Object)
    {
        foreach (var property in element.EnumerateObject())
        {
            // Check if property has [SensitiveData] attribute via reflection
            var propertyInfo = /* get PropertyInfo */;
            if (propertyInfo?.GetCustomAttribute<SensitiveDataAttribute>() != null)
            {
                // Replace value with mask
                writer.WriteString(property.Name, "***MASKED***");
            }
            else if (property.Value.ValueKind is JsonValueKind.Object or JsonValueKind.Array)
            {
                // Recursively process nested objects/arrays
                MaskSensitiveProperties(property.Value, options);
            }
        }
    }
}
```

**Key Points**:
- Attribute-based marking (`[SensitiveData]`)
- Recursive traversal of JSON tree
- Handles nested objects and arrays
- Production-safe (masks before logging/serialization)
- Clear visual indicator (`***MASKED***`)
- Works with System.Text.Json and reflection

**When to Use**:
- Production logging of configuration
- API response serialization with secrets
- Debugging output containing sensitive data
- Telemetry with PII or credentials
- Any JSON serialization that might expose secrets

---

## Async/Await Best Practices

### ConfigureAwait(false) - CRITICAL Requirement

**RULE**: ALL `await` expressions in library code MUST use `.ConfigureAwait(false)`.

**No Exceptions**: Even `Task.CompletedTask` must use `.ConfigureAwait(false)`.

**Rationale**: Library code should never capture synchronization context to prevent deadlocks and improve performance.

```csharp
// ❌ WRONG
await Task.CompletedTask;
await next(context);
await SomeMethodAsync();

// ✅ CORRECT
await Task.CompletedTask.ConfigureAwait(false);
await next(context).ConfigureAwait(false);
await SomeMethodAsync().ConfigureAwait(false);
```

**Why This Matters**:
1. **Deadlock Prevention**: Prevents deadlocks when library consumers use `.Result` or `.Wait()`
2. **Performance**: Eliminates unnecessary context switches
3. **Consistency**: Library code doesn't need UI thread marshaling
4. **Microsoft Guidance**: Official recommendation for all library code

**The Deadlock Scenario**:
```csharp
// Consumer code (blocks UI thread)
public void ButtonClick()
{
    var result = middleware.InvokeAsync(context).Result; // Blocks UI thread
}

// Your library WITHOUT ConfigureAwait(false)
public async Task InvokeAsync(HttpContext context)
{
    await next(context); // Tries to resume on UI thread
    // But UI thread is blocked waiting! DEADLOCK!
}

// Your library WITH ConfigureAwait(false)
public async Task InvokeAsync(HttpContext context)
{
    await next(context).ConfigureAwait(false); // Resumes on thread pool
    // No deadlock! Works correctly!
}
```

---

## Collection Initialization Standards

Use collection expressions `[]` for collections and `new()` for complex types.

```csharp
// ✅ CORRECT: Collections use []
public List<string> Items { get; init; } = [];
public Dictionary<string, int> Map { get; init; } = [];
public int[] Array { get; init; } = [];
public string[] Names { get; init; } = [];

// ✅ CORRECT: Complex types use new()
public NestedSettings Nested { get; init; } = new();
public ClientSettings Client { get; init; } = new();
public RetrySettings Retry { get; init; } = new();
```

**Rationale**:
- Collection expressions `[]` are more concise and modern (C# 12)
- `new()` is clearer for complex object initialization
- Consistency across all settings classes

---

### 6. Handler Pattern (DelegatingHandler)
**Used by**: HttpClient.AspNet

**File**: `Handlers/{Feature}DelegatingHandler.cs`

**Template**:
```csharp
namespace Audabit.Common.{Feature}.AspNet.Handlers;

/// <summary>
/// Delegating handler for {feature description}.
/// </summary>
public class {Feature}DelegatingHandler : DelegatingHandler
{
    /// <summary>
    /// Sends an HTTP request with {feature}.
    /// </summary>
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        // Pre-request logic
        
        var response = await base.SendAsync(request, cancellationToken);
        
        // Post-request logic
        
        return response;
    }
}
```

---

### 6. Event Pattern (Telemetry/Logging)

**Used by**: Observability, HttpClient.AspNet

**File**: `Events/{Feature}{Action}Event.cs` or `Telemetry/{Feature}{Action}Event.cs`

**Template**:
```csharp
using System.Diagnostics.CodeAnalysis;
using Audabit.Common.Observability.Events;

namespace Audabit.Common.{Feature}.AspNet.Events;

/// <summary>
/// Event raised when {description}.
/// </summary>
[ExcludeFromCodeCoverage]
public sealed class {Feature}{Action}Event : LoggingEvent
{
    /// <summary>
    /// Initializes a new instance of the <see cref="{Feature}{Action}Event"/> class.
    /// </summary>
    public {Feature}{Action}Event(string property1, int property2)
        : base(nameof({Feature}{Action}Event))
    {
        Properties.Add(nameof(property1), property1);
        Properties.Add(nameof(property2), property2);
    }
}
```

**Key Points**:
- Inherit from `LoggingEvent`
- Event classes must be `sealed` (final implementation, enables JIT optimization)
- `[ExcludeFromCodeCoverage]` attribute
- Constructor-based property initialization
- Use `nameof()` for type-safe property names
- Clear, descriptive event names

---

## Testing Patterns

### Test Project Structure
```
Audabit.Common.{Feature}.AspNet.Tests.Unit/
├── Extensions/               # Tests for extension methods
├── Handlers/                 # Tests for handlers
├── Middleware/               # Tests for middleware
├── Validators/               # Tests for validators
├── TestHelpers/              # Shared test utilities
└── Usings.cs                 # Global usings
```

### Global Usings Pattern

**File**: `Usings.cs`
```csharp
global using AutoFixture;
global using AutoFixture.Xunit2;
global using NSubstitute;
global using Shouldly;
global using Xunit;
```

### Test Class Template
```csharp
using Audabit.Common.{Feature}.AspNet.{Folder};

namespace Audabit.Common.{Feature}.AspNet.Tests.Unit.{Folder};

public class {ClassName}Tests
{
    [Fact]
    public void {MethodName}_{Scenario}_{ExpectedResult}()
    {
        // Arrange
        var fixture = new Fixture();
        var sut = fixture.Create<{ClassName}>();

        // Act
        var result = sut.Method();

        // Assert
        result.ShouldNotBeNull();
    }

    [Theory]
    [AutoData]
    public void {MethodName}_{Scenario}_{ExpectedResult}(string param)
    {
        // AutoFixture provides test data
    }
}
```

### Test Naming Convention
- **Pattern**: `{MethodName}_{Scenario}_{ExpectedResult}`
- **Examples**:
  - `AddHttpClient_WithValidConfiguration_ShouldRegisterClient`
  - `SendAsync_WhenCorrelationIdExists_ShouldAddHeader`
  - `Validate_WhenPropertyIsNull_ShouldReturnInvalid`

### Testing Tools
- **xUnit**: Test framework
- **NSubstitute**: Mocking library
- **Shouldly**: Assertion library
- **AutoFixture**: Test data generation

---

## Culture-Invariant Formatting

**CRITICAL**: Always use `CultureInfo.InvariantCulture` when formatting numbers for configuration or serialization.

```csharp
using System.Globalization;

// ✅ CORRECT: Culture-invariant formatting
configData["Timeout"] = timeout.ToString(CultureInfo.InvariantCulture);
configData["FailureThreshold"] = threshold.ToString(CultureInfo.InvariantCulture);

// ❌ WRONG: Culture-specific (0.5 vs 0,5 depending on locale)
configData["Timeout"] = timeout.ToString();
```

**Why**: Different cultures use different decimal separators (. vs ,). Configuration binding expects invariant format.

---

## Modern C# Features (Required)

### 1. File-Scoped Namespaces ✅
```csharp
namespace Audabit.Common.{Feature}.AspNet.Extensions;

public class MyClass { }
```

### 2. Primary Constructors ✅
```csharp
public class MyService(ILogger<MyService> logger, IOptions<MySettings> options)
{
    public void DoWork()
    {
        logger.LogInformation("Working");
    }
}
```

### 3. Records for Immutable Data ✅
```csharp
public record MySettings
{
    public string Value { get; init; } = string.Empty;
}
```

### 4. Collection Expressions ✅
```csharp
public List<int> Numbers { get; init; } = [1, 2, 3];
public int[] Array { get; init; } = [4, 5, 6];
```

### 5. Pattern Matching ✅
```csharp
if (value is not null and > 0)
{
    // Process
}
```

---

## NuGet Package Considerations

### Public API Surface
- Mark internal types as `internal`
- Use `[ExcludeFromCodeCoverage]` for generated/simple code
- XML documentation for ALL public APIs
- Follow semantic versioning

### Breaking Changes
- Avoid breaking changes in minor versions
- Clearly document breaking changes
- Consider obsolete attributes before removal

### Dependencies
- Minimize external dependencies
- Target .NET 9.0 (or current LTS)
- Use Microsoft.Extensions.* for DI, configuration, logging

---

## Best Practices Summary

### ✅ Always Do
1. **DRY Principle** - Delegate to core methods, avoid duplication
2. **Argument Validation** - `ArgumentNullException.ThrowIfNull()`
3. **File-Scoped Namespaces** - Modern C# syntax
4. **One Class Per File** - Single responsibility
5. **XML Documentation** - All public APIs
6. **Immutable Settings** - Use `record` and `init`
7. **When() Guards** - FluentValidation nullable safety
8. **Culture-Invariant Formatting** - Configuration/serialization
9. **Comprehensive Tests** - xUnit + NSubstitute + Shouldly
10. **Primary Constructors** - For dependency injection

### ❌ Never Do
1. **Duplicate Code** - Extract and reuse
2. **Multiple Classes Per File** - Violates SRP
3. **Mutable Settings** - Use records with init
4. **Skip Validation** - Always validate configuration
5. **Ignore Nullability** - Use When() guards in validators
6. **Culture-Specific Formatting** - Use InvariantCulture
7. **Skip XML Docs** - All public APIs must be documented
8. **Block Namespaces** - Use file-scoped

---

## Sealed Modifier Pattern

### When to Use `sealed`

**Always Sealed**:
- **Background services**: `public sealed class {Feature}BackgroundService : BackgroundService`
- **Shared context classes**: `public sealed class {Feature}SharedContext`
- **Attributes**: `public sealed class {Feature}Attribute : Attribute`
- **Health checks implementing IHealthCheck**: `public sealed class {Feature}Check : IHealthCheck`

**Recommended Sealed** (use when final implementation):
- **Settings records**: `public sealed record {Feature}Settings`
- **Validators**: `public sealed class {Feature}Validator : AbstractValidator<T>`

**Never Sealed**:
- **Extension method classes** (static classes can't be sealed)
- **Middleware classes** (unless certain they won't be inherited)
- **Builder classes** (unless certain they won't be inherited)

**Rationale**: The `sealed` modifier prevents inheritance, which improves performance (JIT optimizations, devirtualization) and clarifies design intent. Use it when a class is not designed to be extended.

**Examples**:
```csharp
// ✅ Background Service - Always sealed
public sealed class CpuMonitorBackgroundService : BackgroundService { }

// ✅ Shared Context - Always sealed  
public sealed class CpuMonitorSharedContext { }

// ✅ Attribute - Always sealed
public sealed class NotApiKeyProtectedAttribute : Attribute;

// ✅ Health Check - Always sealed
public sealed class HealthStartupCheck : IHealthCheck { }

// ✅ Settings - Recommended sealed
public sealed record HealthChecksSettings { }

// ✅ Validator - Recommended sealed
public sealed class HealthChecksSettingsValidator : AbstractValidator<HealthChecksSettings> { }
```

---

## Argument Validation Patterns

### ArgumentNullException.ThrowIfNull vs ArgumentException.ThrowIfNullOrWhiteSpace

**Use `ArgumentNullException.ThrowIfNull` for**:
- Object parameters (services, configuration, contexts)
- Collections
- Delegates (Action, Func)
- Any reference type where null is invalid

**Use `ArgumentException.ThrowIfNullOrWhiteSpace` for**:
- String parameters that must have content (service names, event descriptions, keys)
- String parameters where empty or whitespace is semantically invalid

**Examples**:
```csharp
public static IServiceCollection AddObservability(
    this IServiceCollection services,
    string serviceName)
{
    // Object validation
    ArgumentNullException.ThrowIfNull(services);
    
    // String with required content validation
    ArgumentException.ThrowIfNullOrWhiteSpace(serviceName);
    
    return services;
}

public class MyMiddleware(RequestDelegate next, IOptions<Settings> settings)
{
    public async Task InvokeAsync(HttpContext context)
    {
        // Object validation
        ArgumentNullException.ThrowIfNull(context);
        
        await next(context).ConfigureAwait(false);
    }
}

protected LoggingEvent(string eventDescription)
{
    // String must have content
    ArgumentException.ThrowIfNullOrWhiteSpace(eventDescription);
    
    Properties.Add("Event description", eventDescription);
}
```

**Key Difference**: 
- `ThrowIfNull` - Only checks for `null`
- `ThrowIfNullOrWhiteSpace` - Checks for `null`, empty string `""`, or whitespace-only strings

---

## Constants for Magic Strings

**Pattern**: Extract magic strings to `private const` fields at the top of the class.

**Use For**:
- HTTP header names
- Endpoint paths
- Event names
- Logging property keys
- Configuration section names
- Status codes or well-known values

**Template**:
```csharp
public class MyMiddleware
{
    private const string HeaderName = "X-Custom-Header";
    private const string EventName = "MyEvent";
    private const string PropertyKey = "Property name";
    
    public async Task InvokeAsync(HttpContext context)
    {
        if (context.Request.Headers.TryGetValue(HeaderName, out var value))
        {
            // Use constant instead of literal string
        }
    }
}
```

**Examples from Codebase**:
```csharp
public class ExceptionMiddleware
{
    private const string EventName = "UnhandledServiceExceptionEvent";
    private const string ServiceNameKey = "Service name";
    private const string HttpMethodKey = "HTTP method";
    private const string PathKey = "Path";
    private const string StatusCodeKey = "Status code";
}

public static class HealthChecksExtensions
{
    private const string HealthEndpoint = "/health";
    private const string StartupEndpoint = "/health/startupz";
    private const string ReadyEndpoint = "/health/readyz";
    private const string LiveEndpoint = "/health/livez";
    private const string CpuEndpoint = "/health/cpu";
}

public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeaderName = "X-Correlation-Id";
}
```

**Benefits**:
- Single source of truth for string values
- Easier refactoring (rename in one place)
- Compile-time checking (typos in constant name caught by compiler)
- IntelliSense support
- Self-documenting code

---

## Custom Validator Methods Pattern

**Pattern**: Create `private static` helper methods for complex validation logic that doesn't fit into standard FluentValidation rules.

**When to Use**:
- Complex format validation (header names, URLs, patterns)
- Business rules that require multiple checks
- Reusable validation logic within the validator
- Domain-specific validation (e.g., "must start with X-")

**Template**:
```csharp
public class MySettingsValidator : AbstractValidator<MySettings>
{
    public MySettingsValidator()
    {
        RuleFor(x => x.Property)
            .NotEmpty()
            .Must(BeValidFormat)
            .WithMessage("Property must follow required format");
    }

    private static bool BeValidFormat(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
        {
            return false;
        }

        // Custom validation logic
        return value.StartsWith("prefix", StringComparison.OrdinalIgnoreCase);
    }
}
```

**Real Example from ApiKeySettingsValidator**:
```csharp
public class ApiKeySettingsValidator : AbstractValidator<ApiKeySettings>
{
    public ApiKeySettingsValidator()
    {
        RuleFor(x => x.HeaderName)
            .NotEmpty()
            .Must(IsValidHeaderName)
            .WithMessage("ApiKeySettings:HeaderName should follow standard conventions (X-* or Authorization)");
    }

    private static bool IsValidHeaderName(string headerName)
    {
        if (string.IsNullOrWhiteSpace(headerName))
        {
            return false;
        }

        return headerName.StartsWith("X-", StringComparison.OrdinalIgnoreCase) ||
               headerName.Equals("Authorization", StringComparison.OrdinalIgnoreCase);
    }
}
```

**Key Points**:
- **private static** methods - no instance state needed
- **Clear naming** - Use Be*, Is*, Validate* prefixes
- **Null checks inside** - Handle edge cases in the validator method
- **StringComparison** - Always specify for culture-aware comparisons
- **Descriptive messages** - Explain what format is expected

---

## Event Pattern - Conditional Properties

**Pattern**: Only add properties to events when they have meaningful values to avoid cluttering logs with null/empty data.

**Use For**:
- Optional request/response bodies
- Optional headers
- Contextual information that may not always be present
- Large or sensitive data that should only be logged when available

**Template**:
```csharp
[ExcludeFromCodeCoverage]
public class MyEvent : LoggingEvent
{
    public MyEvent(string requiredParam, string? optionalParam1, string? optionalParam2)
        : base(nameof(MyEvent))
    {
        // Always add required properties
        Properties.Add(nameof(requiredParam), requiredParam);

        // Conditionally add optional properties
        if (!string.IsNullOrWhiteSpace(optionalParam1))
        {
            Properties.Add(nameof(optionalParam1), optionalParam1);
        }

        if (!string.IsNullOrWhiteSpace(optionalParam2))
        {
            Properties.Add(nameof(optionalParam2), optionalParam2);
        }
    }
}
```

**Real Example from HttpClientRequestEvent**:
```csharp
[ExcludeFromCodeCoverage]
public class HttpClientRequestEvent : LoggingEvent
{
    public HttpClientRequestEvent(
        string clientName, 
        string method, 
        string uri, 
        string? body, 
        string? headers)
        : base(nameof(HttpClientRequestEvent))
    {
        // Required properties - always add
        Properties.Add(nameof(clientName), clientName);
        Properties.Add(nameof(method), method);
        Properties.Add(nameof(uri), uri);

        // Optional properties - only add if present
        if (!string.IsNullOrWhiteSpace(body))
        {
            Properties.Add(nameof(body), body);
        }

        if (!string.IsNullOrWhiteSpace(headers))
        {
            Properties.Add(nameof(headers), headers);
        }
    }
}
```

**When to Use Conditional Addition**:
- Request/response bodies (may be null or empty)
- Headers (may not be set)
- Correlation IDs (may not exist in all contexts)
- Optional contextual information
- Data that may be sensitive or large (only log when needed)

**Benefits**:
- Cleaner logs (no null/empty entries)
- Better performance (less data to serialize)
- Clearer signal-to-noise ratio
- Reduced storage costs for logs

---

## Comprehensive XML Documentation Pattern

### Extension Methods and Public APIs

**Full Template with All Sections**:
```csharp
/// <summary>
/// Brief one-line description of what the method does.
/// </summary>
/// <param name="services">The service collection.</param>
/// <param name="configuration">The configuration section.</param>
/// <returns>The service collection for fluent chaining.</returns>
/// <exception cref="ArgumentNullException">Thrown when services or configuration is null.</exception>
/// <remarks>
/// <para>
/// <b>Feature Overview:</b>
/// Detailed explanation of what this configures and why it's needed.
/// Include architectural context and integration points.
/// </para>
/// <para>
/// <b>Key Behaviors:</b>
/// <list type="bullet">
/// <item><description>Behavior or feature 1</description></item>
/// <item><description>Behavior or feature 2</description></item>
/// <item><description>Behavior or feature 3</description></item>
/// </list>
/// </para>
/// <para>
/// <b>Integration Notes:</b>
/// How this integrates with other systems, middleware order, or dependencies.
/// </para>
/// </remarks>
/// <example>
/// Register in Program.cs:
/// <code>
/// builder.Services.AddMyFeature(configuration.GetSection("MySettings"));
/// </code>
/// </example>
public static IServiceCollection AddMyFeature(
    this IServiceCollection services,
    IConfiguration configuration)
```

### Settings Records

**Full Template with Configuration Examples**:
```csharp
/// <summary>
/// Configuration settings for {feature description}.
/// </summary>
/// <remarks>
/// <para>
/// Detailed description of the settings purpose, usage context, and important considerations.
/// Include security implications if applicable.
/// </para>
/// <para>
/// All settings can be configured via appsettings.json or environment variables.
/// Environment variables use double underscore (__) as the hierarchy separator.
/// </para>
/// </remarks>
/// <example>
/// Configuration in appsettings.json:
/// <code>
/// {
///   "MySettings": {
///     "Property1": "value",
///     "Property2": 42,
///     "NestedSettings": {
///       "Enabled": true
///     }
///   }
/// }
/// </code>
/// 
/// Or using environment variables:
/// <code>
/// MySettings__Property1=value
/// MySettings__Property2=42
/// MySettings__NestedSettings__Enabled=true
/// </code>
/// </example>
public record MySettings
{
    /// <summary>
    /// Gets or initializes the {property description}.
    /// </summary>
    /// <value>
    /// Detailed description of the value, expected format, constraints, and default behavior.
    /// </value>
    /// <remarks>
    /// Additional context: when this is used, security considerations, performance implications.
    /// </remarks>
    public string Property1 { get; init; } = string.Empty;
}
```

### Properties

**Full Property Documentation**:
```csharp
/// <summary>
/// Gets or initializes the {brief description}.
/// </summary>
/// <value>
/// {Detailed description of the value}
/// {Expected format or constraints}
/// {Default behavior if not set}
/// </value>
/// <remarks>
/// {When/how this is used}
/// {Security considerations}
/// {Performance implications}
/// </remarks>
public string MyProperty { get; init; } = string.Empty;
```

**Key Elements**:
1. **`<summary>`** - Brief one-liner (required for all public members)
2. **`<param>`** - All parameters with clear descriptions
3. **`<returns>`** - What the method returns and its purpose
4. **`<exception>`** - All exceptions that can be thrown
5. **`<remarks>`** - Multi-paragraph detailed explanation
   - Use `<para>` for separate paragraphs
   - Use `<b>Title:</b>` for section headers
   - Use `<list type="bullet">` for bullet lists
   - Use `<code>` for inline code
6. **`<example>`** - Real-world usage examples with `<code>` blocks
7. **`<value>`** - For properties (detailed value description)
8. **`<see cref=""/>`** - Cross-references to related types/members

**Formatting Standards**:
- Use `<para>` to separate logical sections
- Bold headers with `<b>Header:</b>`
- Multi-line code blocks within `<code>` tags
- Cross-reference related APIs with `<see cref="TypeName"/>`

---

## Validator Error Message Conventions

### Two Message Format Patterns

#### Pattern 1: Configuration Path Format (Top-Level Settings)

**Use When**: Validating top-level settings classes where error helps identify exact configuration location.

**Format**: `"{SettingsType}:{PropertyName} {description}"`

**Example**:
```csharp
public class ApiKeySettingsValidator : AbstractValidator<ApiKeySettings>
{
    public ApiKeySettingsValidator()
    {
        RuleFor(x => x.Key)
            .NotEmpty()
            .WithMessage("ApiKeySettings:Key is required in configuration");

        RuleFor(x => x.HeaderName)
            .NotEmpty()
            .WithMessage("ApiKeySettings:HeaderName is required in configuration");
    }
}
```

#### Pattern 2: Property Name Only Format (Nested Settings)

**Use When**: Validating nested settings objects or DTOs where error context is clear from nesting.

**Format**: `"{PropertyName} {description}."`

**Example**:
```csharp
public class RetrySettingsValidator : AbstractValidator<RetrySettings>
{
    public RetrySettingsValidator()
    {
        RuleFor(x => x.MaxRetryAttempts)
            .GreaterThanOrEqualTo(0)
            .WithMessage("MaxRetryAttempts must be greater than or equal to 0.");

        RuleFor(x => x.RetryDelayMilliseconds)
            .GreaterThan(0)
            .WithMessage("RetryDelayMilliseconds must be greater than 0.");
    }
}
```

### Guidelines

- **Top-level settings validators**: Use `{SettingsType}:{PropertyName}` format
- **Nested settings validators**: Use `{PropertyName}` only
- **End with period**: For property-only messages
- **No period**: For configuration path messages (unless sentence continues)
- **Clear and actionable**: State what's wrong and what's expected
- **Specific constraints**: Include ranges, formats, or examples when helpful

---

## Common Validation Range Patterns

### Standard Ranges by Use Case

#### Percentage (0-1 Decimal)
```csharp
RuleFor(x => x.FailureThreshold)
    .GreaterThan(0)
    .LessThanOrEqualTo(1)
    .WithMessage("FailureThreshold must be between 0 and 1.");
```

#### Percentage (0-100 Integer)
```csharp
RuleFor(x => x.CpuUpperThreshold)
    .GreaterThan(0)
    .LessThanOrEqualTo(100)
    .WithMessage("CpuUpperThreshold must be between 1 and 100.");
```

#### Retry Attempts (0-10)
```csharp
// Note: >= 0 allows disabling retries
RuleFor(x => x.MaxRetryAttempts)
    .GreaterThanOrEqualTo(0)
    .LessThanOrEqualTo(10)
    .WithMessage("MaxRetryAttempts must be between 0 and 10.");
```

#### Timeout Seconds (1-300)
```csharp
RuleFor(x => x.TimeoutSeconds)
    .GreaterThan(0)
    .LessThanOrEqualTo(300)  // 5 minutes max
    .WithMessage("TimeoutSeconds must be between 1 and 300.");
```

#### Sample Counts (1-100)
```csharp
RuleFor(x => x.CpuMaxSamples)
    .GreaterThan(0)
    .LessThanOrEqualTo(100)
    .WithMessage("CpuMaxSamples must be between 1 and 100.");
```

#### Sample Intervals (1-60 seconds)
```csharp
RuleFor(x => x.CpuSampleIntervalInSeconds)
    .GreaterThan(0)
    .LessThanOrEqualTo(60)
    .WithMessage("CpuSampleIntervalInSeconds must be between 1 and 60.");
```

#### Backoff Power (1-10)
```csharp
RuleFor(x => x.RetryBackoffPower)
    .GreaterThanOrEqualTo(1)  // Must be at least 1
    .LessThanOrEqualTo(10)
    .WithMessage("RetryBackoffPower must be between 1 and 10.");
```

#### Milliseconds (0-60000 for short waits)
```csharp
RuleFor(x => x.RetryDelayMilliseconds)
    .GreaterThan(0)
    .LessThanOrEqualTo(60000)  // 60 seconds max
    .WithMessage("RetryDelayMilliseconds must be between 1 and 60000.");
```

### Comparison Operators Guide

- **`GreaterThan(0)`** - Use for values that must be positive (> 0)
- **`GreaterThanOrEqualTo(0)`** - Use for values that can be zero (>= 0), often to disable a feature
- **`GreaterThanOrEqualTo(1)`** - Use for minimums that start at 1
- **`LessThanOrEqualTo(X)`** - Always specify upper bounds for resource-intensive settings

### Rationale Documentation

Document the rationale for limits in XML comments:
```csharp
/// <summary>
/// Gets or initializes the maximum number of retry attempts.
/// </summary>
/// <value>
/// Valid range: 0-10. Set to 0 to disable retries.
/// Default is 3.
/// </value>
/// <remarks>
/// Upper limit prevents excessive retry storms that could overwhelm downstream services.
/// </remarks>
public int MaxRetryAttempts { get; init; } = 3;
```

---

## Add vs Use Extension Method Pattern

### Naming Convention

- **`Add{Feature}`** - **Additive** method that adds to existing configuration
- **`Use{Feature}`** - **Replacement** method that clears existing configuration first

### Pattern Structure

```csharp
/// <summary>
/// Adds {feature} alongside existing configuration.
/// </summary>
public static IServiceCollection Add{Feature}(
    this IServiceCollection services,
    Action<Builder>? configure = null)
{
    ArgumentNullException.ThrowIfNull(services);

    // Add configuration
    services.Configure(/* configuration */);
    configure?.Invoke(new Builder());

    return services;
}

/// <summary>
/// Replaces all existing {related configuration} with {feature}.
/// </summary>
public static IServiceCollection Use{Feature}(
    this IServiceCollection services,
    Action<Builder>? configure = null)
{
    ArgumentNullException.ThrowIfNull(services);

    // Clear existing configuration
    services.ClearProviders();  // or equivalent

    // Delegate to Add method (DRY principle)
    return services.Add{Feature}(configure);
}
```

### Real Example

```csharp
/// <summary>
/// Adds JSON console logging alongside existing providers.
/// </summary>
/// <param name="services">The service collection.</param>
/// <param name="configureLogging">Optional additional logging configuration.</param>
/// <returns>The service collection for fluent chaining.</returns>
public static IServiceCollection AddJsonConsoleLogging(
    this IServiceCollection services,
    Action<ILoggingBuilder>? configureLogging = null)
{
    ArgumentNullException.ThrowIfNull(services);

    services.AddLogging(builder =>
    {
        builder.AddJsonConsole(options =>
        {
            options.IncludeScopes = true;
            options.TimestampFormat = "yyyy-MM-dd HH:mm:ss ";
        });

        configureLogging?.Invoke(builder);
    });

    return services;
}

/// <summary>
/// Replaces all existing logging providers with JSON console logging.
/// </summary>
/// <param name="services">The service collection.</param>
/// <param name="configureLogging">Optional additional logging configuration.</param>
/// <returns>The service collection for fluent chaining.</returns>
public static IServiceCollection UseJsonConsoleLogging(
    this IServiceCollection services,
    Action<ILoggingBuilder>? configureLogging = null)
{
    ArgumentNullException.ThrowIfNull(services);

    // Clear existing providers first
    services.AddLogging(builder => builder.ClearProviders());

    // Delegate to Add method - DRY principle
    return services.AddJsonConsoleLogging(configureLogging);
}
```

### When to Use

- **Add{Feature}**: When the feature should work alongside existing configuration
  - Example: `AddHttpClient` - adds a named client
  - Example: `AddJsonConsoleLogging` - adds JSON logging provider
  
- **Use{Feature}**: When the feature should replace existing configuration
  - Example: `UseJsonConsoleLogging` - replaces all logging providers
  - Example: `UseRouting` - establishes routing as the middleware

### Critical Rules

1. **Always delegate**: `Use` method should **always** delegate to `Add` method
2. **Clear first**: `Use` method clears existing configuration before calling `Add`
3. **Same parameters**: Both methods should have identical parameters
4. **Document difference**: XML docs must clearly state "adds" vs "replaces"

### Examples

- ✅ `AddHttpClient` / no `Use` variant - always additive
- ✅ `AddJsonConsoleLogging` / `UseJsonConsoleLogging` - logging can be additive or replacement
- ✅ `AddObservability` - no `Use` variant - always additive

---

## IHealthCheck Implementation Pattern

### Complete Template

```csharp
/// <summary>
/// Health check for {description of what is being checked}.
/// </summary>
/// <param name="dependency">The injected dependency.</param>
/// <param name="emitter">The event emitter for telemetry.</param>
public sealed class {Feature}HealthCheck(
    IDependency dependency,
    IEmitter<{Feature}HealthCheck> emitter) : IHealthCheck
{
    /// <summary>
    /// Checks {what is being checked} and returns the health status.
    /// </summary>
    /// <param name="context">The health check context.</param>
    /// <param name="cancellationToken">The cancellation token.</param>
    /// <returns>A health check result indicating the current health status.</returns>
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        // CRITICAL: Task.CompletedTask moves execution off the caller's thread
        await Task.CompletedTask.ConfigureAwait(false);
        
        // Perform health check logic
        if (!condition)
        {
            return HealthCheckResult.Unhealthy("Specific reason for unhealthy status");
        }

        // Emit telemetry event
        emitter.RaiseInformation(new {Feature}HealthCheckEvent("context"));
        
        return HealthCheckResult.Healthy("Specific reason for healthy status");
    }
}
```

### Real Example

```csharp
/// <summary>
/// Health check that verifies startup tasks have completed.
/// </summary>
/// <param name="startupTaskContext">The shared context tracking startup completion.</param>
/// <param name="emitter">The event emitter for telemetry.</param>
public sealed class HealthStartupCheck(
    StartupTaskContext startupTaskContext,
    IEmitter<HealthStartupCheck> emitter) : IHealthCheck
{
    /// <summary>
    /// Checks whether all startup tasks have completed successfully.
    /// </summary>
    /// <param name="context">The health check context.</param>
    /// <param name="cancellationToken">The cancellation token.</param>
    /// <returns>Healthy if startup is complete, unhealthy otherwise.</returns>
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        await Task.CompletedTask.ConfigureAwait(false);
        
        if (!startupTaskContext.IsComplete)
        {
            return HealthCheckResult.Unhealthy("Startup tasks have not completed");
        }

        emitter.RaiseInformation(new HealthCheckStartupCompletedEvent("HealthChecks"));
        return HealthCheckResult.Healthy("All startup tasks completed successfully");
    }
}
```

### Critical Requirements

1. **Sealed class**: Health checks must be `sealed` to prevent inheritance
2. **Primary constructor**: Use primary constructor with dependency injection
3. **Task.CompletedTask requirement**: MUST `await Task.CompletedTask.ConfigureAwait(false)` at the start
   - Moves execution off the health check endpoint's synchronous pipeline
   - Prevents blocking the thread pool
   - Required even if the check is synchronous
4. **ConfigureAwait(false)**: All `await` statements must use `ConfigureAwait(false)`
5. **Factory methods**: Use `HealthCheckResult.Healthy()` / `Unhealthy()` / `Degraded()`
6. **Descriptive messages**: Provide specific reasons for health status
7. **Telemetry**: Inject `IEmitter<T>` and raise events for observability

### Why Task.CompletedTask?

```csharp
// ❌ WITHOUT Task.CompletedTask - runs synchronously on health check thread
public async Task<HealthCheckResult> CheckHealthAsync(...)
{
    // This runs on the health check endpoint thread
    if (!startupTaskContext.IsComplete)
    {
        return HealthCheckResult.Unhealthy("...");
    }
    return HealthCheckResult.Healthy("...");
}

// ✅ WITH Task.CompletedTask - asynchronously yields to thread pool
public async Task<HealthCheckResult> CheckHealthAsync(...)
{
    await Task.CompletedTask.ConfigureAwait(false);
    
    // This runs on a thread pool thread, not blocking the endpoint
    if (!startupTaskContext.IsComplete)
    {
        return HealthCheckResult.Unhealthy("...");
    }
    return HealthCheckResult.Healthy("...");
}
```

**Benefits**:
- Prevents blocking health check endpoint threads
- Allows health checks to scale better under load
- Provides true async execution even for synchronous checks
- Consistent pattern across all health checks

---

## Common Mistakes to Avoid

### 1. Validator Without When() Guard
```csharp
// ❌ WRONG: NullReferenceException if Clients is null
RuleForEach(x => x.Clients.Values)
    .SetValidator(new ClientValidator());

// ✅ CORRECT: Safe with When() guard
When(x => x.Clients != null, () =>
{
    RuleForEach(x => x.Clients.Values)
        .SetValidator(new ClientValidator());
});
```

### 2. Not Delegating in Overloads
```csharp
// ❌ WRONG: Duplicate logic
public static IServiceCollection Method1(this IServiceCollection services, string param)
{
    services.AddSingleton<IService, Service>(); // DUPLICATE
    return services;
}

public static IServiceCollection Method2(this IServiceCollection services)
{
    services.AddSingleton<IService, Service>(); // DUPLICATE
    return services;
}

// ✅ CORRECT: Delegation
public static IServiceCollection Method2(this IServiceCollection services)
{
    return services.Method1("default");
}

public static IServiceCollection Method1(this IServiceCollection services, string param)
{
    ArgumentNullException.ThrowIfNull(services);
    services.AddSingleton<IService, Service>();
    return services;
}
```

### 3. Multiple Classes in One File
```csharp
// ❌ WRONG: Two classes in Extensions.cs
public static class MyExtensions { }
public class MyBuilder { } // Should be in MyBuilder.cs
```

---

## File Organization Checklist

- [ ] One class per file
- [ ] File name matches class name
- [ ] Namespace matches folder path
- [ ] File-scoped namespace
- [ ] XML documentation on public APIs
- [ ] ArgumentNullException.ThrowIfNull() on parameters
- [ ] Settings use `record` with `init`
- [ ] Validators use When() guards for nullables
- [ ] Tests follow naming convention
- [ ] Culture-invariant formatting in tests

---

## Quick Reference

| Pattern | File Location | Example |
|---------|---------------|---------|
| Extension Methods | `Extensions/{Feature}Extensions.cs` | `ResilientHttpClientExtensions.cs` |
| Builder Classes | `Builders/{Feature}Builder.cs` | `ResilientHttpClientBuilder.cs` |
| Settings | `Settings/{Feature}Settings.cs` | `HttpClientSettings.cs` |
| Validators | `Validators/{Type}Validator.cs` | `HttpClientSettingsValidator.cs` |
| Middleware | `Middleware/{Feature}Middleware.cs` | `CorrelationIdMiddleware.cs` |
| Handlers | `Handlers/{Feature}Handler.cs` | `HttpLoggingDelegatingHandler.cs` |
| Events | `Events/{Feature}{Action}Event.cs` | `HttpClientRetryEvent.cs` |
| Tests | `Tests/.../{{ClassName}Tests.cs` | `ResilientHttpClientExtensionsTests.cs` |

---

## Conclusion

This library follows strict patterns for consistency, maintainability, and quality. When adding new code:

1. **Check existing patterns** - Look at similar code in the package
2. **Follow DRY principle** - Delegate, don't duplicate
3. **One class per file** - No exceptions
4. **Use modern C#** - Records, file-scoped namespaces, primary constructors
5. **Validate everything** - FluentValidation with When() guards
6. **Test thoroughly** - xUnit + NSubstitute + Shouldly
7. **Document clearly** - XML docs for all public APIs

**Remember**: This is a library, not a Web API. No controllers, routes, or versioned folders.

---
> Source: [JohnnySchaap/Audabit.Common.ApiVersioning.AspNet](https://github.com/JohnnySchaap/Audabit.Common.ApiVersioning.AspNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-31 -->
