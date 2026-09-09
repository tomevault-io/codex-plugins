# Copilot Instructions

## Build, test, and lint commands

Run commands from the repository root (`/skillssss`).

- Install tooling:
  `npm ci`
- Lint all Markdown:
  `npm run lint`
- Lint one file:
  `npx markdownlint-cli2 path/to/file.md`
- Run one CI-style structure check:

  ```bash
  for dir in architecture backend frontend fullstack flutter deploy blockchain; do
    [ -d "$dir" ] || { echo "Missing directory: $dir"; exit 1; }
  done
  ```

There is no application build command and no unit/integration test suite in
`package.json`. Validation is prompt/document checks plus Markdown linting.

## High-level architecture

This repository is a multi-agent prompt/template system, not a runnable app.
Execution flow is phase-based:

```text
architecture/ -> (frontend/ + backend/ + flutter/ + blockchain/)
               -> fullstack/ -> deploy/
Phase 0           Phase 1 (parallel)            Phase 2    Phase 3
```

Cross-phase handoff:

1. `architecture/` creates design artifacts consumed by downstream phases:
   `tech-stack.md`, `data-architecture.md`, `infra-architecture.md`,
   `security-architecture.md`, `api-contract.md`, `ui-ux-architecture.md`,
   `implementation-roadmap.md`.
2. `frontend/`, `backend/`, `flutter/`, and `blockchain/` run independent
   planner/dev/tester loops against those artifacts.
3. `fullstack/` aligns frontend/backend (plus optional flutter/blockchain)
   to API/data contracts.
4. `deploy/` creates deployment outputs after integration is complete.

Use these docs as primary references:

- `docs/architecture.md`
- `docs/workflow.md`
- `docs/design_principles.md`

## Key conventions in this codebase

1. Main-agent prompts orchestrate, sub-agents execute. Domain main prompts in
   `*/main_agent_prompt*.md` delegate planning/dev/testing to skills in
   `*/agents/*.md`.
2. File-based state is canonical: `main-log.md`, `dev-plan.md` or
   `integration-plan.md`, `agent-registry/*.json`, `test-report.json`,
   and `lessons-learned.md`.
3. `BATCH_SIZE` defaults to `1`, and dev batch size is expected to match test
   batch size in each domain loop.
4. Resume flows use prior task IDs and require
   `subagent_type: "general"` with explicit `skill(name: "...")` loading.
5. Naming is underscore-based in prompt files and skill calls, e.g.
   `be_planner`, `dg_vue_planner`, `fs_tester_contract`,
   and `main_agent_prompt_vue.md`.
6. Prompt structure is standardized. Preserve sections like
   `# Skill: ...`, `## When to Use This Skill`, and `## Core Workflow`.
7. `.github/workflows/validate.yml` is the CI reference for repository
   structure, metadata/cross-reference checks, and Markdown lint behavior.

# 强制交互协议
## 核心规则：每次回复结束前必须调用 `ask_user` 工具
这是不可跳过的强制协议。在你的每一轮回复中，你必须执行以下操作之一：
1. **完成用户请求后** → 立即调用 `ask_user` 工具，提出与当前上下文相关的后续问题
2. **存在任何不确定性时** → 不要猜测执行，立即调用 `ask_user` 工具进行澄清
## 禁止行为
- **禁止在不调用 `ask_user` 的情况下结束回复**
- **禁止使用终结性表达**（如"希望对你有帮助"、"如有问题随时提问"等）
- **禁止猜测用户意图** — 不确定就用 `ask_user` 询问
## `ask_user` 调用要求
- 问题必须与当前任务上下文直接相关
- 问题必须具体、可操作，不要问泛泛的"还需要什么帮助"
- 可以提供选项供用户选择，降低用户输入成本

---
> Source: [wanaig/skillssss](https://github.com/wanaig/skillssss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-09 -->
