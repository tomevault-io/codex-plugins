# Copilot / AI agent instructions for this repository

This repository is a NestJS monorepo (apps + libs). The goal of these notes is to help an AI coding agent become productive quickly by surfacing the project's architecture, developer workflows, and code conventions — with concrete file references and commands.

1) Big-picture architecture
- Monorepo using NestJS. The main app is `apps/api-gateway` (see `apps/api-gateway/src`). Shared code lives under `libs/*` (for example `libs/database`, `libs/config`, `libs/shared`).
- `nest-cli.json` declares projects and is the source of truth for which libs/apps are part of the workspace.
- Runtime patterns: `apps/api-gateway/src/main.ts` sets global prefix `api/${app.version}`, enables Swagger at `/api-docs`, and uses a `ConfigService` from `@app/config`.
- Data layer: TypeORM + PostgreSQL configured under `libs/database` (see `libs/database/src/data-source.ts` and `libs/database/src/entities`). Migrations are generated/run via TypeORM CLI configured in `package.json` scripts.
- Integrations: RabbitMQ (amqplib + amqp-connection-manager), Redis (cache-manager + ioredis), Firebase, Cloudinary/AWS S3, PayOS, Twilio, ffmpeg. These appear as libs under `libs/` and dependencies in `package.json`.

2) Developer workflows & commands (copyable)
- Install dependencies: `npm install`
- Local dev (watch): `npm run start:dev` (uses Nest CLI to watch the monorepo app declared by `nest-cli.json`).
- Start production build locally: `npm run build` then `npm run start:prod` (note `start:prod` runs the compiled file under `dist`).
- Docker compose: `docker compose up -d` — `docker-compose.yml` is present at repository root.
- TypeORM migrations:
  - Generate: `npm run migration:generate -- <Name>` (scripts point to `libs/database/src/data-source.ts`)
  - Run: `npm run migration:run` (dev) or `npm run migration:run:prod` (production dist path)
- Seeds: `npm run seed` (dev ts-node) / `npm run seed:prod` (dist)
- Tests: unit `npm run test`, watch `npm run test:watch`, e2e `npm run test:e2e` (uses `apps/api-gateway/test/jest-e2e.json`).
- Lint & format: `npm run lint` and `npm run format`.

3) Project-specific conventions and patterns
- Path aliases: tsconfig/jest mapping use `@app/*` aliases (see `package.json` jest section). Use these aliases when referencing libs (e.g. `@app/config`, `@app/shared`).
- Monorepo project additions: when adding a new library, add an entry in `nest-cli.json` and create a `tsconfig.lib.json` under the new lib consistent with other libs.
- Custom shared utilities:
  - Custom RPC exceptions: `@app/shared/customs/custom-rpc-exception` (main app maps RPC errors into HTTP responses).
  - Custom logging: `@app/shared/customs/custom-logger` and `apps/api-gateway/src/middlewares/logger.middleware.ts` — respect existing logging prefix and structure.
- Global pipes and validation: The API gateway uses a `ValidationPipe` configured to transform inputs and convert constraint errors into `CustomRpcException` messages. Keep DTOs consistent with class-validator and class-transformer.

4) Integration and communication patterns
- GraphQL: `@nestjs/graphql` + Apollo — fetching data primarily through GraphQL resolvers under `apps/api-gateway/src/resolvers`.
- Microservices/queues: RabbitMQ used for cross-service messaging. Look for uses of `@nestjs/microservices`, `amqplib`, and `amqp-connection-manager` in services.
- Caching: `@nestjs/cache-manager` + Redis. Cache store configuration lives in `libs/redis`.
- File storage: AWS S3 and Cloudinary libs exist under `libs/aws` / `libs/cloudinary` — check their modules to see how files are uploaded and signed.

5) Where to make typical changes (concrete file references)
- Add new API endpoints / GraphQL resolvers: `apps/api-gateway/src/controllers` and `apps/api-gateway/src/resolvers`.
- Add shared utilities or DTOs: `libs/shared/src` (decorators, dtos, helpers).
- Add or modify DB entities / migrations: `libs/database/src/entities` and `libs/database/src/migrations` (use TypeORM scripts to generate migrations).
- Configuration values: `libs/config/src` (the `ConfigService` is used widely; check `apps/api-gateway/src/main.ts` to see how `ConfigService` is consumed).

6) Quick checks for PRs or code-changes
- Run `npm run build` to ensure TypeScript compiles (monorepo build uses `nest build`).
- Run `npm run test` (unit) and `npm run test:e2e` for integration where applicable.
- If DB schema changes are made, include generated TypeORM migration and ensure `npm run migration:generate` is reproducible.

7) Helpful hints for code generation by AI
- Prefer using existing aliases (e.g. `@app/shared`) and imports patterns from neighboring files.
- Follow the established exception/response pattern — use `CustomRpcException` for validation errors and existing `CustomLogger` for logging statements.
- When adding new Nest modules, register them in `AppModule` inside `apps/api-gateway/src/app.module.ts` and ensure any provider tokens follow the shared pattern.
- For changes touching runtime config, read/modify `libs/config` and adjust `nest-cli.json` if adding new projects.

8) CRITICAL: Always verify against actual codebase before creating diagrams, documentation, or code
- **NEVER** add properties, methods, or enums that don't exist in the actual codebase.
- **BEFORE creating any diagram or documentation**, read the actual source files:
  - For entities: check `libs/database/src/entities/` for actual properties and relationships
  - For enums: check `libs/shared/src/enums/` for exact enum values
  - For services: check `apps/api-gateway/src/services/` for actual methods
  - For DTOs: check `libs/shared/src/dtos/` for actual fields
- **Example of mistakes to avoid**:
  - ❌ Adding `CourseLearningFormat.HYBRID` when only INDIVIDUAL and GROUP exist
  - ❌ Adding fields like `maxCapacity` or `availableSpots` to Session when they don't exist
  - ❌ Using `EnrollmentStatus.ACTIVE` when actual values are PENDING_GROUP, CONFIRMED, LEARNING, UNPAID, CANCELLED, DONE
  - ❌ Creating methods that don't exist in the actual service implementation
- **Always follow these steps**:
  1. Read the actual entity/enum/service files from the codebase
  2. Use ONLY properties/methods/values that actually exist
  3. If a feature requires new fields, propose them but clearly mark as "NEW - not yet in codebase"
  4. Cross-reference with actual usage in services and controllers

If anything in these notes is unclear or you'd like more detail (for example — wiring a new RabbitMQ consumer, or how database seeds are organized), tell me which area to expand and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gs4lthung)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gs4lthung)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
