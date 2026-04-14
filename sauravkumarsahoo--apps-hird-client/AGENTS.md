# GitHub Copilot Instructions — HIRD Client

## Project Overview

**HIRD** (HWiNFO Remote Display) Client is a Flutter mobile app (Android + iOS) that
streams real-time PC hardware sensor data from the
[HIRD Server](https://github.com/clicksrv/Apps.HIRD.Server) over a local Wi-Fi network.
Communication uses **gRPC with Protocol Buffers**. The app discovers servers by scanning
the local subnet, lets the user pick one, then displays a live-updating dashboard of CPU,
GPU, RAM, storage, and system readings.

---

## Project Structure

```
lib/
  main.dart               # App entry point, connectivity guard, theme setup
  common.dart             # ThemeData definitions, logger utility
  constants.dart          # App-wide constants (app name, gRPC port 25151)
  settings.dart           # Singleton Settings class (currently hardcoded, no persistence)
  widgets_lib.dart        # Shared widgets: FloatCard, ExpandableFloatCard, CenteredColumn,
                          #   BlendedAssetImage, LinearTempGauge (Syncfusion-backed)
  pages/
    select_server_page.dart    # LAN server discovery and selection UI
    data_visualizer_page.dart  # Main dashboard: CPU/GPU/RAM/Storage/System cards
    no_connectivity_page.dart  # Shown when not on Wi-Fi or Ethernet
    settings_page.dart         # Stub – not yet implemented
  services/
    server_scanner_service.dart  # Singleton: continuous LAN scan, gRPC computer info fetch
    sensor_client_service.dart   # Top-level function: returns a SensorServiceClient
  models/
    server_info.dart             # Plain data class: ip, SensorServiceClient, ComputerInfo
  widgets/
    cpu_temp_gauge.dart          # SfRadialGauge for CPU package temperature
    gpu_temp_gauge.dart          # SfRadialGauge for GPU temperature + hotspot
    storage_temp_gauge.dart      # SfRadialGauge for individual storage device temperature
    cpu_core_temps_widget.dart   # Horizontal SfLinearGauge bars per CPU core temperature
    cpu_core_usages_widget.dart  # Horizontal SfLinearGauge bars per CPU core usage
  helpers/
    device_helper.dart   # Retrieves device name via device_info_plus (registered in GetIt)
    size_helper.dart     # Static screen-size utility (anti-pattern, avoid expanding)
  generated/             # Protobuf/gRPC generated Dart code – DO NOT EDIT manually
    sensorcomms.pb.dart
    sensorcomms.pbgrpc.dart
    sensorcomms.pbenum.dart
    sensorcomms.pbjson.dart
protos/
  sensorcomms.proto      # Source of truth for all data models and service definitions
```

---

## Architecture & Data Flow

1. **Startup**: `main.dart` checks network connectivity. Wi-Fi or Ethernet → `SelectServerPage`;
   anything else → `NoConnectivityPage`.
2. **Server Discovery**: `ServerScannerService` (singleton via factory constructor) starts a
   periodic LAN ping scan (`ping_discover_network`) on port `25151`. Each discovered IP gets a
   gRPC `getComputerInfo` call. Results are emitted over a broadcast stream.
3. **Server Selection**: `SelectServerPage` is a `StreamBuilder` on
   `ServerScannerService.stream`. Tapping a server navigates to `DataVisualizerPage`.
4. **Live Dashboard**: `DataVisualizerPage` calls `SensorServiceClient.monitor()` and wraps
   the response stream in a `StreamBuilder`. On each tick, five expandable cards rebuild:
   CPU, GPU, RAM, System, Storage. The stream has a 10-second timeout per call.
5. **Theme**: `animated_theme_switcher` wraps `MaterialApp`. Theme brightness is toggled on
   `SelectServerPage`.

### Dependency Injection

`GetIt` is registered in `main.dart` via `setup()`, but only `DeviceHelper` is currently
registered. Services use manual factory-based singletons instead of GetIt.

---

## Key Dependencies

| Package | Purpose | Notes |
|---|---|---|
| `grpc` ^3.0.2 | gRPC communication with HIRD Server | Core – keep |
| `protobuf` ^2.0.1 | Protobuf serialisation | Core – keep |
| `get_it` | Service locator / DI | Partially used |
| `connectivity_plus` | Network state detection | Migrate to latest |
| `network_info_plus` | Get device Wi-Fi IP | Migrate to latest |
| `device_info_plus` | Device name for gRPC requests | Migrate to latest |
| `wakelock` | Keep screen awake while viewing dashboard | **Deprecated** → replace with `wakelock_plus` |
| `syncfusion_flutter_gauges` | Radial & linear gauge widgets | **Target for replacement** with custom widgets |
| `syncfusion_flutter_sliders` | Slider widgets (currently unused in UI) | **Remove** |
| `animated_theme_switcher` | Animated light/dark theme toggle | Consider replacing with standard `ThemeMode` |
| `expandable` | Expandable card panel | Consider replacing with standard `ExpansionTile` |
| `back_button_interceptor` | Android back-button handling | Consider replacing with `PopScope` (Flutter 3.x) |
| `fading_edge_scrollview` | Fading scroll edges | Minor visual polish |
| `ping_discover_network` | Subnet ping scan (git dependency) | Keep; no pub.dev alternative |
| `freezed_annotation` / `freezed` | Code generation for immutable models | Set up but **not yet used** |

---

## Known Issues & Technical Debt

### Flutter/Dart Version

- `pubspec.yaml` SDK constraint is `>=2.17.0 <3.0.0` — **must be upgraded to Flutter 3.x**.
- Many deprecated Flutter 2.x APIs are in active use (see below).

### Deprecated TextTheme APIs (Flutter 3.x migration required)

All usages of the old named text styles must be migrated to the new `TextTheme` properties:

| Old (removed) | New equivalent |
|---|---|
| `textTheme.headline4` | `textTheme.headlineMedium` |
| `textTheme.headline5` | `textTheme.headlineSmall` |
| `textTheme.headline6` | `textTheme.titleLarge` |
| `textTheme.subtitle1` | `textTheme.titleMedium` |
| `textTheme.subtitle2` | `textTheme.titleSmall` |
| `textTheme.bodyText1` | `textTheme.bodyLarge` |
| `textTheme.overline` | `textTheme.labelSmall` |

### Deprecated Packages

- `wakelock` → replace with `wakelock_plus`
- `back_button_interceptor` → replace with `PopScope` widget (Flutter 3.4+)

### Settings — No Persistence

`Settings` is a singleton with hardcoded getters (`theme`, `cardsExpandedByDefault`,
`firstLaunch`). It has no read/write layer. When persistence is added, prefer
`shared_preferences` or `flutter_secure_storage` (for sensitive values).

### Structural Issues

- **`widgets_lib.dart`**: Monolithic file containing unrelated widgets. Should be split into
  individual files under `lib/widgets/`.
- **`FloatCard`**: Wraps `Container` with almost identical API surface. Consider using
  `Card` with custom theme, or `DecoratedBox`, instead.
- **`SizeHelper`**: Uses static mutable state — an anti-pattern. Prefer
  `MediaQuery.of(context)` at the call site or `LayoutBuilder`.
- **`sensor_client_service.dart`**: A bare global function, not a class or service. Should
  be promoted to a proper class or merged into `ServerScannerService`.
- **`ServerInfo`**: All fields are mutable (`var`, no `final`). Should be made immutable.
- **`DataVisualizerPage`**: Card-building methods (`buildCpuCard`, `buildGpuCard`, etc.) are
  instance methods on the page class — they should be extracted as dedicated `StatelessWidget`
  subclasses under `lib/widgets/`.
- **`CpuCoreTempsWidget`**: Color interpolation logic (`t * 0.65`, `t / 0.25`) appears
  incorrect — the `t` variable divides the raw temperature by 100, not the normalised 0–1
  range. This should be reviewed.
- **Dead code**: `buildVisualizerDirectly()` in `main.dart` (used only in commented code).
- **`SettingsPage`**: Completely empty stub — shows a blank `Scaffold`.

### Syncfusion Widgets (Primary Replacement Target)

All five sensor display widgets rely on Syncfusion:

| File | Syncfusion widget used |
|---|---|
| `widgets/cpu_temp_gauge.dart` | `SfRadialGauge` (2 axes, needle + marker pointers) |
| `widgets/gpu_temp_gauge.dart` | `SfRadialGauge` (2 axes, needle + marker + hotspot) |
| `widgets/storage_temp_gauge.dart` | `SfRadialGauge` (single device) |
| `widgets/cpu_core_temps_widget.dart` | `SfLinearGauge` per core (vertical bar) |
| `widgets/cpu_core_usages_widget.dart` | `SfLinearGauge` per core (vertical bar) |
| `widgets_lib.dart` → `LinearTempGauge` | `SfLinearGauge` (min/max/current range bar) |

**Replacement plan**: implement custom gauge widgets using `CustomPainter` + the `canvas`
API, or the `fl_chart` / `community_charts_flutter` libraries (no licensing restrictions).
Any replacement widget must accept a `ReadingData` object and expose the same color-coding
logic (green / orange / red thresholds).

---

## Coding Conventions

- Use `const` constructors wherever possible.
- Prefer `final` for fields; avoid mutable state unless justified.
- All new widgets go in `lib/widgets/` (one widget per file).
- All new pages go in `lib/pages/`.
- Register any new singleton service in `GetIt` via `setup()` in `main.dart`.
- Do not edit files under `lib/generated/` — regenerate from `protos/sensorcomms.proto`
  using `generate_from_proto.bat`.
- Avoid `print()` — use the project's `logger()` function from `common.dart`.
- Prefer named constructors and factory constructors over bare constructors for models.
- New state management should use `Riverpod` or plain `ChangeNotifier`/`ValueNotifier`
  rather than expanding `StatefulWidget` usage.

---

## Roadmap

### Phase 1 — Flutter 3.x Upgrade
- [ ] Raise SDK constraint: `sdk: ">=3.0.0 <4.0.0"` and `flutter: ">=3.16.0"`
- [ ] Migrate all deprecated `TextTheme` usages
- [ ] Replace `wakelock` → `wakelock_plus`
- [ ] Replace `back_button_interceptor` → `PopScope`
- [ ] Update all `*_plus` package versions to current

### Phase 2 — Remove Syncfusion
- [ ] Implement `RadialGauge` custom widget using `CustomPainter` (replaces all three `*TempGauge` widgets)
- [ ] Implement `BarChartWidget` custom widget (replaces `CpuCoreTempsWidget` and `CpuCoreUsagesWidget`)
- [ ] Implement `LinearRangeGauge` custom widget (replaces `LinearTempGauge`)
- [ ] Remove `syncfusion_flutter_gauges` and `syncfusion_flutter_sliders` from `pubspec.yaml`

### Phase 3 — Code Quality
- [ ] Split `widgets_lib.dart` into individual widget files
- [ ] Extract card build methods from `DataVisualizerPage` into standalone widgets
- [ ] Make `ServerInfo` immutable
- [ ] Implement `Settings` persistence with `shared_preferences`
- [ ] Implement `SettingsPage` UI
- [ ] Remove dead code (`buildVisualizerDirectly`, unused `freezed` setup if not adopted)
- [ ] Fix color interpolation in `CpuCoreTempsWidget`
- [ ] Promote `sensor_client_service` to a proper class
- [ ] Adopt `freezed` for `ServerInfo` and any new models

### Phase 4 — Features & UX
- [ ] Historical graphs (time-series line chart) per sensor reading
- [ ] Per-sensor threshold configuration (stored in Settings)
- [ ] Named / saved server list
- [ ] Landscape layout improvements
- [ ] Notification / alert when a reading exceeds a threshold

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sauravkumarsahoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
