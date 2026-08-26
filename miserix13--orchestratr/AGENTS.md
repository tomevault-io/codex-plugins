# Copilot Instructions for OrchestratR

## Project Overview
- **OrchestratR** is a distributed job orchestration framework for .NET, designed for high reliability, fault tolerance, and dynamic job reallocation across servers.
- The system is split into two main components:
  - `OrchestratR.Server`: Runs jobs on distributed servers, manages job execution and cancellation.
  - `OrchestratR.ServerManager`: Manages job lifecycle, tracks system/job status, and handles persistence.

## Architecture & Data Flow
- **Job Distribution:** Jobs are sent as messages (RabbitMQ) to a shared queue. Servers fetch jobs (no ACK), enabling instant reallocation if a server fails.
- **Persistence:** Supports SQL Server, PostgreSQL, MySQL, and InMemory (for dev/testing only).
- **Low Coupling:** Server and Manager are decoupled; server failures do not impact job management.

## Key Patterns & Conventions
- **Job Implementation:**
  - Jobs are long-running/infinite tasks, implemented as async methods with cancellation and heartbeat support.
  - Example: See `YourInfiniteJob` pattern in `README.md`.
- **Dependency Injection:**
  - Services are registered via extension methods (e.g., `AddOrchestratedServer`, `AddOrchestratedServerManager`).
- **Transport:**
  - Only RabbitMQ is supported for message brokering (see `OrchestratR.Extension.RabbitMq`).
- **Persistence Extensions:**
  - Each DB backend has its own extension project (e.g., `OrchestratR.ServerManager.Persistence.MsSql`).

## Developer Workflows
- **Build:**
  - Use the solution file: `OrchestratR.slnx`.
  - Standard .NET build commands apply (`dotnet build`).
- **Run/Debug:**
  - Use `docker-compose-simple-example.yml` or `docker-compose-infrastructure.yml` for local orchestration.
  - Each service has its own Dockerfile (see root for `DockerFile-Server`, `DockerFile-ServerManager`).
- **Testing:**
  - No explicit test project found; follow .NET conventions if adding tests.

## Integration Points
- **RabbitMQ:** Required for all orchestration messaging.
- **Database:** Choose and configure one supported DB for persistence.
- **API/Clients:** Use `IOrchestratorClient` and `IOrchestratorMonitor` for job management and monitoring (see `OrchestratR.ServerManager`).

## Notable Files & Directories
- `src/OrchestratR.Server/` — Server logic, job execution, server extensions
- `src/OrchestratR.ServerManager/` — Manager logic, API, monitoring, and job control
- `src/OrchestratR.Extension.RabbitMq/` — RabbitMQ transport integration
- `src/OrchestratR.ServerManager.Persistence.*` — Persistence backends
- `README.md` — Architecture, usage, and code examples

## Project-Specific Advice
- Always handle job cancellation and heartbeat in job implementations.
- Use unique job names for cluster-wide uniqueness.
- For high-availability, ensure RabbitMQ is clustered and persistent storage is reliable.
- When extending, follow the pattern of separate extension projects for new transports or persistence backends.

---
For more details, see the `README.md` and the `assets/images/` diagrams.

---
> Source: [miserix13/OrchestratR](https://github.com/miserix13/OrchestratR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
