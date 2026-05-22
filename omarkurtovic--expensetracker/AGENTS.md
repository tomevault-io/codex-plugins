# Expense Tracker - AI Coding Agent Instructions

## Project Overview
ExpenseTracker is a **multi-tenant Blazor Server web app** for personal finance management. Users track transactions across accounts and categories, with features including tags, budgets, recurring transactions, and dark mode. Architecture combines **CQRS via MediatR** with **vertical slice feature organization**.

**Tech Stack**: Blazor Server, EF Core + SQLite, ASP.NET Identity, MudBlazor, FluentValidation, ApexCharts

---

## Architecture & Vertical Slices

### Feature Organization Pattern
Each feature lives in `ExpenseTrackerWebApp/Features/{FeatureName}/` with vertical structure:
- **Commands/** → MediatR command handlers (create, edit, delete)
- **Queries/** → MediatR query handlers (read operations)
- **Handlers/** → IRequestHandler implementations
- **Components/** → Razor components (UI)
- **Dtos/** → Data transfer objects
- **Validators/** → FluentValidation rules (inline with commands)

Example: `Features/Accounts/` contains Commands, Handlers, Components, Queries, and Dtos all co-located.

### CQRS & MediatR Pipeline
**All business logic routes through MediatR** (`Program.cs` registers assembly-wide):
```csharp
builder.Services.AddMediatR(options => {
    options.RegisterServicesFromAssembly(typeof(Program).Assembly);
    options.AddOpenBehavior(typeof(PerformanceBehaviour<,>));
    options.AddOpenBehavior(typeof(ValidationBehavior<,>));
});
```

**Key Behaviors** (`Features/SharedKernel/Behaviours/`):
- **ValidationBehavior**: Enforces FluentValidation before handler execution
- **PerformanceBehaviour**: Logs requests >500ms with user ID and request details

### Command Pattern Example
```csharp
// Features/Accounts/Commands/CreateAccountCommand.cs
public class CreateAccountCommand : IRequest<int> {
    public required string UserId { get; set; }
    public required AccountDto AccountDto { get; set; }
}

public class CreateAccountCommandValidator : AbstractValidator<CreateAccountCommand> {
    public CreateAccountCommandValidator() {
        RuleFor(x => x.UserId).NotEmpty();
        RuleFor(x => x.AccountDto.Name).NotEmpty();
        // Validators defined inline with commands
    }
}
```

---

## Multi-Tenancy & User Isolation

**All data is user-scoped via `IdentityUserId`** — no explicit per-request filtering needed because:
1. `ICurrentUserService` extracts logged-in user ID from `ClaimTypes.NameIdentifier`
2. DTOs/queries implicitly filter by user (see `GetAccountsWithBalanceQuery` in Accounts)
3. DbContext relationships enforce FK constraints on user data

**Critical**: When querying from database, **always filter by `IdentityUserId`** to maintain isolation:
```csharp
var userAccounts = db.Accounts
    .Where(a => a.IdentityUserId == userId)
    .ToList();
```

---

## Database Models & Relationships

**Core Entities** (`Database/Models/`):
- **Account**: User's financial accounts (linked to user via `IdentityUserId`)
- **Category**: Transaction categories (user-scoped)
- **Transaction**: Records of money movement (references Account + Category)
- **Tag/TransactionTag**: Many-to-many tags on transactions
- **Budget/BudgetAccount/BudgetCategory**: Budget tracking per category or account
- **UserPreference**: Dark mode toggle

**Key Constraint Pattern**:
- `Account.IdentityUserId` → Required FK, Cascade delete
- `Transaction.AccountId` → Required FK, Cascade delete (deleting account removes txns)
- `Transaction.CategoryId` → Required FK, Restrict delete (can't delete category with txns)

---

## Build & Test Commands

**Build**: `dotnet build ExpenseTracker.sln`
**Watch**: `dotnet watch run --project ExpenseTracker.sln`
**Tests**: `dotnet test`

**Test Fixture Pattern** (`ExpenseTrackerTests/`):
- Uses xUnit with `IClassFixture<AccountTestFixture>`
- Fixtures create isolated DbContext instances with seeded test data
- Handlers are unit-tested directly (e.g., `CreateAccountCommandHandlerTests`)

---

## Database Migrations

**Add Migration**: `dotnet ef migrations add MigrationName --project ExpenseTrackerWebApp`
**Update Database**: Applied automatically on app startup in `Program.cs`

**Seeding**: Run in `Program.cs` on startup to ensure default categories/accounts have icons/colors.

---

## Key Service Classes

- **ICurrentUserService** → Gets authenticated user ID/username from HTTP context
- **TagService** → Manages user's tags
- **UserPreferencesService** → Loads/saves dark mode
- **AppDbContext** → EF Core DbContext with all entities + relationships configured

---

## Common Tasks

### Adding a New Command
1. Create `Features/{Feature}/Commands/{CommandName}.cs` with validator
2. Create `Features/{Feature}/Handlers/{CommandHandler}.cs` implementing `IRequestHandler`
3. Inject `AppDbContext`, extract `UserId` via `ICurrentUserService`
4. Call `mediator.Send(command)` from Razor components

### Adding a Query
1. Create `Features/{Feature}/Queries/{QueryName}.cs` implementing `IRequest<T>`
2. Create handler with `Where(x => x.IdentityUserId == userId)` filter
3. Return projected DTOs, never raw entities

### Adding Validation
FluentValidation rules live inline in command validators. Use `.When()` for conditional rules and `.NotEmpty()` for required fields.

---

## UI Components & MudBlazor

Blazor Razor components live in `Components/Pages/` and `Features/{Feature}/Components/`. MudBlazor is configured with bottom-left snackbars and filled variants. ApexCharts are preconfigured with transparent background and light theme.

---

## Docker & Deployment

- **Local**: Run with SQLite (`app.db`)
- **Docker**: App runs from `/home/app.db`
- **Azure**: Live at expensetracker.net (demo: sa / Secret1!)

---

## Testing Conventions

- **Unit Tests**: Test handlers directly with mocked DbContext (fixtures provide test context)
- **Pattern**: Arrange (create context/command), Act (call handler), Assert (verify state)
- **Location**: `ExpenseTrackerTests/Features/{Feature}/*`

---
> Source: [omarkurtovic/ExpenseTracker](https://github.com/omarkurtovic/ExpenseTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
