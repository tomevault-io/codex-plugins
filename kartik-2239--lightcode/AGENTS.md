# Copilot Instructions

## Build and test commands

- Build the main binary: `go build ./cmd/lightcode` or `make build`
- Run the app: `go run ./cmd/lightcode/main.go`
- Run the full test suite: `go test ./...`
- Run the existing focused test file: `go test ./internal/server/tools`
- Run a single test: `go test ./internal/server/tools -run TestSkill_BasicSkill`

There is no repo-defined lint target or linter config in this repository. Do not invent one in a Copilot session.

`go test ./...` is currently not green: `internal/server/tools/skill_test.go` expects `SKILL_PATH`-based behavior, but `internal/server/tools/skill.go` resolves the skills directory through `config.SkillsPath()` from `~/.lightcode/config.json`.

## High-level architecture

`cmd/lightcode/main.go` is the only real entrypoint. By default it starts the local HTTP server and then launches the Bubble Tea TUI; `-server` runs only the server and `-tui` runs only the TUI.

The app is split into two halves that communicate over localhost HTTP instead of direct package calls:

- `internal/server` exposes endpoints for session CRUD, message history, streaming chat completions, and the current todo list.
- `internal/tui` is the Bubble Tea frontend. It talks to the server through `internal/tui/client`, including streaming assistant responses from `/chat-completion`.

Persistent app state lives under `~/.lightcode/`:

- `config.json` stores theme, port, providers, skills path, and the currently selected model.
- `lightcode.db` is a SQLite database managed through GORM.
- `skills/` holds user skills that the `skill` tool loads.

The agent loop is in `internal/server/agent/agent.go`. Each iteration reads the session's stored messages from SQLite, converts them into `llm.Chat` messages, appends the session todo list as an extra user message, calls the model, persists the assistant reply, executes any tool calls, persists tool outputs, and repeats until the model returns plain assistant text or the iteration limit is hit.

Mode selection is a real architectural concept, not just UI state. The TUI cycles between `chat`, `plan`, and `assistant`, sends the selected mode to the server, and `internal/server/llm/client.go` chooses a different system prompt for each mode.

## Key conventions

- Sessions are the main unit of state. A session stores its title, working directory, and serialized todo list in the `sessions` table; individual conversation turns are stored separately in `messages`.
- Message payloads are stored as JSON strings in `models.Message.Data`, using `models.StoredMessageData`. When replaying history to the LLM, stored `tool_call` messages are reintroduced as synthetic user messages that look like `Tool "<name>" (call_id=<id>) output: ...`.
- Tool registration is side-effect driven. Every tool registers itself from `init()` in `internal/server/tools/*`, and `tools.GetToolsForChat()` exposes the whole registry to the model.
- File-system tools are intentionally scoped to the session working directory. Reuse `tools.ValidatePath` for path-based reads/edits instead of adding ad hoc path handling.
- Skills are external content, not repository files. The `skill` tool loads `~/.lightcode/skills/<skillName>/SKILL.md`, strips frontmatter from the main skill body, and also embeds any sibling files from that skill directory inside a `<skill_files>` block.
- The TUI command palette includes `/sessions`, `/new_session`, `/delete_session`, and `/models`; model selection updates `CurrentModel` in `~/.lightcode/config.json`.
- The TUI does not call the LLM packages directly. New user-facing behavior should usually flow through the HTTP server and the persisted session/message model, not by bypassing the server from the UI.

---
> Source: [Kartik-2239/lightcode](https://github.com/Kartik-2239/lightcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-09 -->
