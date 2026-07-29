# LightweightChartsIOS — Coding Instructions

These conventions apply to all work in this repository.

## Project Architecture

This is a Swift wrapper around the TradingView [lightweight-charts](https://github.com/tradingview/lightweight-charts) JavaScript library. Swift code communicates with JS running inside a `WKWebView` via string-based script evaluation.

Key directories:
- `Sources/LightweightCharts/Protocols/` — Public API protocols (`ChartApi`, `SeriesApi`, `PriceScaleApi`, `TimeScaleApi`, `PaneApi`)
- `Sources/LightweightCharts/LightweightChartsModels/` — Codable structs and enums that map to JS option objects
- `Sources/LightweightCharts/Implementations/API/` — Concrete classes that bridge Swift calls to JS
- `Sources/LightweightCharts/Assets/` — The compiled JS library and helper scripts
- `Example/LightweightCharts/Example/` — Demo ViewControllers showcasing features

## Swift-to-JS Bridge Patterns

Every API object has a `jsName: String` property (unique per instance) used to reference the corresponding JS object.

### Fire-and-forget (no return value)
```swift
let script = "\(jsName).someMethod(\(arg));"
context.evaluateScript(script, completion: nil)
```

### Simple result (cast directly)
```swift
let script = "\(jsName).someMethod();"
context.evaluateScript(script) { result, _ in
    completion(result as? Double)
}
```

For `Int` results, use `(result as? NSNumber)?.intValue`.

### JSON-decoded result (Decodable structs)
```swift
let script = "JSON.stringify(\(jsName).someMethod());"
context.decodedResult(forScript: script, completion: completion)
```

### Rules
- Every method that **returns a value** in JS must use a `completion:` handler in Swift — the WKWebView bridge is always asynchronous.
- Fire-and-forget methods pass `completion: nil`.
- Use `decodedResult` when the return type is a struct/object. Use simple casting for primitives (`Double`, `Bool`, `Int`).

## Adding New Model Types (Codable structs/enums)

- All public option structs must conform to `Codable`.
- All properties should be `Optional` (use `?`) so only explicitly set values are serialized to JSON. JS defaults handle the rest.
- If a Swift property name differs from the JS key, add an explicit `CodingKeys` enum.
- Enums representing JS string literals must use `String` raw values and conform to `Codable`.
- JS string literals with hyphens need explicit raw values:
  ```swift
  public enum ConflationPriority: String, Codable {
      case background
      case userVisible = "user-visible"    // hyphenated JS value
      case userBlocking = "user-blocking"
  }
  ```

## Adding New Protocol Methods

1. Add the method signature to the protocol in `Protocols/`.
2. Implement it in the corresponding class in `Implementations/API/`.
3. If the method returns a value, the Swift signature must include a `completion:` parameter.
4. Update any manual test mocks in `Example/Tests/Tests.swift` that directly conform to the changed protocol (e.g., `TestSeries` conforms to `SeriesApi`). These mocks don't inherit the default extension implementations and will fail to compile if new required methods are missing.

## Adding New Example ViewControllers

1. Create a new `*ViewController.swift` file in `Example/LightweightCharts/Example/`.
2. Register it in `TableViewController.swift` → `fillSections()` using the `Row` pattern:
   ```swift
   Row(title: "Feature Name", subtitle: "Short description", viewController: { MyNewViewController() })
   ```
   Place it in the appropriate `Section` (v5 Plugin API, v4 Compatibility, etc.).
3. Add the file to the Xcode project (`LightweightCharts.xcodeproj/project.pbxproj`) in **four** sections:
   - `PBXBuildFile` — `/* MyVC.swift in Sources */ = {isa = PBXBuildFile; fileRef = <fileRefID>; };`
   - `PBXFileReference` — `/* MyVC.swift */ = {isa = PBXFileReference; lastKnownFileType = sourcecode.swift; path = MyVC.swift; sourceTree = "<group>"; };`
   - `PBXGroup` children list — add the `fileRefID` under the `Example` group
   - `Sources` build phase — add the `buildFileID` to the `files` array

## Version Bumps

When releasing a new wrapper version, update **all** of these:
- `LightweightCharts.podspec` → `s.version`
- `CHANGELOG.md` → new section at the top
- `README.md` → version references
- `scripts/build-upstream-umd.sh` → default `VERSION` variable (only if JS version changed)

## CocoaPods

After adding or removing Swift files under `Sources/LightweightCharts/`, re-run `pod install` from `Example/` so CocoaPods picks up the changes:
```bash
cd Example && pod install
```
Do this **before** attempting to build. The podspec uses a glob (`Sources/LightweightCharts/**/*.swift`), but the Pods project file is only regenerated on `pod install`.

## Testing

After any code change, verify the build:
```bash
xcodebuild -workspace Example/LightweightCharts.xcworkspace \
  -scheme LightweightCharts-Example \
  -configuration Debug \
  -sdk iphonesimulator \
  build
```

Run unit tests:
```bash
xcodebuild -workspace Example/LightweightCharts.xcworkspace \
  -scheme LightweightCharts-Example \
  -configuration Debug \
  -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone 16e' \
  test
```
Adjust the `-destination` to match an available simulator (`xcrun simctl list devices available`).

## Test Diagnostics Policy

When running `xcodebuild test`, simulator process logs can include noisy non-fatal messages (for example: `RunningBoard`, `ProcessSuspension`, `WebContent`, watchdog/assertion noise).

Use only these lines as pass/fail source of truth:
- `Executed <n> tests, with <m> failures`
- `** TEST SUCCEEDED **` / `** TEST FAILED **`

Do not classify simulator process-noise logs as test failures unless they are accompanied by XCTest assertion failures.

## Canonical Documentation Policy

Keep one canonical source for migration/release docs at repository root unless explicitly requested otherwise.

- Do not create duplicate docs in `Example/` (or other subfolders) just to satisfy path-sensitive tests.
- If tests are path-sensitive, fix path resolution in tests to locate the canonical root doc robustly.

## Test Quality Policy

Prefer behavior-focused assertions over implementation-detail assertions.

- Avoid tautologies (for example: `XCTAssertNotNil` on non-optional values).
- Avoid brittle coupling to generated/internal naming details (for example: hardcoded `jsName` prefixes).
- Prefer observable behavior/output assertions to no-crash-only checks.

## JS Bridging Safety

When interpolating string arguments into JavaScript, always pass quoted/escaped values using JSON serialization helpers (for example: `.jsonString()`) instead of raw string insertion.

For optional or version-dependent JS APIs (especially plugin methods), guard calls with `typeof ... === 'function'` checks before invoking them.

Do not add defensive availability checks for core runtime globals required by this wrapper (for example `window`); assume the `WKWebView` JS context and bundled lightweight-charts library are present unless the API itself is optional/version-dependent.

## Style

- SwiftLint is configured in `Example/.swiftlint.yml`.
- Line length: warning at 125, error at 180.
- Type nesting: max depth 3.
- Prefer `let` over `var` where possible.
- No unnecessary comments or docstrings on code you didn't change.

---
> Source: [tradingview/LightweightChartsIOS](https://github.com/tradingview/LightweightChartsIOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
