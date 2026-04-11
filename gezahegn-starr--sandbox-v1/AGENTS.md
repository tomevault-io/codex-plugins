# Copilot Instructions

## Project Overview

This is a **monorepo** for running GitHub Copilot CLI agents inside isolated local sandboxes using **Apple's `container` runtime** (not Docker). The project is macOS-specific and relies on the `container` binary being installed on the host.

## Repository Structure

```
packages/
  sandbox-cli/        # Go CLI — manages sandbox lifecycle
  container/          # Dockerfiles for the agent container images
  mise-example/       # Minimal example project for testing the agent:mise image
docs/                 # Documentation (currently empty)
```

---

## `packages/sandbox-cli` — The `sandbox` CLI

A Go CLI tool (using [Cobra](https://github.com/spf13/cobra) + [charmbracelet/huh](https://github.com/charmbracelet/huh)) that wraps the Apple `container` binary to manage sandbox lifecycle for Copilot agents.

**Language:** Go 1.25  
**Entry point:** `main.go` → `cmd.Execute()`  
**Module name:** `sandbox-cli`

### Commands (`cmd/`)

| File | Command(s) | Purpose |
|------|-----------|---------|
| `root.go` | `sandbox` | Root command, global `--debug` flag, `containerBin()` helper |
| `run.go` | `sandbox run` | Create-or-reuse a sandbox and attach to it |
| `create.go` | `sandbox create` | Create a sandbox without attaching |
| `exec.go` | `sandbox exec` | Execute a command inside a running sandbox |
| `ls.go` | `sandbox ls` | List sandboxes |
| `stop.go` | `sandbox stop` | Stop one or more sandboxes |
| `rm.go` | `sandbox rm` | Remove one or more sandboxes |
| `save.go` | `sandbox save` | Save a sandbox snapshot as a template |
| `reset.go` | `sandbox reset` | Remove all sandboxes and clean up state |
| `network.go` | `sandbox network [ls\|connect\|disconnect]` | Manage sandbox networking |
| `signals.go` | — | Signal forwarding helpers |
| `prompt.go` | — | Interactive selection UI (charmbracelet/huh) |
| `types.go` | — | Go structs for JSON output of `container ls --format json` |
| `version.go` | `sandbox version` | Print version |

### Key conventions

- **Sandbox naming:** `copilot-<last-segment-of-workspace-path>` (e.g., `/Users/you/myapp` → `copilot-myapp`)
- **Container binary:** Always accessed via `containerBin()` which returns `"container"` (Apple's binary)
- **Attach pattern:** `run` and `exec` use `syscall.Exec` to replace the Go process with the container binary so signals and terminal control pass directly through
- **Debug logging:** Use `debugLog(format, args...)` — only prints when `--debug` flag is set
- **GITHUB_TOKEN:** Must be set in the host environment; injected into containers via `-e GITHUB_TOKEN=${GITHUB_TOKEN}`
- **Skills mount:** If `~/.copilot/skills` exists on the host it is automatically mounted read-only into the container at `/home/agent/.copilot-host-skills`

### Build & install

```shell
cd packages/sandbox-cli
go build -o sandbox .          # build locally
go build -o ~/go/bin/sandbox . # install globally
```

---

## `packages/container` — Container Images

Base and feature images for the Copilot agent environment.

| File | Tag | Description |
|------|-----|-------------|
| `Dockerfile` | `agent` | Base image (Ubuntu 24.04, Node.js 24, Go, Python 3, Copilot CLI) |
| `Dockerfile.mise` | `agent:mise` | Extends `agent` with [mise](https://mise.jdx.dev) polyglot runtime manager |
| `Dockerfile.playwright-cli` | `agent:playwright-cli` | Extends `agent` with Playwright |

**Base image key details:**
- Non-root user: `agent` (has passwordless sudo)
- Working directory inside container: `/home/agent/workspace`
- Workspace is mounted at both `/home/agent/workspace` and the host absolute path
- Entrypoint: `/home/agent/entrypoint.sh`
- XDG dirs configured under `/home/agent/.config`, `.local/state`, `.local/share`
- Node 24 is mandatory for the Copilot CLI

**Adding a new feature image:** Create `Dockerfile.<feature>` in `packages/container/` starting with `FROM agent`.

### Build commands

```shell
# From repo root
docker build --tag agent --file packages/container/Dockerfile packages/container
docker build --tag agent:mise --file packages/container/Dockerfile.mise packages/container
```

---

## `packages/mise-example` — Test Project

A minimal example used to validate the `agent:mise` feature image. Contains a `.mise.toml` that pins Node 22 and Python 3.12, plus a `check.sh` verification script.

```shell
sandbox run --image agent:mise packages/mise-example
```

---

## Common Workflows

### Running an agent sandbox

```shell
# First time — creates and attaches
sandbox run /path/to/your/project

# Subsequent runs — reuses existing container
sandbox run /path/to/your/project

# With a feature image
sandbox run --image agent:mise /path/to/your/project
```

### Adding a new CLI command

1. Create `packages/sandbox-cli/cmd/<command>.go`
2. Define a `*cobra.Command` variable
3. Register it in `init()` with `rootCmd.AddCommand(...)` (or a subcommand parent)
4. Follow the `runE` pattern: return errors rather than printing them

### Adding a new container feature

1. Create `packages/container/Dockerfile.<feature>` starting with `FROM agent`
2. Document it in the feature table in `Readme.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gezahegn-starr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gezahegn-starr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
