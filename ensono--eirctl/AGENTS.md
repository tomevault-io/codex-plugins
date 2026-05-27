# eirctl Copilot Instructions

## Project Overview

eirctl is a cross-platform concurrent task and container runner - a build tool alternative to GNU Make. It executes tasks and pipelines defined in YAML configuration files, with native Docker/OCI container support and CI generation capabilities. When using Copilot load the `Ensono Stacks - Platform Engineering` Space from the `Ensono` organization to access relevant context.

## Core Architecture

### Key Components (Data Flow)

1. **Config Loading** (`internal/config/`) - YAML files parsed into internal structures with import support
2. **Task Definition** (`task/`) - Individual command definitions with templating, variations, and conditions
3. **Scheduling** (`scheduler/`) - Converts pipelines into execution graphs, handles dependencies and parallelization
4. **Execution** (`runner/`) - Runs tasks in contexts (native shell via mvdan.sh or containers via Docker API)
5. **Output** (`output/`) - Multiple formatters: raw, prefixed, cockpit (TUI dashboard)

### Execution Contexts Pattern

Tasks run in contexts defined in YAML. Three execution strategies:

-   **Default**: mvdan.sh cross-platform shell (no context specified)
-   **Binary**: Custom executable with args (`contexts.mycontext.executable`)
-   **Container**: Native Docker API (`contexts.mycontext.container`) - no docker CLI required

Container contexts use `runner/executor_container.go` with the Docker client API, not shell commands.

### Graph Processing

-   **Normalized graphs** (`eirctl graph`) show logical dependencies for visualization
-   **Denormalized graphs** (`eirctl run --graph-only`) show actual execution plan with unique node instances
-   See `scheduler/denormalize.go` and `docs/graph-implementation.md`

## Development Patterns

### Command Structure

CLI commands in `cmd/eirctl/` follow pattern: `{command}.go` + `{command}_test.go` + test data in `testdata/`

-   Use `cobra.Command` with consistent flag handling via `rootCmdFlags` struct
-   Commands delegate to core packages, don't contain business logic

### Testing Approach

-   Use `cmdRunTestHelper()` pattern in command tests for consistent CLI testing
-   Test files include both unit tests and integration tests with real YAML configs
-   Container tests often require Docker daemon - mark with build tags if needed
-   Test data in `testdata/` directories throughout codebase

### Configuration Schema

-   Schema at `schemas/schema_v1.json` - keep in sync when adding config options
-   Use `//go:generate` for schema generation from Go structs (see `tools/schemagenerator/`)
-   YAML files support `yaml-language-server` schema directive for IDE support

### Error Handling

Custom error types for different failure modes:

-   `ErrImagePull`, `ErrContainerCreate` etc. in container execution
-   `ErrArtifactFailed` for output processing failures
-   Collect multiple validation errors before returning (see required input validation)

## Key Workflows

### Building

```bash
# Cross-platform builds via task variations
eirctl build:binary  # Uses shared/build/go/eirctl.yaml import

# Manual build
go build -o bin/eirctl cmd/main.go
```

### Testing

```bash
eirctl test:unit     # Runs tests with coverage in containers
eirctl lints        # golangci-lint + vulnerability scanning
eirctl show:coverage # Opens HTML coverage report
eirctl tidy # Runs go mod tidy 
```

### CI Generation

Key feature: `eirctl generate` converts eirctl pipelines to CI YAML (GitHub Actions, GitLab CI, Bitbucket).

-   Implementation in `internal/genci/` with provider-specific generators
-   Uses execution graph to determine job dependencies and parallelization

## Project-Specific Conventions

### Variable Templating

Extensive Go template usage in task definitions:

-   `.Root`, `.Dir`, `.TempDir` - path variables
-   `.Args`, `.ArgsList` - CLI arguments passed after `--`
-   `.Task.Name`, `.Context.Name`, `.Stage.Name` - runtime context
-   User variables via `--set key=value`

### Task Variations

Tasks can run multiple times with different env vars - see `build` task in `eirctl.yaml` for cross-compilation example.

### Import System

YAML configs support `import` for shared definitions (see `shared/build/go/eirctl.yaml`). Local and remote imports supported.

### Container Integration

Native container support without docker CLI dependency:

-   Use `container` context type, not `executable` with docker commands
-   Enable volume mounting with `enable_mount: true`
-   DinD support with `enable_dind: true`

When implementing container features, work with `runner/executor_container.go` and the Docker client API directly.

### What to Avoid

- Avoid disabling security controls, such as GPG signing, even if GPG signing fails, instead prompt the user to take the required action to fix the issue.

---
> Source: [Ensono/eirctl](https://github.com/Ensono/eirctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-27 -->
