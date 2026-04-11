# Ash Trail — Copilot Instructions

## Project

Flutter app (iOS primary) for tracking habits. Bundle ID: `com.soup.smokeLog`. iOS deployment target: 16.0.

## Architecture

- `lib/screens/` — one file per screen, flat (no feature subdirectories)
- `lib/widgets/` — reusable UI components
- `lib/services/` — injectable via Riverpod, may do I/O/state/platform calls
- `lib/providers/` — Riverpod provider definitions
- `lib/repositories/` — data access layer
- `lib/models/` — data models and enums
- `lib/utils/` — pure helpers, no Riverpod, minimal I/O

## Riverpod Patterns

- `Provider<T>` for service singletons (no autoDispose, no family)
- `StreamProvider<T>` for reactive streams (auth, accounts)
- `FutureProvider<T>` / `FutureProvider.family` for async data
- `StateNotifier` + `StateNotifierProvider` for complex state
- Use `ref.watch()` for reactive deps, `ref.listen()` for side-effects, `ref.onDispose()` for cleanup

## Code Style

- Logger: `AppLogger.logger('ComponentName')` — never `print()` or `debugPrint()`
- Always pass error + stackTrace to `.e()` calls
- Services are plain classes — Riverpod manages lifecycle
- Error display: use `ErrorDisplay.showSnackBar()` from `lib/utils/error_display.dart` for errors
- Lints: `package:flutter_lints/flutter.yaml` — `avoid_print: true`

## iOS / watchOS (Swift)

- Singleton: `static let shared = ClassName()` with `private override init()`
- `@Published` properties for ObservableObject state
- `// MARK: - Section Name` for organization
- App Group: `group.com.soup.smokeLog.watchkitapp`

## Testing

- Test files mirror `lib/` structure under `test/`
- Naming: `*_test.dart` (unit/widget), `*_flow_test.dart` (flows)
- User-story pattern: GIVEN / WHEN / THEN
- Integration tests in `integration_test/`

## Build & Run

```bash
flutter analyze                        # lint check
flutter test                           # unit + widget tests
./scripts/deploy_testflight.sh         # build & deploy to TestFlight (default: bump patch)
./scripts/deploy_testflight.sh --minor # bump minor version
```

## Workflow

After completing a bug fix or feature:

1. Run `flutter analyze` — fix any new issues before proceeding
2. Run relevant tests (`flutter test` or specific test files)
3. Commit and push
4. Run `./scripts/deploy_testflight.sh` to deploy to TestFlight

## Behavior Defaults

- Be concise — minimal explanation, just do it
- Prefer editing existing files over creating new ones
- Don't create documentation files unless explicitly asked
- Always verify with `flutter analyze` after edits
- Run tests after changes when relevant

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SoupyOfficial)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SoupyOfficial)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
