# Copilot Instructions — Distributed Order Platform

## Project Overview

This is a distributed order processing system built with **Akka.NET**, **RabbitMQ**, and **Kubernetes (k3d)**. It uses the actor model for concurrency and a saga-based orchestration pattern for order lifecycle management. Refer to `docs/architecture.md` for the full design.

## Solution Structure

- **`src/Shared/Contracts/`** — Shared DTOs, events, commands. No logic. Referenced by all services.
- **`src/Shared/Messaging/`** — RabbitMQ abstractions (connection factory, publisher, consumer base). No Akka dependency.
- **`src/Shared/AkkaCore/`** — Shared Akka.NET base actors, configuration options, HOCON builder.
- **`src/ApiGateway/`** — ASP.NET Core Minimal API. Stateless HTTP ingress. NOT part of the Akka cluster.
- **`src/OrderService/`** — Order lifecycle + saga orchestrator. Akka cluster seed node.
- **`src/PaymentService/`** — Payment validation consumer. Akka cluster member.
- **`src/InventoryService/`** — Inventory reservation consumer. Akka cluster member.
- **`src/ShippingService/`** — Shipping dispatch consumer. Akka cluster member.

## Coding Standards

### Language & Framework
- **.NET 9**, C# 13, nullable reference types enabled (`<Nullable>enable</Nullable>`).
- Target `net9.0`. Use `<ImplicitUsings>enable</ImplicitUsings>`.
- ASP.NET Core Minimal APIs (no controllers) for HTTP endpoints.

### SOLID Principles
- **Single Responsibility**: Each actor handles exactly one concern. Each service owns one bounded context.
- **Open/Closed**: Use message-based polymorphism (pattern matching on message types) rather than inheritance hierarchies.
- **Liskov Substitution**: Shared abstractions (e.g., `IRabbitMqPublisher`, `IMessageHandler<T>`) must be substitutable.
- **Interface Segregation**: Keep interfaces small and focused. `IRabbitMqPublisher` publishes. `IRabbitMqConsumer` consumes. Don't merge them.
- **Dependency Inversion**: Services depend on abstractions from `Messaging-private` and `AkkaCore-private`, never on concrete implementations of other services.

### Naming Conventions
- **Namespaces**: `DistributedOrderPlatform.{Layer}.{Feature}` (e.g., `DistributedOrderPlatform.Contracts.Events`).
- **Actors**: Suffix with `Actor` (e.g., `OrderSagaActor`, `PaymentProcessorActor`).
- **Messages** (commands/events): Use past tense for events (`OrderReceived`, `PaymentValidated`), imperative for commands (`RequestPayment`).
- **Options classes**: Suffix with `Options` (e.g., `RabbitMqOptions`).
- **Files**: One type per file. File name matches type name.

### Actor Patterns
- Actors are the unit of concurrency. NEVER use `async/await` inside `Receive<T>` handlers — use `ReceiveAsync<T>` with `PipeTo` pattern.
- Keep actor state private and immutable where possible. Use `Become()` for state transitions.
- Always define supervision strategies explicitly. Default: `OneForOneStrategy` with bounded retries.
- Use `ReceiveTimeout` for saga timeouts. Never use `Thread.Sleep` or `Task.Delay` to block actors.
- Log actor lifecycle events (PreStart, PostStop, PreRestart) at Debug level.

### Messaging Patterns
- All messages inherit from a base `IMessage` interface carrying `MessageId`, `CorrelationId`, `OrderId`, `Timestamp`.
- Use `record` types for messages (immutable by default).
- Serialize with `System.Text.Json`. Register converters for custom types.
- Always set `x-retry-count` header. Increment on retry, route to DLQ on exhaustion.
- Use `BasicQos(prefetchCount: 10)`. Manual ACK only after successful processing.

### Configuration
- Use `IOptions<T>` pattern. Bind from `appsettings.json` sections.
- Environment variables override JSON config (K8s ConfigMap/Secret injection).
- HOCON for Akka.NET cluster config, built programmatically from `AkkaClusterOptions`.
- Never hardcode connection strings, ports, or credentials.

### Error Handling
- Actors: Let exceptions propagate to the supervisor. Don't catch-all inside actors.
- RabbitMQ consumers: Catch processing exceptions, NACK with `requeue: false`, increment retry header.
- HTTP endpoints: Return ProblemDetails (RFC 7807) for errors.
- Log exceptions with structured logging (Serilog). Include `OrderId` and `CorrelationId` in log context.

### Testing
- Unit test actors using `Akka.TestKit.Xunit2`.
- Unit test message handlers in isolation (mock `IRabbitMqPublisher`).
- Integration tests use `Testcontainers` for RabbitMQ.
- Name test methods: `MethodName_Scenario_ExpectedResult`.

### Docker & Kubernetes
- Multi-stage Dockerfiles: `sdk` image for build, `aspnet` image for runtime.
- One Dockerfile per service in `infra/docker/`.
- K8s manifests in `infra/k8s/`. Use `order-platform` namespace.
- All pods expose `/health/live` and `/health/ready` endpoints.
- Resource requests/limits on every container.

### Code Style
- Use `file-scoped namespaces`.
- Prefer `var` when the type is obvious from the right side.
- Use `primary constructors` for DI in services/actors.
- Records for immutable data. Classes for mutable state (actors).
- Prefer pattern matching (`switch` expressions, `is` patterns) over `if/else` chains.
- Keep methods under 30 lines. Extract complex logic into private methods or separate classes.
- XML doc comments on all public types and members.

### Dependencies (NuGet)
- `Akka` + `Akka.Cluster` + `Akka.Cluster.Tools` — Actor system and clustering.
- `RabbitMQ.Client` (7.x) — AMQP client.
- `Serilog.AspNetCore` — Structured logging.
- `prometheus-net.AspNetCore` — Metrics endpoint.
- `Microsoft.Extensions.Hosting` — Generic host for background services.
- Do NOT use MassTransit, NServiceBus, or other heavyweight frameworks. Raw `RabbitMQ.Client` only.

### Git Conventions
- Commit messages: `type(scope): description` (e.g., `feat(order-service): add saga timeout handling`).
- Branch naming: `feature/`, `fix/`, `infra/`.
- All folders inside `distributed-order-platform` must have the `-private` suffix.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/josejalvarezm)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/josejalvarezm)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
