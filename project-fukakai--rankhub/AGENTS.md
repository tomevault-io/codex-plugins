# RankHub AI Coding Instructions

## Project Overview

RankHub is a cross-platform Flutter app for managing rhythm game data (舞萌 DX, 中二节奏, etc.) across multiple platforms. Architecture follows a **Platform-Game two-level abstraction** with plugin-style extensibility.

## Architecture Pattern

### Core Abstractions

- **Platform** (`IPlatform`): Data source handling authentication, credentials, and managing multiple games
- **Game** (`IGame`): Content provider building UI views (Wiki/Rank tabs) and managing game-specific logic
- **Platform Registry** (`PlatformRegistry` in `lib/data/platforms_data.dart`): Centralized registration of all platforms

### Key Flow

```
Page → Controller → Service → Platform/Game → API/Database
```

### Module Structure

Each platform lives in `lib/modules/{platform_name}/` with:

- Platform implementation (extends `BasePlatform`)
- Game implementations (extend `IGame`)
- `services/` folder for platform-specific services (LoginHandler, CredentialProvider, API service)
- `widgets/` for platform-specific UI components

Example: `lib/modules/lxns/` contains LXNS platform with MaimaiDX game support.

## State Management

**GetX Framework** (forked version from git repo, not pub.dev):

- Controllers use `.obs` for reactive state: `var selectedIndex = 0.obs;`
- Controllers injected via `BindingsBuilder` in `lib/routes/app_pages.dart`
- UI updates with `Obx(() => ...)` wrapper
- Lifecycle hooks: `ever()`, `once()`, `debounce()` for state changes

## Authentication & Credentials

### Three Credential Types

1. **API Key** (`ApiKeyCredentialProvider`): Static, stored in `account.apiKey`
2. **OAuth2** (`OAuth2CredentialProvider`): Auto-refreshing tokens in `accessToken`/`refreshToken`
3. **Username/Password** (`UserPasswordCredentialProvider`): Basic auth

### OAuth2 Flow (LXNS Example)

1. Generate PKCE verifier/challenge (`lib/utils/pkce_helper.dart`)
2. Launch WebView with platform auth URL (`flutter_inappwebview`)
3. Extract authorization code from callback
4. Exchange for tokens, save to `Account` model
5. Auto-refresh handled by `CredentialProvider.getCredential()`

## Data Sync System

**SyncManager** (`lib/services/sync_manager.dart`) handles concurrent task execution:

- Max 3 concurrent tasks (`maxConcurrentTasks`)
- Tasks organized in `SyncTaskGroup` with priorities
- Task types: `metadata`, `songs`, `collections`, `scores`, `players`
- Progress tracking via `task.updateProgress(0.5, description: "...")`

### Creating Sync Tasks

Platforms implement `createFullSyncTasks(Account)` returning a `SyncTaskGroup`. Each `SyncTask` has:

- `execute` callback with progress updates
- Task states: `pending` → `running` → `completed/failed/cancelled`

## Database Layer

**Isar Community** (NoSQL database):

- Multi-database architecture: separate DB per game + shared account DB
- Access via `IsarService.instance.maimai.*` or `IsarService.instance.account.*`
- Models use `@collection` annotation, require code generation
- Services extend `BaseIsarService` with custom `databaseName` and `schemas`

## Development Workflows

### Code Generation (Required After Model Changes)

```bash
dart run build_runner build --delete-conflicting-outputs
```

### Running the App

```bash
flutter pub get
flutter run
```

### Adding New Platform

1. Create `lib/modules/{platform}/` directory structure
2. Implement `BasePlatform` with `loginHandler`, `credentialProvider`, `getGames()`
3. Implement `PlatformLoginHandler.showLoginPage()` for auth UI
4. Implement `CredentialProvider` (choose base: API Key/OAuth2/UserPass)
5. Register in `lib/data/platforms_data.dart` → `PlatformRegistry._platforms`

### Adding New Game

1. Implement `IGame` interface with `buildWikiViews()` and `buildRankViews()`
2. Create Isar models with `@collection` if local storage needed
3. Create service extending `BaseIsarService` for data access
4. Add to platform's `getGames()` list
5. Run build_runner if models added

## Conventions

### Routing

- All routes defined as constants in `lib/routes/app_routes.dart`
- Page bindings (controller injection) in `lib/routes/app_pages.dart`
- Use `Get.toNamed(AppRoutes.detail)` for navigation

### File Naming

- Snake_case for all Dart files: `maimai_dx_game.dart`
- Controllers suffix: `main_controller.dart`
- Services suffix: `sync_manager.dart`, `maimai_api_service.dart`

### UI Patterns

- Bottom navigation with `NavigationBar` (3 tabs: Wiki/Rank/Mine)
- Blur effect on bottom bar: `BackdropFilter` with `ImageFilter.blur`
- Tab transitions: `AnimatedSwitcher` with `FadeTransition` + `ScaleTransition`
- Multi-tab game views: `GameContentView` list returned from games

### Error Handling

- Credential errors throw exceptions caught by UI layers
- Sync task failures update task status to `failed` with error message
- Token expiry handled automatically by `CredentialProvider.getCredential()`

## External Dependencies

Key packages (see `pubspec.yaml`):

- `isar_community`: Local database
- `dio`: HTTP client with cache interceptor
- `flutter_inappwebview`: OAuth2 WebView flows
- `cached_network_image`: Image caching
- `screenshot`: Image generation for share features
- `permission_handler`: Storage permissions for image saving

## Testing & Debugging

- Use `flutter analyze` for linting (follows `package:flutter_lints/flutter.yaml`)
- Print debugging common: `print('📦 提交任务组: ...')` with emojis for log categorization
- GetX provides `Get.find<Controller>()` for controller access in debugging

## Important Notes

- README.md contains detailed architecture diagrams and extension guides
- NEVER use GetX from pub.dev - project uses forked version from GitHub
- Isar code gen required before running after model changes
- All platforms must be registered in `PlatformRegistry` to be visible
- Account model fields: `apiKey`, `accessToken`, `refreshToken`, `tokenExpiry`, `username`, `password`, `metadata`

---
> Source: [Project-Fukakai/RankHub](https://github.com/Project-Fukakai/RankHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
