# GitHub Copilot Instructions for Swagger MCP Adapter

## Project Overview

This is a TypeScript-based MCP (Model Context Protocol) server that integrates with Swagger/OpenAPI specifications to expose API endpoints as tools for Large Language Models (LLMs). The project provides a seamless way to connect existing REST APIs to LLMs, allowing them to make requests and receive responses through standardized MCP commands.

The server reads Swagger/OpenAPI files, parses the endpoints, and exposes them as MCP tools that LLMs can invoke. This enables LLMs to interact with your APIs in a structured, validated manner.

## Architecture & Structure

Based on best practices for TypeScript/Node.js projects, this project follows a modular structure optimized for maintainability and scalability:

```
swagger-mcp-adapter/
├─ src/
│  ├─ index.ts              # MCP server entrypoint
│  ├─ server.ts             # MCP server setup (SDK integration)
│  ├─ swagger/
│  │   ├─ loader.ts         # Load & validate swagger/openapi files
│  │   ├─ parser.ts         # Parse specifications into normalized endpoints
│  │   ├─ cache.ts          # Memory state management for loaded specs
│  │   └─ types.ts          # TypeScript type definitions from OpenAPI
│  ├─ commands/
│  │   ├─ listServices.ts   # MCP command to list available services
│  │   ├─ callService.ts    # MCP command to call a specific service
│  │   └─ loadSpec.ts       # MCP command to load a new swagger spec
│  ├─ http/
│  │   ├─ client.ts         # HTTP client wrapper (axios/fetch)
│  │   └─ validator.ts      # Request/response validation with Zod
│  ├─ utils/
│  │   └─ logger.ts         # Structured logging utility
│  └─ config.ts             # Configuration management
├─ test/
│  ├─ swagger.mock.json     # Mock OpenAPI specification for testing
│  └─ server.test.ts        # Unit and integration tests
├─ package.json             # Project dependencies and scripts
├─ biome.json               # Biome configuration
├─ tsconfig.json            # TypeScript configuration
├─ README.md                # Project documentation
```

### Directory Guidelines

#### `/src` - Source Code

- Contains all TypeScript source files
- Organized by feature/domain for better maintainability
- Entry point is `index.ts` for the MCP server

#### `/src/swagger` - OpenAPI/Swagger Handling

- `loader.ts`: Handles loading OpenAPI specs from files or URLs
- `parser.ts`: Parses specifications into normalized data structures
- `cache.ts`: Memory state management for loaded specs and endpoints
- `types.ts`: Generated TypeScript interfaces from OpenAPI schemas

#### `/src/commands` - MCP Commands

- Implements MCP protocol commands as tools for LLMs
- `listServices.ts`: Returns available API endpoints with clean markdown formatting
- `getServiceInformation.ts`: Provides detailed information about a specific service including parameters, schemas, and examples
- `getAllServiceInformation.ts`: Provides comprehensive information about all services from a specification
- `getCacheInformation.ts`: Monitors cache status, expiration times, and performance metrics

#### `/src/http` - HTTP Client & Validation

- `client.ts`: Generic HTTP client for making API requests
- `validator.ts`: Schema validation using Zod based on OpenAPI specs

#### `/src/utils` - Utilities

- Shared utility functions and helpers
- Logging, error handling, and common operations

#### `/test` - Test Files

- Unit tests for individual modules
- Integration tests for MCP server functionality
- Mock data for testing without external dependencies

## TypeScript Modules & Dependencies

- Use npm for dependency management (`package.json`)
- Module should use ESM (`"type": "module"` in package.json)
- Keep dependencies minimal and regularly updated
- Use `npm audit` and `npm update` for security and updates
- Export main functionality through `package.json` exports field

## TypeScript Development Standards

### Code Style & Formatting

- Always use `biome` for code formatting and linting
- Follow TypeScript naming conventions: camelCase for variables/functions, PascalCase for classes/interfaces
- Use meaningful variable names; avoid abbreviations
- Keep functions small and focused on a single responsibility
- Prefer `const` over `let`, use `let` only when reassignment is necessary

### Error Handling

- **Always handle errors appropriately** after async operations
- Use `try/catch` blocks for synchronous errors
- Return errors as rejected Promises for async operations
- Create custom error classes for domain-specific errors
- Use structured logging for error details with context

```typescript
// Good
try {
  const result = await service.callEndpoint(params);
  return result;
} catch (error) {
  logger.error("Failed to call endpoint", { error, params });
  throw new APIError("Service call failed", { cause: error });
}
```

### Memory State Management

- **Spec Cache**: Maintain in-memory cache of loaded Swagger/OpenAPI specifications
- **Endpoint Cache**: Cache parsed endpoints for each loaded spec to avoid re-parsing
- **URL-based Loading**: Support dynamic loading of specs by URL with unique identifiers
- **Cache Invalidation**: Implement TTL-based cache invalidation for stale specs
- **Concurrent Access**: Handle concurrent requests to the same spec safely

### MCP Server Setup

- Use the official `@modelcontextprotocol/sdk` for MCP implementation
- Implement proper tool definitions with schemas
- Handle MCP protocol messages correctly
- Support graceful shutdown with signal handlers
- Validate all inputs according to MCP specifications

### HTTP Requests & Validation

- Use Axios or native fetch for HTTP requests
- Implement proper timeout and retry logic
- Validate request parameters against OpenAPI schemas using Zod
- Handle different content types (JSON, form-data, etc.)
- Return normalized responses to LLMs

### Schema Validation & Type Safety

- Use Zod for runtime validation of OpenAPI schemas
- Generate TypeScript types from OpenAPI specifications
- Validate both incoming parameters and outgoing responses
- Provide clear error messages for validation failures

### Testing

- Write unit tests for all modules using Vitest
- Use mock servers for integration testing
- Test error scenarios and edge cases
- Maintain high test coverage (>80%)
- Run tests in CI/CD pipeline

## Project-Specific Guidelines

### Configuration Management

- Use environment variables for configuration
- Provide sensible defaults for all configuration values
- Validate configuration on startup
- Support multiple environments (development, staging, production)
- Make OpenAPI source configurable (file path or URL)

### API Integration

- Support both JSON and YAML OpenAPI specifications
- Handle authentication requirements (API keys, OAuth, etc.)
- Implement rate limiting to prevent API abuse
- Cache responses when appropriate to reduce load
- Provide fallback mechanisms for API failures

### Security & Validation

- Validate all inputs to prevent injection attacks
- Implement proper CORS handling if needed
- Use HTTPS for all external API calls
- Sanitize and validate OpenAPI specifications
- Implement request/response size limits

### Logging & Monitoring

- Use structured logging with Pino for consistent log format
- Log important events: API calls, errors, performance metrics
- Include contextual information in logs (request IDs, user info)
- Monitor MCP server health and performance
- Alert on critical errors or performance degradation

## Development Workflow

1. **Project Setup**

   - Initialize Node.js + TypeScript project with proper tooling
   - Set up Biome, and Vitest for development

2. **Memory State & Caching System**

   - Implement in-memory cache for loaded Swagger specs
   - Create cache management for parsed endpoints
   - Add URL-based spec identification and loading
   - Implement cache invalidation and cleanup

3. **Swagger/OpenAPI Parser**

   - Implement loader for JSON/YAML specifications from URLs or files
   - Parse endpoints and generate TypeScript interfaces
   - Normalize service metadata for MCP exposure
   - Integrate with caching system for performance

4. **MCP Server Implementation**

   - Set up MCP server with SDK
   - Implement `loadSpec`, `listServices`, and `callService` commands
   - Handle LLM tool invocations with spec-specific context
   - Support multiple concurrent specs in memory

5. **Request/Response Handling**

   - Create generic HTTP client with error handling
   - Implement schema validation with Zod
   - Normalize responses for LLM consumption
   - Handle authentication and rate limiting per spec

   - Create generic HTTP client with error handling
   - Implement schema validation with Zod
   - Normalize responses for LLM consumption

6. **Best Practices Implementation**

   - Add configurable OpenAPI source support
   - Implement comprehensive error handling
   - Add structured logging throughout
   - Ensure strong typing across the codebase

7. **Testing Strategy**

   - Write unit tests for all core functionality
   - Create contract tests with mock APIs
   - Implement end-to-end tests with mock OpenAPI specs

8. **Release Preparation**
   - Configure package.json for ESM and exports
   - Set up GitHub Actions for automated publishing
   - Create comprehensive documentation

## Technology Stack

- **Core:** TypeScript 5.6+, Node.js (ESM)
- **MCP SDK:** @modelcontextprotocol/sdk 1.17.4
- **OpenAPI/Swagger:** swagger-parser 10.0.4, openapi-typescript 7.0+
- **Validation:** zod 3.23+
- **HTTP Client:** axios 1.7+
- **Testing:** vitest 2.0+
- **Logging:** pino 9.0+
- **Build:** tsup 8.5+, Biome 2.2.2
- **Dev Tools:** tsx 4.19+
- **Production:** Optimized ESM builds with external dependency handling

## Development Plan

1. ✅ **Project Setup** - Initialize Node.js + TypeScript project with proper tooling
2. ✅ **Memory State & Caching System** - Implement in-memory cache for loaded Swagger specs
3. ✅ **Swagger/OpenAPI Parser** - Implement loader for JSON/YAML specifications from URLs or files
4. ✅ **MCP Server Implementation** - Set up MCP server with SDK and multiple tools
5. ✅ **Request/Response Handling** - Create generic HTTP client with error handling
6. ✅ **Best Practices Implementation** - Add comprehensive error handling and structured logging
7. ✅ **Testing Strategy** - Write unit tests for all core functionality
8. ✅ **Production Build** - Configure tsup for optimized production builds
9. ✅ **Claude Desktop Integration** - Enable seamless integration with Claude Desktop
10. 🔄 **Release Preparation** - Configure package.json for ESM and exports

This plan allows for incremental development with clear milestones. Each step builds upon the previous, ensuring a solid foundation for the MCP server with proper memory state management.

## Improved Flow Architecture

### Current Flow Issues (Before)

- **No Memory State**: Each command reloads and reparses the entire Swagger spec
- **Static Configuration**: Only supports one hardcoded spec source
- **Inefficient Caching**: No reuse of parsed endpoints across requests
- **Limited Flexibility**: Cannot dynamically load different specs

### Improved Flow (After)

- **Memory State Management**: In-memory cache for loaded specs and parsed endpoints
- **Dynamic Spec Loading**: Support for loading specs by URL with unique identifiers
- **Efficient Caching**: Reuse parsed endpoints to avoid redundant processing
- **Multi-Spec Support**: Handle multiple concurrent specs in memory

### Flow Sequence

1. **List Services Command**

   ```
   User/LLM → list_services(swaggerUrl) → Load/Cache spec → Return formatted service list
   ```

2. **Get Service Information Command**

   ```
   User/LLM → get_service_information(swaggerUrl, serviceId) → Retrieve from cache → Return detailed service info
   ```

3. **Get All Service Information Command**

   ```
   User/LLM → get_all_service_information(swaggerUrl) → Retrieve from cache → Return comprehensive service details
   ```

4. **Get Cache Information Command**
   ```
   User/LLM → get_cache_information() → Analyze cache state → Return performance metrics
   ```

### Memory State Structure

```typescript
interface SpecCache {
  [specId: string]: {
    spec: SwaggerSpec;
    endpoints: ParsedEndpoint[];
    loadedAt: Date;
    ttl: number;
  };
}
```

### Benefits

- **Performance**: Intelligent caching avoids reloading/parsing the same spec multiple times
- **Flexibility**: Support multiple specs simultaneously with URL-based identification
- **User Experience**: Clean markdown formatting and comprehensive service information
- **Monitoring**: Built-in cache monitoring and performance metrics
- **Reliability**: TTL-based cache invalidation prevents stale data issues
- **Production Ready**: Optimized builds with proper external dependency handling
- **Claude Desktop Integration**: Seamless integration with Claude Desktop for production use

---
> Source: [serifcolakel/swagger-mcp-adapter](https://github.com/serifcolakel/swagger-mcp-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-08 -->
