# GitHub Copilot repository instructions — sqlitezstd

These repository instructions are intended to help a Copilot coding agent
understand the purpose, layout, and standard build/test/lint workflow for this
repository so it can make safe, buildable changes with minimal exploratory
searching.

## Short summary

- Repository: `github.com/jtarchie/sqlitezstd`.
- Language: Go (module-based). Declared Go version: 1.24.4 (see `go.mod`).
- Purpose: Provide a read-only SQLite VFS that allows opening Zstandard seekable
  compressed SQLite database files (ZSTD seekable format) via the `sqlite3`
  driver.
- Important: This project implements a read-only VFS. Do not add or assume
  write-support in the VFS or rely on PRAGMAs that would require persistent
  writes.

## What this repository expects (environment)

- Go: use Go 1.24.x (the `go` directive in `go.mod` is 1.24.4). Newer 1.24.x
  builds are expected to be compatible, but verify `go test` and `go build`.
- CGO: the project depends on `github.com/mattn/go-sqlite3` which requires CGO.
  Ensure system has a C toolchain (macOS: Xcode command line tools) and a
  working SQLite development environment if building with static linking.
- The repository uses an external CLI to produce ZSTD seekable files:
  `github.com/SaveTheRbtz/zstd-seekable-format-go/cmd/zstdseek` (version in
  `go.mod` is v0.8.0). Install with
  `go install github.com/SaveTheRbtz/zstd-seekable-format-go/cmd/zstdseek@v0.8.0`
  if you need to generate `.zst` test fixtures.

## Build, lint, test, and benchmark (validated commands)

The canonical tasks are available in `Taskfile.yml`. The commands below are
already validated against this repository and should be used in this order when
preparing changes for a pull request:

- Fetch dependencies:

  - `go mod download`

- Format, lint, and tests:

  - `task`

## Quick validation checklist for PRs produced by an agent

1. Code compiles: `go build ./...` (no unchecked errors).
2. Format, lint, tests `task`
3. Avoid changing public API without a conscious module versioning plan.

## Project layout (high level)

- `go.mod`, `go.sum` — module and dependency pins.
- `README.md` — usage and examples (contains recommended usage for opening
  compressed DBs with `?vfs=zstd`).
- `vfs.go`, `readerat.go`, `file.go` — core VFS and file/reader implementations.
- `build.go` — a small `tools` build stub used to pin tool dependencies.
- `benchmark_test.go` — benchmarks that build temporary DBs and compress them.
- `sqlite_zstd_suite_test.go` — test suite / helper tests used in CI/local runs.
- `Taskfile.yml` — convenience tasks: `format`, `lint`, `test`, `bench`,
  `default`.

When making changes, prefer to keep the main implementation in `vfs.go` and
helpers in `file.go`/`readerat.go`. Tests and benchmarks are in the repo root.

## Notable gotchas and operational notes

- The code relies on SQLite build-time features (e.g., FTS5). Tests and code may
  require the `fts5` build tag. Use `-tags fts5` when running tests.
- Because `mattn/go-sqlite3` uses CGO, cross-compilation or running in CI
  containers may need appropriate system packages or a full toolchain.
- This VFS is explicitly read-only. Avoid creating changes that assume write
  support, temporary file persistence, or modifications to the underlying DB.

## Coding conventions and style

- Follow idiomatic Go. Run `gofmt` and `go vet` as part of local validation.
- Keep exported identifiers documented with Go doc comments.
- Prefer small, focused changes. If a change touches public APIs, add a note in
  the PR description describing the rationale and compatibility impact.

## When to search the repo

Trust these instructions first. Only perform additional repository searches if:

- The instructions above are insufficient to complete the requested task.
- A build or test command specified here fails locally (then search to find the
  source of the failure).

If you must search, prefer these files in order: `README.md`, `Taskfile.yml`,
`go.mod`, `vfs.go`, `file.go`, `readerat.go`, tests in `*_test.go` files.

## Contact / maintainers

If behavior is unclear or tests fail in non-obvious ways, refer to `README.md`.
For questions about design or compatibility (especially the C replacement
mentioned in the README), contact the repository owner (email visible in
`README.md`) or open a draft PR describing the proposed change.

---

These instructions are intentionally short and repository-scoped. If more detail
is required for a particular subdirectory or file pattern, create path-specific
instruction files under `.github/instructions/` per GitHub docs.

---
> Source: [jtarchie/sqlitezstd](https://github.com/jtarchie/sqlitezstd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
