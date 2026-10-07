
# SPASS Converter — Project Context

## Summary
Android (Kotlin/Jetpack Compose) app that converts Samsung Pass export files (.spass) to CSV. Fully offline—nothing leaves the device.

## Key Paths
| Item | Path |
|------|------|
| Main activity | `app/src/main/java/com/stanley/spassconverter/MainActivity.kt` |
| Manifest | `app/src/main/AndroidManifest.xml` |
| Build config | `app/build.gradle.kts` |
| Store assets | `store-assets/` |
| Play guide | `store-assets/play-console-flow.md` |
| Fill-in guide | `store-assets/PHASE5-FILL-IN-GUIDE.md` |
| Privacy / Terms (GitHub Pages) | `docs/index.html` |
| Privacy policy URL | `https://stanley-projects.github.io/SpassConverter/` |

## Signing
- **Keystore:** `C:\Users\HP\Coding Projects\Android Key Stores\Spass Converter\Spass Converter Key`
- **Alias:** `spassconverter-key`
- **Passwords:** In `local.properties` (not in git)

## Build
- Release AAB: `./gradlew bundleRelease`
- Script: `build-release.bat`
- Output typically copied to `C:\Users\HP\Downloads\SpassConverter-release.aab`

## Play Store Status
- Main store listing, screenshots, icon, feature graphic, descriptions: **done**
- Content rating, data safety, privacy policy, target audience (18+): **done**
- Internal test release: **published**
- **Production:** Blocked until closed test with ≥12 opted-in testers for ≥14 days (0 testers opted in so far)
- Testers use Closed testing opt-in link from Play Console

## Important Notes
- **FLAG_SECURE:** Enabled in MainActivity (screenshots blocked on all form factors)
- **Tablet support:** `android:resizeableActivity="true"` for 7", 10", foldables
- **Sensitive (gitignored):** `local.properties`, keystore, `*.spass`, `*.csv`, `*.apk`, `*.aab`
- **Test .spass:** `store-assets/create_test_spass.py` generates `test_export.spass` (password: `test123`)

---
> Source: [stanley-projects/SpassConverter](https://github.com/stanley-projects/SpassConverter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-06 -->
