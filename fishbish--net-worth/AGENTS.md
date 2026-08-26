# Copilot Instructions

## Build, test, and lint commands

- Solution file: `.\net-worth.slnx`
- Web app project: `.\net-worth-app\net-worth-app.csproj`
- Restore: `dotnet restore .\net-worth.slnx`
- Build: `dotnet build .\net-worth.slnx`
- Run locally: `dotnet run --project .\net-worth-app\net-worth-app.csproj --launch-profile https`
- Tests: there is currently no test project in this repository, so there is no `dotnet test` or single-test command to use yet.
- Lint/format: there is currently no repository-specific lint or formatting command checked in.

## High-level architecture

- This repo is a single ASP.NET Core + Blazor Web App targeting `net10.0`. `Program.cs` wires up Microsoft Identity Web authentication, a global fallback authorization policy, EF Core with SQL Server, Razor Pages for the Microsoft Identity UI, and interactive server Razor components.
- The main UI lives under `net-worth-app\Components`. `Components\Pages` contains the routed pages, `Components\Layout` contains the shared navigation shell, and `HistoryLineChart.razor` renders the history SVG chart directly in Razor instead of relying on a JS charting library.
- Business logic lives in scoped services under `net-worth-app\Services`; the Razor pages are intentionally thin and call those services for reads, validation, and persistence.
- The core data model is:
  - `Institution` -> shared catalog of account providers
  - `Account` -> user-owned record that must belong to an `Institution`
  - `Instrument` -> shared catalog record reused across accounts
  - `AccountInstrument` -> link between one account and one shared instrument
  - `AccountSnapshot` -> dated account snapshot header that stores either an account balance or grouped instrument balances
  - `InstrumentSnapshot` -> dated balance for one account-specific instrument link within an account snapshot
- `CurrentUserAccessor` is the user boundary. Services for accounts, instruments, snapshots, and history all resolve the authenticated user id from `oid` or `sub` and must scope reads/writes to that user.
- History is intentionally a single `/history` experience. `HistoryService` supports total net worth, single-account history, and single-instrument history from the same page by aggregating snapshot data with filters.
- `Catalogue` is the shared maintenance surface for institutions and instruments, while the Accounts page handles account-specific linking, shared-instrument editing, and unlinking.
- `NetWorthDbContext` sets all foreign keys to `DeleteBehavior.NoAction` first, then explicitly enables cascade delete only for `AccountInstrument -> Account`.

## Key conventions

- Use `NetWorth` as the root namespace.
- Use SQL Server connection strings with `Server=(local)` unless the user explicitly asks for a different target.
- Use `Institution` as the provider model name; do not replace it with a plain string institution field on accounts.
- `Account` should keep a required `InstitutionId`/`Institution` relationship rather than storing institution names directly.
- Do not add a `Notes` field to `Account`.
- Treat small catalog entities such as `Institution` and shared `Instrument` as direct CRUD over the EF entity, including direct UI binding and bulk `SaveAllAsync` flows on the catalogue page.
- Treat user-owned or more complex areas such as accounts, instruments, snapshots, and history with service-layer models (`Lookup`, `Upsert`, editor/page models) instead of binding EF entities directly.
- Scope all account-, instrument-, snapshot-, and history-related queries and writes to the current user. Validate ownership and FK existence explicitly rather than assuming ids are valid.
- Keep related catalog creation out of complex aggregate save flows; for example, do not create institutions inside account save operations.
- Shared instruments are universal records linked through `AccountInstrument`, not account-owned child rows.
- Removing an instrument from an account should unlink the `AccountInstrument` row only; editing an instrument updates the shared record for every linked account.
- Enforce unique instrument names globally and unique tickers globally when a ticker value is present.
- Manage institutions and instruments from the single `/catalogue` page instead of separate catalogue screens.
- Snapshot rule: for a given account and date, save either the account-level balance or instrument-level balances, never both.
- Instrument-level snapshots should reference `AccountInstrumentId`, not a direct `InstrumentId`, because instruments can be reused across accounts.
- Blank snapshot values mean “leave this account or instrument out of the snapshot”; saving an account with no values removes existing snapshot data for that date.
- Liability balances stay stored as positive values. The sign is flipped only when total net worth is computed across accounts.
- Prefer EF Core data annotations (`[Required]`, `[MaxLength]`, `[Column(TypeName = ...)]`) and `[Index]` attributes over Fluent API when practical.
- Rely on EF key conventions for `Id` properties instead of adding explicit `HasKey` configuration unless it is truly needed.
- Keep enum values stored as integers.
- Keep `DbSet` properties in `Data\NetWorthDbContext.cs` alphabetical.
- In model classes, keep navigation properties directly under their FK properties and preserve blank lines between properties.
- Keep the `NetWorth` connection string in `appsettings.json`; do not duplicate the same value in `appsettings.Development.json`.
- Do not generate or apply EF Core migrations unless the user explicitly asks for it.
- Service methods commonly report validation and not-found problems with `InvalidOperationException`; page components catch those and show the message in Bootstrap alerts.

---
> Source: [fishbish/net-worth](https://github.com/fishbish/net-worth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
