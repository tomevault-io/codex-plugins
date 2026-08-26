<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Phonely Mobile App - Device Inspection Feature Progress

## Completed Sprints

### Sprint 1: Core Infrastructure ✅

- [x] Core services (HardwareTestService, DeviceInfoService)
- [x] Data models (InspectionResult, TestResult, DeviceInfo)
- [x] Repository layer (InspectionRepository)
- [x] Riverpod providers for state management

### Sprint 2: Basic Tests ✅

- [x] Touch Screen Test (TouchGridTestScreen)
- [x] Speaker Test (SpeakerTestScreen)
- [x] Microphone Test (MicrophoneTestScreen)
- [x] Vibration Test (VibrationTestScreen)
- [x] Button Test (ButtonTestScreen)
- [x] Camera Tests (Front/Back Camera)
- [x] Device Info Test (DeviceInfoTestScreen)
- [x] Battery Health Test (BatteryTestScreen)
- [x] Storage Health Test (StorageTestScreen)

### Sprint 3: Sensor Tests ✅

- [x] Accelerometer Test (AccelerometerTestScreen) - Ball visualization
- [x] Gyroscope Test (GyroscopeTestScreen) - 3D rotation visualization
- [x] Magnetometer Test (MagnetometerTestScreen) - Compass with animated needle
- [x] Proximity Sensor Test (ProximityTestScreen) - Tap-to-simulate
- [x] Light Sensor Test (LightSensorTestScreen) - Light level buttons

### Sprint 4: Connectivity Tests ✅

- [x] WiFi Test (WifiTestScreen) - Wave animations, connectivity stream
- [x] Bluetooth Test (BluetoothTestScreen) - Scanning animation
- [x] GPS Test (GpsTestScreen) - Satellite orbit animation
- [x] NFC Test (NfcTestScreen) - Optional hardware check

### Sprint 5: Display Tests ✅

- [x] Dead Pixel Test (DeadPixelTestScreen) - Full screen color test
- [x] Multi-Touch Test (MultiTouchTestScreen) - 5+ touch point detection

### Sprint 6: Integration & Polish ✅

- [x] All services exported via barrel file
- [x] All test screens wired in inspection flow
- [x] UI sections for all test categories
- [x] Consistent error handling and feedback

## Project Structure

```
mobile/lib/features/device_inspection/
├── data/
│   ├── models/
│   │   └── models.dart (InspectionResult, TestResult, etc.)
│   └── inspection_repository.dart
├── services/
│   ├── services.dart (barrel file)
│   ├── battery_service.dart
│   ├── connectivity_test_service.dart
│   ├── device_info_service.dart
│   ├── display_test_service.dart
│   ├── hardware_test_service.dart
│   ├── sensor_test_service.dart
│   └── storage_service.dart
└── presentation/
    ├── screens/
    │   ├── tests/
    │   │   ├── tests.dart (barrel file - 23 test screens)
    │   │   └── [all test screens]
    │   ├── inspection_flow_screen.dart
    │   └── inspection_summary_screen.dart
    ├── widgets/
    └── inspection_provider.dart
```

## Tech Stack

- Flutter SDK: 3.10.4
- Dart: 3.10.0
- State Management: flutter_riverpod, freezed
- Key Packages: sensors_plus, connectivity_plus, battery_plus, disk_space
- Test Device: iPhone 15 Pro Max

---
> Source: [MohdAleeRehman/Phonely](https://github.com/MohdAleeRehman/Phonely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
