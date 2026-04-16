# NullFeed Flutter — Agent Guidelines

This file is the **canonical source** for all AI coding agent instructions in this
repository. The following files are automatically synced copies and must not be
edited directly:

- `.cursorrules`
- `.windsurfrules`
- `.clinerules`
- `.continuerules`
- `CLAUDE.md`
- `.github/copilot-instructions.md`

If you need to update agent instructions, edit **this file** (`AGENTS.md`) and run
`scripts/sync-agent-rules.sh`, or let CI tell you they're out of sync.

---

## Project Overview

NullFeed is a self-hosted YouTube media center. This is the Flutter client
targeting iOS and Apple TV.

**Stack:** Flutter 3.41+, Dart 3.11+, Riverpod 3.x, Freezed 3.x, GoRouter 17.x,
Hive, Dio, video_player.

## CI Pipeline

This repo has 6 CI checks that run on every PR:

| Check | What it does |
|-------|-------------|
| **Format Check** | `dart format --set-exit-if-changed .` |
| **Analyze** | `flutter analyze --fatal-infos --fatal-warnings` |
| **Test** | `flutter test` |
| **Build iOS** | `flutter build ios --release --no-codesign` (runs on `macos-26`) |
| **Dependency Audit** | Warns on major version drift (non-blocking) |
| **Agent Rules Sync** | Verifies all agent instruction files match `AGENTS.md` |

### How to handle CI failures

**CI failures are iterative.** Fixing one failure often reveals the next. Do not
assume CI is green after pushing a fix — always wait for the full run to complete
and check results before moving on.

The correct workflow:

1. Push your fix.
2. **Wait for CI to finish** (use `gh pr checks <number> --watch` or poll with
   `gh pr checks <number>`).
3. Read the results. If something still fails, pull the logs with
   `gh run view <run-id> --log-failed`.
4. Fix the next failure. Repeat until all checks pass.
5. Only then consider the PR ready for review.

**Do not** push a fix and immediately tell the user "CI should pass now." Instead,
confirm it actually passes.

**Common failure chains:**
- SDK version mismatch → dependency resolution fails → all checks fail
- Code changes → format check fails → fix formatting → analyzer finds new
  issues → fix those → tests may need updating
- Dependency upgrades → generated code stale → run `dart run build_runner build
  --delete-conflicting-outputs` → API changes in new versions need migration

**Useful commands:**
```bash
# Check CI status on a PR
gh pr checks <pr-number> --repo windoze95/nullfeed-flutter

# Watch CI until it finishes
gh pr checks <pr-number> --watch --repo windoze95/nullfeed-flutter

# Get failed job logs
gh run view <run-id> --repo windoze95/nullfeed-flutter --log-failed

# Run checks locally before pushing
dart format --set-exit-if-changed .
flutter analyze --fatal-infos --fatal-warnings
flutter test
```

## Branching & PR Strategy

- **`main`** is the stable branch. All PRs target `main`.
- **Feature branches:** `feature/<short-description>`
- **Bug fixes:** `fix/<short-description>`
- **CI/infra fixes that affect `main` directly** (e.g., broken CI on `main`)
  should go in their own PR, not bundled into a feature PR. This unblocks all
  other PRs.
- Keep PRs focused. Don't mix unrelated changes.
- Rebase feature branches on `main` after infra PRs merge.

## Code Conventions

- **Formatting:** `dart format` with default settings. CI enforces this.
- **Analysis:** Zero warnings, zero infos. CI runs `--fatal-infos --fatal-warnings`.
- **Models:** Use `@freezed` with `abstract class`. Regenerate with
  `dart run build_runner build --delete-conflicting-outputs` after model changes.
- **State management:** Riverpod 3.x `Notifier` / `NotifierProvider`. Do not use
  the legacy `StateNotifier` or `StateProvider` APIs.
- **Routing:** GoRouter 17.x.
- **Tests:** Widget tests must initialize Hive boxes in `setUp`.

## Architecture

```
lib/
├── config/          # Theme, routes, constants
├── models/          # Freezed data classes
├── providers/       # Riverpod notifiers and providers
├── screens/         # Full-page widgets
├── services/        # API, storage, offline, websocket
└── widgets/         # Reusable UI components
```

## Agent Rules Sync

All agent instruction files are kept in sync with this file. To update:

```bash
# After editing AGENTS.md:
./scripts/sync-agent-rules.sh

# Or just commit — CI will catch any drift
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/windoze95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-14 -->
