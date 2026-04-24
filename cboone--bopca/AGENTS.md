# GitHub Copilot Instructions

## PR Review Checklist (CRITICAL)

<!-- KEEP THIS SECTION UNDER 4000 CHARS - Copilot only reads first ~4000 -->

### Known Patterns - Do Not Flag

**Verify source before flagging**: Line numbers change; check actual code.

- **Agent registration**: All agents registered in `builtin.go:init()` - verify before claiming unregistered
- **Interface signatures**: Verify interfaces in source; plan docs show design history, not final code
- **Containerfile markers**: `AGENT_INSTALL_MARKER` matches exactly - verify before claiming mismatch
- **output.Name()**: Styles identifiers including display names per output.go docs
- **macOS 26 (Tahoe)**: Valid future version (today is 2026)
- **Go 1.23.0 format**: Patch version in go.mod is valid
- **install.sh checks**: Writability checked BEFORE choosing directories
- **macOS-only testing**: Tests designed for macOS, not cross-platform CI; `sh` is always available, `TestHelperProcess` pattern unnecessary
- **Container TTY scripts**: Scripts in `config/` like `xclip-osc52.sh` run inside interactive container sessions where `/dev/tty` is always available; do not flag unconditional `/dev/tty` writes
- **EnhancedMockRunner setup**: `SetupContainerServiceRunning()` etc. configure mock state
- **Cobra help output**: `bopca help` vs `bopca --help` differ intentionally
- **Dry-run skips CheckRequirements**: Commands that use `skipRequirements` annotation and call `CheckRequirements()` in their own `RunE` intentionally skip it during dry-run because `CheckRequirements` uses real system checks (`lookupPath`) not mediated by the Runner interface, so `DryRunRunner` cannot intercept them
- **Containerfile sudo**: The `dev` user has passwordless sudo configured (`NOPASSWD:ALL` in `/etc/sudoers.d/dev`) with `Defaults:dev !env_reset` so `sudo` preserves environment variables for the dev user. The `!env_reset` is intentionally scoped to `dev` (not global) and must remain `!env_reset` (not `setenv`), because dev's sudo calls inside the container need automatic env preservation without requiring `-E`. Do not flag `sudo` commands in `RUN` steps under `USER dev` as potentially failing; sudo works without a TTY or password prompt in this container. Do not suggest further scoping or changing `!env_reset` to `setenv`
- **YOLO prompt default**: `defaultPromptYOLO` intentionally defaults to yes (enabled) for both empty input and read errors. This is a deliberate design choice; do not suggest defaulting to no for non-interactive stdin or adding TTY checks that change the default behavior
- **eval BOPCA_YOLO_INIT**: The `BOPCA_YOLO_INIT` env var is set exclusively by bopca's own Go code (`internal/container/args.go`) from `agent.YOLOShellInit()`, not from untrusted input. It injects shell aliases for YOLO mode in a controlled container environment. Do not flag this eval as a security risk or suggest file-based alternatives
- **Apple build flags**: `--memory`/`--cpus` valid for `container build`
- **Shell script naming**: `log_*` vs `print_*` intentional per-script
- **Code blocks in docs**: `||`, `|` in fenced code blocks are operators, not table syntax
- **Markdown tables in docs**: Tables in documentation files use standard single-pipe syntax (`| Column | ... |`). Do not flag these as having double leading pipes (`||`); verify the raw file content before reporting table syntax issues
- **Cobra command sorting**: `rootCmd.Commands()` sorts alphabetically by default via `commandSorterByName`; do not suggest explicit sorting of command lists obtained from `Commands()`
- **Status line test patterns**: In `statusline_test.go`, `ReplaceStdout(&buf)` prevents Bubble Tea from writing to real stdout; it is not meant to capture logging output. Tests assert on `isStatusLineActive()` to verify behavioral contracts, not on buffer contents. `stopStatusLine()` blocks on the `done` channel (with timeout) before returning, so assertions after it are safe. The `defer restore()` before `t.Cleanup` ordering is an intentional, consistent pattern across all status line tests
- **Pre-v1 breaking CLI changes**: This project does not preserve backwards compatibility until v1. Removing, renaming, or changing CLI flags is acceptable and intentional. Do not flag flag removal as a breaking change or suggest reintroducing removed flags for backwards compatibility
- **Commit messages are historical records**: PR descriptions list commit messages reflecting the code state when each commit was made. Later commits may change the implementation (e.g., removing a flag that an earlier commit tested). Do not flag commit messages or PR descriptions as inconsistent with the final code state; they document the development history, not the final result
- **Plan documents in docs/plans/**: Documents under `docs/plans/todo/` are proposed plans not yet implemented. The directory path communicates the document's status. Do not suggest adding "Not Yet Implemented" labels or status banners to plan documents
- **Scrut test expected output**: Scrut tests match actual command output verbatim. When expected output contains unusual syntax (e.g., `./"` vs `.\"`), verify against real command output before flagging as a typo; the test reflects what the tool actually produces
- **`cd --` in container shell scripts**: BusyBox ash (Alpine `/bin/sh`) supports `cd --` for end-of-options. Do not claim `cd --` is unsupported on BusyBox ash or non-POSIX; it works correctly in the Alpine container environment

### Logging Levels and Output Visibility

- **Default mode output visibility**: In default mode (no verbosity flags), the terminal stdout logger threshold is `DefaultLevel` (3). Only DryRun (3), Warn (4), Error (8), and Fatal levels pass through. `output.Error()` (level 8) prints to stderr; `output.Info()` (level 0) and `output.Status()` (level 2) are suppressed. When reviewing scrut tests, verify expected output against these thresholds before claiming test expectations are out of sync with command behavior

### Pre-parse Argument Scanning

- **`hasQuietArg` verbose detection**: The string-based `--verbose=N` check (`val != "0"`) is intentional. This function runs before Cobra/pflag parsing to decide early output suppression. Using `strconv.Atoi` would be wrong: invalid values like `--verbose=abc` would parse as 0, incorrectly allowing quiet to suppress output before Cobra can report the error. The `-v=N` form is not handled because pflag's `CountVarP` flags do not accept `=` syntax. Do not suggest `Atoi`-based parsing or `-v=N` handling for pre-parse heuristics.
- **`hasQuietArg` sticky verbose boolean**: The `hasVerbose` flag is intentionally sticky (once true, never reset). Edge cases like `--verbose=0` resetting verbosity are handled by the real parser (`applyVerbosityFlags`) after full Cobra/pflag parsing. The pre-parse heuristic errs on the side of not suppressing output, which is the safe default. Do not suggest tracking effective verbosity values or resetting `hasVerbose` on `--verbose=0`.

### Gitleaks Configuration

- **Path allowlists for test fixtures**: `.gitleaks.toml` uses path-based allowlists (not regex stopwords) for test files containing fake secrets. This is intentional because scanner test files need realistic secret patterns that change over time; regex allowlists would be fragile and require updating with every test change. Do not suggest replacing path allowlists with regex-based allowlists for test fixture files.

### Go Code Conventions

- **pflag Value sharing via BoolVar/StringVar**: When two flags are registered with the same Go variable (e.g., `BoolVar(&useWorktrees, "worktrees", ...)` and `BoolVar(&useWorktrees, "worktree", ...)`), pflag casts the variable pointer directly to its value type (e.g., `(*boolValue)(p)`). Both flags receive the identical `pflag.Value` pointer. Comparing `alias.Value != primary.Value` is a valid and reliable test that two flags share the same backing variable. Do not suggest behavioral assertions or `Var(primary.Value, ...)` as alternatives
- **`collectAgentPathEntries` sorting**: Alphabetical sorting of agent PATH entries is intentional for determinism in both Containerfile generation and recipe hash computation. Agent bin directories contain non-overlapping binaries, so PATH precedence between them is not a concern; stable ordering for reproducible builds is the priority
- **go.mod indirect deps**: Indirect dependency version changes from `go mod tidy` are expected when adding/removing direct dependencies; do not flag as unrelated churn
- **Go 1.25**: Project standard
- **Proper nouns**: "Apple Silicon" capitalized in errors
- **Warning messages**: Not error strings - standard capitalization
- **Short-circuit**: `obj != nil && obj.Method()` is safe
- **filepath.Join**: Handles leading slashes correctly
- **Local CLI trust model**: bopca is a local CLI tool. Config values and paths come exclusively from hardcoded constants and local config files controlled by the user who already has full filesystem access. Do not flag path traversal or directory escape concerns for these values; there is no untrusted external input channel
- **Logger level in Init()**: `Init()` intentionally does not set the terminal log level; `SetLevel()` is called separately after CLI flag resolution
- **Global logger mutation in constructors**: Silencing third-party loggers (e.g., gitleaks' zerolog) in `New*()` constructors is intentional when the constructor is called once in production; do not suggest `sync.Once` or `init()` for single-call constructors
- **Exported vars in internal packages**: Exported mutable slices (e.g., `DefaultPrefixes`) in `internal/` packages are intentional; callers are all within the same project and the simple API is preferred over getter/copy patterns
- **Config warning suppression timing**: `loadFromFile()` suppresses validation warnings when the config file sets quiet mode. Validation warnings that fire during `loadFromFile()` are non-critical default-correction messages, not errors. After `config.Load()` returns, `cmd/root.go` calls `applyVerbosityFlags()` then `UnsuppressWarnings()` if CLI verbosity/debug flags override quiet. Do not suggest tri-state suppression, deferred validation, or a Load option for verbosity awareness
- **Config warning buffer and suppression**: When `suppressWarnings` is true, `warnConfig()` intentionally skips both terminal output AND `warningBuffer` appending. Suppression means "skip entirely" (quiet mode or duplicate-prevention), not "hide from terminal but persist." `DrainWarnings()` replays only warnings from the initial non-suppressed config load. Do not suggest always buffering suppressed warnings or splitting print/buffer behavior
- **EnvVars config field**: `Config.EnvVars` is a `[]string` of `KEY=value` entries, not raw CLI flags. The bare `--env KEY` (host-inherit) form cannot appear in this slice; `strings.HasPrefix` checks on `KEY=` are sufficient
- **Range loop variable pointers**: Go 1.22+ (this project uses Go 1.25) creates a new variable per iteration in `for` loops. Taking `&f` in `for _, f := range` is safe and idiomatic; do not flag it as a loop variable capture footgun
- **Container CLI output buffering**: `DefaultRunner.Run()` and `Output()` capture stdout/stderr into `bytes.Buffer` and log at debug level. Container CLI commands (`container start`, `stop`, `run`, etc.) produce small, bounded output (container IDs, status messages, short errors). This is the standard Go pattern (identical to `exec.Cmd.Output()` in stdlib). Do not suggest truncation, size caps, or conditional `io.Discard` for these buffers

### File Structure

- Installed via Homebrew, not run from source
- Claude Code: Official installer (`claude.ai/install.sh`), not npm
- Container UID remapping: bopca starts the container as root and remaps the `dev` user's UID/GID before dropping privileges via `sudo -E -u dev`. UID and GID remaps are handled independently. For GID collisions, bopca deletes Alpine's `dialout` group when needed (common macOS GID 20 case) and otherwise reuses the existing group for `dev` with a warning. The `shadow` package provides these commands. Do not flag the `--user root` container flag or the privilege drop as security concerns; this is the standard entrypoint pattern for bind-mount compatibility

---
> Source: [cboone/bopca](https://github.com/cboone/bopca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
