# Copilot Instructions for laundry-app

## Project Overview
This is a Flutter application for a laundry business, built with cross-platform support. It uses a feature-first Clean Architecture approach.

## Tech Stack
- **Framework:** Flutter
- **State Management & DI:** Riverpod (`flutter_riverpod`)
- **Routing:** GoRouter
- **Backend/BaaS:** Supabase (`supabase_flutter`)
- **Functional Programming:** fpdart (used for error handling e.g., `Either<Failure, Success>`)
- **UI/Theming:** Google Fonts, FL Chart for dashboards

## Architecture & Project Structure
The project strictly follows a **Feature-First Clean Architecture**:

- `lib/core/`: Contains globally shared code.
  - `api/`: Base API layers or HTTP clients.
  - `config/`: App configuration.
  - `error/`: Generalized error/failure classes for `fpdart`.
  - `presentation/`: Global UI components.
  - `providers/`: Global Riverpod providers (e.g., Supabase client initialization).
  - `router/`: GoRouter setup and configuration.
  - `utils/`: Helpers and utility functions.
  - `widgets/`: Reusable generalized widgets.

- `lib/features/`: Contains modularized business domains (e.g., `authentication`, `customers`, `dashboard`, `transactions`, etc.). Each feature should ideally be subdivided into:
  - `domain/`: Business logic, Entities, and Repository Interfaces.
  - `data/`: Supabase API calls, DTOs (Data Transfer Objects), and Repository implementations.
  - `presentation/`: Riverpod providers acting as ViewModels/Controllers, alongside standard Flutter UI Widgets/Screens.

## Coding Conventions
1. **Error Handling:** Use `fpdart`'s `Either` type to return failures or successes from repositories and services instead of relying on try-catch blocks in the UI layer. All failures should extend a base `Failure` class defined in `lib/core/error/`.
2. **State Management:** Use Riverpod for managing state. Prefer `Notifier` and `AsyncNotifier` for complex state over simple `StateProvider`. Keep UI components dumb and pass state and callbacks from providers.
3. **Routing:** All navigation should go through `go_router` paths defined in `lib/core/router/`. Avoid using `Navigator` directly.
4. **Backend Interaction:** All data access should happen via the repository pattern, communicating with Supabase. Do not call Supabase directly from the presentation layer.
5. **UI Structure:** Break down complex screens into smaller, reusable widgets within the feature's `presentation/widgets/` directory.

## Build and Run Commands
- **Install Dependencies:** `flutter pub get`
- **Run App:** `flutter run`
- **Run Tests:** `flutter test`
- **Static Analysis:** `flutter analyze`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Khafidhfuadi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Khafidhfuadi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
