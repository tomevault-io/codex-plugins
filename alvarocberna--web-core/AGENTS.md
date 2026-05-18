# GitHub Copilot / AI Agent instructions for web-core

Purpose: provide concise, actionable information for AI agents to be productive quickly in this repository.

---

## Big picture / architecture
- Project uses **NestJS** with a light DDD-like structure under `src/`:
  - `presentation/` — controllers, DTOs, filters, guards, strategies (http layer).
  - `domain/` — use-cases, entities, interfaces (business rules). Example: `src/domain/use-cases/create-articulo.use-case.ts`.
  - `infrastructure/` — datasources, repositories, Prisma ORM client wrappers.
- Controllers call *Service* (presentation) → service delegates to **domain use-cases** → repositories talk to DB (Prisma).
- Authentication: **JWT stored in cookies** (`access_token`, `refresh_token`), handled in `src/presentation/auth/` (AuthController, AuthService, JwtStrategy, JwtAuthGuard).

## Key flows & integration points
- Auth flow:
  - Login issues `access_token` and `refresh_token` cookies (httpOnly, sameSite: 'lax'). See `src/presentation/auth/auth.controller.ts`.
  - `JwtStrategy` reads the token from cookie with `cookieExtractor` and verifies it. The `validate(payload)` returns a `user` object which gets attached to `req.user`. See `src/presentation/auth/strategies/jwt.strategy.ts`.
  - Guarded routes use `@UseGuards(JwtAuthGuard)` to ensure `req.user` is present.
  - Frontend must send cookies on requests (`fetch(..., { credentials: 'include' })` or `axios` with `withCredentials: true`). CORS is configured with `credentials: true` using `URL_FRONTEND` from env. See `src/main.ts`.

## DTOs & validation (important)
- Global `ValidationPipe` is enabled in `src/main.ts` with `whitelist: true` and `forbidNonWhitelisted: true` and `transform: true`.
  - Consequence: any body property without validation decorators (class-validator metadata) will be rejected with `property X should not exist`.
- Conventions:
  - DTOs live in `src/presentation/<resource>/dto` and typically have `CreateXDtoImpl` and `UpdateXDtoImpl` types.
  - For PATCH endpoints prefer `PartialType(CreateDto)` (from `@nestjs/mapped-types`) so updates can be partial.
  - Use `class-transformer` `@Type(() => Date)` for dates and `@Type(() => SubDto)` plus `@ValidateNested({ each: true })` for nested arrays (e.g., `sec_articulo`).

## Common pitfalls and project-specific rules
- Exports must be providers in the same module. If you export a strategy or provider, also include it in `providers` of that module (see fix in `src/presentation/auth/auth.module.ts`).
- Routes must use the correct HTTP method; e.g., `@Patch('/editar/:id')` expects `PATCH`, not `POST`.
- `req.user` exists only on routes protected by `JwtAuthGuard` because Passport/Guard performs token extraction and `validate()` and attaches the object.
- Prisma client is used via a wrapper `PrismaService` which points to the generated client under `generated/prisma/client`. If you change the schema run `npx prisma generate` and `npx prisma migrate dev` when appropriate.
- Error formatting: `HttpExceptionFilter` (`src/presentation/filters/http-exception.filter.ts`) normalizes errors; prefer to inspect `message` and `statusCode` from its output when debugging.

## Useful files & locations (quick references)
- Entry & global config: `src/main.ts` (ValidationPipe, cookie parser, CORS, filters)
- Auth: `src/presentation/auth/{auth.controller.ts,auth.service.ts,auth.module.ts,strategies/jwt.strategy.ts,guards/jwt-auth.guard.ts}`
- DTO examples: `src/presentation/articulo/dto/{create-articulo.dto.ts,update-articulo.dto.ts}`
- Domain use-cases: `src/domain/use-cases/*` (e.g., `create-articulo.use-case.ts`)
- Repositories: `src/infrastructure/repository/*` and datasources in `src/infrastructure/datasources/*`.
- Prisma ORM wrapper: `src/infrastructure/orm/prisma/prisma.service.ts` and generated client `generated/prisma/`.

## Developer workflows (commands)
- Install: `npm install`
- Development server: `npm run start:dev`
- Production build/start: `npm run build` then `npm run start:prod` (or `npm run start` to run compiled app)
- Tests: `npm run test` (unit), `npm run test:e2e` (e2e), `npm run test:cov` (coverage)
- Prisma migrations: `npx prisma migrate dev` (local) and `npx prisma generate` when schema or client changes.

## Debugging tips (practical)
- For auth failures: inspect Network tab and check `Cookie: access_token=...` is sent and `credentials` is set.
- For validation errors: compare request payload with DTOs; remember `forbidNonWhitelisted` will reject unknown props.
- For module provider/export issues: confirm provider is listed in `providers` before exporting (UnknownExportException commonly triggers otherwise).
- For DB issues: check generated client and Prisma migrations; `generated/prisma` is committed in this repo, but regenerate after schema changes.

## Code style & conventions
- Types/DTO classes use `Impl` suffix (e.g., `CreateArticuloDtoImpl`).
- Files are organized by layer (presentation, domain, infrastructure). Keep that separation for new features.
- Use `@UseGuards(JwtAuthGuard)` to mark routes that require authentication and use a `@CurrentUser` decorator (recommended) to fetch `req.user` cleanly.

---

If you'd like, I can open a PR with this file added and iterate based on any missing details you want included (e.g. CI commands, environment examples, or additional error patterns to watch). Please tell me any section that needs more examples or clarity.

---
> Source: [alvarocberna/web-core](https://github.com/alvarocberna/web-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
