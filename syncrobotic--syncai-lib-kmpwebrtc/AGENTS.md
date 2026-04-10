# Project Guidelines

## Overview

SyncAI-Lib-KmpWebRTC is a Kotlin Multiplatform WebRTC client library supporting Android, iOS, JVM, JS, and WasmJS platforms. It provides video/audio stream receiving (WHEP), audio sending (WHIP), and DataChannel bidirectional communication.

## Architecture

- **Single-module KMP structure**: All code resides in a single Gradle module
- **expect/actual pattern**: Shared interfaces defined in `src/commonMain/`, each platform provides `actual` implementations in corresponding source sets
- **Compose Multiplatform**: UI components use Compose (`VideoRenderer`, `AudioPushPlayer`); `BidirectionalPlayer` is deprecated
- **Three-layer architecture**: Layer 1 (SignalingAdapter) → Layer 2 (WhepSession/WhipSession) → Layer 3 (Composables)
- **Package**: `com.syncrobotic.webrtc` (sub-packages: `config`, `signaling`, `session`, `audio`, `ui`, `datachannel`)

### Source Set Structure

| Source Set | Platform | WebRTC Implementation |
|------------|----------|----------------------|
| `commonMain` | Shared | Interfaces, config, signaling, state machine |
| `androidMain` | Android | Google WebRTC SDK + ExoPlayer |
| `iosMain` | iOS | GoogleWebRTC CocoaPod |
| `jvmMain` | Desktop | webrtc-java |
| `jsMain` | Browser | Native RTCPeerConnection (partial stub) |
| `wasmJsMain` | Browser | Native RTCPeerConnection (incomplete) |

## Code Style

- Language: Kotlin, following `kotlin.code.style=official`
- JVM target: Java 11
- **All comments, documentation, and commit messages must be written in English**
- Use Kotlin Coroutines and StateFlow for asynchronous operations
- Compose components provided as `@Composable` functions, returning Controllers
- Architecture pattern: State Machine (sealed class states) + expect/actual platform abstraction, not Redux or Clean Architecture

## Docs Reference

Project architecture and specification documents are stored in the `docs/` directory. Verify generated code conforms to design principles before committing:
- [docs/README_V2.md](../docs/README_V2.md): v2 API design and usage examples
- [docs/MIGRATION_PLAN.md](../docs/MIGRATION_PLAN.md): v1 → v2 migration phases and checklist
- [docs/ROADMAP.md](../docs/ROADMAP.md): Feature planning and milestones

## Code Review Checklist

When generating or modifying code, ask yourself:
- [ ] Are platform dependencies (Android/iOS/JVM) properly abstracted with expect/actual?
- [ ] Are performance and resource cleanup considered (PeerConnection, Factory, EglBase)?
- [ ] Is the interface segregation principle maintained (small, focused interfaces)?
- [ ] Are new dependencies introduced? If so, they must be added to `gradle/libs.versions.toml`
- [ ] Does error handling use sealed class states (e.g. `PlayerState.Error`)?
- [ ] Does new signaling code use `SignalingAdapter` interface (not legacy `WhepSignaling`/`WhipSignaling`)?
- [ ] Could this break existing platform actual implementations?

## Commit Rules

Commit messages must be in English, following Conventional Commits (release-please depends on this format):

| Type | Prefix | Example |
|------|--------|--------|
| New feature | `feat:` | `feat: add VideoPushPlayer composable` |
| Bug fix | `fix:` | `fix: resolve EglContext conflict on multi-renderer` |
| Optimization | `perf:` | `perf: reduce memory usage in JVM video decoding` |
| Test | `test:` | `test: add WebRTCConfig unit tests` |
| Documentation | `docs:` | `docs: update ROADMAP phase 1 status` |
| Refactor | `refactor:` | `refactor: extract signaling interface` |

## Build and Test

```bash
# Build all platforms
./gradlew build

# Build specific targets
./gradlew jvmMainClasses          # JVM
./gradlew bundleReleaseAar        # Android AAR
./gradlew jsJar                   # JavaScript
./gradlew wasmJsJar               # WebAssembly
./gradlew linkPodReleaseFrameworkIosArm64  # iOS framework

# Run tests
./gradlew jvmTest

# Publish to local Maven
./gradlew publishToMavenLocal

# Publish to GitHub Packages
./gradlew publish -Pversion=x.y.z
```

## Conventions

- **PeerConnectionFactory management**: Android creates a separate Factory per connection (to avoid EglContext conflicts); JVM uses reference counting to share a single Factory
- **Multi-instance safety**: Multiple `VideoRenderer` / `WebRTCClient` instances can run simultaneously with no shared state conflicts
- **Signaling**: Uses `SignalingAdapter` interface with built-in `WhepSignalingAdapter`/`WhipSignalingAdapter`; legacy `WhepSignaling`/`WhipSignaling`/`WebSocketSignaling` are `@Deprecated` (will be removed in v3.0)
- **Version management**: Managed automatically by release-please; version defined in `gradle.properties`
- **Dependency definitions**: Uses version catalog `gradle/libs.versions.toml`, referenced in code as `webrtcLibs.xxx`
- **iOS Simulator**: GoogleWebRTC does not support iOS Simulator; physical device only
- **Logging**: Currently uses platform-native logging (Android `Log.d`, iOS `NSLog`, JVM `println`); unified Logger abstraction planned (see ROADMAP Phase 2)
- **Error handling**: Uses sealed class states for errors (`PlayerState.Error`, `AudioPushState.Error`); Signaling layer uses custom Exceptions (`WhepException`, `WhipException` with `WhipErrorCode`)
- **Deprecated types (v2 migration)**: `AudioRetryConfig` (use `RetryConfig`), `StreamProtocol`, `StreamDirection`, `SignalingType`, `WebSocketSignalingConfig`, `ServerEndpoints`, `BidirectionalConfig`, and `WebRTCConfig.signalingType/wsConfig/whepEnabled/whipEnabled` — all marked `@Deprecated`, will be removed in v3.0

## Key Dependencies

| Dependency | Purpose |
|------------|--------|
| Compose Multiplatform 1.10.0 | UI framework |
| Kotlin 2.3.0 | Language |
| Ktor 3.0.3 | HTTP/WebSocket signaling |
| webrtc-android (io.github.webrtc-sdk) | Android WebRTC |
| webrtc-java (dev.onvoid.webrtc) | Desktop WebRTC |
| GoogleWebRTC CocoaPod 1.1.31999 | iOS WebRTC |

## Roadmap

See [docs/ROADMAP.md](../docs/ROADMAP.md) for feature planning and milestones.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Syncrobotic)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Syncrobotic)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
