# Copilot Instructions for homebrew-executor

## Repository Purpose

This is a Homebrew tap for the Executor CLI tool. It contains the Formula definition that allows users to install Executor via `brew install jorgefuertes/executor/executor`.

## Architecture

- **Formula/executor.rb** - Ruby DSL defining how to build and install Executor
  - Clones the main [executor repo](https://github.com/jorgefuertes/executor)
  - Builds the Go binary using `go build` with version injection via ldflags
  - Includes bottle (pre-built binary) definitions for macOS architectures

## Testing & CI

### Local Testing

Test the formula locally:
```bash
brew install --build-from-source ./Formula/executor.rb
brew test executor
brew uninstall executor  # cleanup
```

Test only syntax without installing:
```bash
brew audit --strict ./Formula/executor.rb
```

### CI Workflows

- **tests.yml** - Runs on push/PR using Homebrew's `test-bot`
  - Tests on Ubuntu 22.04, macOS 14, and macOS 15
  - Validates tap syntax: `brew test-bot --only-tap-syntax`
  - Builds and tests formula on PRs: `brew test-bot --only-formulae`

- **publish.yml** - Triggered when PR is labeled with `pr-pull`
  - Merges bottles (pre-built binaries) from test-bot
  - Uses `brew pr-pull` to merge PR and update bottles

## Key Conventions

### Version Updates

When updating to a new Executor version:

1. Update `VERSION` constant at top of Formula/executor.rb
2. The `url` tag automatically interpolates: `tag: "v#{VERSION}"`
3. Update bottle SHA256 hashes after building new bottles
4. Bottle root_url also interpolates VERSION for release artifacts

### Bottle Updates

After test-bot builds bottles in CI:
- Download bottle files from PR artifacts
- Run `brew bottle --merge --write *.json` to update Formula
- Bottles are hosted on GitHub releases at: `https://github.com/jorgefuertes/executor/releases/download/v{VERSION}`

### Formula Structure

- Uses `:stable` livecheck with regex to auto-detect new releases
- `depends_on "go" => :build` means Go is only needed at build time
- `std_go_args(ldflags: ...)` injects version into Go binary at build time
- Test block validates both command execution and version output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jorgefuertes)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jorgefuertes)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
