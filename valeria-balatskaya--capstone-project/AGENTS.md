# Copilot Instructions

## Architecture & Data Flow
- `lib/main.dart` boots `LoraTrackApp`, which immediately shows `LiveTrackingScreen`; all other screens are pushed via `AppDrawer` replacements, so keep navigator usage consistent.
- Screens under `lib/screens/` are stateful widgets that call services directly and mutate local state via `setState`; there is no global state management library.
- `LiveTrackingScreen` derives its device list from `DeviceService.loadDevices()` and only displays map placeholders today; any real map integration should preserve the surrounding status badges and bottom stats block.
- `LiveTrackingScreen` also calls `LocationService.getCurrentLocation()` (Geolocator) to read the handset GPS sensor; treat that path as optional and keep it resilient to denied permissions or disabled services.
- `DeviceListScreen` is the source of truth for CRUD flows and persists changes through `DeviceService.saveDevices`; keep new device attributes serializable because they wind up in shared preferences.

## State & Persistence Patterns
- `DeviceService` first tries `fetchDevicesFromChirpStack()` using credentials from `SettingsService`; only if that fails does it read locally cached JSON from SharedPreferences (key `tracked_devices`).
- Each call to `loadDevices()` overwrites the cache with whatever was fetched, so batch mutations (add/update/remove) should operate on in-memory lists already returned by that method to avoid race conditions.
- `SettingsService` persists the `AppSettings` model (`lib/models/app_settings.dart`) as JSON under the `app_settings` key; always go through `saveSettings`/`loadSettings` rather than manipulating SharedPreferences yourself.
- Shared preferences (`shared_preferences`) and HTTP (`http`) are runtime dependencies, so keep them under `dependencies` rather than `dev_dependencies` to avoid build failures.
- Geolocation reads rely on the `geolocator` plugin; Android needs coarse/fine permissions in `android/app/src/main/AndroidManifest.xml` and iOS needs `NSLocationWhenInUseUsageDescription` in `ios/Runner/Info.plist`.

## ChirpStack Integration
- REST calls target `${serverUrl}/api/applications` and `/devices` endpoints with `Grpc-Metadata-Authorization: Bearer <token>` headers; `ChirpStackService.testConnection` automatically retries with standard `Authorization` headers for backward compatibility.
- `DeviceService.getDeviceLocation` hits `/api/devices/<devEui>/events?limit=1&types=up` and expects location metadata inside `event['data']`; guard new consumers against nulls because most events will lack location payloads during development.
- Keep network timeouts short (currently 10s) to avoid freezing the UI; wrap long-running fetches with loading spinners as done in `LiveTrackingScreen` and `DeviceListScreen`.
- Phone GPS pulls should reuse the `_isFetchingGps` flag and `_gpsError` messaging pattern in `LiveTrackingScreen` to give immediate feedback when permissions or services fail.

## Navigation & UI Composition
- `AppDrawer` decides active routes using simple string keys (`position`, `devices`, `settings`); add new sections by extending that enum-like pattern and using `Navigator.pushReplacement` to avoid back-stack explosions.
- Detail pages (`DeviceDetailScreen`, future analytics views) assume the device map is a plain `Map<String, dynamic>`; when introducing typed models, supply back-compat converters so existing screens keep working.

## Build, Run, Test
- Install deps with `flutter pub get`, then run on a device/emulator with `flutter run -d <device>`; use `flutter build apk --release` for Android deliverables.
- Static analysis uses the default `flutter_lints` set (`analysis_options.yaml`); run `flutter analyze` before committing because CI expects a clean analyzer output.
- The lone widget test (`test/widget_test.dart`) is still the Flutter counter sample and fails against the current UI; either update it to exercise `LiveTrackingScreen` or skip it explicitly when running `flutter test`.

## Conventions & Gotchas
- UI colors/font weights are hard-coded; reuse the styles already defined inside each screen instead of introducing theme extensions unless you refactor the whole app.
- `DeviceService` prints errors directly; prefer `debugPrint` if you need more logging but be aware that lint rules currently allow prints.
- When adding real-time updates, respect the existing `_isLoading` flags and snack-bar patterns so users keep receiving consistent feedback.
- Keep README version notes in sync with in-app behavior (version badge at the top) to avoid confusion during demo reviews.

---
> Source: [Valeria-Balatskaya/Capstone-Project](https://github.com/Valeria-Balatskaya/Capstone-Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-28 -->
