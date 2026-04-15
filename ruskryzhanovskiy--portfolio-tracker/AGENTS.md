# Copilot Instructions for AI Agents

## Project Overview
This is a Flutter mobile application project for portfolio tracking. The codebase is structured for cross-platform development (Android/iOS) using Dart and Flutter, following a feature-first architecture with clean separation of concerns.

## Architecture & Key Files
- **lib/main.dart**: Application entry point
- **lib/app.dart**: Root widget with provider setup and global configuration
- **lib/modules/**: Feature modules (markets, portfolio, profile, etc.)
  - **/domain/**: Models and business logic
  - **/data/**: Repositories and data sources
  - **/presentation/**: UI components (pages, widgets) and state management
- **lib/core/**: Shared infrastructure
  - **/network/**: Network layer with error handling
  - **/cache/**: Offline-first caching system
  - **/repository/**: Base repository with caching support
- **lib/services/**: Global services
  - **currency_manager.dart**: Currency preferences management
  - **locale_manager.dart**: Language preferences management
- **lib/l10n/**: Localization files (en, uk)
- **pubspec.yaml**: Project configuration and dependencies
- **analysis_options.yaml**: Linting rules configuration
- **android/** and **ios/**: Platform-specific configuration

## Developer Workflows
- **Build & Run**:
  - Use `flutter run` to launch the app on a connected device/emulator
  - For Android/iOS builds, use `flutter build apk` or `flutter build ios`
- **Dependency Management**:
  - Add packages to `pubspec.yaml` and run `flutter pub get`
- **Localization**:
  - Add strings to `lib/l10n/app_en.arb` and `app_uk.arb`
  - Run `flutter gen-l10n` to regenerate localization files
- **Linting & Analysis**:
  - Run `flutter analyze` to check for code issues
- **Testing**:
  - Add tests in `test/` and use `flutter test` to run them

## Architecture Patterns
- **Feature-First Structure**: Each feature has its own module with domain, data, and presentation layers
- **State Management**:
  - Use BLoC/Cubit for complex feature-level state
  - Use ChangeNotifier for simple global services
- **Repository Pattern**:
  - Extend `BaseRepository` for offline-first data handling
  - Use `CacheService` for local storage
  - Use `NetworkService` for API calls
- **Error Handling**:
  - Use sealed classes for type-safe error handling
  - Handle network and cache errors separately
- **Dependency Injection**:
  - Use `Provider` for global services
  - Use `RepositoryProvider` for repositories
  - Use `BlocProvider` for feature-level state
- **Settings Management**:
  - Use persistent storage (Hive) for user preferences
  - Implement settings in Profile feature
  - Use global services for cross-feature settings

## Code Organization
- **Feature Module Structure**:
  ```
  modules/
    ├── feature_name/
    │   ├── domain/
    │   │   ├── models/
    │   │   └── repositories/
    │   ├── data/
    │   │   ├── data_sources/
    │   │   └── repositories/
    │   └── presentation/
    │       ├── cubit/
    │       ├── pages/
    │       └── widgets/
  ```
- **Global Services Structure**:
  ```
  services/
    ├── currency_manager.dart
    └── locale_manager.dart
  ```

## Examples
- **Adding a New Feature**:
  1. Create feature directory structure
  2. Implement domain models and interfaces
  3. Create data sources and repositories
  4. Add presentation layer with Cubit/BLoC
  5. Connect to global services if needed

- **Adding Global Settings**:
  1. Create a service extending ChangeNotifier
  2. Implement Hive storage for persistence
  3. Add to MultiProvider in app.dart
  4. Add UI controls in ProfilePage
  5. Add strings to localization files

## Environment Setup
- **Environment Variables**: Store in `.env` file
- **API Keys**: Access via `dotenv.env['KEY_NAME']`
- **Dependencies**: All listed in `pubspec.yaml`

## References
- [Flutter Documentation](https://docs.flutter.dev/)
- [BLoC Library](https://bloclibrary.dev/)
- [Provider Package](https://pub.dev/packages/provider)
- [Hive Database](https://docs.hivedb.dev/)

---
_These instructions reflect the current architecture and patterns used in the portfolio tracking app. Update as new patterns emerge._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RusKryzhanovskiy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
