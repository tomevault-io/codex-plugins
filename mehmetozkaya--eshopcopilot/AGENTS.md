# Copilot Instructions for EShopCopilot

## Architecture

This is a **.NET 10 + Aspire 13** distributed eShop with four projects:

| Project | Role |
|---|---|
| `EShop.AppHost` | Aspire orchestrator — wires resources, sets up service discovery and health checks |
| `EShop.ApiService` | Minimal API backend — all business logic and HTTP endpoints |
| `EShop.Web` | Blazor Server (Interactive Server) frontend |
| `EShop.ServiceDefaults` | Shared cross-cutting concerns: OTel, health checks, resilience, service discovery |

**Service wiring (AppHost.cs):** `webfrontend` holds a reference to `apiservice` and waits for it to be healthy before starting. Service discovery resolves the API URL automatically at runtime — never hardcode `localhost` URLs in `EShop.Web`.

`EShop.ServiceDefaults` must be referenced and `builder.AddServiceDefaults()` called in every service (`ApiService`, `Web`). Never duplicate health check or OTel setup outside of `Extensions.cs`.

## Build & Run

```bash
# Run the full distributed app via Aspire CLI (preferred — starts dashboard + all services)
aspire run --project src/EShop.AppHost

# Alternatively, use dotnet run on the AppHost
dotnet run --project src/EShop.AppHost

# Build the entire solution
dotnet build src/EShop.sln

# Run a single project directly (no orchestration, no service discovery)
dotnet run --project src/EShop.ApiService
dotnet run --project src/EShop.Web
```

> Always prefer `aspire run` over running individual services — service discovery, health checks, and the Aspire dashboard are only available when running through the AppHost.

## Aspire Rules

- **Always call `builder.AddServiceDefaults()`** in every service project (`ApiService`, `Web`) immediately after `WebApplication.CreateBuilder(args)`. This wires OTel, health checks, resilience, and service discovery in one call — never replicate any of these manually.
- **Always call `app.MapDefaultEndpoints()`** before `app.Run()` in every service to expose `/health` and `/alive` endpoints that Aspire monitors.
- When adding a new service to the solution, register it in `AppHost.cs` with `.WithHttpHealthCheck("/health")`. If it depends on another service, chain `.WithReference(dep).WaitFor(dep)`.
- Never hardcode service URLs. Use the Aspire-injected service name (e.g., `"apiservice"`) as the `HttpClient` base address — service discovery resolves it automatically at runtime.
- New projects must reference `EShop.ServiceDefaults` — do not copy OTel or health check setup into project-local code.

## ApiService Conventions

**Folder structure:**
- `Models/` — plain domain classes (no attributes, no interfaces)
- `Services/` — business logic with in-memory `List<T>`; registered as **Singleton** so state persists during the app lifecycle
- `Endpoints/` — static classes with `Map*Endpoints(this IEndpointRouteBuilder app)` extension methods

**Adding a new domain entity** always requires these three steps in order:
1. `Models/MyEntity.cs` — plain class, `string` properties default to `string.Empty`
2. `Services/MyEntityService.cs` — in-memory CRUD, singleton-registered in `Program.cs`
3. `Endpoints/MyEntityEndpoints.cs` — Minimal API route group, called in `Program.cs`

**Minimal API structure — always use `MapGroup`:**

```csharp
public static class ProductEndpoints
{
    public static void MapProductEndpoints(this IEndpointRouteBuilder app)
    {
        var group = app.MapGroup("/api/products").WithTags("Products");

        group.MapGet("/", (ProductService service) =>
            Results.Ok(service.GetAll()));

        group.MapGet("/{id:int}", (int id, ProductService service) =>
            service.GetById(id) is Product product
                ? Results.Ok(product)
                : Results.NotFound());

        group.MapPost("/", (Product product, ProductService service) =>
        {
            var created = service.Create(product);
            return Results.Created($"/api/products/{created.Id}", created);
        });

        group.MapPut("/{id:int}", (int id, Product product, ProductService service) =>
            service.Update(id, product) is Product updated
                ? Results.Ok(updated)
                : Results.NotFound());

        group.MapDelete("/{id:int}", (int id, ProductService service) =>
            service.Delete(id) ? Results.NoContent() : Results.NotFound());
    }
}
```

- Route groups are mounted at `/api/{entities}` with `.WithTags("EntityName")`
- Use route constraints (`{id:int}`) on all `id` parameters
- Inject services directly as lambda parameters — do not use `[FromServices]`
- Keep lambda bodies single-expression where possible; extract helpers only when logic is non-trivial

**HTTP result conventions:**
- `GET` (list) → `Results.Ok(...)`
- `GET` (single) → `Results.Ok(...)` or `Results.NotFound()`
- `POST` → `Results.Created($"/api/{entities}/{id}", created)`
- `PUT` → `Results.Ok(updated)` or `Results.NotFound()`
- `DELETE` → `Results.NoContent()` or `Results.NotFound()`

## EShop.Web Conventions

- Blazor Server with **Interactive Server** render mode
- Components live under `Components/` — pages go in `Components/Pages/`
- `@page` directive required on all page components
- Use `OutputCache` for cacheable pages

## Code Style & Best Practices

**C# / .NET 10**
- Target framework is `net10.0` — use the latest language and runtime features freely
- `Nullable` and `ImplicitUsings` are enabled in all projects; do not disable them
- Use C# 12 **collection expressions** (`[...]`) for all list/array initialization — never `new List<T> { ... }`
- Prefer `var` for local variables when the type is clear from the right-hand side
- Prefer expression-bodied members and pattern matching over verbose `if/else` chains
- No `#region` blocks

**Minimal API**
- All endpoints must be grouped with `MapGroup` — never call `app.MapGet(...)` directly in `Program.cs`
- One static `*Endpoints` class per domain entity; expose a single `Map*Endpoints(this IEndpointRouteBuilder app)` extension method
- `Program.cs` should only contain composition root logic: `builder.*`, `app.Use*`, `app.Map*Endpoints()`, `app.Run()` — no inline route handlers
- Register services with the correct lifetime: **Singleton** for in-memory stores, **Scoped** for database-backed services (when introduced)

**Aspire**
- `builder.AddServiceDefaults()` must be the first call after `CreateBuilder` in every service
- `app.MapDefaultEndpoints()` must appear before `app.Run()` in every service
- Use collection expressions when seeding in-memory data in services

---
> Source: [mehmetozkaya/EShopCopilot](https://github.com/mehmetozkaya/EShopCopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-02 -->
