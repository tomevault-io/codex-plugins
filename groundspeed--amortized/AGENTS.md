# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Amortized** (also marketed as "EZ-Payment-Calc") is an iOS app for calculating amortized loan payments and displaying current mortgage rates from the St. Louis Fed (FRED) API. Bundle ID: `com.groundspeedhq.Pay-Calc`. Swift 5.0, deployment target iOS 18.6 (both app and tests).

## Build & Test Commands

Build and test via `xcodebuild` from the repo root:

```bash
# Build
xcodebuild -project Amortized.xcodeproj -scheme Amortized -configuration Debug -sdk iphonesimulator build

# Run tests (use a simulator ID from `xcodebuild -showdestinations`)
xcodebuild -project Amortized.xcodeproj -scheme Amortized -destination 'platform=iOS Simulator,id=<UUID>' test

# Run a single test class
xcodebuild -project Amortized.xcodeproj -scheme Amortized -destination 'platform=iOS Simulator,id=<UUID>' -only-testing:AmortizedTests/ClassName test
```

Prefer building and running through Xcode for the simulator UI.

## Architecture

The app uses **MVVM** with SwiftUI views and `ObservableObject` view models. Entry point is `Amortized/AmortizedApp.swift` — a `@main struct AmortizedApp: App` using the SwiftUI app lifecycle.

### Key Files

| File | Role |
|------|------|
| `Amortized/AmortizedApp.swift` | `@main App` entry point — `TabView` with Payment and Rates tabs |
| `Amortized/PaymentView.swift` | Payment tab UI + `PaymentViewModel` + `InputField` component |
| `Amortized/RatesView.swift` | Rates tab UI + `RatesViewModel` |
| `Amortized/GlobalHelper.swift` | `PaymentService` — amortization PMT calculation |
| `Amortized/ApiService.swift` | `ApiService` — three concurrent FRED API calls for mortgage rates |
| `Amortized/Rates.swift` | `Rates` model + `FREDResponse` / `FREDObservation` Codable types |
| `AmortizedTests/AmortizedTests.swift` | Unit tests for `PaymentService`, FRED parsing, and view models |

### Data Flow

- **Payment tab**: User inputs (amount, down payment, rate, term) → `PaymentViewModel.calculate()` → `PaymentService.shared.calculateMonthlyPayment()` → PMT formula → displayed result. Optional `AVSpeechSynthesizer` reads the result aloud.
- **Rates tab**: On appear → `RatesViewModel.loadRates()` → `ApiService.shared.getRates()` → three concurrent FRED API calls (`MORTGAGE30US`, `MORTGAGE15US`, `MORTGAGE5US`) → decoded into `FREDResponse` → `observations[0]` = today, `observations[1]` = last week → displayed as today/last-week sections.

### FRED API

Mortgage rates are fetched from the [St. Louis Fed FRED API](https://fred.stlouisfed.org/). Each series is a separate call:

```
https://api.stlouisfed.org/fred/series/observations?series_id=MORTGAGE30US&api_key=<KEY>&file_type=json&sort_order=desc&limit=2
```

| Series ID | Rate |
|-----------|------|
| `MORTGAGE30US` | 30 Year Fixed |
| `MORTGAGE15US` | 15 Year Fixed |
| `MORTGAGE5US` | 5/1 ARM |

### Singletons

- `PaymentService.shared` — stateless calculation service
- `ApiService.shared` — `@MainActor` network service; hardcoded FRED API key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GroundSpeed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
