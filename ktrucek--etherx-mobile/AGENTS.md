# Copilot instructions for `etherx-mobile`

## Architecture

- This app is a mostly single-screen React Native browser shell; `App.tsx` contains the browser UI, tab state, URL bar, `WebView`, and the webpage bridge logic.
- `index.js` only registers the root component. There is no active `src/` application layer yet even though `tsconfig.json` includes `src/**/*`.
- The app launches into `INITIAL_URL = https://n8n.kriptoentuzijasti.io/browser.html`; treat that hosted page as part of the product flow when changing startup, auth, or navigation behavior.
- The key cross-boundary contract is the injected JavaScript string in `App.tsx` plus `handleMessage()`. Keep message payloads aligned on both sides for `type: 'localStorage'`, `type: 'web3'`, and `type: 'ready'`.

## WebView and bridge conventions

- Preserve the `window.ETHERX_*` flags and custom `userAgent` in `App.tsx`; downstream web content can detect the mobile shell from those values.
- `window.ethereum` is a lightweight injected stub, not a full wallet SDK. If you add wallet features, extend both the injected `request()` contract and the native `handleMessage()` branch together.
- `AsyncStorage` persistence is mediated through `window.ReactNativeWebView.postMessage(...)`; avoid introducing a second storage path unless you intentionally replace the bridge.
- When changing navigation, update the local input state (`url`), the rendered URL (`activeTab?.url`), and the tab list entry together. `navigateTo()` is the canonical example.
- Tab titles are refreshed from `onNavigationStateChange`; if you add per-tab metadata, keep it synchronized there instead of inventing a second navigation listener.

## Native/build details

- Android release builds are currently debug-signed on purpose in `android/app/build.gradle`; CI produces sideloadable APKs, not Play Store-ready signed releases.
- `android/app/src/main/res/values/styles.xml` references `@drawable/rn_edit_text_material`; keep `android/app/src/main/res/drawable/rn_edit_text_material.xml` in place or Android builds can fail.
- Hermes is enabled and the new architecture is disabled in `android/gradle.properties`; do not enable Fabric/new architecture casually because the project is pinned to a compatibility-first setup.
- iOS pods are declared explicitly in `ios/Podfile` for `react-native-webview`, `RNCAsyncStorage`, and `RNSVG`; run `cd ios && pod install` after native dependency changes.

## Workflows

- Use `npm install` for local setup and `npm run start`, `npm run android`, `npm run ios` for day-to-day development.
- Use `npm run build:android` and `npm run build:ios` for manual release builds; CI mirrors this in `.github/workflows/build.yml` with Node 18 and Java 17.
- There are no test files in the repo today even though `package.json` exposes `npm test`; prefer validating changes with targeted builds/linting unless you also add tests.
- `deploy-mobitel.sh` is the release helper: it repairs the Android drawable, bumps versions in `package.json`, `package-lock.json`, and `android/app/build.gradle`, creates a git tag, and pushes to trigger GitHub Actions.
- The deploy script is written in Croatian and assumes the existing repo/tagging flow; keep edits surgical unless you are intentionally changing the release process.

## Codebase-specific guidance

- Prefer small, localized edits in `App.tsx` over premature abstraction. Most behavior is intentionally centralized there today.
- Do not assume installed packages are already wired into the app. For example, `@react-navigation/*` is present in `package.json`, but the current UI uses a custom tab strip instead of a `NavigationContainer`.
- `loose-globals.d.ts` exists as a typing workaround for the current TS setup and injected JavaScript usage. If TypeScript errors appear, prefer fixing the root typing/reference issue instead of adding more ambient declarations blindly.
- Keep the dark theme palette and compact browser layout consistent with the existing styles in `App.tsx` unless the task explicitly asks for a redesign.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ktrucek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
