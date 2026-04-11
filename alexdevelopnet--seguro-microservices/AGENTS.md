# Copilot Instructions for seguro-microservices

## Big Picture Architecture
- This repository implements a microservices insurance platform using C#/.NET 8, PostgreSQL, and Hexagonal (Ports & Adapters) architecture.
- There are two main services:
  - **PropostaService**: Handles insurance proposal creation, listing, status changes, and exposes a REST API.
  - **ContratacaoService**: Handles contracting of approved proposals, persists contracts, and communicates with PropostaService via HTTP REST.
- Each service is split into four layers: API (controllers, DTOs), Application (services, ports), Domain (entities, repositories), Infrastructure (adapters, persistence).
- Data flows from API → Application → Domain → Infrastructure, with dependencies only pointing inward (classic Clean Architecture).

## Developer Workflows
- **Build**: Use `dotnet build` from the solution or service directory.
- **Run**: Use `dotnet run --project <Service>.API/<Service>.API.csproj` for each service.
- **Database**: PostgreSQL is required. Connection strings are in each service's `appsettings.json`.
- **Migrations**: Use `dotnet ef migrations add <Name> --project <Service>.Infrastructure/<Service>.Infrastructure.csproj --startup-project <Service>.API/<Service>.API.csproj --context <DbContext>` and `dotnet ef database update ...`.
- **Tests**: Run with `dotnet test` in the `PropostaService.Tests` project. Fakes are used for repositories in unit tests.

## Project Conventions & Patterns
- **DTOs/Requests**: API request/response records are in `API/DTOs/`, not inside controllers.
- **Repositories**: Domain interfaces in `Domain/Repositories/`, implementations in `Infrastructure/Repositories/`.
- **DbContext**: Each service has its own DbContext in `Infrastructure/Persistence/`.
- **Adapters**: HTTP and persistence adapters are in `Infrastructure/Adapters` and `Infrastructure/Repositories`.
- **Dependency Injection**: All services, repositories, and adapters are registered in each service's `Program.cs`.
- **Async**: All data access and service methods are async/await.
- **Status transitions**: Proposal status is changed via a PATCH endpoint and domain methods (`Aprovar`, `Rejeitar`).

## Integration & Communication
- **Inter-service**: ContratacaoService calls PropostaService via HTTP using an adapter that implements an Application port interface.
- **Database**: Each service uses its own PostgreSQL database, configured in `appsettings.json`.

## Key Files & Directories
- `src/PropostaService.API/Controllers/PropostasController.cs`: Main API for proposals.
- `src/ContratacaoService.API/Controllers/ContratacoesController.cs`: Main API for contracts.
- `src/PropostaService.Application/PropostaService.cs`: Application service for proposals.
- `src/ContratacaoService.Application/Services/ContratacaoAppService.cs`: Application service for contracts.
- `src/PropostaService.Domain/Entities/Proposta.cs`: Proposal entity and business logic.
- `src/ContratacaoService.Domain/Entities/Contratacao.cs`: Contract entity.
- `src/PropostaService.Infrastructure/Repositories/PropostaRepository.cs`: Proposal repository implementation.
- `src/ContratacaoService.Infrastructure/Repositories/ContratacaoRepository.cs`: Contract repository implementation.
- `src/PropostaService.API/DTOs/`: API request/response records.

## Examples
- To add a new endpoint, create a DTO in `API/DTOs/`, add a method to the controller, and implement the logic in the Application service.
- To add a new integration, define a port in Application, implement it in Infrastructure, and register in DI.

---

If any section is unclear or missing project-specific details, please provide feedback for improvement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexdevelopnet)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alexdevelopnet)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
