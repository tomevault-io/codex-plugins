# Copilot Instructions for AZ-204 Spring Learning Project

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is an AZ-204 (Azure Developer) certification study project built with Spring Boot. The project demonstrates Azure services integration with both real Azure services (free tier) and mock implementations for learning purposes.

## Architecture Principles
- **Multi-profile support**: `local` (mocks), `azure` (real services), `hybrid` (mixed)
- **Service abstraction**: Interface-based design with real and mock implementations
- **Cost-conscious**: Focus on Azure free tier services
- **Learning-oriented**: Comprehensive examples and documentation

## Key Technologies
- **Spring Boot 3.2.0** with Java 17
- **Azure SDK for Java** (Spring Cloud Azure 5.8.0)
- **H2 Database** for local development
- **Spring Data JPA** for data persistence
- **Spring Actuator** for monitoring

## Azure Services Implemented
- **Azure Storage (Blob)** - File upload/download/management
- **Azure Key Vault** - Secret management
- **Azure Service Bus** - Message queuing (planned)
- **Azure Application Insights** - Monitoring (planned)

## Code Guidelines
1. **Service Layer**: All Azure services must have both real and mock implementations
2. **Configuration**: Use conditional beans based on profiles and properties
3. **Error Handling**: Provide clear error messages for both Azure and mock scenarios
4. **Documentation**: Include Portuguese comments for learning purposes
5. **Testing**: Prefer integration tests with the `local` profile

## Profile-Specific Behavior
- **local**: All services use mock implementations, no Azure credentials needed
- **azure**: Real Azure services, requires proper environment variables
- **hybrid**: Mix of real (free) and mock (paid) services

## Environment Variables (for Azure profile)
```
AZURE_KEYVAULT_ENDPOINT=https://your-keyvault.vault.azure.net/
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
AZURE_SERVICEBUS_CONNECTION_STRING=Endpoint=sb://...
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...
```

## When Adding New Features
1. Create service interface first
2. Implement both real and mock versions
3. Add conditional configuration
4. Create controller endpoints
5. Add to simulator for demonstration
6. Update README with new endpoints

## Common Patterns
- Use `@ConditionalOnProperty` for service selection
- Implement `CommandLineRunner` for demonstrations
- Use `ResponseEntity<Map<String, Object>>` for API responses
- Include proper error handling and logging

## Learning Focus Areas
- Azure Storage Blob operations
- Key Vault secret management
- Service Bus message handling
- Application Insights integration
- Azure Functions (planned)
- Cosmos DB integration (planned)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AllanDi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
