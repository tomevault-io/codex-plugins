# Riviera.Contracts — Shared Contracts Library

## Overview
Shared contracts library for the Riviera Digital platform. Defines the public API surface between projects — DTOs, service interfaces, and shared result types. All cross-project communication goes through this library.

## Purpose
- **Single source of truth** for `OperationResult<T>` and shared result types
- **Module contracts** — DTOs and service interfaces per business module
- Consumed by: `Kontrol`, `Dock`, and any future Riviera services

## Solution Structure

```
Riviera.Contracts.slnx
│
└── src/
    ├── shared/
    │   └── Riviera.Shared.Contracts/        ← Universal result types
    │       └── Results/
    │           ├── OperationResult.cs        ← OperationResult + OperationResult<T>
    │           └── PagedResult.cs            ← Pagination wrapper
    │
    └── modules/
        └── Riviera.Modules.{Name}.Contracts/ ← Per-module contracts
            ├── DTOs/
            │   ├── Requests/                 ← *Request.cs — input DTOs
            │   └── Responses/                ← *Response.cs, *Dto.cs — output DTOs
            └── Services/
                └── I{Name}Service.cs         ← Service interface
```

## Coding Rules

### DTOs
- **Request DTOs**: suffix `*Request` — input validation attributes allowed
- **Response DTOs**: suffix `*Response` or `*Dto` — output-only, no logic
- **Never** expose domain entities — only DTOs cross module boundaries
- Use `record` or `class` consistently within a module — check existing pattern first

### Service Interfaces
- **I-prefix**: `IWebUserService`, `ICatalogService`
- All methods `async` — return `Task<T>`, `Task<OperationResult<T>>`, or `Task<OperationResult>`
- XML `<summary>` docs on all interface members — this is a public API surface

### OperationResult Pattern
```csharp
// Write operations — always return OperationResult or OperationResult<T>
Task<OperationResult<Guid>> CreateAsync(CreateRequest request);
Task<OperationResult> UpdateAsync(Guid id, UpdateRequest request);
Task<OperationResult> DeleteAsync(Guid id);

// Read operations — return the type directly (null on not found)
Task<MyDetailDto?> GetByIdAsync(Guid id);
Task<MyListResponse> GetPagedAsync(SearchRequest request);
```

### Naming Conventions
- **PascalCase**: all public types and members
- **Suffix patterns**: `*Request`, `*Response`, `*Dto`, `*ListResponse`
- **Namespace**: `Riviera.Modules.{ModuleName}.Contracts.{Area}` or `Riviera.Shared.Contracts.{Area}`

## Adding a New Module

1. Create folder: `src/modules/Riviera.Modules.{Name}.Contracts/`
2. Add `.csproj` referencing `Riviera.Shared.Contracts`
3. Create `DTOs/Requests/` and `DTOs/Responses/` subfolders
4. Create `Services/I{Name}Service.cs`
5. Add project to `Riviera.Contracts.slnx`

## Build

```powershell
dotnet build Riviera.Contracts.slnx
```

## Notes
- `Vezir.*` namespaces are legacy names — use `Riviera.*` for all new contracts
- Do NOT add business logic to this library — contracts only
- Do NOT add EF Core, infrastructure, or framework-specific dependencies

---
> Source: [Breckey-co/Contracts](https://github.com/Breckey-co/Contracts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-07 -->
