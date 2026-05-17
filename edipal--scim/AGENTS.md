# SCIM 2.0 Playground - Copilot Instructions

## Architecture

Multi-tenant SCIM 2.0 Service Provider (RFC 7643/7644) with five Maven modules:

| Module | Role | Port |
|---|---|---|
| `scim-server-common` | Shared JPA entities, repositories, and common security utilities | - |
| `scim-server-api` | SCIM 2.0 API (Users, Groups, Discovery, Bulk) | 8080 |
| `scim-server-mgmt` | Admin UI + management API (Thymeleaf + vanilla JS) | 8081 |
| `scim-validator` | Groovy/Spock SCIM compliance suite (REST Assured) | - |
| `scim-validator-mgmt` | Validator run/inspection management service | 8082 |

Multi-tenancy is workspace-based. SCIM routes are scoped to `/ws/{workspaceId}/scim/v2/**`,
and the current implementation expects `workspaceId` to be a UUID.

- `BearerTokenAuthFilter` extracts the workspace UUID from the path and validates that the token belongs to that workspace.
- Bearer tokens are validated via SHA-256 hash lookup (`WorkspaceTokenRepository.findByTokenHashAndNotRevoked`).
- There is no `WorkspaceContext` ThreadLocal anymore; controllers resolve the workspace UUID from the route and pass it explicitly into services.
- All core SCIM entities are workspace-scoped with `workspace_id` foreign keys.

Compatibility mode is route-based and extensible:

- Controllers expose both base and compat route forms, for example:
	- `/ws/{workspaceId}/scim/v2/Users`
	- `/ws/{workspaceId}/scim/v2/{compat}/Users`
- `CompatMode` currently supports `MS`.
- `MsScimUserMapper` applies Microsoft validator compatibility tweaks:
	- converts selected `primary` booleans to string values
	- adds flattened enterprise manager alias key

Management security uses Auth0 OIDC:

- Both management modules use standard Spring Security OAuth2 Client with Auth0 as the OIDC provider.
- Each module has its own `AUTH0_CLIENT_ID`, `AUTH0_CLIENT_SECRET`, and `AUTH0_ISSUER_URI`.
- Role claims are read from a configurable OIDC claim (`APP_SECURITY_OIDC_ROLE_CLAIM`, default `https://scimplayground.dev/roles`).
- Management user persistence is email-based in both management modules; resolved emails are normalized and stored as the primary key.
- Management access expects a usable email claim from OIDC principals.
- Shared helpers live in `scim-server-common` (`Auth0OidcSecuritySupport`, `MgmtSecuritySupport`).

Kubernetes support is split into two trees:

- `k8s/app/**` deploys the namespaced SCIM stack in `scim`:
	- CloudNativePG PostgreSQL cluster
	- validator database resource
	- API, management, and validator-mgmt Deployments and Services
- `k8s/cluster/**` deploys supporting cluster resources:
	- local-path storage configuration and `local-path-custom` `StorageClass`
	- `cloudflared` in its own namespace

Kubernetes secrets are stored as `*.sops.yaml` files and rendered through `ksops`.
The root `.sops.yaml` defines the active age recipient.

## Build And Run

```bash
# Full reactor build
mvn clean install

# Full reactor build without running validator specs
mvn clean install -Dskip.validator.tests=true

# API local mode (requires datasource env vars and ACTUATOR_API_KEY)
cd scim-server-api && mvn spring-boot:run

# Mgmt UI/API local mode (requires datasource env vars, ACTUATOR_API_KEY, and
# Auth0 OIDC env vars: AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_ISSUER_URI,
# AUTH0_REDIRECT_URI — or set SPRING_PROFILES_ACTIVE=cloudflare for Cloudflare JWT mode)
cd scim-server-mgmt && mvn spring-boot:run

# Validator management local mode (requires datasource env vars, ACTUATOR_API_KEY, and
# Auth0 OIDC env vars: AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_ISSUER_URI,
# AUTH0_REDIRECT_URI — or set SPRING_PROFILES_ACTIVE=cloudflare for Cloudflare JWT mode)
cd scim-validator-mgmt && mvn spring-boot:run

# Docker stack
docker compose up --build

# Docker stack plus local cloudflared sidecar
docker compose --profile cloudflare up --build

# Kubernetes support resources (requires kubectl, kustomize, ksops, sops, and SOPS_AGE_KEY_FILE)
kustomize build --enable-alpha-plugins --enable-exec k8s/cluster | kubectl apply -f -

# Kubernetes application stack
kustomize build --enable-alpha-plugins --enable-exec k8s/app | kubectl apply -f -
```

Docker default ports:

- API `:8080`
- Mgmt `:8081`
- Validator Mgmt `:8082`
- Playground PostgreSQL `:5432`
- Validator PostgreSQL `:5433`

Operational notes:

- `docker-compose.yml` loads `docker/env/cloudflare.env` into the management apps.
- Kubernetes manifests set `SPRING_PROFILES_ACTIVE=cloudflare` for the management apps.
- Application services in Kubernetes are `ClusterIP`; Cloudflare tunnel is the external-access path in this branch.
- No repository-specific `DOCKER_HOST` or `TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE` overrides are required for local Testcontainers runs; use the default local Docker Desktop / Docker Engine setup.

## Validator Execution

`scim-validator` can either bootstrap its own disposable target via
Testcontainers or run against an already reachable SCIM API.

```bash
cd scim-validator && mvn test
```

Notes from `ScimBaseSpec`:

- By default, the validator can bootstrap PostgreSQL plus `edipal/scim-server-api:latest` when explicit `SCIM_*` settings are not provided.
- Bootstrap selection is based on whether a usable target is configured, not on the presence of the default `SCIM_API_URL` placeholder value.
- Disable automatic bootstrap with `SCIM_TESTCONTAINERS_ENABLED=false` or `-Dscim.testcontainers.enabled=false` when targeting an existing environment.
- You can alternatively set `SCIM_BASE_URL` (full path, including `/ws/{workspaceId}/scim/v2`).
- You can also provide `SCIM_API_URL` together with `SCIM_WORKSPACE_ID`.
- `SCIM_AUTH_TOKEN` is required for validator runs.
- `SCIM_WORKSPACE_ID` is required unless `SCIM_BASE_URL` is provided.
- Validator config is loaded from `validator-application.yml` in test resources to avoid `application.yml` collisions when the validator test JAR is consumed by `scim-validator-mgmt`.
- `ValidatorConfiguration` accepts both `SCIM_*` placeholders and dotted JVM properties such as `scim.baseUrl`, `scim.authToken`, `scim.apiUrl`, and `scim.workspaceId`, because `scim-validator-mgmt` sets dotted properties before loading validator specs.
- The validator `tests` classifier JAR is packaged after test resources are copied (`test-compile`) so downstream consumers receive `validator-application.yml`.

## Code Conventions

- No Lombok in this repository.
- Constructor injection throughout; no `@Autowired` field injection.
- SCIM response payloads are map-built (`Map<String, Object>`, usually `LinkedHashMap`), not dedicated response DTO hierarchies.
- SCIM mapping code uses static utility classes (`ScimUserMapper`, `ScimGroupMapper`, `MsScimUserMapper`).
- Java records are used in DTO layers (notably in mgmt and validator-mgmt modules).
- `@Transactional` appears on service classes and on selected controller classes/methods. Preserve existing boundaries unless intentionally refactoring.
- Flyway migrations are split by concern:
	- shared schema migrations under `db/common`
	- validator schema migrations under `db/validator`
	- module-specific migrations under `db/migration`
- Content types:
  - SCIM endpoints: `application/scim+json`
  - Mgmt endpoints: standard JSON (`application/json`)

## Data Model Patterns

`ScimUser` and `ScimGroup` use optimistic locking via `@Version` (ETag support).

- Workspace-scoped uniqueness:
	- `scim_users`: `(workspace_id, user_name)`
	- `scim_groups`: `(workspace_id, display_name)`
- Management ownership data is email-keyed:
	- `mgmt_users.email` is the primary key
	- `validator_mgmt_users.email` is the primary key
	- `validation_run.created_by_email` is a foreign key to `validator_mgmt_users(email)`
- `workspaces.created_by_username` is sized for email-style owner values (`VARCHAR(500)`).
- `ScimUser` flattens `name.*` and enterprise extension sub-attributes into columns.

- Multi-valued user attributes on `ScimUser` are JSON-backed lists, not `@OneToMany` child entities:
	- `emails`, `phoneNumbers`, `addresses`, `entitlements`, `roles`, `ims`, `photos`, `x509Certificates`

## Key SCIM Components

- `ScimFilterParser`: recursive-descent filter parser to JPA `Specification<T>`.
	- operators: `eq ne co sw ew pr gt ge lt le`
	- logic: `and or not`, grouping with parentheses
	- supports `name.*`, `meta.*`, and enterprise extension attribute paths
- `ScimPatchEngine`: RFC 7644 PATCH processing with path parsing and filtered multi-valued operations.
- `ScimSchemaDefinitions`: source of truth for discovery/schema responses.

When adding or changing attributes, keep parser, mapper, patch, and schema definitions aligned.

## Protocol And API Behavior

- SCIM pagination is 1-based (`startIndex`, `count`). Controllers clamp invalid values and enforce max count (`200`).
- Mgmt pagination input is also 1-based (`page`, `size`) and converted to Spring `PageRequest` (0-based internally).
- PUT/PATCH on Users and Groups support `If-Match` with weak ETags (`W/"<version>"`) and return `412` on mismatch.
- SCIM errors are returned with SCIM error schema and SCIM content type.

## Working In This Codebase

If you modify SCIM behavior, review impact across these areas:

1. API controllers (`Users`, `Groups`, `Bulk`, discovery endpoints)
2. Service layer logic and transactional boundaries
3. Mappers (`ScimUserMapper`, `ScimGroupMapper`, `MsScimUserMapper`)
4. Filter and patch engines
5. Schema definitions and ServiceProviderConfig flags
6. Validator specs (`A1` through `A9`) and compatibility expectations

If you modify management authentication or deployment behavior, also review:

1. Both management modules' `SecurityConfig`
2. Shared helpers in `scim-server-common/src/main/java/.../security`
3. `docker-compose.yml` and `docker/env/*.env`
4. `k8s/app/**` and `k8s/cluster/**`
5. `.sops.yaml` and `age/rotate_sops_age_key.py`

## Adding A New SCIM Attribute

1. Extend `ScimUser`/`ScimGroup` (or add a JSON-backed value object and list field in `scim-server-common` when the attribute is multi-valued).
2. Update mapper read/write paths in `scim-server-api`.
3. Add PATCH support in `ScimPatchEngine` when applicable.
4. Add schema metadata in `ScimSchemaDefinitions`.
5. Extend filter/sort resolution in `ScimFilterParser` when queryable/sortable.
6. Validate ETag, projection (`attributes`/`excludedAttributes`), and compat-mode behavior.
7. Add/adjust validator coverage in relevant `scim-validator` specs.

---
> Source: [edipal/scim](https://github.com/edipal/scim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-17 -->
