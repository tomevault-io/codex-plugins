# Copilot Instructions for Max

## Tool Naming

**Do NOT create tools that share a name with built-in Copilot CLI tools.** Copilot CLI reserves certain tool names (e.g. `list_agents`, `read_agent`, `write_agent`, `task`, `bash`, `grep`, `glob`, `view`, `edit`, `create`, etc.). If Max defines a tool with the same name, it will conflict at runtime.

When naming custom tools, use unique prefixes or alternative verbs to avoid collisions. For example:
- ~~`list_agents`~~ → `show_agent_roster`
- ~~`read_agent`~~ → `get_agent_result` (already done)

If you encounter a "conflicts with built-in tool" error, rename the offending tool and update all references in `src/copilot/tools.ts`, `src/copilot/agents.ts`, and `src/copilot/system-message.ts`.

---
> Source: [burkeholland/max](https://github.com/burkeholland/max) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
