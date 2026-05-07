# Copilot instructions for calsync

Concise guide for AI agents: focus on the notification → parse → calendar flow and follow project conventions (privacy, tests, and background dispatching).

## Big picture
- Flow: NotificationListener (`NotificationMonitorService`) → `NotificationProcessor` → sentence extraction (`DateTimeParser.extractAllSentencesContainingDate`) → parsing (`DateTimeParser` rule set, fallback `TimeNLPAdapter` when enabled) → calendar write (`CalendarHelper.insertEvent`) → user confirmation (`NotificationUtils.sendEventCreated`).
- Entry points you’ll touch most: `NotificationMonitorService`, `NotificationProcessor`, `DateTimeParser`, `TimeNLPAdapter`, `CalendarHelper`, `NotificationUtils`.

## Key conventions & behaviors
- Parsing API: `DateTimeParser.parseDateTime(sentence)` and `DateTimeParser.parseDateTime(context, sentence, baseMillis)` → returns `ParseResult(startMillis, endMillis?, title?, location?)`.
- Prefer-future: `SettingsStore.getPreferFutureOption()` (0=auto, 1=prefer future, 2=disable).
- Defaults: missing time → 09:00; evening tokens map to 19:00; relative expressions ("3天后", "还剩X小时") supported.
- Title/location: lightweight extraction via `JiebaWrapper`; keep titles short and heuristic-driven.
- Privacy: app has NO INTERNET permission — do not add network code or log full notification texts in production.

## Tests & developer workflow
- Environment: JDK 21, AGP 8.13, Kotlin 2.0.21; run via Gradle wrapper.
- Common commands:
  - ./gradlew assembleDebug
  - ./gradlew assembleRelease
  - ./gradlew :app:testDebugUnitTest --no-daemon --info
  - ./gradlew connectedDebugAndroidTest
  - If OOM: set `GRADLE_OPTS=-Xmx3g`.
  - Lint: `./gradlew :app:lintDebug` and view `app/build/reports/lint-results-debug.html`.
  for FOSS flavor:
  - ./gradlew assembleFossDebug
  - ./gradlew :app:testFossDebugUnitTest --no-daemon --info
  
- Testing pattern: parser rules require unit tests with a frozen base time (see `app/src/test/*/DateTimeParserTest*.kt`). Use `DateTimeParser.parseDateTime(..., baseMillis)` or test helpers like `parseWithBase`.
- To run a single test class: `./gradlew :app:testDebugUnitTest --tests "top.stevezmt.calsync.DateTimeParserTest*"`.

## Safe change patterns for agents
- Parser changes: extend `DateTimeParser` first, add targeted unit tests under `app/src/test/java/top/stevezmt/calsync/`, assert start/end/timezone behavior with a fixed base. Only use `TimeNLPAdapter` as fallback and guard with `SettingsStore.isTimeNLPEnabled()`.
- Notification flow: do heavy work on background dispatcher; never block `NotificationListenerService` callbacks. Use `NotificationUtils.ensureChannels` and `safeNotify` for notifications.
- Calendar writes: prefer `SettingsStore.getSelectedCalendarId()` if set; otherwise pick first visible calendar and always set timezone when inserting.
- Confirmation UI: confirmation notifications include delete-action (`EventActionReceiver`) and deep links to created events — update both when changing behavior.

## Integration & third-party
- `timenlp/internal/*`: regex-based normalizer; used only via `TimeNLPAdapter`.

- ML Kit (full flavor)
  - Implementation: `app/src/full/java/top/stevezmt/calsync/MLKitStrategy.kt` (FOSS stub: `app/src/foss/java/.../MLKitStrategy.kt` returns null).
  - Dependency: added as `fullImplementation(libs.mlkit.entity.extraction)` (see `gradle/libs.versions.toml` and `app/build.gradle.kts`).
  - Behavior: uses Google ML Kit Entity Extraction (`EntityExtraction`) and `DateTimeEntity`; calls `extractor.downloadModelIfNeeded()` (via `Tasks.await`) which may trigger a model download via Google Play Services on first run.
  - Testing & agent guidance:
    - **Do not** invoke real ML Kit in unit tests; use the FOSS flavor or mock `EntityExtraction` / `EntityExtractor` in instrumentation tests.
    - Guard model downloads with try/catch and avoid blocking the UI thread — ML Kit calls should run on background dispatcher as parser strategies are run there.
    - If you change ML Kit usage, update settings/UI hints where we check for `com.google.android.gms` presence (see `SettingsActivity.kt`).

- `third_party/llama.cpp` (native module)
  - Location: `third_party/llama.cpp/` — this is a full native C/C++ project with its own CI and `.github/copilot-instructions.md` (read that file before changing native code).
  - Android integration: `app/src/main/cpp/CMakeLists.txt` adds llama.cpp as a subdirectory and builds a `llama_jni` shared library; NDK version is pinned in `app/build.gradle.kts` (`ndkVersion = "29.0.13599879"`).
  - Build notes: changes to `llama.cpp` may require CMake/NDK updates and are often platform-specific; prefer minimal, well-tested changes. Use CPU-only defaults for Android builds; Vulkan/CUDA/Metal require extra toolchains and hardware.
  - Testing & agent guidance:
    - Run native builds with CMake and the Android NDK; ensure `LLAMA_DIR` exists (CMake will fail otherwise).
    - For changes inside `third_party/llama.cpp`, follow its contribution guide and run its CI locally (`ci/run.sh`) or the specific CMake build commands documented in its `.github` docs.
    - Avoid modifying vendored code unless necessary — prefer JNI glue (`app/src/main/cpp/llama_jni.cpp`) for app-specific logic.



## Quick examples
- Extract: `DateTimeParser.extractAllSentencesContainingDate(ctx, title + "。" + content)`
- Parse with base: `DateTimeParser.parseDateTime(ctx, sentence, baseMillis)`
- Insert & notify: `CalendarHelper.insertEvent(ctx, title, desc, start, end, location)` → `NotificationUtils.sendEventCreated(...)`

---
If anything here is unclear or need more examples (tests, failing-case patterns, or ROM quirks), create a new task to investigate and document it.

---
> Source: [SteveZMTstudios/calsync](https://github.com/SteveZMTstudios/calsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
