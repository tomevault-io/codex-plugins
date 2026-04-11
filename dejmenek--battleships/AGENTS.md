## Project Overview

Battleships is a web-based, real-time multiplayer implementation of the classic Battleship game.  
This repository represents the MVP version of the project, focused on correct gameplay, real-time synchronization, and a clean backend architecture.

The backend is authoritative over all game state and rules. Real-time communication is handled via SignalR.

These instructions guide AI-assisted code generation to stay aligned with the project’s architecture, scope, and technical decisions.

---

## MVP Scope & Boundaries

- Implement classic Battleship rules only (10×10 board, standard ship set).
- Support 2-player games only.
- Focus on correctness, rule enforcement, and synchronization.
- Avoid feature expansion beyond the defined MVP.
- Do not introduce scalability optimizations beyond what is required for correctness.
- Frontend behavior should not be assumed or hardcoded into backend logic.

---

## Tech Stack (Authoritative)

- **Backend:** ASP.NET (Minimal APIs)
- **Real-time:** SignalR
- **Persistence:** Entity Framework Core + SQL Server
- **Validation:** FluentValidation
- **Architecture:** Clean Architecture + CQRS (without MediatR)
- **Frontend:** Razor Pages (context only)
- **Containerization:** Docker
- **Hosting:** Azure

Do not introduce alternative frameworks, libraries, or architectural patterns unless explicitly added to the project.

---

## Solution & Project Structure

```text
docs/
  adr/
  diagram/

src/
  Battleships.API
  Battleships.Web
  Battleships.Application
  Battleships.Infrastructure
  Battleships.Shared
  Battleships.Domain

tests/
  Battleships.UnitTests
  Battleships.IntegrationTests
  Battleships.ArchitectureTests
  Battleships.EndTests
```

### Notes
- Internal folder structures inside each project are intentionally undefined.
- Follow existing patterns once they emerge.
- Do not introduce new structural conventions prematurely.

___

## Clean Architecture Guidelines
### Domain
- Contains core game rules, entities, and invariants.

- as no dependencies on frameworks or infrastructure.
- Must remain framework-agnostic.

### Application
- Contains use cases, commands, queries, and orchestration logic.
- Implements CQRS explicitly (no MediatR or mediator-style abstractions).
- Depends only on Domain and shared abstractions.

### Infrastructure
- Contains persistence, database access, external services, and EF Core implementations.
- Implements interfaces defined in Application or Domain.

### API
- Uses ASP.NET Minimal APIs.
- Maps HTTP endpoints and SignalR hubs.
- Contains no business or game logic beyond request mapping and validation.

### Dependency Rule
- Dependencies flow inward only.
- Domain must not depend on any other project.

___

## CQRS Guidance (Without MediatR)
- Commands and queries must be clearly separated by intent.
- No mediator libraries or hidden dispatch mechanisms should be used.
- Command and query handling should be explicit and easy to trace.
- Shared models between commands and queries should be minimized.

___

## Game Logic & Real-Time Rules
- The server is authoritative over all game state.
- All moves, turns, ship placements, and timers must be validated server-side.
- Clients must never be trusted for rule enforcement.
- SignalR hubs are treated as API surfaces and must apply the same validation rules as HTTP endpoints.
- Game logic must not live in API endpoints or SignalR hubs.
- Invalid actions (out-of-turn moves, repeated shots, invalid placements) must be rejected consistently.

___

## Validation
- FluentValidation is the standard validation mechanism.
- Validate all incoming commands, queries, and real-time messages.
- Validation must occur before application logic is executed.
- Domain invariants should be enforced in the Domain layer where applicable.

___

## Testing Strategy
### Battleships.UnitTests
- XUnit-based unit tests.
- Focus on Domain and Application logic.
- No infrastructure or database dependencies.
- Isolation: Each test should be independent and not rely on the outcome of other tests.
- Test Structure: Follow Arrange-Act-Assert (AAA) pattern in all tests.
- Test Naming: Use descriptive method names: MethodName_StateUnderTest_ExpectedBehavior (e.g., GetUserById_ReturnsUser_WhenUserExists).

### Battleships.IntegrationTests
- Use TestContainers for SQL Server-backed tests.
- Validate persistence, EF Core mappings, and application workflows.
- Isolation, AAA, and Test Naming rules apply here as well.

### Battleships.ArchitectureTests
- Enforce Clean Architecture dependency rules.
- Prevent forbidden project references.

### Battleships.EndTests
- End-to-end tests using Playwright.
- Validate critical user flows only.
- Testing should prioritize correctness of game rules and state transitions over coverage metrics.

___

## Coding Standards
### General Principles
- Prefer clarity and explicitness over cleverness.
- Optimize for readability and maintainability.
- Keep methods small and focused on a single responsibility.
- Avoid premature abstractions and generic solutions.
- Fail fast on invalid input and illegal state transitions.
- Favor immutable data where practical.
- Use async/await consistently for I/O-bound operations.
- Treat all external input as untrusted.

### Naming Conventions
- Use PascalCase for:
  - Classes
  - Records
  - Public methods
  - Commands and queries
- Use camelCase for:
  - Method parameters
  - Local variables
  - Private fields (without underscores)
- Use clear, intention-revealing names:
  - Commands should describe actions (e.g. FireShotCommand)
  - Queries should describe data retrieval (e.g. GetActiveGameQuery)
  - Avoid generic names such as Handler, Manager, or Processor
- Boolean names should read naturally:
  - Prefer IsReady, HasEnded, CanFire
- Avoid abbreviations unless they are widely accepted and unambiguous.
- Avoid magic numbers; use named constants or enums where applicable.

## Code Generation Guidelines
- Follow existing naming and structural patterns.
- Keep logic out of API endpoints and SignalR hubs.
- Avoid speculative or future-oriented abstractions.

## What to Avoid
- Introducing MediatR or similar mediator frameworks.
- Overengineering for scalability or extensibility.
- Embedding business logic in API endpoints or SignalR hubs.
- Guessing frontend behavior or UI constraints.
- Inventing internal folder structures without precedent.

## General Guidance
- Favor simplicity.
- Follow existing patterns.
- Respect architectural boundaries.
- Optimize for correctness and maintainability during the MVP phase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dejmenek)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Dejmenek)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
