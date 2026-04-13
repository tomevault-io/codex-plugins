# AI Coding Assistant Instructions for flutter_application

## Project Overview

**Type:** Flutter cross-platform mobile application (iOS, Android, macOS, Windows, Linux, Web)  
**Dart SDK:** ^3.10.0  
**Key Dependencies:** Flutter, Material Design, flutter_lints  
**Current State:** Early-stage project with basic counter app scaffold and empty `Bcalculator.dart` module

This is a bare-bones Flutter template project. The main app structure is in `lib/main.dart` with a simple stateful counter widget. There's a `Bcalculator.dart` file prepared but currently empty, suggesting a calculator feature is planned.

## Architecture & Structure

### Key Files & Directories

- **`lib/main.dart`** - Application root containing `MyApp` (MaterialApp setup) and `MyHomePage` (stateful counter demo)
- **`lib/Bcalculator.dart`** - Empty module file; appears intended for calculator logic
- **`test/widget_test.dart`** - Single widget test verifying counter increment behavior
- **`pubspec.yaml`** - Dependencies: minimal (flutter, cupertino_icons, flutter_lints)
- **`analysis_options.yaml`** - Lint configuration (includes flutter_lints package)

### Platform-Specific Code

Platform folders follow Flutter convention:
- **`android/`** - Android native code (Gradle-based build system)
- **`ios/`** - iOS native code (Xcode workspace)
- **`macos/`**, **`windows/`**, **`linux/`** - Desktop platform implementations
- **`web/`** - Web platform build artifacts

## Development Workflows

### Build & Run
```bash
# Run on default device/emulator with hot reload enabled
flutter run

# Run on specific device
flutter run -d <device_id>

# Build for production (Android/iOS)
flutter build apk    # Android APK
flutter build ios    # iOS app bundle
```

### Testing
```bash
# Run all tests (widget_test.dart)
flutter test

# Run with coverage reporting
flutter test --coverage
```

### Code Quality
```bash
# Analyze code against configured lints (analysis_options.yaml)
flutter analyze

# Format code according to Dart conventions
dart format lib/ test/

# Apply auto-fixes for lint violations
dart fix --apply
```

### Hot Reload vs Hot Restart
- **Hot Reload** (`r` in CLI or IDE button) - Fast rebuild, preserves app state; ideal for UI tweaks
- **Hot Restart** (`R` in CLI) - Full restart, resets state; use when modifying global constants or major logic

## Code Patterns & Conventions

### Widget Structure
Follow Flutter's widget hierarchy:
- **Stateless Widgets** for pure UI (no state mutations)
- **Stateful Widgets** with `State` subclass for mutable state

**Pattern in this project:**
```dart
class MyApp extends StatelessWidget { ... }  // Root, theme setup
class MyHomePage extends StatefulWidget { ... }  // Configurable container
class _MyHomePageState extends State<MyHomePage> { ... }  // Private state class
```

### State Management
Currently uses `setState()` for local widget state (see `_MyHomePageState._incrementCounter()`). For complex features:
- Keep state scoped to the widget that uses it
- Use `const` constructors for immutable widgets (improves performance)
- Document state mutations with comments

### Naming Conventions
- **Private members** prefixed with `_` (e.g., `_counter`, `_MyHomePageState`)
- **Widget files** use PascalCase: `Bcalculator.dart`
- **Dart files** typically one widget per file (future refactor opportunity)

### Material Design
Project uses Material 3 theme with `ThemeData.fromSeed()`:
- Theme seeded from `Colors.deepPurple`
- Uses `ColorScheme.inversePrimary` for AppBar
- Icons from Material Icons library

## Integration Points

### External Dependencies
- **flutter** - Core framework
- **cupertino_icons** - iOS-style icons (added to pubspec but commented asset examples)
- **flutter_lints** - Lint rules (referenced in analysis_options.yaml)

### No Current Third-Party Logic
This is a fresh template with no database, API client, state management (GetX, Riverpod, BLoC), or complex integrations yet.

## Testing Approach

**Current test** (`test/widget_test.dart`) demonstrates the expected pattern:
- Use `testWidgets()` for Flutter widget tests (not pure unit tests)
- Use `WidgetTester` to interact: `tester.pumpWidget()`, `tester.tap()`, `tester.pump()`
- Assert UI state with `find.text()`, `find.byIcon()`, `expect()`

When adding features (e.g., calculator):
- Test business logic separately if extracted to standalone Dart classes
- Test UI behavior with widget tests

## Immediate Work Suggestions

1. **Develop `Bcalculator.dart`** - Implement calculator logic (stateless or extract state to model)
2. **Organize features** - Consider creating `lib/features/` or `lib/widgets/` subdirectories as codebase grows
3. **Add more tests** - Extend `widget_test.dart` for calculator UI and logic
4. **Document theme** - If customizing colors/fonts, centralize in a `lib/theme/` file

## Key Lint Rules Active

From `package:flutter_lints/flutter.yaml`:
- Avoid `print()` statements (use logging package instead if needed)
- Prefer `const` constructors
- Prefer single quotes
- Avoid async gaps in state changes

Disabled/customizable lints are commented in `analysis_options.yaml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ndohruthagbor-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
