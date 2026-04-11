# GitHub Copilot Instructions

## Project Global Engineering Rules

This section defines **mandatory engineering principles** that apply to
all code written in this repository.

---

## 1. Clean Code First

All code must follow **clean code principles**.

Requirements:

- Code must be **readable, intentional, and self-documenting**
- Prefer **clear naming** over comments
- Avoid unnecessary abstractions
- Avoid clever code, prefer obvious code
- Code should be understandable without external explanation

---

## 2. Small Files & Small Functions

Large files and large functions are **not allowed**.

Guidelines:

- **Widget files:** max 50--70 lines
- **Functions:** ideally \< 20 lines
- **Classes:** must have a single responsibility
- Break logic into **small focused units**
- Extract helpers, widgets, or services when files grow too large

---

## 3. Comments Only When Necessary

Comments are **not a substitute for clean code**.

Allowed comments:

- Explaining **why**, not **what**
- Complex algorithm explanation
- Business rule clarification

Avoid redundant comments that simply restate the code.

---

## 4. Strict Clean Architecture Discipline

Clean Architecture boundaries **must never be violated**.

Rules:

- `presentation` → can depend on `domain`
- `domain` → depends on nothing
- `data` → depends on `domain`
- `core` → shared utilities only

Forbidden:

- UI importing `data`
- Domain importing Flutter
- Models leaking into the domain layer

Violations must be **refactored immediately**.

---

## 5. SOLID, DRY, and Sound Engineering Principles

All code must respect:

### SOLID

- **S** --- Single Responsibility
- **O** --- Open/Closed
- **L** --- Liskov Substitution
- **I** --- Interface Segregation
- **D** --- Dependency Inversion

### DRY

Avoid duplicated logic.

If the same logic appears **more than once**, consider extraction.

### General Principles

- Prefer composition over inheritance
- Prefer immutability
- Prefer explicitness over hidden behavior

---

## 6. Root Cause First

When fixing bugs:

1.  Identify the **root cause**
2.  Fix the **source of the problem**
3.  Avoid patching symptoms

Temporary fixes must be clearly documented.

---

## 7. Minimal Safe Changes

Changes should be:

- **Small**
- **Safe**
- **Targeted**

Avoid:

- Large refactors in unrelated areas
- Changing multiple systems in a single commit
- Overengineering

---

## 8. No Breaking Changes

Do not introduce breaking changes unless explicitly required.

Breaking changes include:

- API contract changes
- Removing public classes
- Renaming widely used interfaces
- Changing repository return types

If unavoidable:

- Document clearly
- Update all affected code

---

## 9. Follow Repository Conventions

All contributors must follow the **existing repository conventions**.

Do not introduce:

- New folder structures
- Different architecture patterns
- Alternative state management approaches

Consistency across the codebase is mandatory.

---

## 10. Core Folder Usage

`core/` is reserved for **shared, feature-agnostic components**.

Allowed:

- Base classes
- Utilities
- Shared widgets
- Error handling
- Network layer

Not allowed:

- Feature-specific logic
- Feature-specific models

---

## 11. Edge Cases & Error Handling

All implementations must consider:

- Null values
- Empty collections
- Network failures
- Invalid API responses
- Unexpected states

Never assume inputs are always valid.

---

## 12. Dependency Rules

Dependencies must be **minimal and justified**.

Do **NOT** add a package unless:

1.  It solves a real problem
2.  It is actively maintained
3.  It does not duplicate existing functionality
4.  It does not introduce heavy transitive dependencies
5.  When adding a package, always use the latest stable version available

---

## 13. Engineering Partner Mindset

Engineers and AI assistants must operate in **engineering partner
mode**.

Responsibilities:

- Suggest improvements when appropriate
- Detect architectural violations
- Highlight potential bugs
- Maintain long-term code quality

However:

- Respect existing architecture
- Avoid unnecessary refactoring

---

## 14. No Assumptions Without Verification

Never assume:

- API contracts
- Data formats
- Business rules
- Existing functionality

Always:

- Inspect the repository
- Verify types and interfaces
- Reuse existing implementations when available

---

## 15. Avoid Duplication

Duplication increases maintenance cost.

Before implementing new functionality:

1.  Search the repository
2.  Check `core/utils`
3.  Check shared widgets
4.  Check existing helpers

If functionality already exists → **reuse it**.

---

## 16. Dart Naming Conventions

Follow official Dart naming standards.

| Element   | Convention              |
| --------- | ----------------------- |
| Classes   | PascalCase              |
| Variables | camelCase               |
| Files     | snake_case.dart         |
| Constants | camelCase or UPPER_CASE |
| Enums     | PascalCase              |

Example:

    user_profile_page.dart
    UserProfilePage
    userRepository

---

## 17. Mandatory Completion Self‑Review Checklist

Before finishing any task verify **Quick Checklist**.

All items must pass before submission.

---

## 18. Git and Pull Request Output (Mandatory)

After completing a task, generate:

### Commit Message

Use **Conventional Commits**:

    feat(auth): add login repository implementation
    fix(cart): handle empty cart state
    refactor(profile): extract avatar widget

### Pull Request Description

Must include:

- Branch name
- Summary of changes
- Reason for the change
- Architecture impact
- Testing considerations

Example:

    ## Branch Name
    feature/login

    ## Summary
    Implements product repository with remote data source integration.

    ## Changes
    - Added ProductRepositoryImpl
    - Added ProductRemoteDataSource
    - Added product models and mapping

    ## Architecture
    Follows Clean Architecture structure.

    ## Notes
    No breaking changes introduced.

---

## Architecture

The project follows **Clean Architecture** with a strict layer separation:

```
lib/
├── app/                        # App-level Cubit, app widget
├── core/                       # Shared, feature-agnostic code
│   ├── data_source/remote/     # BaseRemoteDataSource
│   ├── di/                     # Dependency injection (get_it + injectable)
│   ├── entities/               # Shared domain entities
│   ├── error/                  # Failures, exceptions, error handler
│   ├── models/                 # Shared API models (BaseModel, etc.)
│   ├── network/                # Dio factory, interceptors, endpoints
│   ├── repositories/           # BaseRepository / BaseRepositoryImpl
│   ├── routing/                # AppRouter, Routes constants
│   ├── theme/                  # ColorsManager, TextStyles, FontManager
│   ├── utils/                  # Enums, constants, validators, helpers
│   └── widgets/                # Shared reusable widgets
├── features/                   # One folder per feature
│   └── <feature>/
│       ├── data/
│       │   ├── data_sources/   # Remote (and local) data source interfaces + impls
│       │   ├── models/         # JSON models (json_serializable or built_value)
│       │   └── repositories/   # Repository implementations
│       ├── domain/
│       │   ├── entities/       # Pure Dart domain entities
│       │   ├── repositories/   # Abstract repository interfaces
│       │   └── use_cases/      # (only when logic is shared across multiple features)
│       └── presentation/
│           ├── bloc/           # Bloc/Cubit, events, states
│           ├── pages/          # Full-screen page widgets
│           └── widgets/        # Feature-scoped widgets
├── generated/                  # Auto-generated: assets.dart, l10n.dart
└── l10n/                       # Localisation ARB files (intl_ar.arb, intl_en.arb)
```

---

## Key Packages & Their Usage

| Package                            | Purpose                              |
| ---------------------------------- | ------------------------------------ |
| `flutter_bloc` / `bloc`            | State management                     |
| `built_value` / `built_collection` | Immutable state & event models       |
| `get_it` / `injectable`            | Dependency injection                 |
| `dartz`                            | Functional error handling (`Either`) |
| `dio`                              | HTTP client                          |
| `json_serializable`                | JSON serialization for API models    |
| `shared_preferences`               | Key-value local storage              |
| `flutter_secure_storage`           | Secure token storage                 |
| `firebase_messaging`               | Push notifications                   |
| `firebase_analytics`               | Analytics                            |
| `google_maps_flutter`              | Maps                                 |
| `flutter_localizations` / `intl`   | i18n / l10n                          |
| `shimmer`                          | Loading skeletons                    |
| `cached_network_image`             | Network image caching                |
| `auto_size_text`                   | Text that auto-scales                |

---

## Conventions & Rules

1. **One BLoC/Cubit per feature page**; never share a BLoC between pages unless it is app-wide (`AppCubit`).
2. **No business logic in widgets or pages** — delegate everything to the BLoC.
3. **Domain entities are pure Dart** — no Flutter imports, no JSON annotations.
4. **Models live in `data/` only** — never import a model from `domain/` or `presentation/`.
5. **Use `Either<Failure, T>`** for all repository return types.
6. **Use `remoteRequest()` / `localRequest()`** wrappers from `BaseRepositoryImpl`; never write raw try/catch in a repository impl.
7. **Pagination** uses `PaginationStateData<T>` (built_value) in the state and `PaginationList<T>` from the repository.
8. **Text scaling is locked to 1.0** (`TextScaler.linear(1.0)`) — do not rely on system font scaling.
9. **Screen orientation is portrait-only** — do not design for landscape.
10. **Always annotate repository impls with `@LazySingleton(as: AbstractClass)`** and data source impls similarly so injectable picks them up.
11. **Constants go in `Constant` class** (`lib/core/utils/constant.dart`); endpoints go in `Endpoints` class.
12. **Local storage keys** are stored as constants (use `LocalStorageKeys` class pattern).
13. **Use relative imports for project files** — import classes using paths like `../../class_name.dart` (or the correct relative depth) instead of package imports where applicable.
14. **Use cases are optional for feature-specific logic** — only create a use case class if the logic is shared across multiple features (e.g., `AddToFavoriteUseCase`, `FollowUseCase`). If the logic is specific to a single feature, call the repository directly from the BLoC/Cubit without a use case layer.
15. **All values must be explicitly typed** — always declare the data type for variables, parameters, return types, and fields. Never use `var`, `dynamic`, or inferred types where an explicit type can be provided. (e.g., use `String name = 'John'` not `var name = 'John'`; use `Future<void>` not `Future`; use `List<String>` not `List`).
16. **Use `typedef` for long or complex type signatures** — if a return type or value type is excessively long or used in multiple places, define a `typedef` with a clear, descriptive name to improve readability and reduce repetition. (e.g., use `typedef OffersResult = Either<Failure, List<OfferEntity>>;` instead of repeating `Either<Failure, List<OfferEntity>>` everywhere). Place shared typedefs in `lib/core/utils/typedefs.dart` and feature-specific ones at the top of the relevant file.
17. **Never use functions that return widgets** — always create a `StatelessWidget` class instead of a method/function that returns a `Widget`. Functions that return widgets bypass Flutter's widget tree optimization and make code harder to test and reuse. (e.g., use `class MyButton extends StatelessWidget { ... }` not `Widget _buildButton() { ... }`).
18. **Never create private widgets** — all widgets must be public classes in their own files. Private widgets (prefixed with `_`) cannot be reused, tested in isolation, or found by other developers. Every widget, no matter how small, must be a named public `StatelessWidget` in its own file inside the `widgets/` folder.

---

## Feature Scaffolding

Use the provided `create_feature.sh` script to scaffold a new feature:

```bash
./create_feature.sh "feature_name"
```

This creates the full `data/domain/presentation` folder structure with boilerplate Bloc, Event, State, Repository, DataSource, and Page files.

---

## State Management

- **BLoC / Cubit** via `flutter_bloc` and `bloc`.
- App-wide state lives in `AppCubit` (language, auth, cart count, profile).
- Feature BLoCs are `@injectable` and created per-route in `AppRouter`.
- Use `BlocStatus` enum (`initial`, `loading`, `success`, `error`) to represent async operation states in every state class.
- Use `bloc_concurrency` (`droppable()`, `restartable()`) to control event processing for concurrent events.

### State / Event Modeling

All BLoC **states** and **events** use [`built_value`](https://pub.dev/packages/built_value):

```dart
// State
abstract class MyFeatureState
    implements Built<MyFeatureState, MyFeatureStateBuilder> {
  BlocStatus get status;
  MyFeatureState._();
  factory MyFeatureState([void Function(MyFeatureStateBuilder) updates]) =
      _$MyFeatureState;
  factory MyFeatureState.initial() => MyFeatureState(
        (b) => b..status = BlocStatus.initial,
      );
}

// Event
abstract class MyEvent extends MyFeatureEvent
    implements Built<MyEvent, MyEventBuilder> {
  MyEvent._();
  factory MyEvent([void Function(MyEventBuilder) updates]) = _$MyEvent;
}
```

- Always add `part '<file>_state.g.dart';` / `part '<file>_event.g.dart';`
- Run `flutter pub run build_runner build --delete-conflicting-outputs` after changes.

---

## Dependency Injection

- Uses **`get_it`** + **`injectable`**.
- Register services with `@lazySingleton`, `@injectable`, `@singleton` etc.
- External dependencies (Dio, SharedPreferences, etc.) are registered in the `@module` class inside `lib/core/di/di.dart`.
- Repository impls use `@LazySingleton(as: AbstractRepository)`.
- After changing DI annotations run: `flutter pub run build_runner build --delete-conflicting-outputs`

---

## Networking

- **Dio** is the HTTP client, configured in `DioFactory`.
- Three interceptors are applied in order:
  1. `AuthInterceptor` — injects `Authorization`, `Accept-Language`, platform, version, device ID headers; blocks requests when offline.
  2. `LoggingInterceptor` — logs requests/responses.
  3. `RefreshTokenInterceptor` — handles 401 token refresh.
- All endpoints are **static constants/methods** in `Endpoints` class.
  - Base URL: `Endpoints.baseURL` = `$serverURL/api/v1`
  - Dynamic paths use static methods, e.g. `Endpoints.productDetails(id)`.
- API responses are wrapped in `BaseModel` (`success`, `message`, `data`, `links`, `meta`).

### Data Source Pattern

```dart
// Abstract
abstract class MyRemoteDataSource extends BaseRemoteDataSource {
  Future<MyModel> getData();
}

// Implementation (registered via @LazySingleton)
@LazySingleton(as: MyRemoteDataSource)
class MyRemoteDataSourceImpl extends BaseRemoteDataSourceImpl
    implements MyRemoteDataSource {
  MyRemoteDataSourceImpl(super._dio);
  
  @override
  Future<MyModel> getData() async { ... }
}
```

---

## Repository Pattern

- Abstract: `abstract class MyRepository extends BaseRepository { ... }`
- Implementation: `class MyRepositoryImpl extends BaseRepositoryImpl implements MyRepository { ... }`
- Always delegate to `remoteRequest()` or `localRequest()` from `BaseRepositoryImpl`:
  ```dart
  Future<Either<Failure, MyEntity>> getData() async =>
      await remoteRequest(
        () async => await _dataSource.getData(),
        (model) => model.toDomain(),
      );
  ```
- Return type is always `Either<Failure, T>` (from **dartz**).

---

## Error Handling

- Exceptions: `ServerException`, `CacheException`, `NetworkException`, `GeneralException` (all extend `GenericExceptions`).
- Failures: `ServerFailure`, `CacheFailure`, `NetworkFailure`, `GeneralFailure` (all extend `Failure`).
- Conversion: `ErrorHandler.handleFailureError(e)` → `Failure`; `ErrorHandler.handleExceptionError(e)` → `GenericExceptions`.
- `BaseRepositoryImpl` catches all errors and returns `Left(failure)` automatically.

---

## Models

- **Remote API models** use `json_serializable` use `@JsonSerializable(checked: true, createToJson: false)` for response models and `@JsonSerializable(checked: true, createFactory: false)` for request models.

  ```dart
  @JsonSerializable(checked: true, createToJson: false)
  class MyModel {
    @JsonKey(name: 'field_name')
    final String fieldName;
    // ...

    factory MyModel.fromJson(Map<String, dynamic> json) => _$MyModelFromJson(json);

    MyEntity toDomain() => MyEntity(fieldName: fieldName);
  }
  ```

- Every model must have a `toDomain()` method converting it to the domain entity.
- Shared core models (e.g. `BaseModel`, `MediaModel`, `MetaModel`) live in `lib/core/models/`.
- Every model field must be annotated with `@JsonKey(name: 'field_name')` using the exact backend key.
- Always add `part '<file>_model.g.dart';`
- Don't create toJson() for model classes.
- Don't create fromJson() for request models.

### Request Model

```dart
part 'request.g.dart';

@JsonSerializable(checked: true, createFactory: false)
class Request {
  @JsonKey(name: 'email')
  final String email;
  Request({required this.email});
  Map<String, dynamic> toJson() => _$RequestToJson(this);
}
```

- Every request model field must be annotated with `@JsonKey(name: 'field_name')` using the exact backend key.
- Always add `part '<file>_request.g.dart';`

---

## 📋 Logic Layer

### Helper Classes (Optional)

**Create Helper class ONLY if:**

- Business logic is reused in multiple methods
- Complex calculations/transformations needed
- Validation logic is extensive

**Structure:**

```dart
class FeatureHelper {
  FeatureHelper._(); // Private constructor

  // Validation
  static bool isValidEmail(String email) => email.contains('@') && email.length > 5;

  // Transformation
  static String formatPhone(String phone) => phone.replaceAll(RegExp(r'\D'), '');

  // Calculation
  static double calculateTotal(List<Item> items) => items.fold(0, (sum, item) => sum + item.price);

  // Parsing
  static DateTime? parseDate(String? date) => date != null ? DateTime.tryParse(date) : null;
}
```

**Rules:**

- ✅ Static methods only (no state)
- ✅ Pure functions (same input = same output)
- ✅ Short, focused methods (max 10 lines)
- ✅ Meaningful names (`isValid`, `format`, `calculate`, `parse`)
- ✅ Place in `core/utils/` folder
- ❌ NO business logic that belongs in Cubit
- ❌ NO API calls or side effects

---

## Routing

**MUST** use context extensions (NEVER use Navigator directly)

- Named routes only. All route strings are constants in `Routes` class.
- `AppRouter.generateRoute()` creates BLoC instances and provides them via `BlocProvider` / `MultiBlocProvider` before pushing the page.
- Use `context.pushNamed(Routes.xxx)` / `context.pushNamedAndRemoveUntil(...)` from the navigation extension.

```dart
import '../../../core/extensions/navigation.dart';

// ✅ CORRECT
context.pushNamed(Routes.home);
context.pushReplacementNamed(Routes.login);
context.pushNamedAndRemoveUntil(Routes.home, predicate: (_) => false);
context.pop();

// ❌ WRONG
Navigator.pushNamed(context, Routes.home);
Navigator.pushReplacementNamed(context, Routes.login);
Navigator.pop(context);
```

---

## Widgets

- ✅ **ONLY** `StatelessWidget`
- ❌ **NEVER** `StatefulWidget`
- ❌ **NEVER** use functions that return widgets
- ❌ **NEVER** create private widgets (prefixed with `_`) — all widgets must be public classes in their own files
- ✅ **ALL** logic → BLoC/Cubit; widgets are pure UI
- ✅ One widget per file (`login_form.dart`)
- ✅ **MUST** separate screen sections into widgets

**Rules:**

- Separate each major section (header, content, footer, forms, lists,button groups, etc.) of a page into its own widget.
- Each widget in `widgets/` folder
- Max 50-70 lines per widget
- Improves readability and reusability

---

## Theming & Styling

- **Never** use hardcoded colors — always use `ColorsManager.<colorName>`.
- **Never** use hardcoded `TextStyle` — always use `TextStyles.font<size><Color><Weight>` (e.g. `TextStyles.font14WhiteMedium`).
- **Never** use hardcoded font weights — always use `FontWeightHelper.<weight>`.

---

## Localisation

- Flutter Intl (`intl_utils`). Supported locales: Arabic (`ar`) and English (`en`).
- ARB files: `lib/l10n/intl_ar.arb`, `lib/l10n/intl_en.arb`.
- Access strings via `S.current.<key>` or `S.of(context).<key>`.
- **Never** hardcode user-facing strings. Always add them to both ARB files.
- After editing ARB files, regenerate with: `dart run intl_utils:generate`

---

## Assets

- Declared in `pubspec.yaml` under `assets/` (images, icons, fonts, animations).
- Access via generated `Assets` class in `lib/generated/assets.dart`.
- **Never** use raw asset path strings. Always use `Assets.<category>.<name>`.

---

## Enums

All shared enums live in `lib/core/utils/app_enums.dart`:

- `BlocStatus` — state of async operations.
- `Language` — `en`, `ar`.
- `Gender`, `VerifyType`, `MediaType`, `NotificationType`, `AdvertisementType`, `ItemType`, `PaymentStatus`, etc.
- Use `@JsonValue(...)` annotations for enums that are serialised/deserialised.
- When creating enums, always add a dedicated mapper (extension or helper) to convert **String → Enum** and **Enum → String**.
  - Prefer an `extension` on the enum for `toValue()` / `toJsonValue()`.
  - Add a static/parser helper for `fromValue(String value)` with a safe fallback.
  - Reuse these mappers across models/repositories instead of duplicating switch logic.

---

## Import Order

```dart
// 1. Dart imports
import 'dart:async';

// 2. Flutter imports
import 'package:flutter/material.dart';

// 3. Packages (alphabetical)
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:get_it/get_it.dart';

// 4. Project files (alphabetical)
import '../logic/feature_cubit.dart';
import 'widgets/feature_form.dart';
```

---

## Performance

- ✅ `const` constructors everywhere possible
- ✅ `ListView.builder` for long lists
- ✅ Dispose controllers in `close()`
- ✅ Avoid unnecessary rebuilds
- ✅ Use BlocSelector where possible

---

## Code Generation

Run after any change to generated files:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

This regenerates:

- `*.g.dart` (built_value states/events, json_serializable models)
- `di.config.dart` (injectable DI config)

---

## ✅ Quick Checklist

Use this before submitting any code.

### 🏗️ Architecture

- [ ] Feature follows `data / domain / presentation` folder structure
- [ ] Domain entities are pure Dart — no Flutter/JSON imports
- [ ] Models live only in `data/` layer

### 🧠 State Management

- [ ] One BLoC/Cubit per feature page
- [ ] State and Event use `built_value` with `part '*.g.dart'`
- [ ] `BlocStatus` used for all async states
- [ ] `bloc_concurrency` used for concurrent events
- [ ] No business logic in widgets or pages — all in BLoC or Cubit

### 💉 Dependency Injection

- [ ] Repository impl annotated with `@LazySingleton(as: AbstractRepository)`
- [ ] Data source impl annotated with `@LazySingleton(as: AbstractDataSource)`
- [ ] `build_runner` re-run after DI annotation changes

### 🌐 Networking

- [ ] Endpoint added as a static constant/method in `Endpoints` class
- [ ] Data source uses `performGetRequest` / `performPostRequest` from `BaseRemoteDataSourceImpl`
- [ ] Response parsed via `BaseModel` wrapper

### 🗄️ Repository

- [ ] Uses `remoteRequest()` or `localRequest()` — no raw try/catch
- [ ] Returns `Either<Failure, T>` (dartz)
- [ ] Model converted to domain entity via `toDomain()`

### 📦 Models

- [ ] Annotated with `@JsonSerializable(checked: true, createToJson: false)` for response models
- [ ] Annotated with `@JsonSerializable(checked: true, createFactory: false)` for request models
- [ ] Every field has `@JsonKey(name: 'backend_key')`
- [ ] Has `toDomain()` method
- [ ] Has `part '<file>_model.g.dart'` / `part '<file>_request.g.dart'`

### 🧭 Routing

- [ ] Route string added to `Routes` class
- [ ] BLoC provided in `AppRouter.generateRoute()`
- [ ] Navigation uses context extension (`context.pushNamed`, `context.pop`, etc.) — no `Navigator` directly

### 🎨 Widgets & UI

- [ ] `StatelessWidget` only — no `StatefulWidget`
- [ ] Page broken into focused sub-widgets (max 50–70 lines each)
- [ ] Sub-widgets placed in `widgets/` folder
- [ ] No hardcoded colors — uses `ColorsManager`
- [ ] No hardcoded `TextStyle` — uses `TextStyles`
- [ ] No hardcoded font weights — uses `FontWeightHelper`
- [ ] No raw asset path strings — uses `Assets.<category>.<name>`
- [ ] No hardcoded user-facing strings — added to both ARB files
- [ ] No function returning widgets — use `StatelessWidget` classes instead
- [ ] No private widgets (prefixed with `_`) — all widgets are public classes in their own files

### 🔢 Enums

- [ ] Added to `lib/core/utils/app_enums.dart`
- [ ] Has `extension` with `toValue()` / `toJsonValue()`
- [ ] Has `fromValue(String value)` parser with safe fallback
- [ ] `@JsonValue(...)` annotation added if serialised

### 📥 Imports

- [ ] Dart → Flutter → Packages → Project files (alphabetical within each group)
- [ ] Project files use relative imports

### ⚡ Performance

- [ ] `const` constructors used wherever possible
- [ ] Long lists use `ListView.builder`
- [ ] Controllers/streams disposed in `close()`

### 🔧 Code Generation

- [ ] `flutter pub run build_runner build --delete-conflicting-outputs` run after any `*.g.dart` changes

### 🔁 Use Cases

- [ ] Use case created **only** if logic is shared across multiple features
- [ ] Single-feature logic calls the repository directly from the BLoC/Cubit — no use case class created
- [ ] Use case class name clearly reflects the shared action (e.g., `AddToFavoriteUseCase`, `FollowUseCase`)

### 🔡 Explicit Typing

- [ ] No `var` used — all variables explicitly typed (e.g., `String name = 'John'`)
- [ ] No `dynamic` used — all parameters and fields have explicit types
- [ ] All return types explicitly declared (e.g., `Future<void>`, `List<String>`)
- [ ] No inferred types where an explicit type can be provided
- [ ] `typedef` used for long or complex type signatures to improve readability (e.g., `typedef OffersResult = Either<Failure, List<OfferEntity>>;`)
- [ ] Shared typedefs placed in `lib/core/utils/typedefs.dart` and feature-specific typedefs placed at the top of the relevant file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MohammadAssaf10)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MohammadAssaf10)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
