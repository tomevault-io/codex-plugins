# Walmart Node.js Microservice Kit - Copilot Instructions

## Architecture Overview

This is a **TypeScript-first microservice** with a clear monorepo structure:
- `backend/` - Node.js GraphQL API service  
- `k8s/` - Kubernetes deployment manifests
- `frontend/src/` - Frontend components (React-style structure)

**Key Stack**: Express + Apollo Server + TypeGraphQL + KafkaJS + Azure Cosmos DB

## Development Workflow

### Quick Start
```bash
cd backend
npm install
npm run dev  # Uses ts-node-dev for hot reload on port 4000
```

**GraphQL Playground**: http://localhost:4000/graphql

### Build & Deploy
- `npm run build` → TypeScript compilation to `dist/`
- `npm start` → Production mode from compiled JS
- K8s deployment expects image at `your-registry/order-service:latest`

## Code Patterns & conventions

### GraphQL Schema (TypeGraphQL Style)
- **Decorators required**: `@Resolver`, `@Query`, `@Mutation`, `@Field`  
- **Types**: Define in `src/graphql/types.ts` with `@ObjectType` and `@InputType`
- **Resolvers**: Place in `src/graphql/resolvers/` following `[Entity]Resolver.ts` pattern

**Example resolver pattern**:
```typescript
@Resolver(Order)
export class OrderResolver {
  @Query(() => [Order])
  async orders(@Arg("status", { nullable: true }) status?: string) { ... }
}
```

### Server Architecture
- **Entry point**: `src/server.ts` - Express app with Apollo middleware
- **Required imports**: `import "reflect-metadata"` must be first (TypeGraphQL requirement)
- **Middleware order**: CORS → Rate limiter → JSON parsing → GraphQL
- **Schema building**: Use `buildSchema({ resolvers: [...] })` pattern

### Database Integration 
- **Cosmos DB**: Uses `@azure/cosmos` package (connection setup in deployment env vars)
- **Mock data pattern**: In-memory arrays for development (see `OrderResolver.ts`)
- **Data persistence**: Replace mock arrays with Cosmos DB calls in production

### Rate Limiting
- **Location**: `src/middleware/rateLimiter.ts`
- **Implementation**: Global in-memory counter (100 requests/minute per IP)
- **Production note**: Replace with Redis-backed limiter for multiple instances

### TypeScript Configuration
- **Decorators enabled**: `experimentalDecorators` and `emitDecoratorMetadata` required for TypeGraphQL
- **Strict mode**: All strict TypeScript checks enabled
- **Build target**: ES2020 with CommonJS modules

## Integration Points

### Kafka Messaging
- **Producer location**: `src/kafka/producer.ts` (needs implementation)
- **Usage pattern**: Event-driven communication between services
- **Config**: Uses KafkaJS client library

### Kubernetes Deployment
- **Replicas**: 3 instances configured in `k8s/deployment.yaml`
- **Container port**: 4000 (matches Express server)
- **Environment**: Expects `COSMOS_ENDPOINT` for database connection
- **Image registry**: Update `your-registry/order-service:latest` to actual registry

### Frontend Integration
- **Structure**: Components in `frontend/src/components/`, custom hooks in `hooks/`
- **API consumption**: GraphQL queries against backend `/graphql` endpoint

## Key Files for Context
- `backend/src/server.ts` - Application entry point and middleware setup
- `backend/src/graphql/resolvers/OrderResolver.ts` - Example resolver with mock data pattern
- `backend/src/graphql/types.ts` - TypeGraphQL schema definitions
- `k8s/deployment.yaml` - Production deployment configuration

## Development Notes
- Hot reload via `ts-node-dev` watches all TypeScript files
- GraphQL schema auto-generates from TypeScript decorators
- In-memory mock data allows development without external dependencies
- All folders (ci/, monitoring/, scripts/) are structured but empty - ready for expansion

---
> Source: [loteknowledG/walmart-node-microservice-kit](https://github.com/loteknowledG/walmart-node-microservice-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-10 -->
