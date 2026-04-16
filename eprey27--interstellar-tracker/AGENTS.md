# Interstellar Tracker - Copilot Instructions

## Project Overview
3D visualization system for tracking interstellar object 2I/Borisov passage through our solar system.

## Architecture
- Clean Architecture with microservices
- .NET 8.0 (or latest LTS)
- OpenGL rendering via Silk.NET
- Keycloak authentication
- Docker + Kubernetes deployment

## Coding Standards
- Follow C# conventions and .NET best practices
- Use async/await for I/O operations
- Implement comprehensive XML documentation
- Write unit tests for all business logic (xUnit)
- Follow SOLID principles
- Use dependency injection

## Project Structure
- `src/Domain/` - Core business models and interfaces
- `src/Application/` - Use cases and application logic
- `src/Infrastructure/` - External concerns (DB, APIs)
- `src/Services/` - Microservices (API Gateway, Auth, Calculation, Visualization)
- `src/Web/` - Frontend with Silk.NET
- `tests/` - Unit and integration tests
- `docker/` - Dockerfiles and compose files
- `k8s/` - Kubernetes manifests
- `.github/workflows/` - CI/CD pipelines

## Documentation Requirements
- Every public class/method needs XML comments
- Complex algorithms need inline explanations
- Architecture decisions go in `docs/adr/`
- Target audience: junior developers

## Quality Gates
- Code coverage > 80%
- No critical security issues
- All tests passing
- No code smells (maintainability rating A)

## Progress Tracking
- [x] Create copilot-instructions.md
- [x] Scaffold .NET solution structure
- [x] Setup Docker Compose
- [x] Implement domain models
- [x] Add unit testing infrastructure
- [x] Fix hyperbolic orbit calculations
- [ ] Create microservices
- [ ] Build 3D visualization
- [ ] Configure CI/CD
- [ ] Setup Kubernetes
- [ ] Document everything

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Eprey27) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
