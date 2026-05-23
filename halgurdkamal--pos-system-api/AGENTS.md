# POS System API - AI Coding Agent Instructions

## Product Overview

**Multi-Tenant Pharmacy POS System** - A centralized Point-of-Sale API for pharmaceutical retail chains.

### Core Concept

This system implements a **shared database architecture** where:

- **Centralized Drug Database**: One master database contains all drug information (barcodes, formulations, regulatory info, pricing guidelines)
- **Multi-Shop Registration**: Multiple pharmacy shops can register and use the system
- **Shop-Specific Inventory**: Each shop maintains its own stock levels, batches, and local pricing while referencing the central drug catalog
- **Shared Drug Catalog**: All shops access the same verified drug information (brand names, generic names, manufacturers, barcodes)

### Key Features

1. **Central Drug Repository**: Master drug catalog with complete pharmaceutical data (formulations, regulations, suppliers)
2. **Shop Registration System**: Pharmacies register and get isolated inventory management
3. **Per-Shop Inventory**: Each shop tracks its own:
   - Stock levels (quantity on hand)
   - Batch numbers and expiry dates
   - Purchase prices and selling prices
   - Storage locations
4. **Barcode Management**: Universal barcode system across all registered shops
5. **Multi-Tenant Data Isolation**: Shop-specific data segregation while sharing drug catalog

### Business Model

- **For Pharmacy Chains**: Corporate chains can manage multiple locations with consistent drug data
- **For Independent Pharmacies**: Individual shops benefit from centralized drug database without managing it
- **For Distributors**: Can provide verified drug information to multiple retail partners

## Architecture Overview

This is a **Clean Architecture ASP.NET Core 6.0 API** implementing the multi-tenant POS system. The codebase follows strict separation of concerns with dependency flow always pointing inward (API → Application → Domain).

### Layer Structure

```
src/
├── Core/Domain/              # Pure business entities (NO dependencies)
│   ├── Common/BaseEntity.cs  # All entities inherit from this
│   └── {Feature}/            # Feature-based organization
│       ├── Entities/
│       │   ├── Drug.cs       # Shared drug catalog (all shops)
│       │   ├── Shop.cs       # Shop registration & profile
│       │   ├── ShopInventory.cs  # Per-shop stock levels
│       │   └── Order.cs      # Shop-specific transactions
│       └── ValueObjects/
│           ├── Pricing.cs    # Base pricing + shop-specific pricing
│           ├── Formulation.cs # Drug formulation details
│           ├── Batch.cs      # Shop-specific batch tracking
│           └── Inventory.cs  # Stock management per shop
├── Core/Application/         # Use cases via CQRS (depends ONLY on Domain)
│   ├── Common/
│   │   ├── Interfaces/       # Repository contracts
│   │   │   ├── IDrugRepository.cs       # Central drug catalog
│   │   │   ├── IShopRepository.cs       # Shop management
│   │   │   └── IInventoryRepository.cs  # Per-shop inventory
│   │   └── Models/           # Result<T>, PagedResult<T>
│   └── {Feature}/
│       ├── Queries/          # Read operations (GetDrug, GetShopInventory)
│       ├── Commands/         # Write operations (RegisterShop, UpdateStock)
│       └── DTOs/             # API response models
├── Infrastructure/           # External concerns (implements Application interfaces)
│   ├── Data/
│   │   ├── ApplicationDbContext.cs  # EF Core DbContext
│   │   ├── Configurations/          # Entity configurations
│   │   │   ├── DrugConfiguration.cs      # Central catalog config
│   │   │   ├── ShopConfiguration.cs      # Shop entities
│   │   │   └── InventoryConfiguration.cs # Per-shop inventory
│   │   ├── Repositories/    # EF Core implementations
│   │   └── Migrations/      # Database migrations
│   └── SampleData/          # Test data generators
└── API/                     # HTTP layer
    ├── Controllers/         # REST endpoints (use MediatR, NOT repositories directly)
    │   ├── DrugsController.cs      # Central drug catalog APIs
    │   ├── ShopsController.cs      # Shop registration & management
    │   └── InventoryController.cs  # Per-shop inventory management
    └── Extensions/          # DI configuration per layer
```

### Multi-Tenant Data Model

**Shared Data (All Shops)**:

- Drug catalog (DrugId, BrandName, GenericName, Barcode, Formulation, Regulatory)
- Manufacturer information
- Supplier contacts

**Shop-Specific Data**:

- Inventory (TotalStock, ReorderPoint, Batches per shop)
- Local pricing (can override suggested pricing)
- Orders and transactions
- Customer data (per shop)
- Shop profile (name, address, license)

## Critical Patterns

### 1. CQRS with MediatR (ALWAYS use this pattern)

**Queries (Read):**

```csharp
// 1. Define query as record
public record GetDrugQuery(string DrugId) : IRequest<DrugDto?>;

// 2. Create handler in same folder
public class GetDrugQueryHandler : IRequestHandler<GetDrugQuery, DrugDto?>
{
    private readonly IDrugRepository _repository;

    public async Task<DrugDto?> Handle(GetDrugQuery request, CancellationToken ct)
    {
        var entity = await _repository.GetByIdAsync(request.DrugId, ct);
        return entity == null ? null : MapToDto(entity);
    }
}
```

**Commands (Write):**

```csharp
public record CreateDrugCommand(...) : IRequest<DrugDto>;
public class CreateDrugCommandHandler : IRequestHandler<CreateDrugCommand, DrugDto> { }
```

### 2. Feature Organization (Vertical Slices)

When adding a new feature like "Orders":

1. **Domain**: `src/Core/Domain/Orders/Entities/Order.cs`
2. **Interface**: `src/Core/Application/Common/Interfaces/IOrderRepository.cs`
3. **DTOs**: `src/Core/Application/Orders/DTOs/OrderDto.cs`
4. **Queries**: `src/Core/Application/Orders/Queries/GetOrder/` (Query + Handler)
5. **Commands**: `src/Core/Application/Orders/Commands/CreateOrder/` (Command + Handler)
6. **Repository**: `src/Infrastructure/Data/Repositories/OrderRepository.cs`
7. **Controller**: `src/API/Controllers/OrdersController.cs`
8. **Register**: Add to `ServiceCollectionExtensions.AddInfrastructureLayer()`

## Current State & Migration Notes

- **Root `Program.cs`**: Legacy minimal API endpoints (`/drug/{id}`, `/drugs`) - being phased out
- **New Clean Architecture**: All code in `src/` folder structure (see README.md)
- **Storage**: Currently in-memory `List<T>` - migrating to Entity Framework Core + PostgreSQL
- **Database**: PostgreSQL via Npgsql.EntityFrameworkCore.PostgreSQL
- **No tests yet**: Unit/integration tests should be added to `tests/` folder

## Developer Workflows

### Build & Run

```powershell
dotnet build                    # Build entire solution
dotnet run                      # Run API (http://localhost:5000)
dotnet watch run                # Auto-reload on changes
.\start-api.ps1                 # PowerShell quick-start script
```

### Testing Endpoints

- **Swagger UI**: http://localhost:5000 (auto-opens)
- **Health Check**: GET http://localhost:5000/health
- **Drugs API**: GET http://localhost:5000/api/drugs?page=1&limit=20

### Adding Packages

```powershell
dotnet add package PackageName
```

### Database Migrations (EF Core + PostgreSQL)

```powershell
# Install required packages
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools

# Create initial migration
dotnet ef migrations add InitialCreate --project . --output-dir src/Infrastructure/Data/Migrations

# Update database
dotnet ef database update

# Add new migration after model changes
dotnet ef migrations add AddOrdersTable

# Revert last migration
dotnet ef migrations remove

# Generate SQL script
dotnet ef migrations script --output migrations.sql
```

## Common Issues & Solutions

1. **MediatR handlers not found**: Ensure handler class is in same namespace hierarchy as query/command
2. **Repository not injected**: Check `ServiceCollectionExtensions.AddInfrastructureLayer()` includes configuration parameter
3. **Build errors after adding feature**: Verify all 7 steps in feature checklist completed
4. **Null reference in DTO mapping**: Check if value objects are initialized (`= new()`)
5. **EF Core migration errors**: Ensure `Microsoft.EntityFrameworkCore.Design` is installed and migrations assembly is set correctly
6. **Database connection fails**: Verify PostgreSQL connection string in `appsettings.json` and ensure PostgreSQL is running
7. **Owned entities not saving**: Configure value objects using `OwnsOne` or `OwnsMany` in entity configuration
8. **Tracking issues**: Use `AsNoTracking()` for read-only queries to improve performance

## Key Conventions

- **Namespaces**: Follow folder structure exactly (`pos_system_api.Core.Domain.Drugs.Entities`)
- **Naming**: Queries end in `Query`, Commands in `Command`, Handlers in `Handler`
- **DTOs**: Never use domain entities in API responses - always create separate DTOs
- **Async**: All repository methods are async with `CancellationToken` parameter
- **Nullability**: Enabled - use `?` for nullable reference types
- **Records**: Use for immutable queries/commands (`public record GetDrugQuery(...)`)
- **Value Objects**: Configure as owned entities in EF Core using `OwnsOne`/`OwnsMany` in entity configurations
- **Migrations**: Store in `src/Infrastructure/Data/Migrations/`, always specify output directory
- **Decimal Precision**: Use `.HasColumnType("decimal(18,2)")` for monetary values in PostgreSQL

## What NOT to Do

- ❌ Don't add dependencies to Domain layer (keep it pure)
- ❌ Don't inject repositories directly in controllers (use MediatR)
- ❌ Don't expose domain entities in API responses (use DTOs)
- ❌ Don't skip the handler when adding queries/commands
- ❌ Don't put business logic in controllers (belongs in handlers)
- ❌ Don't create repositories in Application layer (interfaces only, implement in Infrastructure)
- ❌ Don't forget to add `AsNoTracking()` for read-only queries (performance!)
- ❌ Don't use `.Result` or `.Wait()` on async methods (use `await` properly)
- ❌ Don't forget to configure owned entities in entity configurations

## Future Roadmap

- ✅ Entity Framework Core with PostgreSQL (implemented)
- Add FluentValidation for command validation
- Add authentication/authorization (JWT)
- Add unit tests (xUnit + Moq)
- Add integration tests for controllers
- Add database seeding for initial data
- Implement more features: Orders, Customers, Inventory, Reports
- Add database connection resilience (Polly)
- Add audit logging for entity changes

---
> Source: [halgurdkamal/pos_system_api](https://github.com/halgurdkamal/pos_system_api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-23 -->
