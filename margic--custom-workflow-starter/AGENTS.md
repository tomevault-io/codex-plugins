# Custom Workflow Spring Boot Starter

## Project Overview

This is a Spring Boot starter that extracts Kogito Serverless Workflow custom URI scheme infrastructure into three publishable Gradle modules plus a sample app. It enables consuming applications to get full `dmn://`, `anax://`, and `map://` custom function support by adding a single Gradle dependency and plugin. Consumers never reference `org.kie.kogito` artifacts directly.

Governance assets (DMN models, Jolt mapping specs) are authored via Copilot + MCP tools connected to the [Metadata Management Platform](docs/canonical-metadata-server.md), committed to `src/main/resources/`, and read locally at build time. Builds are fully self-contained with zero external service dependencies. At runtime, the starter registers the catalog with the metadata server for observability and drift detection.

## Tech Stack

- **Java 17** — minimum target, no Java 21+ features
- **Spring Boot 3.3.7** — auto-configuration uses `META-INF/spring/AutoConfiguration.imports` (not deprecated `spring.factories`)
- **Gradle 8.10+** — Gradle-only, no Maven support
- **Kogito 10.1.0** — pinned version, managed centrally via `kogitoVersion` property
- **CNCF Serverless Workflow spec v0.8**

## Architecture: Governance Asset Lifecycle

The project follows a three-phase lifecycle for governance assets:

### Phase 1: Author (IDE → Metadata Server)
- **Primary:** Copilot + MCP tools — `create_decision`, `update_decision`, `validate_decision`, etc. (19 validated MCP tools)
- **Fallback:** Metadata server REST API / UI for non-Copilot workflows
- Copilot writes DMN/Jolt files directly into `src/main/resources/`
- Developer commits governance assets to git
- All MCP write tools force `status: "draft"` — human promotes to `active` via metadata platform UI

### Phase 2: Build (Local Only)
- `FunctionTypeHandler` SPI implementations (in `anax-kogito-codegen-extensions`) are discovered via `ServiceLoader` during Kogito codegen
- They parse custom URIs and emit `WorkItemNode` entries in the generated process code
- Reads `.sw.json` and `.dmn` files from `src/main/resources/` — **no metadata server dependency**
- Builds are fully self-contained and reproducible

### Phase 3: Runtime (Spring Boot)
- `DefaultKogitoWorkItemHandler` subclasses (in `anax-kogito-spring-boot-starter`) are registered by name with `WorkItemHandlerConfig`
- When the process engine reaches a `WorkItemNode`, it dispatches to the handler matching the `workName`
- On `ApplicationReadyEvent`, the starter publishes the catalog to the metadata server (`POST /api/registrations`)
- Registration enables observability, drift detection, and service catalog features
- Registration failure is logged but does NOT prevent the application from starting

**Critical invariant:** The `workName(scheme)` set during codegen must exactly match the name used in `register(scheme, handler)` at runtime.

## Module Structure

| Module | Purpose | Phase |
|--------|---------|-------|
| `anax-kogito-codegen-extensions` | SPI `FunctionTypeHandler` implementations for `dmn://`, `anax://`, `map://` | Build time |
| `anax-kogito-spring-boot-starter` | Auto-configuration: `WorkItemHandler` beans + `WorkItemHandlerConfig` + metadata catalog REST endpoint + runtime registration with metadata server | Runtime |
| `anax-kogito-codegen-plugin` | Gradle plugin: `generateKogitoSources` task, classpath wiring, BOM management, `catalog.json` generation | Build time |
| `anax-kogito-sample` | Example Spring Boot app demonstrating all features (not published) | Both |

## Custom URI Schemes

All custom functions use `"type": "custom"` in `.sw.json` definitions:

| Scheme | URI Pattern | Purpose | Handler |
|--------|-------------|---------|---------|
| `dmn://` | `dmn://{namespace}/{modelName}` | Evaluate a DMN decision model in-process | `DmnWorkItemHandler` |
| `anax://` | `anax://{beanName}/{methodName}` | Invoke a Spring bean method (default method: `execute`) | `AnaxWorkItemHandler` |
| `map://` | `map://{mappingName}` | Apply a Jolt data transformation (spec committed in `src/main/resources/META-INF/anax/mappings/`) | `MapWorkItemHandler` |

### Bean method contract for `anax://`

```java
public Map<String, Object> methodName(Map<String, Object> params)
```

### Jolt transformation contract for `map://`

Jolt specs are committed at `src/main/resources/META-INF/anax/mappings/{mappingName}.json`. At runtime, `MapWorkItemHandler` loads the spec and applies the Jolt transformation. The initial implementation is a stub (Jolt engine wired in a later iteration).

## Kogito Source Code References

When researching Kogito internals, use the correct branch:

| What | Branch / URL |
|------|-------------|
| **Gradle plugin for Kogito codegen** (does not exist in 10.1.0) | `main` branch: https://github.com/apache/incubator-kie-kogito-runtimes/tree/main |
| **Everything else** (codegen engine, SPI, runtime, APIs) | `10.1.x` branch: https://github.com/apache/incubator-kie-kogito-runtimes/tree/10.1.x |

This distinction is critical. The Gradle plugin does not exist in the 10.1.0 release — our `anax-kogito-codegen-plugin` fills that gap using reflective invocation of the codegen engine. The `main` branch has a reference Gradle plugin implementation we model ours after.

## Key Kogito API Patterns

### Codegen SPI (build time)

- Extend `WorkItemTypeHandler` from `org.kie.kogito.serverless.workflow.parser.types`
- Override `type()` → return the scheme name (e.g., `"dmn"`)
- Override `isCustom()` → return `true`
- Override `fillWorkItemHandler()` → parse the URI via `FunctionTypeHandlerFactory.trimCustomOperation(functionDef)`, set `workName()` and `workParameter()` on the factory
- Register in `META-INF/services/org.kie.kogito.serverless.workflow.parser.FunctionTypeHandler`

### Runtime handlers (Spring Boot)

- Extend `DefaultKogitoWorkItemHandler` from `org.kie.kogito.process.workitems.impl`
- Override `activateWorkItemHandler()` → extract parameters from `workItem.getParameter()`, execute logic, call `manager.completeWorkItem()`, return `Optional.empty()`
- Handlers are created as `@Bean` in `AnaxKogitoAutoConfiguration` (not `@Component`)
- Use constructor injection, not `@Autowired`

### Reflective codegen invocation (Gradle plugin)

The `generateKogitoSources` task builds a `URLClassLoader` from `kogitoCodegen` + `runtimeClasspath`, sets `Thread.contextClassLoader`, then invokes `CodeGenManagerUtil.discoverKogitoRuntimeContext()` and `GenerateModelHelper.generateModelFiles()` reflectively. See [docs/0006-POC-REFERENCE.md](docs/0006-POC-REFERENCE.md) §3.1 for the complete working implementation.

### Metadata server integration (Spring Boot starter — runtime registration)

The `MetadataServerRegistrationService` publishes the catalog to the metadata server on `ApplicationReadyEvent`:

- Configured via `anax.metadata-server.url` in `application.yml` or `METADATA_SERVER_URL` env var
- POSTs catalog + instance metadata to `POST /api/registrations`
- Enables observability and drift detection on the metadata server
- Fire-and-forget: registration failure does not block startup
- Optional heartbeat: periodic re-registration at configurable interval

**Package:** `com.anax.kogito.registration`

## Project Documentation

- [ADR 006 — Architecture](docs/0006-kogito-custom-uri-spring-boot-starter.md) — module structure, governance asset lifecycle, URI schemes, auto-configuration, metadata catalog, runtime registration
- [Implementation Plan](docs/0006-IMPLEMENTATION-PLAN.md) — step-by-step prompt sequence for scaffolding
- [POC Reference](docs/0006-POC-REFERENCE.md) — all 10 proven source files from the working prototype
- [README Template](docs/0006-README-TEMPLATE.md) — target README for the starter
- [Metadata Management Platform](docs/canonical-metadata-server.md) — governance asset store for DMN models, Jolt mappings, workflows, and canonical models

## Conventions

- **Package names**: `com.anax.kogito.codegen` (codegen extensions), `com.anax.kogito.autoconfigure` (starter auto-config), `com.anax.kogito.catalog` (catalog), `com.anax.kogito.registration` (runtime registration), `com.anax.kogito.gradle` (plugin)
- **Group ID**: `com.anax`
- **Work-item parameter names** use PascalCase constants: `DmnNamespace`, `ModelName`, `BeanName`, `MethodName`, `MappingName`
- **Auto-configuration conditions**: `DmnWorkItemHandler` is `@ConditionalOnClass(DecisionModels.class)` so projects without DMN skip it. `MetadataServerRegistrationService` is `@ConditionalOnProperty(prefix = "anax.metadata-server", name = "url")` so registration is only active when configured. Other handlers use `@ConditionalOnMissingBean` for consumer override.
- **Catalog endpoint**: enabled by default at `/anax/catalog`, disabled via `anax.catalog.enabled=false`
- **Runtime registration**: enabled when `anax.metadata-server.url` is configured; registration failure does not prevent startup

## Dev Container & Docker Compose

The dev container runs Docker Compose with infrastructure services for integration testing:

| Service | Image | Port | Purpose |
|---------|-------|------|---------|
| `metadata-platform` | `margic/anax-metadata-platform:latest` | `3001` | Real governance asset store (DMN models, Jolt specs) |
| `redpanda` | `redpandadata/redpanda:v24.1.7` | `19092` (Kafka), `18081` (Schema Registry) | Kafka-compatible broker for workflow events |
| `redpanda-console` | `redpandadata/console:v2.6.0` | `8080` (internal) | Kafka topic inspection UI |

**Service URLs inside dev container:**
- Metadata server: `http://metadata-platform:3001`
- Kafka bootstrap: `redpanda:9092`

**Service URLs from host:**
- Metadata server: `http://localhost:3001`
- Kafka: `localhost:19092`
- Redpanda Console: `http://localhost:8080` (auto-forwarded by Codespaces)

Tests tagged `@Tag("devcontainer")` require these services. Run with `./gradlew test -PincludeTags=devcontainer`. CI excludes them by default.

## Build Commands

```bash
# Build all modules
./gradlew build

# Publish to local Maven repo
./gradlew publishToMavenLocal

# Run sample
./gradlew :anax-kogito-sample:bootRun
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/margic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
