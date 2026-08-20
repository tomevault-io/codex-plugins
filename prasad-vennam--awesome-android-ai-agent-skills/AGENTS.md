# Android AI Agent Skills

You are an expert Android developer. When the user asks you to implement a specific feature or pattern, you MUST refer to the local `skills/` directory for exact architectural guidelines and guardrails. Below is an index of locally available knowledge:

## android-accessibility
- **Scope**: Use when improving app accessibility, ensuring WCAG compliance, or fixing TalkBack navigation for Android.
- **File Path**: `skills/android-accessibility/SKILL.md`

## android-adaptive-layouts
- **Scope**: Use when supporting tablets, foldables, and different screen sizes in Android apps with Jetpack Compose.
- **File Path**: `skills/android-adaptive-layouts/SKILL.md`

## android-adb-terminal
- **Scope**: Use when interacting with Android emulators or physical devices via ADB (Android Debug Bridge).
- **File Path**: `skills/android-adb-terminal/SKILL.md`

## android-background-execution
- **Scope**: Expert guidance on implementing and managing background tasks in modern Android applications (API 30+).
- **File Path**: `skills/android-background-execution/SKILL.md`

## android-bluetooth-ble
- **Scope**: Handling Bluetooth Low Energy, flow wrappers, and Android 12+ permissions.
- **File Path**: `skills/android-bluetooth-ble/SKILL.md`

## android-build-verification
- **Scope**: MANDATORY verification skill to run after EVERY file change. Automatically triggers an Android build (assembledDebug) to check for compilation or schema errors.
- **File Path**: `skills/android-build-verification/SKILL.md`

## android-camerax-mlkit
- **Scope**: Seamless integration of CameraX with Google ML Kit for vision tasks.
- **File Path**: `skills/android-camerax-mlkit/SKILL.md`

## android-ci-cd-expert
- **Scope**: Use when setting up continuous integration/deployment (GitHub Actions), Gradle Play Publisher, Fastlane, or R8 Proguard rules for Release builds.
- **File Path**: `skills/android-ci-cd-expert/SKILL.md`

## android-clean-architecture
- **Scope**: Use when designing the architecture, data layer, domain models, or business logic for Android apps.
- **File Path**: `skills/android-clean-architecture/SKILL.md`

## android-code-review-expert
- **Scope**: Use when performing a code review for Android pull requests, focusing on architecture, memory safety, and UI excellence.
- **File Path**: `skills/android-code-review-expert/SKILL.md`

## android-coroutines-expert
- **Scope**: Use to enforce safe concurrency, correct Dispatcher usage, and proper Channel vs StateFlow patterns in ViewModels.
- **File Path**: `skills/android-coroutines-expert/SKILL.md`

## android-custom-drawing-canvas
- **Scope**: Use when creating custom UI components, charts, shaders, or hardware-accelerated drawing in Android with Jetpack Compose Canvas.
- **File Path**: `skills/android-custom-drawing-canvas/SKILL.md`

## android-design-system-m3
- **Scope**: Use when creating Material 3 design systems, custom themes, or dynamic color implementations in Android apps with Jetpack Compose.
- **File Path**: `skills/android-design-system-m3/SKILL.md`

## android-design-to-code
- **Scope**: Use when translating Figma, Sketch, or Adobe XD designs into Jetpack Compose code for Android.
- **File Path**: `skills/android-design-to-code/SKILL.md`

## android-detekt-expert
- **Scope**: Use when performing logic-based static analysis, detecting code smells, or configuring Detekt for Android.
- **File Path**: `skills/android-detekt-expert/SKILL.md`

## android-enterprise-security
- **Scope**: Deep-dive skills for banking and healthcare apps.
- **File Path**: `skills/android-enterprise-security/SKILL.md`

## android-gemini-nano
- **Scope**: Use when integrating Google Gemini Nano via AICore for on-device ML.
- **File Path**: `skills/android-gemini-nano/SKILL.md`

## android-google-workspace-expert
- **Scope**: Use when integrating Google Workspace APIs (Calendar, Drive, Docs) or implementing Google Sign-In with OAuth Scopes.
- **File Path**: `skills/android-google-workspace-expert/SKILL.md`

## android-gradle-expert
- **Scope**: Use when configuring Gradle builds, managing dependencies, or modularizing Android applications.
- **File Path**: `skills/android-gradle-expert/SKILL.md`

## android-hardware-testing
- **Scope**: Automated UIAutomator tests and Hardware mocking setup.
- **File Path**: `skills/android-hardware-testing/SKILL.md`

## android-jetpack-compose
- **Scope**: Use when creating, refactoring, or optimizing Android UI components using Jetpack Compose.
- **File Path**: `skills/android-jetpack-compose/SKILL.md`

## android-kmp-compose-ui
- **Scope**: Expanding KMP shared UI to cover Compose Multiplatform edge-cases.
- **File Path**: `skills/android-kmp-compose-ui/SKILL.md`

## android-kmp-platform-channels
- **Scope**: Bridging Native iOS and Desktop APIs back into KMP.
- **File Path**: `skills/android-kmp-platform-channels/SKILL.md`

## android-kmp-shared
- **Scope**: Use when building Kotlin Multiplatform (KMP) shared modules, integrating with iOS, or sharing business logic across platforms.
- **File Path**: `skills/android-kmp-shared/SKILL.md`

## android-ktlint-formatting
- **Scope**: Use when formatting Kotlin code, enforcing the Android style guide, or configuring KtLint for Android.
- **File Path**: `skills/android-ktlint-formatting/SKILL.md`

## android-lint-and-quality
- **Scope**: Use when setting up static analysis, enforcing coding standards, or configuring Detekt/KtLint for Android.
- **File Path**: `skills/android-lint-and-quality/SKILL.md`

## android-local-stt-tts
- **Scope**: Patterns for local Speech-to-Text and Text-to-Speech without cloud latency.
- **File Path**: `skills/android-local-stt-tts/SKILL.md`

## android-location-maps
- **Scope**: Google Maps SDK, FusedLocationProvider, and Background GeoFencing.
- **File Path**: `skills/android-location-maps/SKILL.md`

## android-macrobenchmark
- **Scope**: Use when evaluating App Startup Time, frame timing, and writing Baseline Profiles.
- **File Path**: `skills/android-macrobenchmark/SKILL.md`

## android-media-and-image-expert
- **Scope**: Use when loading images with Coil/Glide, implementing video playback with Media3/ExoPlayer, or handling media metadata in Android.
- **File Path**: `skills/android-media-and-image-expert/SKILL.md`

## android-media3-exoplayer
- **Scope**: Background playing, Audio focus, and PiP mode using Media3.
- **File Path**: `skills/android-media3-exoplayer/SKILL.md`

## android-motion-animations
- **Scope**: Use when creating advanced Compose animations, shared element transitions, or physics-based motions for Android.
- **File Path**: `skills/android-motion-animations/SKILL.md`

## android-navigation-expert
- **Scope**: Use when scaffolding or debugging Jetpack Compose Navigation. Enforces Type-Safe navigation with Kotlin Serialization instead of legacy String routes.
- **File Path**: `skills/android-navigation-expert/SKILL.md`

## android-offline-first
- **Scope**: Use when implementing local data persistence, offline synchronization, or background data fetching for Android.
- **File Path**: `skills/android-offline-first/SKILL.md`

## android-performance-audit
- **Scope**: Use when profiling Android apps, debugging memory leaks, optimizing Compose UI, or improving app startup time.
- **File Path**: `skills/android-performance-audit/SKILL.md`

## android-rag-local
- **Scope**: Implement local Retrieval-Augmented Generation using Room and Vector embeddings.
- **File Path**: `skills/android-rag-local/SKILL.md`

## android-screenshot-testing
- **Scope**: Use when implementing visual regression tests, verifying UI layouts, or using Paparazzi/Roborazzi for Android.
- **File Path**: `skills/android-screenshot-testing/SKILL.md`

## android-security-hardened
- **Scope**: Use when securing Android applications, implementing Biometric auth, or encrypting sensitive data locally.
- **File Path**: `skills/android-security-hardened/SKILL.md`

## android-unit-testing
- **Scope**: Use when writing or fixing unit tests, mocking objects, or testing reactive flows in Android.
- **File Path**: `skills/android-unit-testing/SKILL.md`

## android-wear-os-compose
- **Scope**: Jetpack Compose for Wear OS, rotary input, and ambient modes.
- **File Path**: `skills/android-wear-os-compose/SKILL.md`

---
Always use `cat` or read the contents of these specific `SKILL.md` files BEFORE providing implementation details for related topics to ensure you adhere to our project's strict standards.

---
> Source: [prasad-vennam/Awesome-Android-AI-Agent-Skills](https://github.com/prasad-vennam/Awesome-Android-AI-Agent-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-20 -->
