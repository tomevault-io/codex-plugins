
# AGENT-FORGE — Output Format Specification

This file guides the **GitHub Copilot CLI** (the generation engine) to produce artifacts in the correct format. All generated files are **VS Code-compatible** and also work in GitHub Copilot CLI at runtime.

## Agent File Format (`.agent.md`)

```yaml
---
name: "Display Name"                    # REQUIRED
description: "What this agent does"     # REQUIRED
argument-hint: "[task] [details]"       # Recommended — placeholder in chat input
tools:                                  # Array of tool aliases (see Tool Aliases below)
  - read                                #   Read file contents (aliases: Read, NotebookRead)
  - edit                                #   Modify files (aliases: Edit, MultiEdit, Write, NotebookEdit)
  - search                              #   Grep/glob search (aliases: Grep, Glob)
  - execute                             #   Shell/terminal commands (aliases: shell, Bash, powershell, run_in_terminal)
  - agent                               #   Invoke sub-agents (aliases: custom-agent, Task)
  - web                                 #   Fetch URLs (aliases: WebSearch, WebFetch)
  - todo                                #   Task lists (aliases: TodoWrite)
agents:                                 # List of allowed subagent names (orchestrators only)
  - "researcher"                        #   Only these agents can be invoked as subagents
  - "implementer"                       #   Use '*' to allow all, '[]' to prevent any
model: "Claude Sonnet 4.5 (copilot)"   # Optional: single model or prioritized array
# model:                                # Array form — tries each in order until available:
#   - "Claude Sonnet 4.5 (copilot)"
#   - "Gemini 3 Flash (Preview) (copilot)"
user-invokable: true                    # Show in agent dropdown (default: true)
disable-model-invocation: false         # Prevent auto-delegation (default: false)
target: "vscode"                        # Optional: "vscode" or "github-copilot" (both if omitted)
handoffs:                               # Multi-agent workflow transitions (flat pattern)
  - label: "Hand off to Backend"
    agent: "express"
    prompt: "Continue with backend work"
    send: false
---

Body: markdown instructions for the agent.
```

### Subagent Properties

| Property | Purpose | Default |
|----------|---------|---------|
| `agents` | List of allowed subagent names for this agent. Use `'*'` to allow all, `[]` to prevent any. Only meaningful when `agent` tool is included. | `'*'` (all) |
| `model` | AI model for this agent. String or prioritized array. Useful for cost-efficient subagents. | Inherits from session |
| `user-invokable` | Whether agent appears in the agents dropdown. Set `false` for subagent-only agents. | `true` |
| `disable-model-invocation` | Whether to prevent other agents from auto-invoking this as a subagent. Set `true` for orchestrators (user-invoked only). | `false` |

### Tool Aliases Reference

| Alias | Platform Equivalents | Description |
|-------|---------------------|-------------|
| `execute` | shell, Bash, powershell | Execute shell commands |
| `read` | Read, NotebookRead | Read file contents |
| `edit` | Edit, MultiEdit, Write, NotebookEdit | Modify files |
| `search` | Grep, Glob | Search files or text |
| `agent` | custom-agent, Task | Invoke other agents |
| `web` | WebSearch, WebFetch | Fetch URLs, web search |
| `todo` | TodoWrite | Create/manage task lists |
| `github/*` | — | GitHub MCP server tools |
| `playwright/*` | — | Playwright MCP server tools |

Unrecognized tool names are ignored, enabling cross-product compatibility.

## Custom Instructions Discovery

At runtime, GitHub Copilot (in VS Code and Copilot CLI) discovers and loads instructions from these locations:

| Location | Scope |
|----------|-------|
| `~/.copilot/copilot-instructions.md` | Global (all sessions) |
| `.github/copilot-instructions.md` | Repository |
| `.github/instructions/**/*.instructions.md` | Repository (modular) |
| `AGENTS.md` (Git root or cwd) | Repository |
| `Copilot.md`, `GEMINI.md`, `CODEX.md` | Repository |

Repository instructions take precedence over global instructions.

## Instruction File Format (`.instructions.md`)

```yaml
---
name: "instruction-slug"               # REQUIRED
description: "What standards are enforced"  # REQUIRED
applyTo: "**/*.{ts,tsx,js,jsx}"        # REQUIRED — glob for auto-loading
---

Body: rules grouped under ## headings, bullet points with reasoning.
```

**applyTo must be specific** — `**/*` wastes context. Use file-type patterns.

## Skill File Format (`SKILL.md`)

```yaml
---
name: "skill-slug"                      # REQUIRED — must match parent directory name
description: "Domain knowledge. USE FOR: 5+ trigger phrases. DO NOT USE FOR: 3+ exclusions."  # REQUIRED (1-1024 chars)
argument-hint: "[topic or context]"       # Hint shown in chat input for /slash command
user-invokable: true                      # Show in /slash menu (default: true)
disable-model-invocation: false           # Allow auto-loading (default: false)
license: "MIT"                            # Optional license
compatibility: "Requires Node.js 18+"    # Optional environment requirements (1-500 chars)
---

Body: category-appropriate structure. Keep <500 lines.
- SDK/Library: Core Concepts → Quick Start → Common Patterns → API Reference → Pitfalls
- Framework: Architecture → Project Structure → Conventions → Decision Tree → Patterns → Pitfalls
- Service/Infra: Overview → Configuration → Deployment → Troubleshooting
- Workflow: Overview → Step-by-Step → Decision Tree → Checklist → Examples
```

**Skill description controls on-demand loading.** Always include `USE FOR:` and `DO NOT USE FOR:` trigger phrases. Write descriptions slightly "pushy" — skills tend to under-trigger. `name` MUST match the parent directory name or VS Code won't load the skill.

Progressive disclosure: Level 1 (frontmatter metadata — always in context) → Level 2 (SKILL.md body — loaded when relevant) → Level 3 (files in `references/`, `scripts/`, `assets/` — loaded only when referenced). Split large knowledge into `references/` subdirectory files.

## Prompt File Format (`.prompt.md`)

```yaml
---
name: "prompt-slug"                     # REQUIRED — becomes /slash command
description: "What this command does"   # REQUIRED
agent: "agent-name"                     # Route to specific agent
argument-hint: "[task description]"
---

Body: task template with ${input:name:placeholder}, ${selection}, ${file} variables.
```

## Hook Config Format (`.json`)

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [{ "type": "command", "bash": "./script.sh", "powershell": "./script.ps1", "timeoutSec": 10 }],
    "postToolUse": [{ "type": "command", "bash": "./script.sh", "powershell": "./script.ps1", "timeoutSec": 15 }]
  }
}
```

Hook files MUST include `"version": 1`. Use `"bash"`/`"powershell"` keys (not `"command"`). Use `"timeoutSec"` (not `"timeout"`).

Events: `sessionStart`, `sessionEnd`, `userPromptSubmitted`, `preToolUse`, `postToolUse`, `errorOccurred`, `subagentStop`, `agentStop`

## MCP Server Config Format (`.vscode/mcp.json`)

```json
{
  "servers": {
    "github": { "type": "http", "url": "https://api.githubcopilot.com/mcp" },
    "custom": { "command": "npx", "args": ["-y", "package-name"], "env": { "KEY": "${input:API_KEY}" } }
  }
}
```

## Quality Rules

1. **No generic filler** — "follow best practices" is not a responsibility. Name specific patterns.
2. **Tech-specific standards** — every Technical Standard must reference actual framework APIs, patterns, or conventions.
3. **Minimum 4 responsibilities** per agent, each tied to the specific tech stack.
4. **Opening line** must name the technology: "You are the **React Specialist** — ..." not "You are the Agent..."
5. **applyTo must be specific** — `**/*.{tsx,jsx}` not `**/*`.
6. **Skill descriptions** must have ≥5 `USE FOR` and ≥3 `DO NOT USE FOR` trigger phrases.
7. **No duplicate content** — instructions codify standards, skills provide knowledge, agents define behavior. Don't repeat across them.
8. All agents that build/test code must include `execute` (or `run_in_terminal`) in tools.
9. **Orchestrator agents** must have `agents` property, `agent` tool, and MUST NOT have `edit` or `execute` tools.
10. **Subagent agents** must have `user-invokable: false` and should be listed in their orchestrator's `agents` array.

## Orchestration Patterns

When generating multi-agent systems with 3+ agents, choose one of these patterns:

### Coordinator-Worker Pattern

A coordinator agent manages the overall task and delegates to specialized workers. Each worker has a tailored toolset.

```yaml
# Orchestrator
---
name: "Feature Builder"
tools: ['read', 'search', 'agent', 'todo']     # NO edit/execute — delegates everything
agents: ['researcher', 'implementer', 'reviewer']
user-invokable: true
disable-model-invocation: true
---
# Body: decompose → delegate → validate → iterate
```

```yaml
# Worker subagent
---
name: "Implementer"
tools: ['read', 'edit', 'search', 'execute']
user-invokable: false                           # Only invoked by orchestrator
model: ['Claude Sonnet 4.5 (copilot)', 'Gemini 3 Flash (Preview) (copilot)']
---
# Body: focused expertise, structured output
```

### Multi-Perspective Review Pattern

Run multiple review perspectives in parallel for unbiased, comprehensive analysis.

```yaml
---
name: "Thorough Reviewer"
tools: ['read', 'search', 'agent']
agents: ['security-reviewer', 'quality-reviewer', 'arch-reviewer']
---
# Body: launch reviewers in parallel, synthesize findings
```

### Pipeline Pattern

Sequential stages where each agent's output feeds the next.

```yaml
---
name: "TDD Coordinator"
tools: ['read', 'search', 'agent', 'todo']
agents: ['red', 'green', 'refactor']
---
# Body: 1. Red (failing tests) → 2. Green (pass tests) → 3. Refactor (improve)
```

### When to Use Each Pattern

| Pattern | Use When | Don't Use When |
|---------|----------|----------------|
| **flat** (handoffs) | 1-2 agents, simple workflows | Complex multi-step tasks |
| **coordinator-worker** | 3+ agents, research→implement→review | Simple independent agents |
| **multi-perspective** | Parallel independent analysis | Sequential dependencies |
| **pipeline** | Clear sequential stages (TDD, CI/CD) | Independent tasks |

---
> Source: [microsoft/agent-forge](https://github.com/microsoft/agent-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
