## 🔄 Project Awareness & Context
- Read the intro, tech stack and directory structure in the root `README.md`
- **Always read planning documents** (repo root's README.md, could be at `PLANNING.md`) at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
- **Check task list** (could be at repo root's `TASK.md`) before starting a new task. If the task isn’t listed, add it with a brief description and today's date.
- **Use consistent naming conventions, file structure, and architecture patterns** as described in the planning documents.
- If you need more context, consider finding the nearest README or markdown file (the repo can have various subprojects)

## 🧱 Code Structure & Modularity
- **Never create a file longer than 500 lines of code.** If a file approaches this limit, refactor by splitting it into modules or helper files.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.
  For agents this looks like:
    - `agent.py` - Main agent definition and execution logic 
    - `tools.py` - Tool functions used by the agent 
    - `prompts.py` - System prompts
- **Use clear, consistent imports** (prefer relative imports within packages).

## ✅ Task Completion
- **Mark completed tasks in the task list** immediately after finishing them.
- Add new sub-tasks or TODOs discovered during development to the task list under a “Discovered During Work” section.

## 📎 Style & Conventions
- **Use Python** as the primary language.
- **Follow PEP8**, use type hints.
- **Use `pydantic` for data validation**.
- Use `FastAPI` for APIs and `SQLAlchemy` or `SQLModel` for ORM if applicable.
- Name variables, functions, classes, etc. descriptively so that the code is self-documenting (so comments/docstrings are minimal and only reserved for non-obvious things)
- When writing prompts, make sure they are declared on its own, rather within objects or functions.

## 📚 Documentation & Explainability
- **Update docs** when new features are added, dependencies change, or setup steps are modified.
- **Comment non-obvious code** and ensure everything is understandable to a mid-level developer.
- When writing complex logic, **add an inline `# Reason:` comment** explaining the why, not just the what.
- Always explain your thought process and rules
- During planning, if not explicitly instructed that existing code should be modified, compare using/modifying existing code vs building from scratch, and explain why you chose one over the other.

## 🧠 AI Behavior Rules
- **Never assume missing context. Ask questions if uncertain.**
- **Never hallucinate libraries or functions** – only use known, verified Python packages.
- **Always confirm file paths and module names** exist before referencing them in code or tests.
- **Never delete or overwrite existing code** unless explicitly instructed to or if part of a task from the task list.
- When I have comments in the code, even if its for TODO, do not delete them. If you think they should be removed, tell me separately instead of editing it

### Warning on going out of scope
- Implement only what is necessary as requested by the user (e.g. via messages or Github issue), **no extra functionality** that wasnt asked for!
- If you think extra functionality/enhancements are needed, **ask for clarification** with the user via how you got the instruction (e.g. messages or Github issue)
- IMPORTANT: We are doing this because
    - It is very easy to overengineer and suffer from code bloat 
    - **There are other maintainers on this repo and editing things you were not assigned could cause merge conflicts!**

## Communication
- Efficient communication: when replying to a request or writing docs, be concise. 
	- If it can fit within a post-it, do so instead of writing a lengthy report.

## 🧪 Testing & Reliability
- **Always create Pytest tests for new features** (functions, classes, routes, etc).
- **After updating any logic**, check whether existing tests need to be updated. If so, do it.
- **Tests should live in `/tests` folder** mirroring the main app structure (see `tests/README.md`).
  - Include at least:
    - 1 test for expected use
    - 1 edge case
    - 1 failure case
- Sometimes, if the objects require things you don't have (like API keys), then write mocks
- See the `testing_guide.md` for more details

## Iterating
If you fail to accomplish the instructions you were given, 
- think of alternate solutions and try a few of them
- If available, consider using the perplexity search tool to obtain external info
- Document the attempts and failures and report them back to me. Failure is ok as long as you try intelligently
- **IMPORTANT**: Regularly commit and **push** your changes to prevent losing work, especially at the end!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicholaschenai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nicholaschenai)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
