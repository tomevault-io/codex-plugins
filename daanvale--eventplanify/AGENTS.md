# EventPlanify - AI Coding Instructions

## Architecture Overview

**EventPlanify** is an Angular 18 event planning application with a **microservices backend architecture**. The frontend consumes three main microservices:

- **MS-Reserves** (`localhost:8081`) - Event reservations, clients, services, terraces
- **MS-Templates** (`localhost:8082`) - Event templates, cities, service types  
- **Auth Service** (`localhost:8080`) - Authentication and user management

## Key Structural Patterns

### Microservice Organization
- **Models**: `src/app/models/ms_reserve/` and `src/app/models/ms_template/` mirror backend entities
- **Services**: `src/app/shared/ms_reserve/` and `src/app/shared/ms_template/` contain HTTP services
- **Environment URLs**: Configured in `src/app/enviroments/enviroment.local.ts`

```typescript
// Service pattern example from clientService.service.ts
private API = environment.msReservesUrl + '/client';
// All services follow: {microservice-url}/{entity} URL structure
```

### Fallback Data Strategy
**Critical Pattern**: All services implement offline fallback using test data from `src/app/assets/`:

```typescript
catchError(() => {
  return this.loadLocalClientsWithFallback().pipe(
    tap((data) => { 
      console.warn('✅ Cargando clients locales como fallback')
      this.fallbackClient = data.filter(a => a.killed == 0);
    })
  );
})
```

### Component Architecture
- **Lazy Loading**: All routes use `loadComponent()` for code splitting
- **Standalone Components**: No NgModules, all components are standalone
- **PrimeNG UI**: Complete UI library integration via `src/app/imports.ts`
- **Custom Theming**: PrimeNG theme configured in `src/app/enviroments/enviroment.style.ts`

## Authentication & Security

### JWT Interceptor Pattern
`src/app/auth/jwt-interceptor.ts` automatically adds Bearer tokens to backend calls:
- Only applies to `localhost:8080` (main backend)
- Excludes auth endpoints
- Token stored in `sessionStorage`

### Service Structure
```typescript
// Standard service pattern
@Injectable({ providedIn: 'root' })
export class {Entity}Service {
  private API = environment.{microservice}Url + '/{endpoint}';
  {entities}: {Entity}Model[] = [];
  fallback{Entity}: {Entity}Model[] = [];
}
```

## Development Workflow

### Available Commands
- `npm start` - Development server on `localhost:4200`
- `npm test` - Karma unit tests
- `ng build` - Production build
- `ng serve` - Development with auto-reload

### Backend Dependencies
**Required**: All three microservices must be running locally:
1. Auth Service: `localhost:8080`
2. MS-Reserves: `localhost:8081` 
3. MS-Templates: `localhost:8082`

### Testing Strategy
- **Offline Development**: Extensive test data in `src/app/assets/` allows frontend development without backend
- **Data Files**: `reserve-test-data.ts`, `template-test-data.ts` contain realistic sample data
- **Database Pipelines**: JSON files in `assets/database json pipelines/` for data seeding

## Code Conventions

### File Naming
- **Services**: `{entity}Service.service.ts` (note: `.service.ts` suffix)
- **Models**: `{Entity}Model.ts` with PascalCase
- **Components**: `{feature}.component.ts` with kebab-case directories

### Data Models
- **Soft Delete Pattern**: All entities have `killed?: number` field (0 = active, 1 = deleted)
- **User Association**: Most entities link via `idUser: number` field
- **Pagination**: Services expect `page` and `size` parameters

### Error Handling
- **Console Patterns**: Use `✅` for success fallbacks, `❌` for errors
- **Fallback Strategy**: Always provide local data alternatives
- **Observable Chaining**: Prefer RxJS operators over nested subscribes

## Integration Points

### Cross-Service Communication
- **Templates → Reserves**: Event types from MS-Templates used in MS-Reserves
- **Auth → All**: JWT tokens required for all backend communication
- **Shared Models**: `ResponsePage.ts` for paginated responses across services

### External Dependencies
- **PrimeNG**: Complete UI framework, import modules via `src/app/imports.ts`
- **RxJS**: Reactive patterns throughout, especially for fallback strategies
- **Angular Router**: Lazy-loaded standalone components for all routes

### Future Enhancements
- **Expand local developent**:  
npm uninstall -g localtunnel
npx lt --port 4200

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DAANVALE)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DAANVALE)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
