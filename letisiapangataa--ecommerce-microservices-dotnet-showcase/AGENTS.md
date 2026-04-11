# Copilot Instructions for E-commerce Microservices Application

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a microservices-based e-commerce application built with .NET 9, designed to showcase Azure Container Orchestration capabilities for AZ-204 certification.

## Architecture Guidelines
- **Service-based Architecture**: Each microservice should be independent and loosely coupled
- **Domain-Driven Design**: Services should be organized around business domains (User, Product, Order)
- **API-First**: All services expose RESTful APIs
- **Container-Ready**: All services should be containerized with Docker
- **Azure-Native**: Designed for deployment on Azure Container Instances/AKS

## Technology Stack
- **.NET 9**: Primary framework for all microservices
- **ASP.NET Core Web API**: For REST API development
- **Entity Framework Core**: For data access
- **Azure Service Bus**: For inter-service messaging
- **Application Insights**: For monitoring and telemetry
- **Docker**: For containerization
- **Azure Container Registry**: For image storage
- **Azure Kubernetes Service (AKS)**: For orchestration

## Service Responsibilities
- **User Service**: User authentication, profile management, user data
- **Product Service**: Product catalog, inventory management, product search
- **Order Service**: Order processing, order history, payment coordination
- **API Gateway**: Request routing, authentication, rate limiting, API composition

## Coding Standards
- Use **Controllers** for API endpoints
- Implement **Repository Pattern** for data access
- Use **DTOs** for API contracts (defined in Shared.Models)
- Follow **RESTful conventions** for API design
- Implement proper **error handling** and **logging**
- Use **async/await** for I/O operations
- Include **health checks** for each service

## Azure Integration Patterns
- Use **Azure Service Bus** for asynchronous communication between services
- Implement **Application Insights** for distributed tracing
- Use **Azure Key Vault** for secrets management
- Implement **Azure Monitor** for health monitoring
- Use **Azure Container Registry** for container images

## Container Guidelines
- Create optimized **multi-stage Dockerfiles**
- Use **.NET runtime images** for production
- Implement **health checks** in containers
- Use **non-root users** for security
- Optimize for **small image sizes**

## Security Considerations
- Implement **JWT authentication**
- Use **HTTPS** for all communications
- Validate all **input data**
- Implement **CORS** properly
- Use **Azure Active Directory** integration where applicable

## Testing Strategy
- Unit tests for business logic
- Integration tests for APIs
- Container tests for deployments
- Load tests for performance validation

When generating code, prioritize cloud-native patterns, scalability, maintainability, and Azure best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/letisiapangataa)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/letisiapangataa)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
