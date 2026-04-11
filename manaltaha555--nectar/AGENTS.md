<!--
Instructions for AI coding agents working on the Nectar Flutter app.
Keep this file concise and focused on repository-specific facts an agent needs
to be productive without asking trivial questions.
-->

# Copilot instructions for the Nectar repository

Summary
- This is a Flutter mobile app (multi-platform) named `nectar` with a simple
  feature-driven layout under `lib/features/` and lightweight view-models in
  `lib/Logic/` that call HTTP client code in `lib/core/services`.

Quick goals for edits
- Preserve platform-neutral code (Dart/Flutter). Avoid changing Android/iOS
  native config unless asked. Prefer edits in `lib/` and `assets/`.

Project structure (key folders/files)
- `lib/main.dart` — app entry and top-level ThemeData.
- `lib/features/*` — feature modules split into subfolders (e.g. `home`,
  `auth`, `start`) containing UI screens.
- `lib/Logic/` — simple ViewModel-like classes (e.g. `products_view_model.dart`) that
  fetch data from `lib/core/services` and expose plain lists/fields.
- `lib/core/services/` — API and integration code (e.g. `products_api.dart`).
- `lib/data/models/` — JSON -> model mapping (e.g. `product_model.dart`, `products.dart`).
- `lib/core/constants/` and `lib/core/themes/` — visual constants used across UI.

Important conventions and patterns
- Minimal state management: view models are plain classes, not Providers or
  Bloc instances. UI widgets instantiate a view-model (see `home_page.dart`) and
  call async fetch methods directly followed by `setState()` to re-render.
- Networking: `http` package is used directly in `lib/core/services/*`. APIs
  return typed model lists (e.g. `List<ProductModel>`). Handle HTTP non-200
  by returning empty lists (existing pattern).
- Assets: images/icons live under `assets/` and are referenced by relative
  paths in code (see `AppImages` constants). When adding images update
  `pubspec.yaml` only if adding new top-level asset directories.

Build / run / test workflows (developer commands)
- Typical local run (device or emulator):
  - flutter run
- Build a release APK / iOS: use standard Flutter commands (e.g. `flutter build apk`)
  — native android gradle wrappers exist under `android/gradlew(.bat)` if needed.
- Run unit/widget tests:
  - flutter test

Files and examples to reference when changing behavior
- Add or modify models: `lib/data/models/product_model.dart` and
  `lib/data/models/products.dart` show how JSON is parsed and lists are
  propagated to the UI.
- Networking pattern: `lib/core/services/products_api.dart` — follow its
  use of `http.get`, `jsonDecode`, and returning typed model lists.
- UI + view-model interaction: `lib/features/home/screens/home_page.dart` uses
  a `ProductsViewModel` instance, awaits `fetchProducts()`, then calls
  `setState` to stop a Skeletonizer loading indicator. Follow that pattern
  when adding similar pages.

Edge-cases & expectations for changes
- Keep null-safety: Dart SDK is ^3.8.1 — use non-nullable types and `required`.
- Avoid introducing new global state management libraries. If a change
  requires Provider/Bloc, document why and add migration notes.
- When altering API response handling, preserve the current fallback of
  returning an empty list for non-200 responses unless a caller requires
  exceptions. Update calling UI to handle an empty list (shows empty UI).

Where to look for CI / automation
- There are no repository workflows or `.github/workflows` present. Do not
  assume CI. If adding automation, create a single GitHub actions workflow
  that runs `flutter test` and `flutter analyze` on PRs.

Style & lint
- Project uses `flutter_lints` via `analysis_options.yaml`. Keep changes
  lint-clean. Prefer small focused edits and keep formatting consistent with
  existing code (2-space indentation, trailing commas in Flutter widgets).

When adding or changing features
- Update `AppImages` / `AppColors` if new visual assets are added (see
  `lib/core/constants/`). Update `pubspec.yaml` only for new asset directories.
- Update `lib/main.dart` ThemeData if global typography or button theme
  changes are required (see `lib/core/themes/`).

If you need clarification
- Ask for which target platforms the change should support (Android, iOS,
  web, desktop) before editing native or build files.

End of instructions — please run changes locally (`flutter analyze` and
`flutter test`) and report any new lint or test failures in the PR.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manaltaha555)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/manaltaha555)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
