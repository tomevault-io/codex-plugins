<!-- Copilot/AI agent instructions for this Flutter feedback system -->
# AI Coding Agent Project Guide

## Overview
This is a Flutter app for managing feedbacks, built around a single main file ([lib/main.dart](../lib/main.dart)) with additional pages and services under `lib/`. The app uses a BottomNavigationBar for navigation between Home, Exercises, Feedbacks, Statistics, and Materials. Feedback data is loaded from Firestore and cached in-memory (`feedbackStorage`).

## Key Files & Structure
- [lib/main.dart](../lib/main.dart): App entrypoint, navigation, all main pages, feedback model, Firestore integration, and most business logic.
- [pubspec.yaml](../pubspec.yaml): Declares dependencies (Firebase, shared_preferences, etc.).
- `lib/pages/`, `lib/services/`: Place new UI pages and service logic here for non-trivial features.
- Platform folders (`android/`, `ios/`, etc.) are generated—**never edit plugin registrant files directly**.

## Data Flow & Architecture
- **Feedbacks**: Modeled by `FeedbackModel`, stored in Firestore (`feedbacks` collection), loaded into global `feedbackStorage`.
- **User roles**: `currentUser` is set after Firebase Auth; role-based access (Admin/Instructor) controls data visibility and actions.
- **Navigation**: Uses nested `Navigator` widgets and `MaterialPageRoute`. Add new pages using this pattern.
- **State**: Most state is managed in-memory; no Redux/Provider. Use local state or extract to services as needed.

## Developer Workflows
- **Build**: `flutter pub get`, then `flutter run -d <device>`
- **Test**: `flutter test` (see [test/widget_test.dart](../test/widget_test.dart))
- **Analyze**: `flutter analyze` for static checks
- **Export**: Admins can export feedbacks to XLSX via UI (see `FeedbackExportService`)

## Project Conventions
- **Hebrew UI**: All UI strings are in Hebrew; maintain UTF-8 and RTL layout.
- **Const Widgets**: Prefer `const` constructors for widgets.
- **Small PRs**: Extract new logic to `lib/services/` or `lib/widgets/` instead of growing `main.dart`.
- **No direct DB/network in widgets**: Use service wrappers for persistence/networking.
- **Feedback criteria**: Scoring uses discrete values (1, 3, 5); keep this logic consistent.

## Integration & Extensibility
- **Persistence**: To add new storage, update `pubspec.yaml` and create a service in `lib/services/` (see Firestore usage in `main.dart`).
- **Voice Assistant**: Voice command logic is in `voice_assistant.dart` and integrated in `MainScreen`.
- **Export/Import**: Use `feedback_export_service.dart` for XLSX export; follow this pattern for new export features.

## Examples
- **Feedback submission**: See `FeedbackFormPage` in [lib/main.dart](../lib/main.dart).
- **Feedback display**: See `FeedbacksPage` and `FeedbackDetailsPage` in [lib/main.dart](../lib/main.dart).
- **Statistics**: See `StatisticsPage` and related pages for filtering/aggregation patterns.

## Do / Don't Checklist
- **Do**: Run `flutter analyze` and `flutter test` before PRs.
- **Do**: Add new files under `lib/` for new features/services.
- **Don't**: Edit generated platform files or plugin registrants.
- **Don't**: Replace Firestore/in-memory feedback logic without migration/tests.

---
If any section is unclear or missing, please request clarification or suggest additions for future updates.

---
> Source: [yotalon-alt/magen](https://github.com/yotalon-alt/magen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
