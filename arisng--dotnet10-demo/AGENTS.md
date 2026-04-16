# .NET 10 demos — Copilot coding-agent notes

## Overview & Scope

This repository is a **progressive workshop** demonstrating modern .NET 10 patterns across 8 incremental demos:

| Demo        | Focus                        | Key Patterns                                                   |
| ----------- | ---------------------------- | -------------------------------------------------------------- |
| **demo1**   | Identity Foundation          | ASP.NET Core Identity, Bootstrap integration                   |
| **demo2**   | Dual-Mode Handoff (Baseline) | Blazor Server ↔ WASM transition, prerender-safe DI             |
| **demo3**   | BFF + RBAC                   | Permission-based authorization, role-permission junction table |
| **demo4**   | Entra ID Integration         | OBO flow, Microsoft Graph, enterprise auth                     |
| **demo5**   | Downstream API               | Server-to-server communication, protected scopes               |
| **demo5.1** | Distributed Modular Monolith | .NET Aspire, YARP Proxy, "Two Locks" security                  |
| **demo6**   | Multi-Tenant SaaS            | Finbuckle, Multi-Identity, Data Isolation                      |
| **demo7**   | Feature Flags & Hardening    | Microsoft.FeatureManagement, Azure AppConfig                   |

**Key assumption:** demo2 is the baseline for monolithic demos; demo5.1 is the baseline for Aspire-based distributed demos.

## Workshop Project Structure

### Standard Layout (Demos 1-5)

```
demo<N>/
├── .docs/                      # Demo-specific documentation
│   ├── issues/                 # Demo-specific markdown-based issue (deprecated)
│   └── research/               # Demo-specific implementation notes
├── Demo<N>.<Name>/             # Server (Blazor Server, APIs, Identity)
├── Demo<N>.<Name>.Client/      # Client (Blazor WASM, prerendered)
├── Demo<N>.<Name>.Shared/      # Shared DTOs, models, interfaces
└── README.md                   # Demo-specific walkthrough
```

### Aspire-Based Layout (Demos 5.1+)

```
demo<N>/
├── Demo<N>.<Name>.AppHost/          # .NET Aspire Orchestrator
├── Demo<N>.<Name>.ServiceDefaults/  # Shared telemetry/discovery
├── Demo<N>.<Name>.ApiService/       # Modular Monolith Backend
├── Demo<N>.<Name>.Web/              # Blazor Frontend (BFF with YARP)
├── Demo<N>.<Name>.Web.Client/       # Blazor WASM Client
└── Demo<N>.<Name>.Shared/           # Shared models/interfaces
```

## Creating a New Demo

### Quick Start

Use the VS Code task:
1. `Tasks: Run Task` → "Create New Demo (Copy Previous)"
2. Enter demo number and name (PascalCase)
3. Task auto-copies `demo<n-1>` → `demo<n>`, renames namespaces, cleans build artifacts

### Manual Script Approach

```powershell
scripts/copy-demo.ps1 -NewDemoNumber <n> -DemoName <PascalCase>
```

### Post-Copy Steps

1. Update namespaces to match new project name exactly
2. Update `demo<n>/README.md` with goal, prerequisites, and what's new
3. Update root `README.md` demo table with new demo entry
4. Test the build: `dotnet build demo<n>/Demo<n>.<Name>.slnx`

### Pattern Selection Rules (Required)

- Every new demo **must** introduce at least one pattern from the patterns catalog.
- Pattern selection **must** build on prior demos and reflect a strategic progression (foundation → integration → distribution → hardening).
- Document the chosen patterns in the demo README and link back to the catalog entry for each pattern.

### Namespace Convention

- Namespace follows project name exactly: `Demo<N>.<Name>` → `namespace Demo<N>.<Name>;`
- Permissions use lowercase dot-notation: `weather.read`, `users.delete`, `forecast.write`

## Authentication & Authorization Patterns

### Identity Foundation (demo1+)

- **Auth Method:** ASP.NET Core Identity cookie-based authentication
- **Schema Version:** Must use `IdentitySchemaVersions.Version3` for passkey support (demo2+)
- **Endpoints:** Keep `app.MapAdditionalIdentityEndpoints()` to wire passkey endpoints and `/Account/*` pages
- **Default Users (demo3+):**
  | Email             | Password    | Role    |
  | ----------------- | ----------- | ------- |
  | admin@local.app   | Admin123!   | Admin   |
  | manager@local.app | Manager123! | Manager |
  | user@local.app    | User123!    | User    |

### Permission-Based RBAC (demo3+)

**Data Model:**
- Role → Permissions via junction table (`Data/RolePermission.cs`)
- Permissions seeded in `Data/DbSeeder.cs`

**Request Flow:**
```
HTTP Request
  ↓
IClaimsTransformation (adds "permission" claims)
  ↓
PermissionAuthorizationHandler (enforces PermissionRequirement)
  ↓
Minimal API / Controller returns response
```

**Implementation Checklist** when adding a new permission:
1. Add to seed data in `Data/DbSeeder.cs`
2. Add server policy in `Program.cs`: `AddAuthorizationBuilder().AddPolicy("...", ...)`
3. Add client policy in `.Client/Program.cs`: `AddAuthorizationCore()` requires the `permission` claim
4. Reference in minimal APIs: `.RequirePermission("...")`  (see `Authorization/AuthorizationExtensions.cs`)
5. Reference in Razor components: `<AuthorizeView Resource="..." Action="...">`

### Entra ID Integration (demo4+)

- **Protocol:** OpenID Connect (OIDC) with Microsoft Identity Web
- **Token Flow:** OAuth 2.0 Authorization Code flow with PKCE
- **OBO (On-Behalf-Of):** Used to call Microsoft Graph and downstream APIs with user's delegated permissions
- **Downstream APIs:** Protected with `AddMicrosoftIdentityWebApi()`, validate Bearer tokens and scopes

## Blazor Architecture & DI

### Prerender-Safe Service Abstraction

Routable components inject shared interfaces, not implementations:

**Interface Definition** (`.Client/Services/IAppServices.cs`):
```csharp
public interface IWeatherService { ... }
public interface IUserService { ... }
```

**Server Registration** (`Program.cs`):
```csharp
builder.Services.AddScoped<IWeatherService, ServerWeatherService>();
builder.Services.AddScoped<IUserService, ServerUserService>();
```

**Client Registration** (`.Client/Program.cs`):
```csharp
builder.Services.AddScoped<IWeatherService, ClientWeatherService>();
builder.Services.AddScoped<IUserService, ClientUserService>();
```

**Why:**
- Prerender phase runs on server (uses `ServerWeatherService`)
- Interactive phases on WASM (uses `ClientWeatherService` + `HttpClient`)
- Single component code works in both contexts

### Blazor Interactivity (InteractiveAuto)

- **First Visit:** 4 phases (SSR → SignalR handshake → WASM init → WASM render)
- **Cached Visit:** 3 phases (skip WASM init if already cached)
- **Caching Headers:** `max-age=31536000` in published mode; `max-age=0` in dev
- **Static Web Assets:** Non-dev needs `StaticWebAssetsLoader.UseStaticWebAssets()` for `.Client` assets

## Server Communication

### BFF (Backend-for-Frontend) Pattern (demo3+)

- Server provides `/api/*` endpoints for client WASM to call
- All cross-origin auth handled on server (cookies are httpOnly)
- Minimal APIs use `RequireAuthorization()` and permission policies

### Server-to-Server Communication (demo5)

**Pattern:**
- BFF calls downstream API via `IDownstreamApi` service
- Uses OBO (On-Behalf-Of) flow to exchange user token for downstream token
- Downstream API validates Bearer tokens via `AddMicrosoftIdentityWebApi()`

### Distributed Modular Monolith (demo5.1+)

**Pattern:**
- **.NET Aspire:** Orchestrates service discovery and environment configuration.
- **YARP Proxy:** Implemented in the Frontend (BFF) to forward `/api/*` requests to the `ApiService`, removing manual `HttpClient` wrappers.
- **"Two Locks" Security:**
    - **Outer Lock:** Entra ID / JWT verification at the proxy/API entry point.
    - **Inner Lock:** Fine-grained Permission-Based RBAC (from demo3) enforced within vertical slices.
- **Identity Shift Left:** Identity logic move to `ApiService` while Frontend handles authentication.

## SaaS & Multi-Tenancy (demo6+)

**Pattern:**
- **Finbuckle.MultiTenant:** Handles tenant resolution (host, header, etc.) and data isolation.
- **Multi-Identity:** Toggles auth providers (Passkey vs. Entra) based on tenant configuration.
- **Data Isolation:** Supports both Shared DB (query filters) and Dedicated DB (dynamic connection strings).
- **Tenant Context in Blazor:** Uses `ITenantProvider` to persist context across SignalR circuit state.

## Feature Flags & Hardening (demo7)

**Pattern:**
- **Microsoft.FeatureManagement:** Toggles features per tenant using the configuration layer.
- **Azure App Configuration:** Centralized management of flags and secrets.
- **Hardening:** CSP, rate limiting, and OpenTelemetry observability.

## Modular Monolith Architecture

**Vertical Slices:**
Organize code by feature/domain rather than technical layer. Each slice contains its own:
- Data access (EF Core or Adapter)
- Service logic
- API endpoints (mapped via `IEndpointRouteBuilder` extensions)
- UI components

**Integration Patterns (demo6):**
1.  **Greenfield (User Service):** Direct EF Core access to local DB with tenant filters.
2.  **Legacy (Order Service):** `LegacyOrderAdapter` wraps calls to legacy HTTP APIs.
3.  **Modern (Graph Service):** `IDownstreamApi` wraps calls to Microsoft Graph via OBO.

## Port Conventions

- **Main App:** `https://localhost:7<N>10` (+ `http://localhost:5<N>10`) via `Properties/launchSettings.json`
- **demo5 Downstream API:** `https://localhost:7220` (+ `http://localhost:5220`)
- **demo6 Legacy API:** `https://localhost:7230`

**Example Dev Loop:**
```bash
cd demo5/Demo5.DownstreamApi
dotnet ef database update
dotnet watch
# In another terminal:
cd demo5/Demo5.DownstreamApi.WeatherApi
dotnet watch
```

## .NET 10 Framework Notes

- **Identity SchemaVersion:** Must use `IdentitySchemaVersions.Version3` for passkeys
- **Blazor InteractiveAuto:** 4 phases on first visit (SSR→SignalR→WASM init→WASM render), 3 phases when cached
- **WASM Caching:** HTTP caching (`max-age=31536000`) only works in published mode; dev uses `max-age=0`
- **Entra ID:** Uses `Microsoft.Identity.Web` with OBO flow for Graph/downstream APIs
- **Minimal APIs:** Use `AddAuthorizationBuilder()` fluent API for policy registration (.NET 10 pattern)
- **Static Web Assets:** Non-dev environments need `StaticWebAssetsLoader.UseStaticWebAssets()` for .Client assets

## Documentation Requirements

Each demo README must include: Goal, Prerequisites, How to Run, What's New (for demo2+).
Root `README.md` maintains the demo lineup table with focus, dependencies, and highlights.

### Patterns Catalog (Single Source of Truth)

The workspace treats `.docs/reference/patterns/` as the living reference for architectural, design, and system patterns. It is continuously updated and should guide implementation decisions and documentation updates.

- Entry point: `.docs/reference/patterns/index.md`
- Pattern entries: `.docs/reference/patterns/catalog/`
- Guidance: `.docs/reference/patterns/guidance/`

### `.docs/` Folder Convention

The `.docs/` folder supports AI-driven development workflow at two levels:

#### Root `.docs/` (Workspace-Level)
Cross-cutting documentation that applies to the entire workspace:

| Folder      | Purpose                                                                    | Naming Pattern                 |
| ----------- | -------------------------------------------------------------------------- | ------------------------------ |
| `issues/`   | markdown-based issue (deprecated, should use beads issues system instead)  | `YYMMDD_<kebab-case-title>.md` |
| `research/` | Research findings that ground implementation plans with verified facts     | `YYMMDD_<kebab-case-title>.md` |
| `agent/`    | Agentic AI workflow docs—custom agent architecture, orchestration patterns | Descriptive names              |

#### Demo-Level `.docs/` (demo5+)
Starting from demo5, each demo folder may contain its own `.docs/` folder for demo-specific documentation:

```
demo<N>/
├── .docs/
│   ├── issues/      # Demo-specific markdown-based issue (deprecated, should use beads issues system instead)
│   └── research/    # Demo-specific research and implementation notes
├── Demo<N>.<Name>/
├── Demo<N>.<Name>.Client/
└── ...
```

**When to use demo-level `.docs/`:**
- Research relevant only to that demo's feature set
- Implementation decisions that don't affect other demos

**When to use root `.docs/`:**
- Cross-demo patterns and conventions
- Workspace-wide tooling or infrastructure issues
- Research that informs multiple demos

**Usage:** Consulting both `demo<N>/.docs/research/` and root `.docs/research/` for orientation on what's next to implement. Document new discoveries at the appropriate level based on scope.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arisng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-14 -->
