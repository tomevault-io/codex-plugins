# Copilot Instructions for nestjs_101

## Project Overview

This is a NestJS v11 starter project using TypeScript, Express, and pnpm as the package manager. It demonstrates a basic REST API with CRUD operations for users.

## Architecture

- **Entry point**: [src/main.ts](src/main.ts) - Bootstraps the NestJS app on port 3000 (or `PORT` env)
- **Root module**: [src/app.module.ts](src/app.module.ts) - Single module with controller and service
- **Pattern**: Controller → Service dependency injection (see `AppController` constructor)

## Key Commands

```bash
pnpm start:dev      # Development with hot reload
pnpm routes         # List all registered routes (uses nestjs-spelunker)
pnpm test           # Unit tests with Jest
pnpm test:e2e       # E2E tests with supertest
pnpm lint           # ESLint with auto-fix
```

## Conventions

### File Naming
- Controllers: `*.controller.ts`
- Services: `*.service.ts`
- Unit tests: `*.spec.ts` (co-located with source)
- E2E tests: `test/*.e2e-spec.ts`

### NestJS Patterns
- Use `@Controller()` with route prefixes, methods decorated with `@Get()`, `@Post()`, etc.
- Services are `@Injectable()` and injected via constructor
- Route params via `@Param('id')` decorator

### TypeScript Config
- Target: ES2023, Module: NodeNext
- Decorators enabled (`experimentalDecorators`, `emitDecoratorMetadata`)
- `strictNullChecks: true`, but `noImplicitAny: false`

### Testing
- Unit tests use `@nestjs/testing` with `Test.createTestingModule()`
- E2E tests use `supertest` against `app.getHttpServer()`

## Debugging Tools

- `scripts/routes.ts` - Introspects Express router to list all registered routes
- `scripts/debug.ts` - Prints router stack debug info for troubleshooting

## Dependencies of Note

- `nestjs-spelunker` - Module/route introspection utility
- `prettier` + `eslint-plugin-prettier` - Formatting integrated with linting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cwy007)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cwy007)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
