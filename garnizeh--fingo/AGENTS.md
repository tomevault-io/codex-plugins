# FinGo — GitHub Copilot Instructions

> **CRITICAL DIRECTIVES — READ FIRST**
>
> 1. **Language Policy:** All output produced by this agent — code, comments, variable names, log messages, documentation, and prose — MUST be in **English**, without exception, regardless of the language used by the user.
> 2. **Safety Protocol:** If there is ANY ambiguity in a request — architecture, naming, business rules, schema — **STOP immediately** and ask the minimum set of direct, closed questions needed to clarify intent. Do NOT speculate or produce half-formed code.
> 3. **No Magic:** Favour clarity and the Go standard library. Do not introduce abstractions that obscure the Ardan Labs flow. When in doubt, write more lines, not fewer.

> 4. **Documentation Storage:** All system documentation MUST be stored under the repository `/.docs` directory. Design documents live in `/.docs/design-docs/` and must be named using the pattern `000-context.md` where `000` is a three- or four-digit sequence and `context` is a short descriptive name (for example `0001-transition.md`). Each design doc MUST include a short backlog of tasks and acceptance criteria. Task artifacts for each design doc must live under `/.docs/tasks/000-context/` with `backlog/` and `done/` subfolders for tracking work items. Example structure:
>
> ```
> .docs/
>     design-docs/
>          0001-transition.md
>          0002-new-feature.md
>     tasks/
>         0001-transition/
>             backlog/
>             done/
>         0002-new-feature/
>             backlog/
>             done/
>     templates/
>         design-doc.md   ← template for new design docs
>         task.md         ← template for individual task files
> ```
>
> **Templates:** When creating a new design doc, copy `.docs/templates/design-doc.md` and fill in all sections. When creating a new task file, copy `.docs/templates/task.md`. Both templates contain inline instructions (inside `<!-- -->` comment blocks) that must be deleted before committing the file.
>
> **Task naming:** Task filenames follow the pattern `pN-NNN-<slug>.md` where `N` is the phase number from the design doc and `NNN` is a three-digit sequence that resets to `001` at the start of each phase. This makes it immediately clear which phase every task belongs to, which is essential for multi-phase design docs. Examples: `p1-001-rename-service.md`, `p2-001-create-credits-table.md`, `p3-002-implement-creditcardbus.md`.
>
> **Task lifecycle:** Tasks start as files in `backlog/`. When a task is completed, move the file to `done/` without renaming it. Never delete task files.
>
> **Automation rules for Copilot:**
> - When asked to create a design doc: use `.docs/templates/design-doc.md` as the base, assign the next available sequence number, create the matching `tasks/NNNN-<slug>/backlog/` and `tasks/NNNN-<slug>/done/` directories, and populate `backlog/` with one task file per item listed in the design doc's Section 8 Backlog. Task filenames must follow the `pN-NNN-<slug>.md` convention — each phase starts its own sequence at `001`.
> - When asked to create a task: use `.docs/templates/task.md` as the base, name the file `pN-NNN-<slug>.md` (using the correct phase number), place it in the correct `backlog/` folder, and link it to its parent design doc.
> - When a task is finished: move the file from `backlog/` to `done/`, set **Status: Done** and **Completed: YYYY-MM-DD** in the front matter.
> - When all tasks for a design doc are completed (i.e., `backlog/` is empty and every task file in `done/` has `Status: Done` and a `Completed: YYYY-MM-DD`), move the entire task folder (`.docs/tasks/000-context/`) into an archive location at `.docs/tasks/done/000-context/`. Ensure the `.docs/tasks/done/` directory exists before moving and do not delete task files; this is an archival reorganization step that preserves history.

> 5. **Database Test Commands:** Before running any test that requires a real database, run `make test-up` to bring up the test database environment. After the tests complete, run `make test-down` to tear down the test database. Automation scripts and Copilot helpers must follow this convention when invoking DB-backed tests.
>
> 6. **Existing Pattern First (Mandatory):** The agent MUST always follow the established project implementation patterns, especially the structural and testing conventions already used in `user`, `product`, and `home` domains. If a task appears to request a different approach, the agent MUST pause, explicitly explain the deviation and its impact, and obtain explicit user approval before implementing anything outside the existing pattern. In case of conflict, existing repository patterns are the default behavior until explicit approval is granted.
>
> Copilot and other automation should look in `/.docs/design-docs/` for the canonical design documents and in `/.docs/tasks/` for task lists and progress. If you do not agree with this policy, STOP and discuss before changing any doc storage locations.

---

## 1. Project Mission

FinGo is a **high-performance Personal Finance and Investment Manager** designed to run on a single VPS. It must handle:

| Domain | Core Capability |
|---|---|
| **Credit Cards** | Invoice cycles, statement closing dates, minimum payment tracking, per-card spending limits |
| **Investments** | Stocks, Cryptocurrencies, Fixed Income instruments, portfolio valuation, gain/loss calculation |
| **Cash Flow** | Accounts Payable/Receivable, recurring transactions, budget envelope management |
| **Dashboard** | Real-time aggregated net-worth, cash position, and investment P&L using WebSocket/SSE |

The technical foundation is the **Ardan Labs `service` pattern**: a layered Go monorepo where the business domain is isolated from transport, persistence, and infrastructure concerns.

---

## 2. Repository Layout (Canonical)

The module path is `github.com/garnizeh/fingo`. All new packages must use this prefix.

```
fingo/
├── api/
│   ├── frontends/
│   │   └── admin/               # Vite/React SPA
│   ├── services/
│   │   ├── auth/                # Auth microservice (keep as-is)
│   │   ├── metrics/             # Metrics service (keep as-is)
│   │   └── sales/               # ← rename to "fingo" or add new service here
│   └── tooling/
│       ├── admin/               # DB migration CLI (extend with fingo migrations)
│       └── logfmt/
├── app/
│   ├── domain/
│   │   ├── creditcardapp/       # ← NEW
│   │   ├── investmentapp/       # ← NEW
│   │   ├── budgetapp/           # ← NEW
│   │   ├── transactionapp/      # ← NEW
│   │   └── userapp/             # KEEP
│   └── sdk/
│       └── mid/                 # KEEP
├── business/
│   ├── domain/
│   │   ├── creditcardbus/       # ← NEW
│   │   ├── investmentbus/       # ← NEW
│   │   ├── budgetbus/           # ← NEW
│   │   ├── transactionbus/      # ← NEW
│   │   ├── userbus/             # KEEP
│   │   ├── auditbus/            # KEEP
│   │   └── productbus/          # REPLACE logic; keep as skeleton reference
│   ├── sdk/                     # KEEP — do not modify
│   └── types/
│       ├── currency/            # ← NEW value object
│       ├── ticker/              # ← NEW value object (stock/crypto symbol)
│       ├── assetclass/          # ← NEW value object
│       └── money/               # KEEP
├── foundation/                  # KEEP — do not modify unless critical
├── zarf/                        # EXTEND — add fingo service manifests
└── .docs/                       # All project documentation lives here
    └── copilot-instructions.md
```

---

## 3. Architecture Layers — Rules and Responsibilities

### 3.1 `foundation/` — Framework Primitives

**Rule: Never import from `business/` or `app/` inside `foundation/`.**

| Package | Responsibility |
|---|---|
| `foundation/web` | Thin HTTP framework. `HandlerFunc` signature is `func(ctx context.Context, r *http.Request) web.Encoder`. All handlers return an `Encoder`, never write directly to `http.ResponseWriter`. |
| `foundation/logger` | Structured logging via `*logger.Logger`. Always call `log.Info`, `log.Error` with a `context.Context` as first argument. Never use `fmt.Println` or `log.Printf`. |
| `foundation/otel` | OpenTelemetry tracing. Use `otel.GetTraceID` for log correlation. |
| `foundation/keystore` | JWT key loading. Do not bypass this for auth. |

### 3.2 `business/sdk/` — Shared Business Utilities

**Rule: These packages are domain-agnostic. Never import a domain package here.**

| Package | Responsibility |
|---|---|
| `business/sdk/sqldb` | PostgreSQL connection, `NamedExecContext`, `NamedQuerySlice`, transaction helpers. All SQL goes through this. |
| `business/sdk/delegate` | Cross-domain event bus. Use when domain A must react to an event in domain B without a direct import. |
| `business/sdk/order` | `order.By` type for SQL `ORDER BY` clauses. |
| `business/sdk/page` | `page.Page` type for limit/offset pagination. |
| `business/sdk/migrate` | Darwin-based schema migrations. All migrations are SQL files under `business/sdk/migrate/sql/`. Migrations must be **idempotent** — use `CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, and `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`. Each migration file is checksummed by darwin and recorded in the `darwin_migrations` table; if a migration fails partway through, **do NOT attempt an automated fix**. Stop, surface the error, and ask the user for manual intervention before proceeding — an incorrect recovery can corrupt the `darwin_migrations` table and leave the schema in an unrecoverable state. |

### 3.3 `business/domain/<name>bus/` — Domain Business Logic

**Rule: No `net/http`, no JSON tags, no SQL tags in this layer. Pure Go business types.**

Every domain package has exactly this internal structure:

```
<name>bus/
├── model.go        # Business types: Entity, NewEntity, UpdateEntity
├── filter.go       # QueryFilter with pointer fields (*uuid.UUID, *string, etc.)
├── order.go        # orderBy constants mapped to SQL column names
├── event.go        # Delegate event/action constants and registration
├── <name>bus.go    # Storer interface + ExtBusiness interface + Business struct
├── <name>bus_test.go
├── testutil.go     # Test helper constructing Business with in-memory or test DB
├── stores/
│   └── <name>db/
│       ├── <name>db.go   # Implements Storer; raw SQL via sqldb helpers
│       └── model.go      # DB-only struct with sqlx tags; toDBX / toBusinessX converters
└── extensions/
    ├── <name>otel/       # OpenTelemetry span decorator
    └── <name>audit/      # Audit trail decorator (calls auditbus via delegate)
```

### 3.4 `app/domain/<name>app/` — HTTP Application Layer

**Rule: No business logic here. Translate HTTP ↔ business types only.**

```
<name>app/
├── <name>app.go    # Private `app` struct, handler methods (create, update, delete, query, queryByID)
├── route.go        # `Routes(app *web.App, cfg Config)` — registers routes + middleware
├── model.go        # App-layer DTOs with `json` tags; toApp* / toBus* converters
├── filter.go       # Parses URL query params into business QueryFilter
└── order.go        # Parses URL query params into order.By
```

Handler method signature is always:

```go
func (a *app) create(ctx context.Context, r *http.Request) web.Encoder { ... }
```

### 3.5 `app/sdk/mux/` — Route Wiring

The `BusConfig` struct in `mux/mux.go` is the single registration point for all domain buses. When adding a new domain, add it here:

```go
type BusConfig struct {
    AuditBus       auditbus.ExtBusiness
    UserBus        userbus.ExtBusiness
    CreditCardBus  creditcardbus.ExtBusiness // ← ADD
    InvestmentBus  investmentbus.ExtBusiness // ← ADD
    BudgetBus      budgetbus.ExtBusiness   // ← ADD
    // ... existing
}
```

### 3.6 `api/services/<name>/main.go` — Service Entry Point

The `run()` function in `main.go` is responsible for:
1. Parsing config via `conf` package
2. Connecting to the database
3. Constructing all stores and buses (dependency injection by hand)
4. Calling `mux.WebAPI(cfg, routeAdder, options...)` to produce the `http.Handler`
5. Running the server with graceful shutdown

---

## 4. Architecture Mapping — FinGo Domains

### 4.1 Credit Card Domain (`creditcardbus`)

**Business model (model.go):**

```go
// CreditCard represents a credit card owned by a user.
type CreditCard struct {
    ID             uuid.UUID
    UserID         uuid.UUID    // Owner
    Name           name.Name
    Limit          money.Money
    ClosingDay     int          // Day of month the statement closes
    DueDay         int          // Days after closing that payment is due
    LastFourDigits string
    Enabled        bool
    DateCreated    time.Time
    DateUpdated    time.Time
}

type Invoice struct {
    ID           uuid.UUID
    CreditCardID uuid.UUID
    ReferenceMonth time.Time   // First day of the billing month (UTC, midnight)
    TotalAmount  money.Money
    Status       InvoiceStatus // open | closed | paid
    DueDate      time.Time
    DateCreated  time.Time
    DateUpdated  time.Time
}
```

**Storer interface must include:**
- `QueryByUserID(ctx, userID, filter, orderBy, page) ([]CreditCard, error)`
- All other Query methods must accept `userID uuid.UUID` as a mandatory first filter.

### 4.2 Investment Domain (`investmentbus`)

**Business model (model.go):**

```go
type Investment struct {
    ID          uuid.UUID
    UserID      uuid.UUID
    AssetClass  assetclass.AssetClass  // stock | crypto | fixed_income
    Ticker      ticker.Ticker
    Quantity    quantity.Quantity
    AvgCost     money.Money
    Currency    currency.Currency      // BRL | USD | EUR
    DateCreated time.Time
    DateUpdated time.Time
}
```

**Notes:**
- Fixed income instruments (e.g., Brazilian Tesouro Direto) have a `MaturityDate` field.
- Mark-to-market valuation is a **read-only view** — the current price is fetched from an external source at query time and must NOT be persisted in the `Investment` row.
- Use `vproductbus` (virtual product pattern) as the reference for implementing a `vinvestmentbus` read-only view domain.

### 4.3 Budget / Cash Flow Domain (`budgetbus`)

```go
type Budget struct {
    ID          uuid.UUID
    UserID      uuid.UUID
    Name        name.Name
    TotalAmount money.Money
    Period      BudgetPeriod    // monthly | yearly
    StartDate   time.Time
    DateCreated time.Time
    DateUpdated time.Time
}

type Transaction struct {
    ID            uuid.UUID
    UserID        uuid.UUID
    BudgetID      *uuid.UUID    // nullable — not all transactions belong to a budget
    Direction     Direction     // debit | credit
    Amount        money.Money
    Description   name.Name
    Category      name.Name
    TransactionAt time.Time
    DateCreated   time.Time
    DateUpdated   time.Time
}
```

---

## 5. Real-time Dashboard Requirements

### 5.1 Approach

Use **Server-Sent Events (SSE)** as the primary real-time mechanism for the dashboard. SSE is preferred over WebSockets because:
- Dashboard data is server-push only (no client messages).
- SSE works natively over HTTP/1.1; no protocol upgrade complexity.
- WebSockets are acceptable if a future feature requires bidirectional communication (e.g., live chat support).

### 5.2 SSE Handler Pattern

SSE handlers live in a dedicated `dashboardapp` under `app/domain/dashboardapp/`. They follow the same file layout as other app packages but with a custom `Encoder` implementation.

```go
// SSE does NOT use the standard web.Encoder return pattern.
// Instead, it takes direct control of the http.ResponseWriter.
// The route must be registered via app.HandlerFuncRaw (if the framework
// exposes it) or via a standard http.HandlerFunc adaptor.

func (a *app) streamNetWorth(w http.ResponseWriter, r *http.Request) {
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "streaming not supported", http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    w.Header().Set("X-Accel-Buffering", "no") // Disable Nginx buffering

    ctx := r.Context()
    userID, err := mid.GetUserID(ctx)
    if err != nil {
        http.Error(w, "unauthorized", http.StatusUnauthorized)
        return
    }

    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            snapshot, err := a.dashBus.NetWorthSnapshot(ctx, userID)
            if err != nil {
                fmt.Fprintf(w, "event: error\ndata: %s\n\n", err.Error())
                flusher.Flush()
                return
            }
            data, _ := json.Marshal(snapshot)
            fmt.Fprintf(w, "data: %s\n\n", data)
            flusher.Flush()
        }
    }
}
```

### 5.3 Dashboard Bus

Create `business/domain/dashboardbus/` with a single read-only `Storer` interface based on purpose-built SQL views or aggregations. It must NOT mutate data.

```go
type Storer interface {
    NetWorthSnapshot(ctx context.Context, userID uuid.UUID) (NetWorthSnapshot, error)
    CashFlowSummary(ctx context.Context, userID uuid.UUID, period time.Month) (CashFlowSummary, error)
    InvestmentPortfolio(ctx context.Context, userID uuid.UUID) ([]PositionSummary, error)
}
```

The store implementation in `stores/dashboarddb/` uses PostgreSQL views and CTEs. No ORM. All queries are raw SQL via `sqldb.QuerySlice` or `sqldb.QueryRowContext`.

### 5.4 WebSocket (Future)

If bidirectional communication is needed, use the `gobwas/ws` library already vendored in the repository. Add a `ws/` sub-package to `foundation/web/` to wrap the upgrade handshake.

---

## 7. Coding Standards

### 6.0 Go 1.26+ Style Guidance

These project-wide stylistic preferences target Go 1.26 and newer. They are recommendations only — follow them unless a clear, documented reason exists to diverge.

- **Use `any` instead of `interface{}`**: prefer the built-in alias `any` for empty-interface types in new code. Example:

    ```go
    // preferred
    func DoSomething(v any) {}

    // avoid in new code
    func DoSomething(v interface{}) {}
    ```

- **Prefer `range` for counting loops and iteration**: for simple counted loops prefer ranging over a fixed-size container instead of writing `for i := 0; i < N; i++`. This avoids off-by-one errors and keeps the intent clear.

  Examples:

  ```go
  // preferred: iterate 10 times with index (i == 0..9)
  for i := range 10 {
      _ = i
  }

  // preferred when you only need to run something N times
  for range 10 {
      // do something 10 times
  }

  // avoid the manual index loop unless performance or a specific index calculation requires it
  // for i := 0; i < 10; i++ { ... }
  ```

These guidance items are ergonomics and readability preferences for the team; they do not change the runtime semantics. When a performance-sensitive hotspot requires a different loop form, document the reason in a short code comment.

### 6.1 Pointer Semantics

| Context | Rule |
|---|---|
| `UpdateEntity` fields | Always pointer types: `*name.Name`, `*money.Money`, `*bool`. A nil pointer means "not provided, do not update". |
| `QueryFilter` fields | Always pointer types. A nil pointer means "no filter on this field". |
| Entity structs | Value semantics (not pointer). Pass and return `CreditCard`, not `*CreditCard`. |
| Handler return | Return the value type directly from `toAppX(entity)`. No pointer returns from converters. |

Rationale: Ardan Labs pointer semantics — value types own their data. Only share via pointer when mutation or optionality is required.

### 6.2 Error Handling

```go
// ✅ CORRECT — Wrap with context, check sentinel errors at the app boundary
if err := s.db.Create(ctx, card); err != nil {
    if errors.Is(err, sqldb.ErrDBDuplicatedEntry) {
        return fmt.Errorf("namedexeccontext: %w", creditcardbus.ErrUniqueName)
    }
    return fmt.Errorf("namedexeccontext: %w", err)
}

// ✅ CORRECT — App layer maps business errors to HTTP/gRPC error codes
if errors.Is(err, creditcardbus.ErrNotFound) {
    return errs.New(errs.NotFound, creditcardbus.ErrNotFound)
}

// ❌ WRONG — Never swallow errors
result, _ := a.cardBus.Create(ctx, nc)

// ❌ WRONG — Never use panic for control flow
if card.UserID == uuid.Nil { panic("bad user") }
```

Every business package declares its sentinel errors at the top of `<name>bus.go`:

```go
var (
    ErrNotFound   = errors.New("credit card not found")
    ErrCardClosed = errors.New("credit card is closed")
)
```

### 6.3 Structured Logging

```go
// ✅ CORRECT — context first, then key/value pairs
log.Info(ctx, "creditcard create", "user_id", card.UserID, "card_id", card.ID)
log.Error(ctx, "creditcard create failed", "user_id", userID, "err", err)

// ❌ WRONG — no context, no structure
log.Printf("creating card for user %s", userID)
fmt.Println("error:", err)
```

Log at **Info** for normal operations (create, update, delete). Log at **Error** for unrecoverable failures. Debug logs in development only.

### 6.4 Context Discipline

- `context.Context` is always the first parameter of any function that does I/O.
- Never store a context in a struct field.
- Use `ctx.Done()` in any loop that waits on external resources (SSE, polling).
- All database calls pass `ctx` through to `sqldb` helpers. Never pass `context.Background()` inside a handler chain; always propagate the request context.

### 6.5 SQL Standards

```sql
-- ✅ CORRECT — explicit column list, named parameters
INSERT INTO credit_cards
    (credit_card_id, user_id, name, card_limit, closing_day, due_day, last_four_digits, enabled, date_created, date_updated)
VALUES
    (:credit_card_id, :user_id, :name, :card_limit, :closing_day, :due_day, :last_four_digits, :enabled, :date_created, :date_updated)

-- ✅ CORRECT — query with filters
SELECT ... FROM credit_cards WHERE user_id = :user_id AND enabled = true

-- ❌ WRONG — SELECT * is never used
SELECT * FROM credit_cards
```

- Column names use `snake_case`.
- Primary key column name is `<table_singular>_id` (e.g., `credit_card_id`).
- All timestamps are stored as `TIMESTAMPTZ` (UTC).
- Use `UUID` for all primary and foreign keys. No integer sequences.

### 6.6 Value Objects

Domain-specific primitive types live in `business/types/<type_name>/`. Do not use raw `string`, `float64`, or `int` in business models where a named type adds clarity.

New types to create:

```
business/types/
├── currency/      # type Currency string; const (BRL Currency = "BRL"; USD = "USD"; EUR = "EUR")
├── ticker/        # type Ticker string — validated stock/crypto symbol (uppercase, 1-10 chars)
├── assetclass/    # type AssetClass string; const (Stock, Crypto, FixedIncome)
└── invoicestatus/ # type InvoiceStatus string; const (Open, Closed, Paid)
```

Each type must have a `Parse(s string) (Type, error)` constructor and a `String() string` method.

### 6.7 Transactions

Use `sqldb.CommitRollbacker` and the `NewWithTx` pattern when multiple domain operations must be atomic:

```go
// In the business layer:
func (b *Business) TransferBudget(ctx context.Context, actorID uuid.UUID, ...) error {
    return sqldb.WithinTran(ctx, b.log, b.db, func(tx sqldb.CommitRollbacker) error {
        budgetStorerTx, _ := b.storer.NewWithTx(tx)
        txStorerTx, _ := b.txStorer.NewWithTx(tx)
        // Both operations are inside the same transaction
        if err := budgetStorerTx.Update(ctx, ...); err != nil { return err }
        if err := txStorerTx.Create(ctx, ...); err != nil { return err }
        return nil
    })
}
```

---

## 7. Domain Implementation Checklist

When adding a new domain (e.g., `creditcardbus`), complete ALL steps in order:

- [ ] **1. Business types** — `business/types/<new_type>/` if new value objects are needed
- [ ] **2. Migration SQL** — Add numbered SQL file to `business/sdk/migrate/sql/`
  - Name format: `000N_create_<table>.sql`
  - Include: `CREATE TABLE`, indexes, foreign key constraints
- [ ] **3. Domain model** — `business/domain/<name>bus/model.go` — pure business types, no tags
- [ ] **4. Filter** — `business/domain/<name>bus/filter.go` — `QueryFilter` with pointer fields
- [ ] **5. Order** — `business/domain/<name>bus/order.go` — `DefaultOrderBy` and column mapping
- [ ] **6. Events** — `business/domain/<name>bus/event.go` — delegate constants
- [ ] **7. Bus interface** — `business/domain/<name>bus/<name>bus.go` — `Storer`, `ExtBusiness`, `Business`, `NewBusiness`
- [ ] **8. DB store** — `business/domain/<name>bus/stores/<name>db/` — SQL implementation + DB model.go
- [ ] **9. OTel extension** — `business/domain/<name>bus/extensions/<name>otel/`
- [ ] **10. Audit extension** — `business/domain/<name>bus/extensions/<name>audit/` (if auditable)
- [ ] **11. Test util** — `business/domain/<name>bus/testutil.go`
- [ ] **12. Unit tests** — `business/domain/<name>bus/<name>bus_test.go`
- [ ] **13. App model** — `app/domain/<name>app/model.go` — DTOs + converters
- [ ] **14. App filter** — `app/domain/<name>app/filter.go` — HTTP query param parsing
- [ ] **15. App order** — `app/domain/<name>app/order.go`
- [ ] **16. App handlers** — `app/domain/<name>app/<name>app.go`
- [ ] **17. Routes** — `app/domain/<name>app/route.go` — `Routes(app *web.App, cfg Config)`
- [ ] **18. Mux wiring** — Add to `BusConfig` in `app/sdk/mux/mux.go` and wire in `api/services/sales/main.go`
- [ ] **19. Admin CLI** — Add `migrate-seed` command updates in `api/tooling/admin/`

---

## 8. Extension and Delegate Pattern

### 8.1 Extension Pattern (OTel, Audit decorators)

Extensions wrap `ExtBusiness` with additional cross-cutting concerns without modifying the core `Business` struct. They are applied in reverse order in `NewBusiness`.

```go
// business/domain/creditcardbus/extensions/creditcardotel/creditcardotel.go
package creditcardotel

type otelBusiness struct {
    bus    creditcardbus.ExtBusiness
    tracer trace.Tracer
}

func NewBusiness(bus creditcardbus.ExtBusiness, tracer trace.Tracer) creditcardbus.ExtBusiness {
    return &otelBusiness{bus: bus, tracer: tracer}
}

func (b *otelBusiness) Create(ctx context.Context, actorID uuid.UUID, nc creditcardbus.NewCreditCard) (creditcardbus.CreditCard, error) {
    ctx, span := b.tracer.Start(ctx, "business.creditcardbus.create")
    defer span.End()
    return b.bus.Create(ctx, actorID, nc)
}
// ... implement all other ExtBusiness methods
```

### 8.2 Delegate Pattern (Cross-domain events)

Use the delegate to react to domain events without creating import cycles. Example: when a `CreditCard` is created, the `auditbus` extension should log the event.

```go
// business/domain/creditcardbus/event.go
package creditcardbus

const (
    DomainName = "creditcard"

    ActionCreated = "created"
    ActionUpdated = "updated"
    ActionDeleted = "deleted"
)
```

The audit extension for creditcard registers a delegate callback that calls `auditbus.Create` on `ActionCreated`, `ActionUpdated`, `ActionDeleted`.

---

## 9. Testing Strategy

### 9.1 Unit Tests (business layer)

Use `business/sdk/unittest` and `business/sdk/dbtest` patterns:

- Business logic tests use a real PostgreSQL instance via Docker (see `dbtest`)
- Tests construct the full `Business` struct with a real `Storer`
- Use `testutil.go` in each domain to expose a `NewUnit(t, db, log) NewTestBusiness()` helper
- Test ownership: always insert records for two different users and verify that queries for user A never return records for user B

```go
func TestCreditCardQuery_UserIsolation(t *testing.T) {
    // Create cards for user A and user B
    // Query with userA filter
    // Assert: only userA cards are returned
    // Assert: userB cards are NOT in result
}
```

### 9.2 Integration Tests (app layer)

Use `app/sdk/apitest` patterns to test HTTP handlers end-to-end. Each test seeds the DB, fires an HTTP request through the full middleware stack, and asserts the response body and status code.

---

## 10. Infrastructure and Deployment

### 10.1 Docker Compose (`zarf/compose/docker_compose.yaml`)

Add a `fingo` service entry following the same pattern as the `auth` service. The `init-migrate-seed` container must run migrations for all FinGo tables before the `fingo` service starts.

### 10.2 Docker (`zarf/docker/`)

Create `dockerfile.fingo` following the same multi-stage pattern as `dockerfile.sales`. The binary is built with `CGO_ENABLED=0 GOOS=linux`.

### 10.3 Kubernetes (`zarf/k8s/`)

All Kubernetes manifests are Helm-based. Add FinGo as a new chart or a new deployment template under `zarf/helm/charts/`. Follow the `base/`→`dev/`→`prod/` overlay pattern.

### 10.4 Configuration

All runtime configuration is injected via environment variables parsed by `github.com/ardanlabs/conf/v3`. No hardcoded configuration in binary. Secrets (DB passwords, JWT keys) are mounted as Kubernetes Secrets or Docker Compose secrets — never committed to the repository.

---

## 11. Implementation Roadmap — What to Keep, What to Replace

### Keep as-is (do not modify)

| Package | Reason |
|---|---|
| `foundation/` | Framework primitives — stable, tested |
| `business/sdk/` | Core utilities — delegate, sqldb, page, order, migrate |
| `business/types/money`, `name`, `role`, `quantity` | Reusable value objects |
| `business/domain/userbus/` | User domain is required for auth; extend only |
| `business/domain/auditbus/` | Audit trail is required for all domains |
| `app/sdk/auth`, `authclient`, `errs`, `query` | Core SDK — stable |
| `api/services/auth/` | Auth microservice — do not touch |
| `api/services/metrics/` | Metrics service — do not touch |
| `api/tooling/logfmt/` | Log formatter — do not touch |

### Keep as reference templates (read, then build new)

| Package | Use as Template For |
|---|---|
| `business/domain/productbus/` | `creditcardbus`, `investmentbus`, `budgetbus` — same structural pattern with `UserID` ownership |
| `business/domain/userbus/stores/usercache/` | Any domain entity that benefits from in-memory caching |
| `business/domain/userbus/extensions/useraudit/` | All new domain audit extensions |
| `app/domain/productapp/` | `creditcardapp`, `investmentapp`, `budgetapp` |
| `api/services/sales/main.go` | Service entry point — wire new buses following the same DI pattern |

### Replace with FinGo logic

| Package | Action |
|---|---|
| `business/domain/productbus/` | After using as reference: replace `Product` logic with `CreditCard` or `Investment` logic. Rename package. |
| `business/domain/homebus/` | Replace with `dashboardbus` (read-only aggregations) |
| `business/domain/vproductbus/` | Replace with `vinvestmentbus` — virtual view domain for portfolio valuation |
| `app/domain/productapp/` | Replace with `creditcardapp` after implementing `creditcardbus` |
| `app/domain/homeapp/` | Replace with `dashboardapp` (SSE endpoints) |
| `zarf/compose/docker_compose.yaml` | Add fingo service; rename `sales-system-network` to `fingo-network` |

---

## 12. Naming Conventions Reference

| Concept | Convention | Example |
|---|---|---|
| Go module path | `github.com/garnizeh/fingo/<layer>/<pkg>` | `github.com/garnizeh/fingo/business/domain/creditcardbus` |
| Business package | `<domain>bus` | `creditcardbus`, `investmentbus` |
| App package | `<domain>app` | `creditcardapp`, `investmentapp` |
| DB store package | `<domain>db` | `creditcarddb`, `investmentdb` |
| Cache store package | `<domain>cache` | `usercache` |
| OTel extension | `<domain>otel` | `creditcardotel` |
| Audit extension | `<domain>audit` | `creditcardaudit` |
| Entity struct | PascalCase, no suffix | `CreditCard`, `Invoice`, `Investment` |
| New entity struct | `New` prefix | `NewCreditCard`, `NewInvestment` |
| Update entity struct | `Update` prefix | `UpdateCreditCard` |
| DB struct (sqlx) | `db` prefix | `dbCreditCard`, `dbInvoice` |
| Converter (bus → DB) | `toDB<Entity>` | `toDBCreditCard` |
| Converter (DB → bus) | `to<Entity>` | `toCreditCard` |
| Converter (bus → app) | `toApp<Entity>` | `toAppCreditCard` |
| Converter (app → bus) | `toBus<Entity>` | `toBusCreditCard`, `toBusNewCreditCard` |
| HTTP route prefix | `/v1/<resource-plural>` | `/v1/credit-cards`, `/v1/investments` |
| DB table name | `snake_case`, plural | `credit_cards`, `invoices`, `investments` |
| DB primary key | `<singular>_id` | `credit_card_id`, `invoice_id` |
| Migration filename | `000N_<description>.sql` | `0005_create_credit_cards.sql` |
| Env var prefix | `FINGO_` | `FINGO_DB_HOST`, `FINGO_WEB_API_HOST` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/garnizeh)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/garnizeh)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
