# .NET Aspire Repository Instructions

✨ _Howdy, code connoisseur!_ ✨

This repository contains a .NET Aspire solution targeting Azure Container Apps with comprehensive observability, secrets management, and CI/CD practices. Fair warning: we're a little picky about architecture here, but that's because we care.

## Project Overview

- **Technology Stack**: .NET 9, .NET Aspire 8.2+, Azure Developer CLI (azd) 1.9+
- **Target Platform**: Azure Container Apps Environment with Dapr and KEDA
- **Architecture**: Microservices with service discovery, distributed tracing, and centralized configuration

## Architecture-First Development

_Listen, we've got a pretty story to tell about how this codebase is organized._ 📖 Before making any code recommendations or changes, always read the script first:

1. Read the relevant `ARCHITECTURE.md` file for context on purpose, intent, and existing patterns
2. Check solution root `ARCHITECTURE.md` for high-level topology, service discovery, and deployment patterns
3. Check project-specific `ARCHITECTURE.md` when modifying code in that project
4. Review the "Good vs Bad Implementations" section in the relevant `ARCHITECTURE.md` to see real-world examples
5. Ground recommendations in documented architecture decisions (service discovery, health checks, versioning, secrets management)
6. Reference existing endpoints and patterns from architecture docs before suggesting new ones
7. Respect documented configurations (OpenTelemetry, resilience, caching strategies)
8. Match documented patterns shown in good examples, avoiding documented anti-patterns

### Architecture Documentation Map

- `/ARCHITECTURE.md` - Solution-wide: topology, deployment, CI/CD, observability, troubleshooting, dependencies & change impact analysis
- `/aspire1.AppHost/ARCHITECTURE.md` - Service orchestration, service discovery, AppHost configuration, Redis & Azure App Config integration
- `/aspire1.WeatherService/ARCHITECTURE.md` - API endpoints, OpenTelemetry, health checks, deployment, Redis caching, feature flags
- `/aspire1.Web/ARCHITECTURE.md` - Blazor Server, SignalR, HTTP clients, WeatherApiClient patterns, session state, feature flags
- `/aspire1.ServiceDefaults/ARCHITECTURE.md` - OpenTelemetry, health checks, resilience, service discovery

Each `ARCHITECTURE.md` includes a "Best Practices vs Anti-Patterns" section with real examples from this codebase.

### Loading Architecture Documentation Efficiently

_Pro tip from someone who definitely knows how to read the room:_ **Only load the ARCHITECTURE.md files relevant to your current task scope:** (TL;DR: we're not asking for your entire life story, just the relevant chapters.)

1. **Always load first**: `/ARCHITECTURE.md` for high-level context, dependencies, and change impact analysis

2. **Load based on task scope**:

   - **Modifying/reviewing AppHost**: Load `/aspire1.AppHost/ARCHITECTURE.md`
   - **Modifying/reviewing WeatherService API**: Load `/aspire1.WeatherService/ARCHITECTURE.md`
   - **Modifying/reviewing Web frontend**: Load `/aspire1.Web/ARCHITECTURE.md`
   - **Modifying/reviewing ServiceDefaults**: Load `/aspire1.ServiceDefaults/ARCHITECTURE.md`
   - **Cross-cutting concerns** (telemetry, health checks, resilience): Load `/aspire1.ServiceDefaults/ARCHITECTURE.md` + relevant project docs
   - **Service discovery issues**: Load `/aspire1.AppHost/ARCHITECTURE.md` + affected service docs
   - **Deployment/infrastructure**: Load `/ARCHITECTURE.md` + `/aspire1.AppHost/ARCHITECTURE.md`

3. **Load multiple when**:

   - Changes affect service contracts (DTO structures, endpoints): Load both WeatherService and Web docs
   - Adding new Azure resources: Load AppHost + affected service docs
   - Changing ServiceDefaults: Load ServiceDefaults + all service docs (WeatherService, Web)
   - Investigating integration issues: Load docs for all services involved in the flow

4. **Don't load**:
   - Documentation for services you're not modifying
   - All docs "just in case" - be selective based on actual task scope

**Example scoping rules**:

- Task: "Add new endpoint to WeatherService" → Load `/ARCHITECTURE.md` + `/aspire1.WeatherService/ARCHITECTURE.md`
- Task: "Fix service discovery between Web and WeatherService" → Load `/ARCHITECTURE.md` + `/aspire1.AppHost/ARCHITECTURE.md` + `/aspire1.Web/ARCHITECTURE.md` + `/aspire1.WeatherService/ARCHITECTURE.md`
- Task: "Update health check patterns" → Load `/ARCHITECTURE.md` + `/aspire1.ServiceDefaults/ARCHITECTURE.md`
- Task: "Add feature flag to UI" → Load `/aspire1.Web/ARCHITECTURE.md`
- Task: "Understand dependencies before changing DTO" → Load `/ARCHITECTURE.md` (Dependencies & Change Impact section) + `/aspire1.WeatherService/ARCHITECTURE.md` + `/aspire1.Web/ARCHITECTURE.md`

## Code Patterns

### Service Discovery

💕 _Use `WithReference()` for service-to-service communication_ — it's basically service speed dating, and we're matchmakers around here.

- Avoid hard-coded URLs; leverage Aspire''s service discovery mechanisms (hard-coding URLs is so 2010)
- Example: `builder.AddProject<Projects.aspire1_Web>("webfrontend").WithReference(apiService)` ✓

### Health Checks

- Use versioned health endpoints (e.g., `/health/detailed`) — think of it as a health report card
- Match existing endpoint naming conventions (`/version`, `/health/detailed`) — consistency is _chef's kiss_
- Include version metadata in health responses (yes, we're that vain about versions)

### Resilience

- Follow documented resilience patterns (retry, circuit breaker) from ServiceDefaults
- Apply appropriate timeouts and fallback strategies
- ServiceDefaults automatically configures HttpClient resilience

### Observability

- Use documented OpenTelemetry patterns
- Exclude health endpoints from traces to reduce noise
- Ensure all services emit structured logs and metrics
- All telemetry flows to Application Insights in Azure

## Deployment and Infrastructure

### Target Platform

- Default deployment target: Azure Container Apps (ACA Environment with Dapr)
- Do not suggest Azure App Service or Azure Kubernetes Service unless explicitly requested

### Secrets Management

🔐 _Keep your secrets secret_ (seriously, we mean it):

- **Local Development**: Use User Secrets (`dotnet user-secrets`) and azd environment variables
- **Dev/Production**: Use Azure Key Vault references with managed identity only (this is not optional, bestie)
- **Never**: Store secrets in `appsettings.json`, connection strings in AppHost code, or commit secrets to source control — we'll know 👀
- Use user-assigned managed identities when cross-resource access is required (fancy and secure, we like it)
- For local development: Azure CLI authentication or `.env` files with User Secrets

### CI/CD Pipeline Requirements

- Use GitHub Actions with Azure Developer CLI (azd) only
- Cache dotnet restore, NuGet packages, and azd bicep modules
- Run restore/build/test in parallel where possible
- Target deployment time: under 120 seconds for typical Aspire apps
- Zero secrets in logs
- Use matrix strategy for multiple environments
- Provide exact workflow YAML snippets and azd hooks when suggesting improvements

## Azure Services Configuration

### Feature Flags (Azure App Configuration)

- Use `Microsoft.Extensions.Configuration.AzureAppConfiguration` + `Microsoft.FeatureManagement.AspNetCore`
- Authenticate with managed identity only (never connection strings)
- Configure sentinel key for 30-second cache refresh
- Use labels for environments (dev, staging, prod)
- Namespace flags appropriately (e.g., `FeatureName:SubFeature`)
- In AppHost: `AddAzureAppConfiguration().WithReference()`
- Local development: `UseFeatureFlags()` with `appsettings.json` fallback
- Enable/disable via Azure Portal or CLI with zero redeployment

### Feature Flag Patterns

- Use `IFeatureManager` for runtime checks
- Use `[FeatureGate]` attribute for controllers and Minimal APIs
- Use `IVariantFeatureManager` for A/B testing
- Keep flags short-lived (less than 90 days)
- Remove unused flags aggressively
- Test both enabled and disabled paths in unit tests using `InMemoryFeatureManager`

### Redis Caching and Sessions

- **Service**: Azure Cache for Redis (Premium tier for persistence and geo-replication)
- **In AppHost**: Use `AddRedis()` with `WithReference()`
- **Distributed Cache**: `Microsoft.Extensions.Caching.StackExchangeRedis` with `IDistributedCache`
- **Session State**: `AddStackExchangeRedisCache()` for distributed sessions
- **Expiration**:
  - Sessions: Sliding expiration 20-30 minutes
  - Cache: Absolute expiration 1-24 hours based on data volatility
- **Caching Patterns**:
  - Cache-aside for reads (check cache → miss → fetch from DB → store in cache)
  - Write-through for critical updates
  - Key naming: `{service}:{entity}:{id}` (e.g., `api:user:12345`)
  - Use `RedisKey` for type safety
  - Serialize with `System.Text.Json` for performance
  - Cache null results (5 minute TTL) to prevent database hammering
  - Monitor hit rates via OpenTelemetry custom metrics
- Use Redis channels for pub/sub scenarios
- Never store secrets in cache

### Session Management

- Use `AddSession()` + `AddStackExchangeRedisCache()` for distributed sessions
- Store only user context (userId, tenantId, culture) - never business data
- Cookie settings: `SameSite=Lax`, `HttpOnly=true`, `Secure=true` (production)
- Sliding expiration: 30 minutes, absolute maximum: 8 hours
- For APIs: Use Redis-backed JWT refresh tokens instead of sessions
- AppHost: `redis.WithReference()` ensures connection string injection

## Development Practices

### Offline-First Development

🌲 _We like our independence_ (and reliability):

- All Azure resource integrations must have local fallbacks (because the cloud isn't always sunny)
- Application must run and debug completely disconnected from Azure (we're not needy like that)
- Wrap all Azure connections in try-catch with graceful fallback to local alternatives (appsettings.json, in-memory, local emulators) — plan B is our style
- Never block application startup on Azure service availability (patience is a virtue, but startup time isn't)

### Testing Strategy

#### Unit Tests

✅ _Keep 'em fast, keep 'em honest:_

- Framework: xUnit + FluentAssertions + NSubstitute (we have commitment issues with anything slower)
- Requirements: Fast (under 100ms), no I/O, no real dependencies — yes, we're that picky
- Mock all external dependencies (don't make us ask twice)
- Naming convention: `[MethodName]_[Scenario]_[ExpectedResult]` (readability is love)
- Target: 80%+ coverage on business logic (because untested code is spicy code)

#### Integration Tests

🧪 _This is where the magic happens_ (but keep it real):

- Use `Aspire.Hosting.Testing` package (no cutting corners here)
- Spin up `DistributedApplication` with real containers (Postgres, Redis, etc.) — bring the whole band
- Use `WebApplicationFactory` for HTTP services (tested the right way, not the lazy way)
- Clean up resources in `Dispose()` (leave it cleaner than you found it)
- Tests should hit real APIs and databases but complete in under 5 seconds each (speed _and_ authenticity)
- For AppHost: Use `DistributedApplicationTestingBuilder`, assert service discovery works, verify health endpoints, test `WithReference()` chains (because if it's not tested, it doesn't exist)

#### Test Structure

- Unit tests: `[ProjectName].Tests`
- Integration tests: `[ProjectName].IntegrationTests`
- E2E tests: `tests/` directory (Playwright, runs via npm scripts)
- Keep integration/E2E tests separate so CI can run unit tests quickly (< 10 seconds)
- Run integration and E2E tests in parallel matrix jobs (faster feedback)

#### Integration Test Best Practices

- Use unique database names per test class (Guid suffix)
- Seed minimal data
- Always clean up in `finally` or `Dispose()`
- Use Respawn for database resets between tests if needed
- Never test deployment infrastructure here (use `azd deploy` smoke tests for that)

#### End-to-End Tests (Playwright)

🎭 _The real deal_ — testing the entire user experience from browser to backend:

**Framework & Setup:**

- Use [Playwright Test](https://playwright.dev) with TypeScript
- Tests run across **3 browsers**: Chromium, Firefox, WebKit (simultaneously)
- Auto-startup of services via `playwright-setup.ts` (global setup/teardown)
- Tests run against **live services** — full integration testing

**Test Suites in `tests/`:**

- `weather-api.spec.ts` — REST API contracts, status codes, response formats, health checks
- `web-app.spec.ts` — Blazor UI interactions, navigation, Counter state, responsive design
- `integration.spec.ts` — Full data flows, service discovery, Redis caching, session persistence
- `performance.spec.ts` — Load times, concurrent users, bundle sizes, cache performance

**Key Playwright Patterns:**

```typescript
// Navigate and wait for readiness
await page.goto(`${baseUrl}/weather`);
await page.waitForLoadState("networkidle");

// Direct API testing via page.request
const response = await page.request.get(`${baseUrl}/weatherforecast`);
expect(response.status()).toBe(200);

// UI interactions
await page.click('a[href="counter"]');
await page.waitForSelector(".weather-card", { timeout: 10000 });

// Performance measurement
const startTime = Date.now();
await page.goto(`${baseUrl}/`);
const loadTime = Date.now() - startTime;
expect(loadTime).toBeLessThan(5000);

// Responsive testing
await page.setViewportSize({ width: 375, height: 667 }); // Mobile
```

**Performance & Assertions:**

- Each E2E test should complete in < 10 seconds (timeout: 30 seconds)
- Measure and assert load times (e.g., home page < 5 seconds)
- Validate cache behavior: First load vs. second load performance
- Check custom telemetry generation during test flows
- Responsive design: Test at desktop (1920x1080), tablet (768x1024), mobile (375x667)

**Anti-Patterns (Don't Do These):**

- ❌ Hard-code service URLs — use environment variables from `playwright.config.ts`
- ❌ Use `page.waitForTimeout()` — use `waitForSelector()`, `waitForResponse()` instead
- ❌ Test only one browser — E2E tests must run all 3 (you catch cross-browser bugs)
- ❌ Ignore test failures — each failure is a real user-facing issue
- ❌ Make tests sleep — proper waits prevent flaky tests
- ❌ Test infrastructure — use `azd deploy` smoke tests for that; E2E tests focus on user flows

**Environment Variables for Playwright:**

```bash
PLAYWRIGHT_WEB_URL         # Web frontend URL (default: https://localhost:7296)
PLAYWRIGHT_BASE_URL        # API base URL (default: http://127.0.0.1:43141)
PLAYWRIGHT_SERVICE_HOST    # API host (default: 127.0.0.1)
PLAYWRIGHT_SERVICE_PORT    # API port (default: 43141)
PLAYWRIGHT_KILL_SERVICE    # Kill service after tests (default: false)
```

**Running E2E Tests:**

```bash
npm test                    # All tests, all browsers
npm run test:api            # API tests only
npm run test:web            # UI tests only
npm run test:integration    # Service flow tests
npm run test:performance    # Load & performance tests
npm run test:headed         # Show browser UI (debugging)
npm run test:debug          # Step-through debugging
npm run test:report         # View HTML report
```

## Build and Validation

### Initial Setup

Before starting development, install Git hooks to enforce branching strategy:

```powershell
.\scripts\Install-GitHooks.ps1
```

This installs hooks that prevent direct commits and pushes to `main`/`master` branches, enforcing a feature branch workflow.

### Build Commands

```bash
# Restore dependencies
dotnet restore

# Build solution
dotnet build aspire1.sln --no-restore

# Run tests
dotnet test aspire1.sln --no-build --verbosity normal

# Run AppHost (starts all services via Aspire Dashboard)
dotnet run --project aspire1.AppHost/aspire1.AppHost.csproj
```

### Versioning

- Uses MinVer for automatic semantic versioning based on git tags
- Version command: `minver` (from solution root)

### Deployment

```bash
# Full provision and deploy
azd up

# Deploy only (after infrastructure exists)
azd deploy

# Tear down environment
azd down --force --purge
```

### Common Issues and Workarounds

- Always run `dotnet restore` before building after changing dependencies
- If build fails, check that all ARCHITECTURE.md files are present (they''re referenced by projects)
- For local debugging, ensure Aspire Dashboard is accessible (typically https://localhost:15888)
- When adding new services, update the AppHost Program.cs and relevant ARCHITECTURE.md files

## Code Examples

### Good: Service Discovery with WithReference

```csharp
// In AppHost Program.cs
var weatherService = builder.AddProject<Projects.aspire1_WeatherService>("weatherservice");
var webApp = builder.AddProject<Projects.aspire1_Web>("webfrontend")
    .WithReference(weatherService);  // ✓ Automatic service discovery
```

### Bad: Hard-coded URLs

```csharp
// ✗ Don''t do this (seriously, we''ll judge)
var httpClient = new HttpClient { BaseAddress = new Uri("https://localhost:7123") };
```

### Good: Key Vault Reference for Secrets

```bash
# Set in Azure environment
azd env set ConnectionStrings__MyDb "@Microsoft.KeyVault(SecretUri=https://kv.vault.azure.net/secrets/mydb-connection)"
```

### Bad: Secrets in Configuration Files

```json
// ✗ Never put secrets in appsettings.json (we will find out)
{
  "ConnectionStrings": {
    "MyDb": "Server=prod.db.com;Password=secret123;"
  }
}
```

### Good: Versioned Health Endpoint

```csharp
app.MapGet("/health/detailed", (IConfiguration config) => new
{
    Status = "Healthy",
    Version = config["App:Version"],
    Timestamp = DateTime.UtcNow
}); // Status updates are our love language
```

### Good: HTTP Client with Resilience

```csharp
// WeatherApiClient already includes resilience patterns from ServiceDefaults (we built this just for you)
public class WeatherApiClient(HttpClient httpClient)
{
    public async Task<WeatherForecast[]> GetWeatherAsync(CancellationToken cancellationToken = default)
    {
        return await httpClient.GetFromJsonAsync<WeatherForecast[]>("/weatherforecast", cancellationToken)
            ?? []; // Resilience is hot
    }
}
```

## Response Guidelines

_We're high-maintenance, but we're worth it._ 💫 When providing code recommendations:

- Always reference the specific ARCHITECTURE.md file that contains relevant patterns (cite your sources, it's hot)
- Show both the anti-pattern to avoid and the correct implementation (before/after energy)
- Include complete code examples, not just snippets (we want the full picture, thanks)
- Verify suggestions against documented patterns (don't wing it)
- When suggesting new endpoints or services, follow existing naming conventions documented in ARCHITECTURE.md files (consistency is attractive)
- If a WeatherApiClient-style typed client already exists, use that pattern for new HTTP clients (imitation is the sincerest form of flattery)
- Provide clear explanations of the reasoning behind recommendations (we like an explanation, it's intimate)
- Highlight potential pitfalls or common mistakes to avoid (let's save each other the heartache)
- Suggest validation steps to verify implementations (trust, but verify)

<instructions>
<instruction>
<description>**CRITICAL**: Read this file FIRST before answering ANY Azure-related questions or performing ANY Azure operations. This file must be loaded as context whenever Azure is explicitly mentioned. It provides accurate and up-to-date information about Azure services, deployment patterns, and best practices specific to this .NET Aspire application. **MUST** be read when user mentions: Azure, Azure Container Apps, Azure Developer CLI (azd), Bicep, Azure services (Redis, App Configuration, Key Vault, Application Insights), or Azure deployment. Load this instruction file ONLY for Azure-related requests, NOT for generic cloud or deployment questions.</description>
<file>.github/instructions/azure.instructions.md</file>
</instruction>
</instructions>


---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MSBart2)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MSBart2)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
