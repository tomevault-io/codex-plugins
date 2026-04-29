# SafeVision AI Agent Instructions

## Project Overview

SafeVision is a Flutter mobile app that helps visually impaired users identify objects
in their surroundings using a real-time camera feed with a YOLOv8 model via TFLite.

**Target Platforms:** Android, iOS, Windows, macOS, Linux, Web

## Architecture Pattern: Clean Architecture + BLoC

The codebase follows Clean Architecture split into three layers:

- **Presentation** (`presentation/`): UI, widgets, BLoC event handlers
- **Domain** (`domain/`): Business logic, entities, abstract repositories
- **Data** (`data/`): Data sources, concrete repository implementations

**Feature Structure:**
```
lib/features/{feature_name}/
├── presentation/  (pages, widgets, bloc)
├── domain/        (entities, repositories, usecases)
└── data/          (datasources, repositories)
```

## Core Dependencies & Patterns

### State Management: Flutter BLoC
- Uses `flutter_bloc` (v9.1.1) for state management
- Event-driven: UI dispatches events → BLoC processes → state updates
- Example: [lib/features/tts/presentation/bloc/tts_event.dart](../lib/features/tts/presentation/bloc/tts_event.dart)

### Key External Services

1. **AI Model:** YOLOv8 TFLite (`tflite_flutter: ^0.11.0`)
   - Model path: `assets/models/yolov8n_safevision.tflite`
   - Labels: `assets/models/labels.txt`
   - Inference runs in a separate Dart isolate to avoid blocking the UI thread
   - Initialization: [lib/features/detection/data/datasources/detection_local_datasource_impl.dart](../lib/features/detection/data/datasources/detection_local_datasource_impl.dart)

2. **Camera:** Real-time capture for object detection
   - Managed by `CameraService` in [lib/core/services/camera_service.dart](../lib/core/services/camera_service.dart)
   - Resolution: Medium preset for a balance of speed vs. quality
   - Frame rate: 6 FPS (configured via `AppConstants.activeInferenceFps`)

3. **Text-to-Speech (TTS):** Voice feedback for accessibility
   - Service: [lib/features/tts/data/datasources/tts_service.dart](../lib/features/tts/data/datasources/tts_service.dart)
   - Helper: [lib/core/utils/voice_helper.dart](../lib/core/utils/voice_helper.dart)

4. **Permissions:** Camera access is required at startup
   - Utility: [lib/core/utils/permission_handler.dart](../lib/core/utils/permission_handler.dart)

## Critical Implementation Details

### Detection Flow

1. `CameraService.startImageStream` sends frames via the `onFrame` callback
2. `onFrame` dispatches `DetectionFrameReceived` to `DetectionBloc`
3. `DetectionBloc` calls the `DetectionObjectFromFrame` use case
4. Inference runs in an isolate — the model outputs `List<DetectionObject>`
5. Bounding boxes are rendered via `BoundingBoxPainter` (custom painter)
6. TTS announces detected objects via `TtsBloc` → `TtsService`

**Frame Lock Pattern:** `CameraService` holds a lock `_isProcessingFrame`.
`DetectionFrameReceived` carries an `onDone` callback that MUST be called
after inference completes (in a `finally` block) to release the lock.
If `onDone` is never called, the camera stream will stall permanently.

### Recognition Entity
```dart
class Recognition {
  final int id;
  final String label;
  final double score;
  final BoundingBox location; // Bounding box for UI rendering
}
```

### UseCase Pattern

`CloseModelUsecase` and `LoadModelUsecase` implement
`UseCase<void, NoParams>` — they do not expose convenience methods (`close()`,
`load()`). Always call them via `usecase.call(const NoParams())`.

`DetectionObjectFromFrame` does not implement `UseCase` because it requires
two parameters — it is called directly from `DetectionBloc`.

### UI Accessibility Requirements

- **High Contrast Theme:** [lib/config/theme/app_theme.dart](../lib/config/theme/app_theme.dart)
- **Large Buttons:** Minimum 80dp height, 24pt font size
- **Voice Feedback:** All key actions announced via TTS
- **Haptic Feedback:** Vibration when a dangerous object is detected (only when
  TTS actually passes the cooldown — not on every event)

### BoundingBoxPainter

- `shouldRepaint` uses an O(1) `version` counter — does not iterate the list
- TextPainter is cached per label in the static `_textCache`
- Call `dispose()` when the painter is no longer in use to clear its cache entry
- In tests: call `BoundingBoxPainter.clearCacheForTesting()` in `tearDown`
  to reset the static cache between test groups

## Build & Development

### Flutter Commands
```bash
flutter pub get           # Install dependencies
flutter analyze           # Check code quality
flutter build apk         # Build Android APK
flutter build ios         # Build iOS app
flutter test              # Run all tests
flutter test --coverage   # Run with coverage report
```

### Code Quality Standards
- Lint rules: [analysis_options.yaml](../analysis_options.yaml) (extends flutter_lints)
- Dart version: >=3.0.0 <4.0.0
- Coverage target: minimum 80% (enforced in CI)
- Comments describe ROLE and INVARIANTS — not change history

### Asset Configuration
- Models, icons, images, and sounds are declared in [pubspec.yaml](../pubspec.yaml)
- Runtime asset loading: `rootBundle.loadString()` for labels,
  `Interpreter.fromBuffer()` for the model (via TransferableTypedData to the isolate)

## Common Workflows

### Adding a New Object Detection Feature
1. Create the feature folder: `lib/features/{feature}/` with domain/data/presentation
2. Define an entity based on `DetectionObject`
3. Add BLoC events in `presentation/bloc/{feature}_event.dart`
4. Implement the repository in `domain/repositories/`
5. Update the AI model in `assets/models/` if required

### Fixing Permission Issues
- Check [lib/core/utils/permission_handler.dart](../lib/core/utils/permission_handler.dart)
- `AppPermissionHandler.requestCamera()` throws `PermissionException` if denied
- Call and await it before `CameraService.initialize()`

### Accessibility Improvements
- Modify the theme in [lib/config/theme/app_theme.dart](../lib/config/theme/app_theme.dart)
- Add TTS calls via `TtsBloc.add(TtsSpeak(...))` for new actions
- Test with high contrast + large font settings

## Known Conventions

- **Code Comments in English:** All code comments explain application-specific
  context and must be written in English
- **Task References:** Task IDs (SV-XX) are included in commit messages and
  branch names — not inside source code
- **Singleton vs LazySingleton:** Services requiring warm-up
  (`TtsService.initialize()`) use `registerSingleton`. BLoC and objects that
  do not need early initialization use `registerLazySingleton`

## CI/CD

GitHub Actions pipeline at `.github/workflows/ci.yml`:
1. `analyze` — format check + flutter analyze
2. `test` — unit + widget tests, coverage ≥ 80%
3. `build-android` — debug + release APK/AAB
4. `build-ios` — no-codesign build
5. `security` — dart pub audit + secret scan
6. `release` — GitHub Release (main branch only)

Flutter version: stable channel (not pinned to a specific version to
avoid build failures when a version is unavailable on the runner).

---
> Source: [nguyenkien06032005-design/TraingModel](https://github.com/nguyenkien06032005-design/TraingModel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
