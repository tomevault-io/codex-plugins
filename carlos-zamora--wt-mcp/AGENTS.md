# Copilot Instructions for wt-mcp

## What this is

An MCP (Model Context Protocol) server for **Windows Terminal**, built with C# and the [ModelContextProtocol C# SDK](https://www.nuget.org/packages/ModelContextProtocol). It exposes tools that let AI assistants read and modify Windows Terminal settings, fragment extensions, Oh My Posh prompt configs, shell integration, and per-directory snippets (wt.json).

## Build & Run

```bash
dotnet build            # Build the project (targets net10.0)
dotnet run              # Run locally via stdio transport
dotnet pack -c Release  # Create NuGet package for distribution
```

There are no tests or linters configured in this project.

## Architecture

**Entrypoint:** `Program.cs` — Configures a .NET Generic Host with the MCP stdio transport and registers all tool classes.

**Tool classes** (`Tools/`) — Each file is a static class annotated with `[McpServerToolType]` containing `[McpServerTool]` methods. Each public static method becomes an MCP tool. Tool classes are:

| Class | Purpose |
|---|---|
| `SettingsTools` | Read/preview/apply JSON Patch (RFC 6902) changes to `settings.json` |
| `FragmentTools` | CRUD for fragment extensions (portable profiles, schemes, actions) |
| `OhMyPoshTools` | Detect, read, and patch Oh My Posh prompt theme configs |
| `ShellIntegrationTools` | Check/configure shell integration (OSC 133 sequences) |
| `SnippetTools` | Manage `.wt.json` per-directory snippets and `sendInput` actions |

**Helper classes** (`Helpers/`) — Stateless utility logic shared across tools:

| Class | Purpose |
|---|---|
| `SettingsHelper` | Load/patch/diff settings.json; deterministic GUID generation (UUID v5) |
| `FragmentHelper` | Fragment file I/O across user-scope and machine-scope directories |
| `WtJsonHelper` | Find/read/write `.wt.json` snippet files with parent-directory walk |
| `OhMyPoshHelper` | Locate Oh My Posh executables, themes, and config from PS profile |
| `SchemaValidator` | Structural validation of fragment JSON (profiles, schemes, actions) |
| `ProcessHelper` | Run subprocesses with timeout + async I/O to avoid deadlocks |
| `TerminalRelease` | Enum + extensions for Terminal release channels (Stable/Preview/Canary/Dev) |
| `WtJsonHelper` | Snippet file discovery using parent-directory walk matching Terminal's behavior |

## Key Conventions

- **Preview-then-apply pattern**: All mutating tools follow a two-step flow — a `Preview*` method returns a unified diff or validation result, then the corresponding `Apply*`/`Create*`/`Update*` method writes the change. The preview must always be shown to the user first.
- **JSON Patch (RFC 6902)**: Settings and Oh My Posh config modifications use JSON Patch via the `JsonPatch.Net` library, not direct JSON manipulation.
- **Backup before write**: All write operations create a `.bak` copy of the original file before overwriting.
- **Release channel resolution**: When no release channel is specified, the server auto-detects the most-preview installed channel (Dev > Canary > Preview > Stable).
- **No namespaces**: All types are in the global namespace — no `namespace` declarations.
- **Implicit usings + nullable enabled**: The project uses `<ImplicitUsings>enable</ImplicitUsings>` and `<Nullable>enable</Nullable>`.
- **Logging to stderr**: All logging goes to stderr because stdout is reserved for MCP protocol messages.
- **Static classes**: Tool and helper classes are entirely static with no instance state.
- **Tool descriptions**: `[Description(...)]` attributes on tools and parameters are critical — they're surfaced to the AI client to guide tool usage. Use triple-quoted raw string literals for multi-line descriptions.

---
> Source: [carlos-zamora/wt-mcp](https://github.com/carlos-zamora/wt-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
