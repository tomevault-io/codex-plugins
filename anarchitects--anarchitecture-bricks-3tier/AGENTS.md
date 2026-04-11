# Copilot Instructions - Anarchitecture Bricks 3-Tier

This repository is implementation-first, library-focused, and Nx-driven.

## Global Rules

- Prefer reusable packages in `libs/`.
- Example applications are allowed only under `examples/`.
- Shared DTOs/models live in `libs/*/ts`.
- HTTP documentation is generated from Nest implementation (`@RouteSchema`) into `docs/openapi`.
- Do not hand-edit generated OpenAPI artifacts.
- Use Nx targets for build/test/lint/docs workflows.

## Layering

### Angular

- `ui/`: presentational components only
- `feature/`: orchestration and façade-level behavior
- `state/`: signal-store state management
- `data-access/`: HTTP adapters and API integration
- `config/`: typed providers/tokens
- `util/`: pure helper code

Dependency direction:

`ui <- feature -> state -> data-access` | `config`, `util`: available to all layers

### Nest

- `presentation/`: controllers and route schemas
- `application/`: use-cases and service abstractions
- `infrastructure-*`: adapters (persistence, mailer, external APIs)
- `config/`: typed runtime configuration
- `util/`: pure helper code
- package root (`@anarchitects/<domain>-nest`): facade module for full-stack, minimal-import consumption

Dependency direction:

`presentation -> application <- infrastructure`

### Nest Consumption Pattern

- Prefer root facade module imports in quick starts and examples when the full stack is intended.
- Keep secondary entry points (`/application`, `/presentation`, `/infrastructure-*`, `/config`) available for advanced/partial composition.
- Do not remove existing layered entry points when introducing facade modules.
- Keep presentation modules backward-compatible unless a breaking change is explicitly planned.
- Configure shared transports once at app root (for example mail via `CommonMailerModule`), not inside domain infrastructure modules.
- Keep domain infrastructure modules adapter-only and use facade feature flags (for example `features.mailer`) for optional domain capabilities.

## TypeORM Cross-Domain Relationship Rule

Domain libraries must **not define TypeORM relations across domains**.

### Rule

If an entity references another domain’s entity:

- Use a **scalar foreign key field** (`userId`, `authorId`, etc.)
- Do **not import the other domain’s entity**
- Do **not define `@ManyToOne`, `@OneToMany`, etc. across domains**
- Do **not extend another domain’s entity**

### Example

Correct:

```ts
@Entity()
export class PostEntity {
  @Column('uuid')
  authorId!: string;
}
```

Incorrect:

```ts
@ManyToOne(() => UserEntity)
author!: UserEntity;
```

### Database Foreign Keys

Cross-domain FKs may exist in the database but **must not be defined in domain entities**.

Instead:

- define them in **integration schemas** used only for migrations
- place those schemas in:

```text
tools/typeorm/schemas/
```

### Data Access Across Domains

When data from another domain is needed, use:

- service composition
- query builder joins
- integration views or read models

### Summary

| Case                 | Rule                                    |
| -------------------- | --------------------------------------- |
| Same domain          | TypeORM relations allowed               |
| Cross domain         | Use scalar FK only                      |
| DB FK                | Define in integration migration schemas |
| Reads across domains | Join/query/service composition          |

## Angular signalStore Rule

Domain stores must **not use `providedIn: 'root'`.**

### Rule

- State stores in domain libraries must be created **without** `providedIn: 'root'`
- They must be registered through **provider helpers**
- App-wide or cross-domain state must be registered explicitly in app or feature `providers`
- Domain libraries must not implicitly create global singleton state

### Correct

```ts
@Injectable()
export class FormsStore {}
```

```ts
export function provideFormsState(): Provider[] {
  return [FormsStore];
}
```

### Incorrect

```ts
@Injectable({ providedIn: 'root' })
export class FormsStore {}
```

### State Registration

State must be registered explicitly:

- in route providers
- in feature providers
- in application bootstrap providers

### Shared TS

- `dtos/`: TypeBox DTO contracts
- `models/`: domain models
- `builders/validators/util/`: schema tooling and helpers

## API and Docs Workflow

- Route metadata comes from Nest controllers and `@RouteSchema`.
- Nest controllers must keep `@RouteSchema` pure Fastify schema fields only (`body`, `params`, `querystring`, `headers`, `response`).
- Do not place `operationId` or `tags` in controllers. OpenAPI metadata is owned by `tools/api-specs/route-metadata.ts`.
- Nest controllers must not define inline TypeBox route schemas. Define schemas in domain TS DTO libraries (`libs/<domain>/ts/src/dtos`) and import them into controllers.
- When adding or changing a route, update `tools/api-specs/route-metadata.ts` with the `METHOD + path` operationId mapping.
- Generate and validate OpenAPI with:
  - `nx run api-specs:generate`
  - `nx run api-specs:lint`
  - `nx run api-specs:verify`
- Use `nx run api-specs:diff` to inspect compatibility vs `origin/main`.
- Use `nx run api-specs:mock` for Prism-based mocking.
- Generate Angular technical docs with `nx run angular-docs:generate`.
- Storybook is the default UI documentation front end and consumes Compodoc metadata.

## Release Workflow

- Do not rely on local `nx release` for normal releases.
- Run full releases from GitHub Actions via `.github/workflows/release.yml` (`workflow_dispatch`) with an explicit domain group input.
- Release command model is domain-scoped: `nx release --groups=<domain> --yes`.
- Use `.github/workflows/publish.yml` only as manual recovery to retry publishing when needed.

## New Domain Rule

- If a new domain is created under `libs/<domain>`, you must add a matching release group in `nx.json` within the same workstream.
- You must also update `tools/release/validate-domain-tags.mjs` with the folder-to-tag mapping for that domain.
- Keep `README.md` and `CONTRIBUTING.md` release instructions current when domain groups change.

## Do

- Keep module boundaries clean and tag-aware.
- Add/update tests when changing DTOs, route schemas, or domain behavior.
- Keep example apps in sync with library integration contracts.
- Keep APIs and documentation deterministic and reproducible.

## Project Planning Workflow

For issue-driven implementation work, agents must keep the GitHub planning board in sync with execution progress.

- Project: https://github.com/orgs/anarchitects/projects/15
- Working view: Board
- Sprint field: Milestone
- Status options: Backlog, Todo, In Progress, In Review, Blocked, Done

### Status Transition Rules

- Move issue to In Progress when active coding starts.
- Move issue to Blocked when external dependencies prevent progress.
- Move issue to In Review when implementation is ready for review or an implementation PR exists.
- Move issue to Done when completion criteria are met.
- Use Backlog or Todo for not-yet-started work based on planning intent.

### Operational Rules

- Include board updates as part of the implementation routine, not as a separate optional step.
- Preserve existing Priority and Milestone unless user asks to change them.
- For new sub-issues, add them to the project and assign Milestone according to the current sprint plan.

## Human In The Loop Workflow

AI coding agents operate in assistive mode and do not replace human code ownership.

### Review And Delivery

- Human developers review all AI-proposed code changes.
- Human developers perform git commit operations.
- Human developers create and merge pull requests.
- AI coding agents may suggest commit messages, but must not treat suggestions as committed history.

### Breaking Change And Deprecation Handling

- AI coding agents must explicitly state when a change introduces potential breaking behavior.
- AI coding agents must explicitly state when a bug fix may justify npm package deprecation of previously published versions.
- AI coding agents may prepare deprecation instructions, but execution is allowed only after explicit human approval.

## Don't

- Reintroduce `contracts/openapi.yaml` as source-of-truth.
- Hand-edit generated `docs/openapi/*` files.
- Mix Angular and Nest concerns in the same library layer.
- Add framework-coupled logic into shared TS packages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anarchitects)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/anarchitects)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
