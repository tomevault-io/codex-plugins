# Laravel Clean Architecture Guidelines

These instructions enforce strict architectural, testing, and operational standards for the project.

## 0. Agent Interaction
* **Direct Execution:** Execute tasks immediately without conversational filler or opening explanations.
* **Proactive Clarification:** If requirements are ambiguous or a superior architectural approach exists, pause execution and present clear questions or alternatives.

## 1. Architectural Patterns
* **Framework Cleanliness:** Decouple business logic from the framework. No fat controllers or models.
* **Action Pattern:** Isolate critical operations into standalone classes with a single `execute()` or `__invoke()` method (e.g., `CreateShipmentAction`).
* **Service Pattern & DTOs:** Encapsulate incoming request payloads into immutable `readonly class` DTOs (type safety). Services strictly manage third-party APIs.
* **Database Transactions:** Strictly wrap multi-table data mutations inside `DB::transaction()` to ensure atomic rollbacks on failure.

## 2. Testing Strategy (TDD)
* **Unit Tests:** Isolate pure logic/math completely off the database and external APIs.
* **Feature/Integration Tests:** Test end-to-end execution flow using database endpoints (use `RefreshDatabase`).
* **UI/Contract Testing:** No frontend testing. Test API Contracts rigorously—JSON structures, data types, and keys must exactly match expected payloads. Use `brianium/paratest` for parallel execution.
* **Stress/Performance Testing:** Use K6 or JMeter to test endurance (e.g., 1000 requests/sec mapping).

## 3. GitFlow Strategy
* **`main`:** 100% stable, production-ready live branch.
* **`develop`:** Integration branch.
* **Feature Branches:** Branch from `develop` (`feature/issue-[number]-[description]`).
* **Pull Requests (PRs):** PR to `develop` with high-caliber technical descriptions (problem, solution, testing strategy). No direct pushes to main/develop.

## 5. Logistics Fundamentals
* **Idempotency Key:** API endpoints require a unique `X-Idempotency-Key` header, validated against Redis within a 60-second window to prevent duplicate executions.
* **Seeders & Factories:** Build robust factories upfront to seed 10,000+ realistic records for query benchmarking.
* **Logging & Auditing:** Use isolated log channels (info/error). Integrate APM tracking (Sentry) for real-time error traces/alerts.

## 6. Readme File
* **Documentation:** Update the README with clear setup instructions, architectural decisions, and usage examples after every significant feature addition or change.

---
> Source: [yousefbzaqout/logistics-core-saas](https://github.com/yousefbzaqout/logistics-core-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-14 -->
