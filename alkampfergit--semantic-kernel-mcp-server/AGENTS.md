# Copilot/Agent Instructions — Semantic Kernel MCP Server

Purpose: Fast-orient AI coding agents (Copilot/agent) to be productive in this repository.

Summary (big picture):
- This repo implements an MCP (Model Context Protocol) server using the C# MCP SDK and integrates Microsoft Semantic Kernel. The server runs as a stdio tool and exposes tools (MCP endpoints) that clients (e.g., VS Code Copilot) can call.
- Major components:
  - `SemanticKernelMcpServer/Program.cs`: MCP server host configuration, registers tools and stdio transport.
  - `SemanticKernelMcpServer/Tools/`: MCP tools. Key files:
    - `SemanticKernelAgentTools.cs`: Agent-based tools using Semantic Kernel (ChatCompletionAgent).
    - `RandomNumberTools.cs`: simple sample tool (non-AI).
  - `SemanticKernelMcpServer/Plugins/TaskManagementPlugin.cs`: Semantic Kernel plugin with `[KernelFunction]` methods for CRUD operations on in-memory tasks.
  - `.mcp/server.json` + `.vscode/mcp.json`: server metadata and VS Code config (environment variables and runtime args).

What to know first (immediate priorities):
- `Program.cs` adds the server and registers tools: `.AddMcpServer().WithStdioServerTransport().WithTools<...>()`.
- Tools are annotated with `[McpServerTool]`. These methods are published to MCP clients.
- The agent uses `ChatCompletionAgent` and `Kernel` (Semantic Kernel). Plugins are registered via `builder.Plugins.AddFromType<T>("PluginName")`.
- Kernel functions are annotated with `[KernelFunction]` and are available for function calling. Example: `TaskManagementPlugin.CreateTask`.

Development workflows & commands agents should use:
- Build:
  - `dotnet build` (solution or project folder `SemanticKernelMcpServer`).
- Run locally for testing (stdio MCP server):
  - `dotnet run --project SemanticKernelMcpServer/SemanticKernelMcpServer.csproj`.
- VS Code (developer experience):
  - Update `.vscode/mcp.json` if using a local build; set environment variables for AI if needed.
  - Use GitHub Copilot Agent Mode and `Select tools` to see the available MCP tools.
- Package & Publish (optional):
  - `dotnet pack -c Release` to produce `.nupkg` files.
  - `dotnet nuget push bin/Release/*.nupkg --api-key <key> --source https://api.nuget.org/v3/index.json`.

Key repo-specific conventions & patterns:
- Tools: Methods annotated with `[McpServerTool]` are canonical MCP entry points. Keep them high-level and thin; routing to the plugin or agent inside the method.
- Plugins: `TaskManagementPlugin` demonstrates the plugin pattern (kernel functions) that is safe for tool calling. Prefer registering plugins with `builder.Plugins.AddFromType<T>()`.
- AI Service Configuration: The agent is created in `SemanticKernelAgentTools()`; the `Kernel` builder needs `AddAzureOpenAIChatCompletion()` or `AddOpenAIChatCompletion()` to enable AI features. If those are not set, the agent will return a friendly message and fall back to direct tools.
- Function Calling Behavior: The `OpenAIPromptExecutionSettings` with `FunctionChoiceBehavior.Auto()` is used to auto-invoke kernel functions; keep argument shapes and descriptions clear.

Integration points & environment variables:
- `.mcp/server.json` declares environment variables for `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_DEPLOYMENT`, and `AZURE_OPENAI_API_KEY`. These map to the runtime environment for the MCP server for enabling AI features.
- `.vscode/mcp.json` contains a `SemanticKernelMcpServer` configuration pointing to `dotnet run` and environment placeholders.

Testing & debugging guidance:
- When the server is running, logs go to stderr (stdout is reserved for MCP protocol messages). `Program.cs` configures logging to send console logs to stderr.
- Reprocamp: If build fails due to app lock, stop running server process (`Stop-Process` on Windows) — we saw this lock during build earlier.
- No unit tests yet — CI runs `dotnet test` but doesn't fail the build if tests are missing. If adding tests, include them under a `tests/` project and reference the solution.

Patterns to follow when editing code:
- Register new tools under `Program.cs` with `WithTools<ToolClass>()`.
- Expose behavior to agents via `TaskManagementPlugin` by annotating with `[KernelFunction]`.
- Make sure public tool method signatures are documented using `[Description]` and parameter descriptions so agents know how to use them.
- Avoid modifying MCP transport or protocol logic unless necessary — the `ModelContextProtocol` library handles message framing; prefer adding tools and functions.

Important TODOs an agent may perform (examples):
- Add new kernel plugin that integrates with a data store (replace in-memory `List<TaskItem>` with persistence; update `TaskManagementPlugin` to use DI for storage).
- Add AI service configuration with environment-driven secrets in `.vscode/mcp.json` and documentation.
- Implement unit tests and add `dotnet test` expectations in CI.
- Add proper error handling and logging to tools that call agent methods.

Where to look for examples:
- Native MCP tool example: `SemanticKernelMcpServer/Tools/RandomNumberTools.cs`.
- AI agent example: `SemanticKernelMcpServer/Tools/SemanticKernelAgentTools.cs`.
- Plugin example with kernel functions & outputs: `SemanticKernelMcpServer/Plugins/TaskManagementPlugin.cs`.

File references:
- `SemanticKernelMcpServer/Program.cs` — server bootstrap & tool registration.
- `SemanticKernelMcpServer/Tools/SemanticKernelAgentTools.cs` — how to build `Kernel`, create `ChatCompletionAgent`, and call agent.
- `SemanticKernelMcpServer/Plugins/TaskManagementPlugin.cs` — pattern for kernel functions (`[KernelFunction]`).
- `.mcp/server.json` & `.vscode/mcp.json` — MCP metadata and VS Code integration config.

If you need help performing an operation, suggest this:
- For code changes: create a focused PR describing the intent and include examples of expected MCP responses and how to test them locally via `dotnet run` and VS Code Agent Mode.
- For adding AI configuration: add `AddAzureOpenAIChatCompletion(...)` calls to `Kernel` builder and update `.vscode/mcp.json` environment variables.

If something is ambiguous or missing here, ask which area to expand (run patterns, plugin conventions, test wiring, or CI updates).  

---

Now I'll commit this file and push it to the repository. (I can further refine if you want more detail or examples.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alkampfergit)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alkampfergit)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
