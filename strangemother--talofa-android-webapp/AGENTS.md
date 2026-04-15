# Talofa Android WebApp - AI Agent Instructions

## Project Overview
This is a customizable Android WebView wrapper that transforms web applications into native Android 10+ apps. The core architecture uses a single-activity WebView with JavaScript-to-Android bridges for native functionality.

## Single-Source Configuration Pattern
**Critical**: All configuration flows through `app/app.properties` → Gradle → BuildConfig + AndroidManifest
- Edit only `app/app.properties` to change `DEFAULT_SCHEME` and `DEFAULT_URL`
- Gradle build reads properties and injects via:
  - `buildConfigField()` → generates `BuildConfig.DEFAULT_SCHEME` / `BuildConfig.DEFAULT_URL` 
  - `manifestPlaceholders["defaultScheme"]` → replaces `${defaultScheme}` in AndroidManifest.xml
- Never hardcode these values in Kotlin/XML - always reference `BuildConfig.*`

Example flow:
```properties
# app/app.properties
DEFAULT_SCHEME=myapp123
DEFAULT_URL=file:///android_asset/demo.html
```
↓
```kotlin
// MainActivity.kt - automatically generated
val url = BuildConfig.DEFAULT_URL  // NOT hardcoded
```

## Build & Release Workflow
```bash
# Local builds (requires Android SDK licenses accepted)
./gradlew :app:assembleDebug
./gradlew :app:assembleRelease

# GitHub Actions - two trigger modes:
1. Manual: Actions tab → "Build Release APKs" → "Run workflow" button
2. Automatic: git tag -a v1.0.X -m "..." && git push webapp v1.0.X
```

Output: `app/build/outputs/apk/{debug,release}/talofa-webview-v*.apk`

## JavaScript Bridge Architecture
**Pattern**: `@JavascriptInterface` methods in `WebAppInterface` inner class expose native Android to web context as `window.Android.*`

Key implementation details:
- Vibration: Use `VibratorManager` (API 31+) with fallback to `Vibrator` (older)
- Always check `hasVibrator()` and wrap in try-catch
- Toast/device info calls must handle context lifecycle
- All bridge methods are synchronous - return strings/primitives only

Example:
```kotlin
@JavascriptInterface
fun vibrate(strength: Float) {
    val vibrator = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        (context.getSystemService(Context.VIBRATOR_MANAGER_SERVICE) as VibratorManager).defaultVibrator
    } else {
        context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    }
    // ... vibration logic
}
```

## Deep Link Configuration System
**URL format**: `<scheme>://setup?url=<target>&scheme=<new_scheme>`

Flow:
1. User scans QR code → Android opens app via intent filter
2. `onNewIntent()` → `handleDeepLink()` extracts params
3. Saves to SharedPreferences: `KEY_SITE_URL`, `KEY_CUSTOM_SCHEME`
4. Calls `webView.loadUrl()` if WebView initialized (`if (::webView.isInitialized)`)

**Critical bug fix**: Always call `setIntent(intent)` in `onNewIntent()` and check `::webView.isInitialized` before reload to prevent crashes when app is already running.

## Asset Management
Demo page bundled as `app/src/main/assets/demo.html` → accessible via `file:///android_asset/demo.html`
- Use this pattern for offline-first content
- WebView loads assets with full JavaScript/WebGL support

## Version Bumping
Update both in `app/build.gradle.kts`:
```kotlin
versionCode = 4        // Increment integer
versionName = "1.0.3"  // Semantic version
```
GitHub Actions extracts `versionName` via: `grep 'versionName = ' app/build.gradle.kts`

## Project-Specific Conventions
1. **No separate modules** - single `app/` module structure (migrated from multi-module)
2. **No signing in CI** - releases are unsigned, users sign locally if needed
3. **Gradle wrapper version**: 8.2 (don't upgrade without testing Android plugin compatibility)
4. **Manifest placeholders** - only for `defaultScheme`, not URL (URLs go in BuildConfig)
5. **SharedPreferences keys** - use `companion object` constants: `KEY_SITE_URL`, `KEY_CUSTOM_SCHEME`

## Common Tasks
- **Add JS bridge method**: Add `@JavascriptInterface fun` in `WebAppInterface` class
- **Change default URL**: Edit `app/app.properties` only, rebuild
- **Test locally**: Build debug APK, install via `adb install -r app/build/outputs/apk/debug/*.apk`
- **Create release**: `git tag -a v1.0.X -m "message" && git push webapp v1.0.X` (triggers CI)

## WebView Configuration
The WebView is configured in `MainActivity.onCreate()` with specific settings for web app compatibility:

**JavaScript & Storage**:
```kotlin
javaScriptEnabled = true           // Required for all JS functionality
domStorageEnabled = true           // localStorage/sessionStorage support
databaseEnabled = true             // IndexedDB/WebSQL support
```

**File & Content Access**:
```kotlin
allowFileAccess = true             // Access to file:// URLs (for assets)
allowContentAccess = true          // Access to content:// URLs
mixedContentMode = MIXED_CONTENT_COMPATIBILITY_MODE  // Allow HTTP in HTTPS
```

**Performance & Graphics**:
```kotlin
// Hardware acceleration enabled via AndroidManifest:
android:hardwareAccelerated="true"  // Essential for WebGL/Canvas

// Cache for better performance:
cacheMode = WebSettings.LOAD_DEFAULT  // Use cached resources when available
```

**Viewport & Display**:
```kotlin
useWideViewPort = true             // Support viewport meta tag
loadWithOverviewMode = true        // Fit content to screen width
builtInZoomControls = false        // Disable zoom for app-like feel
displayZoomControls = false
```

**Critical for Canvas/WebGL apps**: Hardware acceleration MUST be enabled in both manifest (`android:hardwareAccelerated="true"`) and is inherited by WebView. Without this, canvas rendering will be slow/broken.

## Critical Files
- `app/app.properties` - Single source of truth for configuration
- `app/src/main/java/me/talofa/webview/MainActivity.kt` - Core WebView + bridge implementation
- `app/src/main/AndroidManifest.xml` - Deep link intent filters (uses placeholders)
- `.github/workflows/build-release.yml` - CI/CD pipeline (action-gh-release@v2, upload-artifact@v4)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Strangemother) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
