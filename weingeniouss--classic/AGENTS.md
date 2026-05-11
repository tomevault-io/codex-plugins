<!-- Repo-specific Copilot/AI instructions. Keep concise and focused. -->
# Copilot instructions for classic

This file tells AI coding agents how to be productive in this repository. Keep responses concise, pragmatic, and repository-aware.

1. Big picture
- This is a Flutter app (`lib/`, `android/`, `ios/`, `windows/`, `macos/`, `web/`). The app uses GetX for navigation/state (`get` package) and organizes code under `lib/` into `controller/`, `view/`, `modal/` (models), and `utils/`.
- UI code frequently uses top-level Widget functions (not always StatefulWidget classes). Example: `lib/view/screen/menu/addCustomJewellery/.../addCustomJewelleryWidget.dart` exports many helper widget functions (e.g., `productType(...)`, `uploadYourFile(...)`).

2. Project conventions and patterns
- State management: GetX controllers live under `lib/controller/` (e.g., `controller/user_Interface/menu/...`). Expect controllers to be injected or passed to widgets, not always via Provider.
- Widget helpers: many files export helper functions that return `Widget` (not classes). When adding new UI, follow that style for small parts.
- File naming: code contains mixed naming (underscores + PascalCase) and files use `// ignore_for_file: file_names`. Do NOT rename files to satisfy lint rules — preserve existing filenames.
- Utility widgets: reusable controls live under `lib/view/utils/widget/` (e.g., `dropdownSelected.dart`, `inputfield.dart`, `checkbox.dart`). Reuse them instead of duplicating behavior.
- Assets: assets and fonts are declared in `pubspec.yaml` under `assets/` and `fonts:`. Use the same asset paths (e.g., `assets/fonts/IBMPlexSans-*.ttf`).

3. Important files to inspect (examples)
- `lib/main.dart` — app entry: uses `GetMaterialApp`, theme uses font family `Sans-Bold`.
- `pubspec.yaml` — dependencies (notably `get`, `video_player`, `image_picker`) and asset declarations.
- `lib/controller/` — controllers and business logic; look here to understand flow and API calls.
- `lib/view/` — UI screens and widgets (follow existing helper/widget patterns).

4. Build / run / test commands
- Install deps: `flutter pub get`
- Run on device: `flutter run -d <device-id>` (Windows PowerShell: `flutter run -d emulator-5554` or use `flutter devices`).
- Build APK: `flutter build apk --release`
- Run tests: `flutter test` (project has `test/widget_test.dart`).
- iOS: from repo root run `cd ios; pod install` if editing native pods.

5. PR & coding guidance for AI
- Keep changes small and focused. Update only the files needed to implement a feature/bugfix.
- When adding widgets, prefer composing existing `view/utils/widget` components and using GetX controllers under `lib/controller`.
- Preserve existing imports and relative paths. Example import pattern:
  - `import 'package:classic/controller/user_Interface/menu/addCustomJewellery/addCustomJewellery_Controller.dart';`
- Do not change public APIs unless the change is small and accompanied by updates to all call sites.

6. Tests & verification
- Run `flutter test` after changes touching logic. For UI changes, run the app on an emulator and verify flows manually.

7. Known repo quirks
- Many files include `// ignore_for_file: file_names` — filenames are intentionally non-standard.
- Large generated/build artifacts exist under `build/` and `app/` — avoid editing those files.

8. If you need more context
- Inspect `lib/controller/` to find how data flows from UI to API or local state.
- Search for `Get.put` / `Get.find` to locate where controllers are registered.
- Check `pubspec.yaml` for packages that introduce special behavior (e.g., `webview_flutter`, `image_picker`).

If anything here is unclear or you want more examples from specific features/screens, tell me which screen or controller to analyze next.

---
> Source: [Weingeniouss/classic](https://github.com/Weingeniouss/classic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-10 -->
