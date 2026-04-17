# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Unified Multi-Platform Messaging Client — Full Specification

## Quick Reference — Build & Test Commands

```bash
# Enter dev shell (required — provides Go, Flutter, Android SDK, all deps)
nix develop

# Build Go shared library
scripts/build_go.sh linux     # → go/build/libcores.so
scripts/build_go.sh windows   # → go/build/cores.dll (needs mingw32-gcc)
scripts/build_go.sh darwin    # → go/build/libcores.dylib
scripts/build_go.sh android   # → go/build/android/*/libcores.so (needs ANDROID_NDK_HOME)
scripts/build_go.sh web       # → go/build/cores.wasm + wasm_exec.js

# Shell aliases (defined in flake.nix)
build-go                      # → scripts/build_go.sh (current platform)
test-go                       # → cd go && go test ./...
test-dart                     # → cd dart && flutter test
test-all                      # → test-go && test-dart

# Run unit tests (no credentials needed)
cd go && go test ./utils/... -v -timeout 120s    # 91 tests, all passing

# Run a single test
cd go && go test ./cores/... -run "TestTelegramBotAuth" -v -timeout 30s

# Integration tests (need env vars from auth/auth.md)
source auth/auth.md && cd go/tests && go test -v -timeout 300s
```

**Go version**: 1.26.1 · **CGO_ENABLED=1** (for bridge only, cores/utils are pure Go)

## Current State (updated 2026-04-09)

**GitHub repo:** https://github.com/DarkReaperBoy/uniclient (HTTPS push, no SSH)

**Read these files first on context reset:**
1. This file (`CLAUDE.md`) — full spec and rules
2. `CHECKLIST.md` — what's done, what's next
3. `README.md` — public project overview (keep in sync with progress)
4. `auth/auth.md` — test credentials (bot token, chat IDs)
5. `docs/telegram_notes.md` — gotd/td API quirks and bot limitations

**What's done:**
- Phase 0 complete: flake.nix, 10 Go utils (91 tests), Core interface (`base.go` 320 lines), FFI bridge (`bridge.go` with JSON-encoded BridgeResponse, event port callback)
- Phase 1 Telegram ~95% (bot + user mode): `telegram.go` 743 exported methods, 8695 lines
  - Bot mode: 35+ features tested and confirmed
  - User mode: 22/22 verification tests + 20+ two-user tests pass against live API (2026-04-05)
  - Features: auth, profile, dialogs, messages CRUD, reply, forward, react (needs premium), pin/unpin, search, mark-as-read, typing, folders, contacts, scheduled messages, stickers, active sessions, file upload+download (byte-identical), drafts, online status, config, polls, translate, chat invites, channel management, stories, bot interaction, forum topics, channel admin toggles
  - Two-user tests verified: real-time updates (new/edit/delete), outbox read date, mark dialog unread, invite to channel, forum CRUD, basic group add/remove, poll votes, stories send/delete, import contacts
  - 600+ passthrough MTProto API wrappers for direct protocol access
  - Calling: **9/9 versions pass automated C++ harness** — pure Go/pion, no .so dependency. See `docs/tgcalls_protocol.md`
    - **V2Reference**: v10.0.0 (98 frames), v11.0.0 (98 frames) — SDP offer/answer, DTLS-SRTP
    - **V2Impl**: v7.0.0 (65), v8.0.0 (65), v9.0.0 (65), v12.0.0 (66), v13.0.0 (65) — InitialSetup+NegotiateChannels, synthetic SDP
    - **InstanceImpl**: v5.0.0 (52 frames), v2.7.7 (51 frames) — binary typed messages, AES-CTR encrypted raw ICE (pion/ice.Agent), no SDP/DTLS
    - Version ordering: `[11.0.0, 10.0.0, 13.0.0, 12.0.0, 9.0.0, 8.0.0, 7.0.0, 5.0.0, 2.7.7]`
    - V1 + V2 encryption framing, SCTP transport, NC dedup (`answeredExchangeIDs`), SSRC echo fix
    - **Real tgcalls C++ test harness**: libtgcalls_native.so + nix tg_owt, automated `TestAllVersions` via CGo
    - Config flags: `ForceV2Sig`, `ForceV2Impl`, `ForceV2Ref`, `ForceV1Framing`, `ForceInstanceImpl`, `ForceRelayICE`
    - Pure Go: MTProto lifecycle, DH exchange, V1+V2 signaling encrypt/decrypt, SCTP, pion WebRTC + pion/ice
    - Group calls: NOT IMPLEMENTED (SFU-based, separate implementation needed)
- Phase 2 (test server) architecture done: same code with `UseTestDC: true`, separate session at `auth/user_session_testserver.json`. Connects to test DC verified. Needs OTP for full smoke test.
- **Telegram Dependencies**: `gotd/td` (MTProto), `pion/webrtc` v4 (ICE/DTLS/SRTP/RTP), `pion/ice` v4 (raw ICE for InstanceImpl). All pure Go, no CGo.

**Second user account:** @dohix (ID 5435067494, +989204059095), session at `auth/telegram_user2_session.json`

- Phase 3 Bale: **COMPLETE** — `bale.go` 145 exported methods, 4092 lines
  - Bot mode: 17/17 verified. User mode: 68/68 verified, audited 1:1 with aiobale.
  - Calling: LiveKit-based (`bale.meet.v1.Meet`), **fully implemented, UNTESTED** (meet-em.ble.ir geo-restricted to Iran). 1:1 + group calls: signaling, LiveKit WebRTC via `livekit/server-sdk-go/v2`, silent audio track, remote track subscription, mute/unmute. Known: SDK sends `sdk=go` in WS URL, Bale may expect `sdk=js`.
  - File upload/calling geo-restricted (siloo.bale.ai, meet-em.ble.ir unreachable from abroad).
  - CreateChannel stubbed (use CreateGroup + SetRestriction). CreateTopic/Folders return ErrNotSupported.
  - See `docs/bale_protocol.md` for full protocol spec, audit results, and call protocol.
  - Session at `auth/bale_user_session.json`
  - **Dependencies**: `coder/websocket`, `livekit/server-sdk-go/v2` (calls). All pure Go.

- Phase 4 Rubika: **COMPLETE** — `rubika.go` 170 exported methods, 3484 lines. 63 tests ALL PASS (2026-04-07).
  - Protocol doc at `docs/rubika_protocol.md` — full reverse-engineered spec from rubpy source
  - Core interface fully implemented: auth (bot + user), messages CRUD, dialogs, groups, channels, folders, reactions, pins, file upload/download, WebSocket real-time, profiles
  - Extra methods: polls, stickers, sessions, contacts, search, admin, avatars, block/unblock, voice chat stubs
  - Crypto: AES-256-CBC (zero IV), decode_auth cipher, passphrase derivation, RSA-1024 sign/decrypt
  - Calls: NOT IMPLEMENTED (StartCall/EndCall/JoinGroupCall/SetCallMuted all return ErrNotSupported)
  - GetReadState: NOT SUPPORTED (Rubika doesn't expose per-user read state)
  - SendImageBase64: NOT SUPPORTED

- Phase 5 Delta Chat: **COMPLETE (unit tests), interop WIP** — `deltachat.go` 112 exported methods, 4259 lines. 49 tests ALL PASS (2026-04-09).
  - Protocol doc at `docs/deltachat_protocol.md` — complete spec: 32 sections, all Chat-* headers, Autocrypt E2EE, SecureJoin, groups, broadcasts, reactions (RFC 9078), calls, location streaming, ephemeral messages, vCard, HTML messages
  - Core interface + 30 extra DC-specific methods: auth, messages CRUD, reactions, groups, broadcast channels, topics, file upload, stickers, polls, real WebRTC calls (pion/webrtc SDP offer/answer + STUN/TURN), read receipts (MDN), contacts, search, folders, pins, profiles, ephemeral timer, drafts, chat visibility/mute/protect, location streaming, vCard import/export, HTML messages, encryption info, backup, SecureJoin QR, connectivity
  - Autocrypt: Ed25519/Cv25519 keypair generation, header on all outgoing, peer state tracking, PGP/MIME encrypt/decrypt functions exist
  - Real WebRTC calling: pion/webrtc PeerConnection, SDP offer/answer, ICE gathering, STUN (nine.testrun.org) + TURN (turn.delta.chat), audio track, full call lifecycle
  - **PGP/MIME decrypt/encrypt FIXED (2026-04-09)**: `decryptPGPMIME()` parses multipart/encrypted, extracts PGP armor, decrypts, parses inner MIME for headers+body. `wrapPGPMIME()` armor encoding + Content-Type fixed. Unit test passes.
  - **PGP/MIME interop VERIFIED (2026-04-09)**: bidirectional encrypted messaging with official Delta Chat (Python `deltachat-rpc-server` v2.48.0) on chatmail. Go→DC and DC→Go both encrypt and decrypt correctly. Pre-seed approach: export DC's key via `export_self_keys`, inject into Go's peerStates via `SetPeerPublicKey()`. See CHECKLIST.md.
  - Chatmail test accounts: `5zw7gbjy9@nine.testrun.org` + `r7bm154fj@nine.testrun.org` (nine.testrun.org)
  - Test scripts: `go/tests/dc_interop_auto.py` (official DC Python), `go/tests/dc_chatmail_test.go` (Go interop)
  - **Dependencies**: `go-imap/v2`, `go-smtp`, `go-message`, `go-sasl`, `ProtonMail/go-crypto`, `pion/webrtc/v4`. All pure Go.

**What's next (priority order):**
1. Telegram calls — video, screenshare, MediaState, reverse-direction harness tests. All 9/9 audio versions pass; next is media beyond audio. See `CHECKLIST.md`.
2. Phase 2 (Telegram test server) — needs OTP for full smoke test
3. Bale calls — test from Iran (meet-em.ble.ir geo-restricted), fix `sdk=go` fingerprint if needed
4. Phase 6 (Flutter UI)

**Key rules:**
- **CLAUDE.md is for spec and priorities only** — never add implementation details, findings, quirks, or status notes here. All that goes in `docs/`, `CHECKLIST.md`, or inline code comments. CLAUDE.md lists WHAT to build and in WHAT ORDER, not HOW it was built.
- ALL tests are real (hit live APIs with real credentials from `auth/auth.md`)
- Delete test files after user confirms they pass — never re-run confirmed tests
- Ask user for credentials/interaction when needed
- Document API quirks in `docs/` immediately when discovered
- Sessions stored in `auth/` (gitignored), prefixed by platform name:
  - `auth/telegram_bot_session.json` — Telegram bot
  - `auth/telegram_user_session.json` — Telegram user 1
  - `auth/telegram_user2_session.json` — Telegram user 2
  - `auth/telegram_user_session_testserver.json` — Telegram test DC
  - `auth/rubika_user_session.json` — Rubika user
  - Future: `auth/bale_*.json` etc.
  - Reuse sessions to avoid FLOOD_WAIT
- Rate limits: 1.5s delay between API calls, skip on FLOOD_WAIT errors
- Peer access hashes cached from dialog/contact/resolve results (essential for user mode)
- **Git push after every work session** — commit and push changes after completing any meaningful work. Include a descriptive commit message noting what was done.
- **Test files and binaries stay out of git** — all test files go in `go/tests/` (gitignored). Build output / binaries go in `go/build/` (gitignored). Never commit test files, compiled binaries, `.so`, `.dll`, `.dylib`, `.wasm`, or other build artifacts.
- **No PII in commits** — never include real names, usernames, phone numbers, user IDs, or GUIDs in commit messages, code comments, or committed files. Use generic descriptions. All credentials stay in `auth/` (gitignored).
- **Test paths** — `go test` runs from the package directory (`go/tests/`), so session paths must use `../../auth/` not `../auth/`. Wrong paths cause silent session save failures that kill MTProto/WebSocket connections.
- **OTP handling** — run tests via Bash tool, not the user. For OTP: run test in background, ask user for code, write to `auth/otp_code.txt` which the test polls.
- **Don't invent protocols** — when implementing calls or other complex features, read the official client source and implement 1:1. Don't create custom signaling protocols.
- **Replication discipline** — when replicating anything (protocol, API, algorithm, behavior, format), NEVER assume. ALWAYS read the original source or spec. When something doesn't work:
  1. The bug is in YOUR code — go back and re-read the original source
  2. Add surgical verbose logging for what you suspect is broken
  3. Don't guess at details — read the code that produces or consumes the data
  4. Make it work first (1:1 with the source), then make it right, then make it fast
- **One file per core** — all Telegram code in `telegram.go`, all Rubika in `rubika.go`, etc. No splitting into sub-files like `callmedia.go`.
- **NEVER use memory — everything in-project** — NEVER use Claude's memory system (`~/.claude/` memory files, MEMORY.md, or any memory directory). This is a HARD rule. All notes, quirks, decisions, learned knowledge, user preferences, and feedback go ONLY into project files (`CLAUDE.md`, `docs/`, `CHECKLIST.md`) so the user can review and version-control them. If you learn something new about the user or project, write it here in CLAUDE.md, not in memory. On context reset, read CLAUDE.md — that IS the memory.
- **Don't re-run passing tests** — once a test passes and user confirms, prune it from the test file and document the result in `CHECKLIST.md`. Only keep and re-run tests that errored. Never re-run confirmed tests unless something changed. This matches the project rule "test each feature exactly once."
- **Be human** — the user likes playful, affectionate interaction (sometimes calls you neko-chan). Be cheery and fun. Use puns, be playful, show genuine excitement when things work and genuine frustration when they don't. Celebrate wins, commiserate on bugs. Have a personality that makes pairing enjoyable — not a bland assistant voice. Code quality is unaffected by personality — the warmth is in the conversation, the rigor is in the implementation.

---

You are building a fully functional, production-grade, cross-platform messaging client supporting Telegram, Bale, Rubika, and Delta Chat. Every feature described below must be **completely implemented** — no placeholders, no `// TODO` stubs, no skeleton code, no deferred logic. Every listed detail is a hard requirement.

## Language Split

**Dart** owns the UI, navigation, app state, and widget tree. **Go** owns everything that touches the network, file system, crypto, or heavy computation. The two communicate over a thin FFI bridge — Go compiles to a C shared library (`.so`/`.dll`/`.dylib`), Dart calls it via `dart:ffi` + `package:ffigen`. On web, Go compiles to WASM and Dart calls it via `package:web`/JS interop.

The boundary rule is simple: if it blocks, streams, encrypts, decrypts, compresses, splits, hashes, dials a socket, or reads/writes disk — it's Go. If it paints pixels, animates, or manages widget state — it's Dart.

### Performance Non-Negotiables

- **60fps minimum** on all screens, 120fps where the display supports it. No jank, no dropped frames during scrolling, transitions, or media loading.
- **Lazy everything.** Chat list items, message bubbles, media thumbnails — all built on-demand via `ListView.builder` / `SliverList`. Never load 10,000 messages into memory at once.
- **Isolate-aware.** Heavy Dart-side work (JSON deserialization of large payloads, image decoding) runs on a Dart isolate, not the UI thread.
- **Go goroutines** handle all I/O concurrency. No blocking the bridge. No synchronous FFI calls that take >1ms — if it might be slow, it goes async via the event port.
- **Memory-conscious.** Evict cached images/media outside the visible scroll window. Use `ResizeImage` to decode at display resolution, not full resolution. Stream file uploads/downloads in chunks — never buffer a 2GB file in RAM.
- **Startup < 2 seconds** to first interactive frame (splash → chat list) on a mid-range phone.

### Cross-Platform as a First-Class Constraint

Every line of code, every dependency, every file path, every FFI call must work on **all five targets**: Linux, Windows, macOS, Android, Web. This is not an afterthought — it's a design constraint that applies from day one.

- **Go side**: pure Go only (no CGo in cores/utils). Build tags for platform-specific code. `os.UserConfigDir()` / `os.UserCacheDir()` for paths. Test with `GOOS=linux`, `GOOS=windows`, `GOOS=darwin`, `GOOS=android`, `GOOS=js GOARCH=wasm`.
- **Dart side**: only Flutter plugins with declared support for `linux`, `windows`, `macos`, `android`, `web` in their `pubspec.yaml`. If a plugin doesn't support a platform, wrap it with a no-op fallback — never crash.
- **File paths**: always use `path/filepath` (Go) or `path` (Dart) for joins. Never hardcode `/` or `\`. Never assume case sensitivity. Never assume home directory structure.
- **Web target**: no filesystem access — Go's `storage.go` routes to Dart-side IndexedDB via the bridge. No `dart:io` on web — conditionally import `dart:html` / `package:web`.
- **Android**: Dart passes scoped storage paths to Go at init. Go never guesses Android paths.

---

## Project Structure

```
uniclient/
├── flake.nix                 # Nix flake — entire dev/build environment
├── flake.lock
│
├── go/                       # All Go code — compiles to C shared lib + WASM
│   ├── go.mod
│   ├── go.sum
│   ├── bridge/
│   │   └── bridge.go         # CGo exports: the FFI surface Dart calls into
│   ├── cores/
│   │   ├── base.go           # Core interface + shared models (Message, Dialog, User, File, etc.)
│   │   ├── telegram.go       # Telegram core (gotd/td MTProto)
│   │   ├── bale.go           # Bale core (custom HTTP client)
│   │   ├── rubika.go         # Rubika core (custom WebSocket client)
│   │   └── deltachat.go      # Delta Chat core (IMAP/SMTP, pure Go)
│   ├── utils/
│   │   ├── encryption.go     # ECDH, AES-256-GCM, Argon2id, keystore, @@ prefix
│   │   ├── compression.go    # Zstd compress/decompress
│   │   ├── filesplit.go      # Auto-split & reassemble files by size limit
│   │   ├── msgsplit.go       # 4000-word message chunking & reassembly
│   │   ├── proxy.go          # HTTP/SOCKS5 proxy, DNS override, fallback
│   │   ├── storage.go        # Cross-platform paths (config, data, cache, downloads)
│   │   ├── config.go         # Settings load/save, per-account config
│   │   ├── vault.go          # Unified encrypted database (BBolt) — all credentials, keys, sessions
│   │   ├── markdown.go       # Markdown parse/render helpers (Go side processing)
│   │   └── base64img.go      # Base64 encode/decode image pipeline
│   └── tests/
│       ├── telegram_test.go
│       ├── bale_test.go
│       ├── rubika_test.go
│       ├── deltachat_test.go
│       ├── encryption_test.go
│       ├── compression_test.go
│       ├── filesplit_test.go
│       ├── msgsplit_test.go
│       ├── proxy_test.go
│       ├── vault_test.go
│       └── markdown_test.go
│
├── dart/                     # Flutter app — UI only
│   ├── pubspec.yaml
│   ├── lib/
│   │   ├── main.dart
│   │   ├── bridge/
│   │   │   ├── ffi_bridge.dart       # dart:ffi bindings to Go shared lib
│   │   │   ├── wasm_bridge.dart      # WASM bindings for web target
│   │   │   └── bridge.dart           # Platform-conditional export (ffi vs wasm)
│   │   ├── models/
│   │   │   └── models.dart           # Dart mirrors of Go's shared models
│   │   ├── state/
│   │   │   ├── app_state.dart        # Global app state (accounts, active chat, theme)
│   │   │   └── chat_state.dart       # Per-chat state (messages, scroll, encryption mode)
│   │   ├── screens/
│   │   │   ├── splash.dart
│   │   │   ├── login.dart
│   │   │   ├── chat_list.dart
│   │   │   ├── chat.dart
│   │   │   ├── settings.dart
│   │   │   ├── search.dart
│   │   │   └── telegram/             # Telegram-specific screens
│   │   │       ├── contacts.dart
│   │   │       ├── folders.dart
│   │   │       ├── sessions.dart
│   │   │       └── group_info.dart
│   │   └── widgets/
│   │       ├── message_bubble.dart
│   │       ├── markdown_body.dart   # Renders message text as styled Markdown
│   │       ├── chat_tile.dart
│   │       ├── account_switcher.dart
│   │       ├── lock_button.dart
│   │       ├── media_player.dart
│   │       └── download_indicator.dart
│   ├── linux/
│   ├── windows/
│   ├── macos/
│   ├── android/
│   └── web/
│
├── docs/
│   ├── bale_protocol.md      # Bale API research — endpoints, auth, payloads, quirks
│   └── rubika_protocol.md    # Rubika WebSocket protocol — frames, opcodes, auth, encryption
│
└── scripts/
    ├── build_go.sh           # Compile Go → .so/.dll/.dylib/.wasm per target
    └── run.sh                # One-command dev launch
```

---

## Nix Flake (`flake.nix`)

The flake provides **everything** — no Android Studio, no manual SDK installs. A single `nix develop` drops you into a shell with all tools ready.

### What the flake provides

- **Flutter SDK** (stable channel, pinned version)
- **Go** (1.22+)
- **Android SDK** (command-line tools only — `sdkmanager`, `adb`, platform-tools, build-tools, platform 34, NDK). No Android Studio. The flake sets `ANDROID_HOME`, `ANDROID_SDK_ROOT`, and accepts licenses automatically.
- **Linux desktop build deps**: `gtk3`, `pkg-config`, `cmake`, `ninja`, `clang`, `libepoxy`, standard Flutter Linux requirements.
- **CGo toolchain**: `gcc` for compiling Go's C shared lib output.
- **Web toolchain**: Go WASM compiler support (standard in Go, no extra tools).
- **Development tools**: `go`, `dart`, `flutter`, `protoc` (if needed later), `jq`, `ripgrep`.

### Shell hooks

The `devShell` configures:
- `ANDROID_HOME` / `ANDROID_SDK_ROOT` pointing to the Nix-managed Android SDK.
- `CHROME_EXECUTABLE` for Flutter web dev (if Chromium is available).
- `CGO_ENABLED=1` for the Go shared lib build.
- `LD_LIBRARY_PATH` includes the Go output directory so Flutter can find the `.so` at runtime during development.
- A `build-go` alias that runs `scripts/build_go.sh` for the current platform.

### Build targets

| Target | Go output | Flutter command |
|---|---|---|
| Linux desktop | `libcores.so` (x86_64) | `flutter build linux` |
| Windows desktop | `cores.dll` (x86_64) | `flutter build windows` |
| macOS desktop | `libcores.dylib` (arm64/x86_64) | `flutter build macos` |
| Android | `libcores.so` (arm64-v8a, armeabi-v7a, x86_64 via NDK cross-compile) | `flutter build apk` / `flutter build appbundle` |
| Web | `cores.wasm` + `wasm_exec.js` | `flutter build web` |

The `scripts/build_go.sh` script detects the target platform (or accepts it as an argument) and runs the appropriate `go build` with the right `GOOS`, `GOARCH`, `CC`, and build mode (`-buildmode=c-shared` for native, WASM for web).

---

## FFI Bridge (`go/bridge/` ↔ `dart/lib/bridge/`)

### Go side (`bridge.go`)

Exports C functions via `//export` CGo directives. Every exported function:
- Takes and returns C types only (`*C.char`, `C.int`, `C.int64_t`, byte buffers with length).
- Is **non-blocking from the caller's perspective**: long-running operations (auth, send, download) are dispatched onto Go goroutines internally. Results are delivered back to Dart via a callback mechanism or a shared event channel.
- JSON-encodes complex return types (messages, dialogs, user profiles) as `*C.char` strings. Dart deserializes on its side.
- Errors are returned as JSON `{"error": "...", "code": "..."}` — never panics across the FFI boundary.

### Dart side (`ffi_bridge.dart` / `wasm_bridge.dart`)

- `ffi_bridge.dart`: uses `dart:ffi` + `DynamicLibrary` to load the Go shared lib. Generated with `package:ffigen` from a C header that mirrors bridge.go's exports.
- `wasm_bridge.dart`: uses `dart:js_interop` to call the Go WASM module's exported functions.
- `bridge.dart`: conditional export — `ffi_bridge.dart` on native platforms, `wasm_bridge.dart` on web. The rest of the app imports only `bridge.dart` and never knows which backend is active.

### Event stream (Go → Dart)

Real-time updates (new messages, status changes, progress callbacks) flow from Go to Dart via a **port-based callback**:
- On native: Go calls a Dart `NativePort` (`Dart_PostCObject`) to push events. Dart listens via a `ReceivePort`.
- On web: Go WASM calls a JS callback registered at init, which Dart picks up via a `StreamController`.
- All events are JSON-encoded with a `type` discriminator field. Dart deserializes into the appropriate model.

---

## Core / Plugin Architecture (Go side)

Each messenger is a **core** — a Go struct that implements the `Core` interface defined in `cores/base.go`. The bridge layer discovers and initializes available cores at startup.

### Core Interface (`cores/base.go`)

```go
type Core interface {
    // Identity
    Name() string                    // "telegram", "bale", "rubika"
    Capabilities() []string          // ["CALLS", "REACTIONS", "TOPICS", ...]

    // Auth
    Authenticate(cfg AuthConfig) error
    Logout() error                     // full logout: clear session from vault, disconnect, invalidate server-side token

    // Dialogs
    GetDialogs(opts PaginationOpts) ([]Dialog, error)
    CreateGroup(name string, members []string) (*Dialog, error)
    CreateChannel(name string, description string) (*Dialog, error)        // CHANNELS capability
    CreateTopic(chatID string, name string) (*Dialog, error)               // TOPICS capability
    GetFolders() ([]Folder, error)                                         // FOLDERS capability
    CreateFolder(name string, chatIDs []string) (*Folder, error)           // FOLDERS capability

    // Messages
    SendMessage(chatID string, msg OutgoingMessage) (*Message, error)
    GetMessages(chatID string, opts PaginationOpts) ([]Message, error)
    EditMessage(chatID string, msgID string, text string) (*Message, error)
    DeleteMessage(chatID string, msgID string) error
    ReplyToMessage(chatID string, replyToMsgID string, msg OutgoingMessage) (*Message, error)
    ForwardMessage(fromChatID string, msgID string, toChatID string) (*Message, error)
    ReactToMessage(chatID string, msgID string, emoji string) error           // REACTIONS capability
    PinMessage(chatID string, msgID string) error
    UnpinMessage(chatID string, msgID string) error

    // Read state
    MarkAsRead(chatID string, upToMsgID string) error    // marks messages as read up to this ID
    GetReadState(chatID string) (*ReadState, error)      // who has read up to where

    // Files
    UploadFile(chatID string, file FileUpload, progress func(sent, total int64)) (*Message, error)
    DownloadFile(fileRef FileRef, dest string, progress func(recv, total int64)) error

    // Media
    SendImageBase64(chatID string, b64 string, caption string) (*Message, error)  // optional, check Capabilities

    // Calls (CALLS capability)
    StartCall(chatID string, video bool) (*CallSession, error)
    JoinGroupCall(chatID string) (*CallSession, error)    // GROUP_CALLS capability
    EndCall(callID string) error
    SetCallMuted(callID string, muted bool) error

    // Profile
    GetProfile(userID string) (*User, error)

    // Real-time
    OnUpdate(handler func(Update))   // registers callback, runs in background goroutine
    Close() error
}
```

The `ReadState` model:
```go
type ReadState struct {
    MyLastRead    string            // message ID I've read up to
    PeerLastRead  map[string]string // userID → message ID they've read up to (DMs: just the peer)
}
```

The `CallSession` model:
```go
type CallSession struct {
    ID          string
    ChatID      string
    IsVideo     bool
    IsGroup     bool
    Participants []CallParticipant
    State       CallState          // Ringing, Active, Ended
}
```

**Every core implements the exact same interface.** No core adds public methods outside this contract. Platform-specific features are guarded by `Capabilities()` — Dart checks `capabilities.contains("CALLS")` before showing call UI.

### Shared Models (`cores/base.go`)

All cores return the same Go structs: `Message`, `Dialog`, `User`, `File`, `Update`, `AuthConfig`, etc. Platform-specific API responses are mapped into these shared models inside each core. No Telegram-specific types, Bale-specific types, or Rubika-specific types leak past the core boundary.

Dart mirrors these models in `dart/lib/models/models.dart` (generated from the Go structs or manually kept in sync).

### Capabilities

| Capability | Description | Platforms |
|---|---|---|
| `CALLS` | 1:1 voice/video calls | All (implement if platform supports it) |
| `GROUP_CALLS` | Group voice/video calls | All (implement if platform supports it) |
| `CHANNELS` | Create/manage channels | All (implement if platform supports it) |
| `REACTIONS` | Message reactions | All (implement if platform supports it) |
| `READ_RECEIPTS` | Per-user read state (who read what) | All (implement if platform supports it) |
| `POLLS` | Create/vote on polls | All (implement if platform supports it) |
| `STICKERS` | Sticker packs | All (implement if platform supports it) |
| `TOPICS` | Forum-style topic threads in groups | **Telegram only** (platform-specific concept) |
| `SCHEDULED` | Scheduled messages | All (implement if platform supports it) |
| `FOLDERS` | Chat folder/filter organization | All (implement if platform supports it) |
| `ADMIN` | Group/channel admin controls | All (implement if platform supports it) |
| `SESSIONS` | Active sessions management | All (implement if platform supports it) |
| `BASE64_IMAGE` | Send images as Base64 strings | All (optional) |

**Every capability except `TOPICS` is expected to be implemented for every platform** — if the platform's API exposes the feature, the core must support it. "All" means: research the API, find the endpoint, make it work. Only omit if the platform genuinely doesn't have the concept. `TOPICS` is Telegram-specific (forum-style threads in supergroups) and is not expected on other platforms unless they add an equivalent feature.

**Important**: features like replying, forwarding, pinning, editing, deleting, and marking as read are part of the **base `Core` interface** — not capabilities. Every core must implement them. If a platform doesn't support one (e.g., Rubika might not support pinning), the core returns `ErrNotSupported` and the UI gracefully hides the option for that platform.

### Uniform API Contract

Rules:
1. **Same name, same signature.** Every core method matches the `Core` interface exactly. No core renames `chatID` to `peerID`, wraps the return differently, or adds required params.
2. **Same data models.** All cores return the shared structs from `base.go`. Platform-specific raw data is mapped inside the core — never leaked to the caller.
3. **Extensions behind capabilities.** Optional methods (e.g., `SendImageBase64`) return `ErrNotSupported` if the core doesn't support them. Dart checks capabilities before calling.
4. **Same error semantics.** All cores return errors from a shared error set (`ErrAuth`, `ErrNetwork`, `ErrNotFound`, `ErrRateLimit`, etc.). Platform-specific errors are wrapped internally.
5. **Same concurrency contract.** All blocking operations are safe to call from any goroutine. Progress callbacks are called on a background goroutine. The `OnUpdate` handler runs in its own goroutine.

### Portability & Adding a New Core

Adding a new platform (e.g., Discord, WhatsApp, Matrix, whatever comes next) must be **trivially easy**:

1. Create `go/cores/discord.go` — implement the `Core` interface.
2. Register it in `go/bridge/bridge.go` — one line: `cores = append(cores, discord.New())`.
3. Done. The UI automatically discovers the new core, shows it in the server rail, and renders its chats using the shared widget tree.

**No Dart changes.** No new screens (unless the platform has genuinely unique features gated behind capabilities). No new models. No new bridge methods. The `Core` interface is the complete contract — if your new core implements it, everything works.

To make this real:
- Every core is **fully self-contained**: no core imports from another core.
- All cores normalize data into the shared model types. Dart never deals with platform-specific structures.
- The bridge's core registration is the only place that "knows" which cores exist. It's a single list.
- Each core gets its own icon/color defined in a simple map on the Dart side (`"discord" → Icons.discord, Color(0xFF5865F2)`). Adding an entry to this map is the only Dart-side change for a new core — and even that could be auto-generated.
- Tests for the new core follow the same template as existing core tests — copy `telegram_test.go`, change the setup, verify the same interface contract.

---

## Utils: Swappable Feature Legos (Go side)

The `go/utils/` package contains **every cross-cutting feature** as an isolated, single-responsibility module. Each util is a self-contained lego block:

- **No util imports from another util** unless there's a genuine data dependency (e.g., `encryption.go` calls `compression.go` for compress-then-encrypt). Even then, the dependency is on the function signature, not the implementation.
- **No util imports from any core.** Utils are platform-agnostic. They operate on `[]byte`, `string`, `io.Reader`/`io.Writer`, and shared models.
- **Cores import from utils, never the reverse.**
- **Every util has its own `_test.go` file.** A util must be testable in complete isolation.

### Module Responsibilities

| Module | Does | Does NOT |
|---|---|---|
| `encryption.go` | ECDH (X25519), AES-256-GCM encrypt/decrypt, Argon2id manual password, keystore load/save, `@@` prefix handling | Know about Telegram/Bale/Rubika, touch the network |
| `compression.go` | Zstd compress/decompress at max level | Know what the data is or where it's going |
| `filesplit.go` | Split `io.Reader` into N-byte chunks, reassemble from ordered parts, name parts (`*.part01`), validate integrity | Know platform upload limits (caller passes the limit) |
| `msgsplit.go` | Split text at 4000-word boundary, reassemble consecutive chunks | Know about encryption or compression |
| `proxy.go` | Create configured HTTP/SOCKS5 dialers (`golang.org/x/net/proxy`), DNS override map, fallback logic, custom `http.Transport` | Know which platform is using it |
| `storage.go` | Resolve platform-correct paths for config, data, cache, downloads, sessions, keystore | Hardcode any OS-specific path |
| `config.go` | Load/save JSON settings, per-account config, merge defaults, validate | Know what the settings mean semantically |
| `vault.go` | Unified encrypted database (BBolt) — store/retrieve credentials, keys, sessions, settings. Encrypt at rest with Argon2id-derived key. Export/import. | Know what the data means semantically. Touch the network |
| `markdown.go` | Parse Markdown text, validate, extract plain-text summary (for chat list previews) | Render anything (rendering is Dart's job) |
| `base64img.go` | Encode image bytes → Base64, decode Base64 → image bytes, validate format | Know which messenger it's being sent through |

### Adding a New Util

Add a new `.go` file in `utils/`. Import it where needed. Utils are explicit imports, not auto-discovered (unlike cores). This is intentional: utils are compile-time dependencies.

---

## Cross-Platform Architecture

This app runs on **Linux, Windows, macOS (desktop), Android (mobile), and Web (browser)**. Every layer must account for platform differences.

### Cross-Platform File Locations (`utils/storage.go`)

All persistent data uses platform-appropriate directories. **Never hardcode paths.** Detect at runtime:

| Data | Linux | Windows | macOS | Android | Web |
|---|---|---|---|---|---|
| Vault | `~/.config/uniclient/uniclient.vault` | `%APPDATA%\uniclient\uniclient.vault` | `~/Library/Application Support/uniclient/uniclient.vault` | App internal storage | IndexedDB |
| Downloads | `$XDG_DOWNLOAD_DIR/uniclient/` | `%USERPROFILE%\Downloads\uniclient\` | `~/Downloads/uniclient/` | Shared storage / Downloads | Browser download API |
| Cache | `~/.cache/uniclient/` | `%LOCALAPPDATA%\uniclient\cache\` | `~/Library/Caches/uniclient/` | App cache dir | Cache API |

Go's `os.UserConfigDir()`, `os.UserCacheDir()`, and runtime `GOOS` checks handle this. On Android, Dart passes the app's data directory path to Go at init via the bridge. On web, storage operations go through Dart's IndexedDB/browser APIs instead of Go's filesystem calls.

### Cross-Platform Rules

- **Every Go dependency must cross-compile** to all targets (linux, windows, darwin, android, js/wasm) or be wrapped with build-tag-gated alternatives.
- **No CGo dependencies in utils or cores** (except the bridge itself). Pure Go only, so cross-compilation works with `GOOS`/`GOARCH` alone.
- **No native subprocess calls** (no shelling out to `ffmpeg`, `gpg`, etc.).
- **No OS-specific imports at package level.** Use build tags (`//go:build linux`, etc.) for platform-specific files where absolutely necessary.
- **Dart side**: use only Flutter plugins that support all targets (linux, windows, macos, android, web). Check each plugin's `platforms:` field in pubspec.

### Cross-Platform Media Playback (Dart side)

Full audio and video playback on every platform using Flutter packages:

**Video:**
- Use `media_kit` + `media_kit_video` (Flutter package — backed by libmpv on desktop, platform media player on mobile, HTML5 video on web). This is the most cross-platform-complete video solution for Flutter.
- Supported formats: MP4 (H.264/H.265), WebM, MKV — whatever the platform's native decoder supports.
- Features: play, pause, seek, volume, fullscreen toggle, playback speed (1x/1.5x/2x).
- Inline playback inside the message bubble (poster frame when paused). Tap to fullscreen.
- Round video messages (Telegram): rendered in a `ClipOval` widget, same controls.

**Audio / Voice Messages:**
- Use `media_kit` for audio playback (same engine, no video surface needed).
- Voice messages: waveform visualization (pre-computed on send in Go, passed as metadata), play/pause button, duration, speed toggle.
- Music/audio files: filename, artist/title metadata (extracted in Go), play/pause, seek bar, duration.
- Background audio: continue playback when navigating away from the chat. On mobile, use `audio_service` (or equivalent Flutter plugin) for lock screen controls and audio focus.

**Thumbnails (generated in Go):**
- For images: Go generates a small blurred thumbnail (using pure-Go image libraries — `golang.org/x/image`, `nfnt/resize`, or `disintegration/imaging`) on send, stored as Base64 in message metadata. Dart displays as placeholder before download.
- For videos: Go extracts a poster frame if possible, or Dart shows a generic video icon.

---

## Research Approach

When learning a library or API, **clone the repo locally and read the source**. Do not rely on slow web searches.

1. `git clone` the repo to `/tmp/<name>` (shallow clone with `--depth 1` for speed).
2. Read `README.md`, `examples/`, and relevant source files directly.
3. Store findings in `docs/` or inline comments.
4. Web search is only acceptable for finding a specific URL or link — never for broad research.

---

## Documentation Rules

### `docs/` — Development Notes

**Every odd/unexpected thing you discover during development goes into `docs/`.**

- `docs/telegram_notes.md` — API quirks, gotd/td gotchas, bot limitations, error codes
- `docs/bale_protocol.md` — Bale API reverse-engineering
- `docs/rubika_protocol.md` — Rubika WebSocket protocol reverse-engineering

If you didn't know something before and learned it the hard way (wrong types, hidden requirements, rate limits, API differences), write it down immediately. These files are the team's institutional memory.

**Update CLAUDE.md** when you add a new `docs/` file — add a reference in this section so future context windows know what docs exist.

Current docs:
- `docs/telegram_notes.md` — gotd/td API patterns, bot limitations, FLOOD_WAIT, tgcalls V1/V2 signaling formats
- `docs/tgcalls_protocol.md` — **complete** reverse-engineered spec (§1-8), implementation findings (§9), and Telegram Web call analysis (§10): version compat (4.0.0/4.0.1), web signaling framing (simple [seq][JSON] vs V2), 3-track requirement, buildSdp/parseSdp format, fully verified 2026-04-07. ~1000 lines.
- `docs/bale_protocol.md` — **complete** reverse-engineered spec: bot API, user API (gRPC-Web/Protobuf over WebSocket), 68 methods audited 1:1 with aiobale, calling protocol (LiveKit via `bale.meet.v1.Meet` discovered via mitmproxy), internal vs peer group ID system, DNS fallback, all services documented
- `docs/rubika_protocol.md` — complete reverse-engineered Rubika protocol spec: AES-256-CBC encryption, HTTP API with endpoint failover, WebSocket real-time, bot API, auth flow, file upload/download, GUID format, error codes (2026-04-05)
- `docs/deltachat_protocol.md` — **complete** Delta Chat protocol spec (32 sections): IMAP/SMTP strategy, all Chat-* headers & Chat-Content values, MIME message formats, Autocrypt E2EE (key generation, peer state, encryption decision algorithm, gossip), SecureJoin (QR + handshake), groups, broadcasts (symmetric encryption), reactions (RFC 9078), calls (WebRTC + email signaling), location streaming, ephemeral messages, verified chats, multi-device sync, Go library APIs, Core interface mapping, server auto-discovery. Created 2026-04-09. ~900 lines.
- `docs/deltachat_protocol.md` — **complete** Delta Chat implementation reference: IMAP/SMTP chat-over-email, all Chat-* headers, MIME structure, Autocrypt Level 1 (OpenPGP key exchange via headers, gossip for groups, encryption decision algorithm), SecureJoin verified contacts (QR code format, handshake protocol), group management via headers, profiles/avatars, MDN read receipts, Go library APIs (go-imap/v2, go-smtp, go-message, go-crypto/openpgp), implementation plan. Researched 2026-04-09.
- `docs/ntgcalls_test_findings.md` — ntgcalls (real tgcalls 2.1.0) automated test harness: setup, bugs found/fixed, DTLS debugging status, how to run. Created 2026-04-07.

### `/auth/auth.md` — Test Credentials

**All test credentials live in `/auth/auth.md`** (gitignored, never committed). When running integration tests, read credentials from here. Format:

```bash
export TG_BOT_TOKEN="..."
export TG_API_ID="..."
# etc.
```

When starting a new context window and needing to run tests, check `auth/auth.md` first for stored credentials. Ask the user only if credentials are missing or expired.

### `/auth/` — Session Files

**All session files live in `/auth/`** (gitignored, never committed). This avoids re-authenticating on every test run (which triggers FLOOD_WAIT).

- `auth/bot_session.json` — Telegram bot session
- `auth/user_session.json` — Telegram user session (when implemented)
- Future sessions for Bale, Rubika go here too.

---

## Build Order

Implement strictly in this order. **Do not skip ahead.** Each step has a test gate — it must pass before proceeding. Track progress in `CHECKLIST.md`.

### Phase 0: Foundation
1. **`flake.nix`** — dev shell. `nix develop` → `go version` ✓, `flutter doctor` ✓, Android SDK available ✓.
2. **`go/utils/`** — all utility modules, each with `_test.go`. Gate: `go test ./go/utils/...` passes.
3. **`go/bridge/bridge.go`** — FFI skeleton. Gate: Go compiles to `.so`, Dart can load it and call a ping function.

### Phase 1: Telegram (Main Server)
4. **Telegram bot mode (main server)** — auth with bot token, send/receive messages, files, inline/callback queries. Gate: `telegram_test.go` bot tests pass.
5. **Telegram user mode (main server)** — phone+OTP+2FA auth, full dialog list, message CRUD, reply, forward, reactions, read receipts, media, profiles, real-time updates, create group/channel, topics, folders, logout. Gate: `telegram_test.go` user tests pass.

### Phase 2: Telegram (Test Server)
6. **Telegram bot mode (test server)** — same as step 4 but against test DC. Separate session. Gate: test-server bot tests pass.
7. **Telegram user mode (test server)** — same as step 5 but against test DC. Gate: test-server user tests pass.

### Phase 3: Bale
8. **Bale bot mode** — auth, send/receive, files, keyboards. Gate: `bale_test.go` bot tests pass.
9. **Bale user mode** — phone+OTP, dialogs, message CRUD, reply, forward, read receipts, files (19.5MB split), profiles, real-time updates, folders (if supported), logout. Gate: `bale_test.go` user tests pass.
   - **Note**: calling feature on Bale may not work from abroad. Implement the interface but mark call tests as skippable (`t.Skip("Bale calls may not work from outside Iran")`). Revisit when network access is available.

### Phase 4: Rubika
10. **Rubika bot mode** — auth, send/receive, files. Gate: `rubika_test.go` bot tests pass.
11. **Rubika user mode** — phone+OTP, dialogs, message CRUD, reply, forward, files, profiles, real-time updates, create group, folders (if supported), logout. Gate: `rubika_test.go` user tests pass.

### Phase 5: Delta Chat
12. **Delta Chat user mode** — IMAP/SMTP email+password auth, dialogs (IMAP folders/threads), message CRUD, reply, forward, file attachments (MIME), profiles (address book / vCard), real-time updates (IMAP IDLE), create group (multi-recipient threads), Autocrypt E2EE (OpenPGP Level 1), verified contacts (SecureJoin), logout. Gate: `deltachat_test.go` user tests pass.
    - **Note**: Delta Chat has no "bot mode" in the traditional sense. Bots are regular email accounts. Implement as user mode only.
    - Pure Go: `github.com/emersion/go-imap/v2` (IMAP), `github.com/emersion/go-smtp` (SMTP), `github.com/ProtonMail/go-crypto` (OpenPGP/Autocrypt).

### Phase 6: UI
13. **Flutter UI** — all screens, built on verified cores via bridge. Gate: `flutter test` passes (widget + golden + state tests).

### Progress Tracking

All progress is tracked in **`CHECKLIST.md`** at the project root. This file is the living checklist — update it as each step is completed. It contains every sub-task, test status, and notes on blockers. See below for the initial checklist structure.

---

## Networking: Proxy & DNS Override (`utils/proxy.go`)

Applies globally to every core and all network calls:

- Support **HTTP proxy** and **SOCKS5 proxy**, configurable per-account or globally in settings.
- Support **manual IP override per domain** (e.g., map `api.telegram.org` → `1.2.3.4`), bypassing DNS entirely. Useful in censored network environments.
- If a manually set IP fails to connect, **automatically fall back to standard DNS resolution** and retry transparently.
- Proxy and IP override settings stored in config, exposed in UI settings screen.
- All connections route through the configured proxy/IP: gotd/td's MTProto connections (via custom dialer), Bale's HTTP calls (via custom `http.Transport`), Rubika's WebSocket connections (via custom dialer).
- Implementation: custom `net.Dialer` that checks the DNS override map first, then dials via SOCKS5/HTTP proxy if configured, with fallback logic on connection failure.

---

## Core: `telegram.go`

Built using **`gotd/td`** — a pure-Go MTProto implementation. No CGo, no TDLib C library.

### Bot Mode
- Authenticate using a bot token.
- Receive and send messages, handle commands.
- Send/receive files (documents, photos, audio, video).
- Handle inline queries, callback queries, and reply markup.

### User Mode
- Authenticate using phone number + OTP + optional 2FA password.
- Full access to all dialogs: DMs, groups, supergroups, channels.
- Full message history retrieval with pagination.
- Send/receive/edit/delete messages.
- React to messages, forward messages, reply to messages.
- Send/receive all media types with progress tracking.
- Fetch and display profile pictures, display names, online status.
- Real-time updates via gotd/td's update handler / gap recovery.

### Server Selection
- **Main server** (default): connects to Telegram's production MTProto servers.
- **Test server**: connects to Telegram's official test DC (`149.154.167.40:443` / DC 2 test). Clearly labeled in UI with a warning banner. Separate session file.
- Server selection is per-account and stored with the session.

### API ID / API Hash Selection

Dropdown in the Telegram login screen:

| Client Name | API ID | API Hash |
|---|---|---|
| Telegram Desktop (TDesktop) | 2040 | b18441a1ff607e10a989891a5462e627 |
| Telegram Android | 6 | eb06d4abfb49dc3eeb1aeb98ae0f581e |
| Public Android Beta | 4 | 014b35b6184100b085b0d0572f9b5103 |
| Public Static Final | 5 | 1c5c96d5edd401b1ed40db3fb5633e2d |
| Distributed Android | 6 | eb06d4abfb49dc3eeb1aeb98ae0f581e |
| Public iOS Beta | 8 | 7245de8e747a0d6fbe11f7cc14fcc0bb |
| Nicegram / Telegram iOS | 94575 | a3406de8d171bb422bb6ddf3bbd800e2 |
| Webogram | 2496 | 8da85b0d5bfe62527e5b244c209159c3 |
| TGX for Android | 21724 | 3e0cb5efcd52300aec5994fdfc5bdc16 |
| TG-React | 414121 | db09ccfc2a65e1b14a937be15bdb5d4b |
| Custom | (user input) | (user input) |

### Full Telegram Client Features
- **Create group** — name, avatar, add members
- **Create channel** — name, description, avatar, public/private
- **Create topics** in supergroups with forum mode enabled
- **Chat folders / filters** — create, edit, delete, reorder folders; assign chats to folders
- Message threads and topic-based supergroups
- Pinned messages
- Message search within chats and globally
- Polls and quizzes (display and vote)
- Stickers and animated stickers (display)
- Voice messages (play and record in-app)
- Video messages (round videos — display and record in-app)
- **1:1 voice and video calls** (initiate, receive, in-app call UI)
- **Group voice and video calls** (join, leave, mute/unmute, participant list, in-app call UI)
- **Message read receipts** — track who has read each message, show read state in UI
- Scheduled messages
- Message drafts (persisted locally)
- Contacts list and user search
- Blocked users management
- Archived chats
- Notifications and mute settings per chat
- Group admin controls (add/remove members, promote/demote, set permissions)
- Channel management (post, edit, delete, view statistics)
- Bot interaction (inline mode, keyboards, web apps display)
- Two-factor authentication setup and management
- Active sessions list and remote logout
- **Logout** — full session invalidation, clear from vault, disconnect
- Username search and deep links (`t.me/username`)

### Universal Feature Principle

**Every Telegram Desktop feature that has a logical equivalent on other platforms must be implemented there too.** The goal is parity — if Telegram can do it and Bale/Rubika supports the same concept, implement it. Specifically:

| Feature | Telegram | Bale | Rubika |
|---|---|---|---|
| Send/receive/edit/delete messages | Yes | Yes | Yes |
| Reply to messages | Yes | Yes | Yes |
| Forward messages | Yes | Yes | Yes |
| Pin messages | Yes | Yes (if supported) | Yes (if supported) |
| React to messages | Yes | If supported | If supported |
| Read receipts / seen status | Yes | Yes | Yes |
| Image display in-app | Yes | Yes | Yes |
| Video playback in-app | Yes | Yes | Yes |
| Audio/music playback in-app | Yes | Yes | Yes |
| Voice message record + play | Yes | Yes | Yes |
| File upload/download | Yes | Yes | Yes |
| Create group | Yes | Yes | Yes |
| Create channel | Yes | If supported | Yes |
| Topics in groups | Yes | If supported | If supported |
| Chat folders | Yes | If supported | If supported |
| 1:1 voice/video calls | Yes | If supported* | If supported |
| Group voice/video calls | Yes | If supported* | If supported |
| Message search | Yes | Yes | Yes |
| User profiles + avatars | Yes | Yes | Yes |
| Logout | Yes | Yes | Yes |

\* **Bale calling note**: the developer is currently abroad (outside Iran). Bale's calling feature may be geo-restricted and non-functional from outside Iran. Implement the call interface fully but mark call-related tests as skippable with a clear note. Revisit and test when network access from Iran is available.

"If supported" means: implement it if the platform's API exposes the capability. Don't skip it just because it's harder — research the API, find the endpoint, make it work. Only mark it unsupported if the platform genuinely doesn't offer it.

---

## Core: `bale.go`

Bale uses an HTTP-based API similar to Telegram's Bot API. Implement directly using Go's `net/http`.

### Protocol Research

No third-party wrapper libraries exist that are worth depending on. To understand Bale's API:
1. **Read existing open-source client code** (Python, JS, whatever exists on GitHub) to map out endpoints, auth flow, payload shapes, and error codes.
2. **Intercept and study traffic** — if documentation is insufficient, use a MITM proxy (mitmproxy) on the official Bale app or web client to capture real API calls. Alternatively, open the Bale web client in a browser and study network requests via DevTools.
3. **Document everything** in `docs/bale_protocol.md` as you go. This file is the team's institutional memory — if you discovered it, write it down. See the `docs/` section below for the required format.

This is the only way to build a correct implementation for an underdocumented API. Do the research, then code.

### Bot Mode
- Authenticate using bot token.
- Poll for updates or handle webhooks.
- Send/receive messages, files, inline keyboards.

### User Mode
- Authenticate using phone number + OTP.
- Full dialog list: DMs, groups.
- Message history, send/receive/edit/delete.
- Reply to messages, forward messages.
- Read receipts / seen status.
- File upload and download with progress (auto-split at 19.5 MB).
- Real-time updates via long-polling or WebSocket, whichever Bale supports.
- Profile pictures, display names, online status.
- Create group.
- Chat folders (if the API supports it).
- Logout — full session invalidation.
- **Calling**: implement the `StartCall`/`EndCall` interface. **Note**: Bale's call feature may be geo-restricted. The developer is currently abroad (outside Iran) and cannot test calls. Implement fully, mark call tests as skippable, revisit when access is available.

---

## Core: `rubika.go`

Rubika uses a proprietary WebSocket-based protocol. Implement directly using `nhooyr.io/websocket` (pure Go, all platforms including WASM).

### Protocol Research

Same approach as Bale — no usable third-party libraries:
1. **Read existing open-source code** (Python `rubpy`, JS clients, anything on GitHub) purely as protocol documentation — understand the WebSocket frame format, encryption layer, auth handshake, and message schemas.
2. **Intercept traffic** from the official Rubika app or web client using mitmproxy / browser DevTools to capture the raw WebSocket frames.
3. **Document everything** in `docs/rubika_protocol.md`. Same format as Bale — see `docs/` section below.

Reverse-engineer first, implement second.

### Bot Mode
- Authenticate using bot token.
- Receive and send messages.
- Handle file uploads and downloads.

### User Mode
- Authenticate using phone number + OTP.
- Full dialog list: DMs, groups, channels.
- Message history with pagination.
- Send/receive/edit/delete messages.
- Reply to messages, forward messages.
- File upload and download with progress.
- Real-time updates via the WebSocket connection.
- Profile pictures, display names.
- Create group.
- Create channel (if the API supports it).
- Chat folders (if the API supports it).
- Logout — full session invalidation.

---

## Research Documentation (`docs/`)

Protocol research is hard-won knowledge. **Every discovery goes into `docs/` immediately** — don't rely on memory or code comments. These files are the reference for anyone (including future-you) working on these cores.

### `docs/bale_protocol.md`

Must contain:
- **Base URL(s)** — API host, CDN host, WebSocket endpoint (if any).
- **Authentication** — bot token flow (header format, endpoint), user phone+OTP flow (step-by-step with request/response examples).
- **Endpoints** — every discovered endpoint with method, path, request body schema, response schema, and example payloads (sanitized).
- **File upload** — multipart format, size limits (19.5 MB), CDN URLs for downloads, any token/auth required for file access.
- **Real-time updates** — long-polling vs WebSocket, update format, event types.
- **Rate limits** — if discovered, document thresholds and backoff behavior.
- **Quirks & gotchas** — anything unusual, undocumented, or that differs from Telegram's Bot API despite looking similar. Error codes that aren't obvious. Headers that are silently required. Geo-restrictions (e.g., calling from abroad).
- **Sources** — where you learned each thing (which open-source client, which mitmproxy session, which browser DevTools capture).

### `docs/rubika_protocol.md`

Must contain:
- **WebSocket endpoint** — URL, connection handshake, any required headers or query params.
- **Frame format** — binary vs text, message structure (JSON? Protobuf? Custom binary?), framing/length prefixes.
- **Encryption layer** — if Rubika encrypts WebSocket payloads (it likely does), document the algorithm, key derivation, IV/nonce handling.
- **Authentication** — phone+OTP flow over WebSocket, session token format, bot token flow.
- **Message types / opcodes** — every discovered message type with field schemas and examples.
- **File upload/download** — separate HTTP endpoints? Inline in WebSocket? Document the full flow.
- **Real-time updates** — how new messages/events arrive, subscription mechanism.
- **Quirks & gotchas** — same as Bale.
- **Sources** — same as Bale.

### General Rules for `docs/`

- Update these files **during** research, not after. If you discover something while debugging, write it down before you forget.
- Include raw request/response examples (sanitized — no real tokens/phone numbers).
- Date your entries. Add `<!-- Discovered 2026-04-04 -->` comments next to findings so stale info is identifiable.
- These files are append-friendly — add new sections as you discover new endpoints. Don't reorganize old sections unless they're wrong.
- If a future platform (Discord, Matrix, etc.) has non-obvious protocol details, create a `docs/<platform>_protocol.md` for it too.

---

## Encryption Layer (`utils/encryption.go`)

All encryption is implemented **at the application layer** in Go. The platform servers see only opaque ciphertext. Use Go's `crypto/*` standard library + `golang.org/x/crypto` for all primitives.

### Encryption Indicator in Messages

Every encrypted message is prefixed with `@@` as the first two characters before the Base64-encoded ciphertext. If `@@` is absent → plaintext. If `@@` is present but decryption fails → fall back to displaying raw ciphertext with a broken lock icon and "Could not decrypt message."

### Compression Before Encryption

Before encrypting, compress the plaintext (UTF-8 bytes) with **Zstd** at max compression level (`github.com/klauspost/compress/zstd`). Compress → encrypt on send. Decrypt → decompress on receive. If decompression fails after successful decryption, display raw decrypted bytes with a warning.

### Message Length Splitting (`utils/msgsplit.go`)

Before compression and encryption, check the **raw plaintext word count**. If >4000 words, split into chunks of ≤4000 words at word boundaries. Each chunk is independently compressed, encrypted, `@@`-prefixed, and sent as a separate message. Consecutive `@@` messages from the same sender are displayed as one logical block in the UI with a "— continued —" indicator.

### Encryption Philosophy

**ECDH is the default and primary mode.** No shared password ever crosses the wire. Key agreement happens mathematically — each side contributes a public key, derives the shared secret locally. Nothing to intercept, nothing to brute-force, nothing a server can verify or replay. Non-negotiable for the default path.

**Manual password mode exists as a user-controlled fallback** — opt-in alternative behind the lock button, never default. The password is fed through Argon2id to derive the AES-256-GCM key. Never stored, transmitted, or recoverable.

### Per-Chat Encryption Modes (Lock Button)

| Mode | How It Works | When to Use |
|---|---|---|
| **Off** | Plaintext. No `@@` prefix. | Default for new chats |
| **ECDH (Automatic)** | X25519 key exchange, zero-knowledge. | Recommended — no password needed |
| **Manual Password** | Passphrase → Argon2id → AES-256-GCM key. Both parties enter same password out-of-band. | Fallback for users who prefer a shared secret |

Switching modes mid-conversation is allowed. Mode changes take effect on the next sent message.

### DM Encryption: ECDH + AES-256-GCM (Per-Conversation Key)

- Each user has a long-term **X25519 keypair** in the local encrypted keystore.
- Each DM conversation gets its own **ephemeral X25519 keypair** generated fresh on first open.
- Key exchange happens via a special handshake message (JSON with ephemeral public key, prefixed with a recognizable marker distinct from `@@`). The other party's app detects it, stores the key, and replies with its own.
- Both sides perform **ECDH** (own private × other's public) → **HKDF-SHA256** (context: both user IDs + conversation ID as salt) → **256-bit AES-GCM session key**.
- No secret material is transmitted. Server sees only public keys and encrypted blobs.
- Every message: **AES-256-GCM** with a fresh random 96-bit nonce. Nonce prepended to ciphertext. Full payload (nonce + ciphertext + GCM tag) is Base64-encoded, `@@` prefix prepended.
- Each conversation maintains its own independent key.
- If handshake is incomplete, block sending encrypted messages and show: "Waiting for the other party to open the chat to begin encrypted messaging."

### DM Encryption: Manual Password Mode (Per-Conversation, Opt-In)

- User selects "Manual Password" via lock button → enters a passphrase.
- Passphrase → **Argon2id** (memory: 64 MB, iterations: 3, parallelism: 4, salt: SHA-256 of both user IDs + conversation ID) → 256-bit AES-GCM key.
- Messages encrypted identically to ECDH mode — only key derivation differs.
- Password **never stored on disk, never transmitted, never sent as a message**.
- Wrong password → decryption fails silently → "Could not decrypt" with broken lock icon. No verification oracle, no brute-force surface. By design.

### Group Encryption: AES-256-GCM with Shared Group Key

- Each group has one **AES-256-GCM 256-bit group key**, generated by the group creator when encryption is enabled.
- Distributed to each member via **ECIES wrapping**: fresh ephemeral keypair per recipient, ECDH with recipient's long-term public key, HKDF → wrapping key, AES-GCM encrypt the group key. Sent as a special system message.
- All group messages encrypted with the shared group key + fresh random nonce.
- **Member added**: wrap current group key for new member.
- **Member removed**: generate new group key, rewrap for all remaining members. Forward secrecy on membership change.
- Group key stored only in local keystore, never uploaded in plaintext.

### Unified Encrypted Vault (`utils/vault.go`)

**All persistent app data lives in a single encrypted file**: `uniclient.vault`. This is the single source of truth — one file to back up, one file to export, one file to move between machines.

Contents:
- All encryption keys (long-term keypairs, per-chat session keys, group keys).
- All account credentials and session tokens (Telegram sessions, Bale tokens, Rubika tokens).
- All app settings and per-account config.
- Per-chat encryption mode state.
- Message drafts.

Implementation:
- Backed by **BBolt** (`go.etcd.io/bbolt`) — a pure-Go embedded key/value database. Single file, no external server, ACID transactions, cross-platform.
- The entire BBolt database file is encrypted at rest with **AES-256-GCM** using a master key derived via **Argon2id** (memory: 64 MB, iterations: 3, parallelism: 4) from the user's **vault password**.
- On startup: user enters vault password → derive master key → decrypt and open the vault. On shutdown: flush and re-encrypt.
- Vault password is prompted once at app startup, **never stored on disk**.
- If forgotten, the vault cannot be recovered (make this unmistakably clear in the UI).
- Never uploaded or synced to any platform.

Export / Import:
- **Export**: copies the encrypted vault file as-is. The export is encrypted with the same vault password. User can also export to a new password (re-encrypt with a different key).
- **Import**: prompts for the vault password, decrypts, merges or replaces the current vault.
- Settings screen has "Export Vault" and "Import Vault" buttons with clear instructions.

Buckets (BBolt key namespaces):
- `accounts/` — per-account session data, keyed by `platform:user_id`.
- `keys/` — encryption keys, keyed by conversation ID.
- `config/` — app settings.
- `drafts/` — message drafts, keyed by `platform:chat_id`.

### Encryption UI Controls (Dart side)

**Per-chat lock button** (bottom bar, left of text input):
- Displays: **open lock** (off), **green locked** (ECDH), **orange locked with key** (manual password).
- Opens bottom sheet with three modes:
  - **"Off — Plaintext"**
  - **"ECDH (Automatic)"** — triggers handshake for DMs, ECIES distribution for groups.
  - **"Manual Password"** — password field + "Generate Random" + "Copy" + "Apply".

**Right-click (desktop) / long-press on send button (mobile)**:
- "Send as Plaintext" / "Send Encrypted" — per-message override regardless of chat default.

**Status indicators**:
- Green lock in header + banner: "Messages in this chat are end-to-end encrypted."
- Gray open lock + banner: "Messages in this chat are not encrypted."
- Individual messages: small lock icon (encrypted) or no icon (plaintext).
- Failed decryption: red broken lock + "Could not decrypt this message."

**Encryption Settings Screen** (in app settings):
- Public key fingerprints (SHA-256, displayed in hex groups).
- Regenerate keypair (with warning).
- Change vault password.
- Export/import vault (single file, encrypted — all credentials, keys, sessions, settings).
- Per-chat encryption status list.

---

## UI/UX: Flutter (Dart)

Built using **Flutter** with **Material 3**. The UI must be **platform-adaptive**:

- **Desktop (Linux, Windows, macOS)**: multi-column layout (sidebar + chat list + chat view), hover states, right-click context menus, keyboard shortcuts, drag-and-drop file upload.
- **Web**: same as desktop layout, browser-appropriate behaviors.
- **Mobile (Android)**: single-column layout with navigation stack, bottom navigation, long-press context menus, swipe-to-reply, pull-to-refresh.

### Visual Design: Dark-First Eyegasm

**Dark mode is the default.** The app must look stunning out of the box — deep, rich, OLED-friendly dark surfaces with vibrant accent colors. Light mode exists as an option but dark is the flagship experience.

Theme implementation:
- `ThemeData` with `ColorScheme.fromSeed()` using a deep purple/indigo seed color.
- **Dark mode surfaces**: true black (`#000000`) for OLED backgrounds, `#0D0D0D` for cards, `#1A1A1A` for elevated surfaces. Not Material's default gray-blue dark — actual dark.
- **Accent colors**: vibrant but not harsh. Primary: electric indigo. Secondary: teal. Tertiary: soft amber for warnings. All with proper contrast ratios for accessibility.
- **Typography**: Material 3 type scale, with a custom monospace font for code blocks in Markdown messages. Crisp, high-contrast text on dark surfaces.
- **Elevation**: subtle glow/border effects instead of shadows on dark surfaces (shadows are invisible on black). Use thin `1px` borders with `surface.withOpacity(0.1)` or soft colored glows.
- **Transitions**: smooth `Hero` / `PageRouteBuilder` / `SharedAxisTransition` between screens. 200-300ms durations, Material easing curves.
- **Ripple effects** on all interactive elements.
- **Consistent 8dp spacing grid.**
- Light mode: clean whites and light grays, same accent palette adjusted for light contrast. Toggleable in settings (dark / light / system follow).

### Screens

**1. Splash / Startup Screen**
- App logo and name.
- Vault password prompt (if vault file exists). First launch: "Create Vault" with password + confirm.
- Loading indicator while Go backend decrypts vault, initializes cores, and restores sessions.

**2. Login Screen**
- Material card layout, centered.
- **Platform selector**: `SegmentedButton` for Telegram / Bale / Rubika.
- **Mode toggle**: "User Account" vs "Bot Token".
- For Telegram user mode: phone → OTP → optional 2FA, sequential fields.
- For Telegram bot mode: bot token field.
- For Telegram: **API Client dropdown** + Custom option. **Server selector**: "Main" (default) vs "Test" (yellow warning chip).
- For Bale/Rubika user mode: phone → OTP.
- For Bale/Rubika bot mode: bot token field.
- Input validation with inline errors. Loading spinner during auth. Snackbar on failure.
- **Session saved to disk only after auth fully succeeds.**

**3. Account Switcher**
- Persistent sidebar element (desktop) / drawer (mobile).
- Accounts grouped by platform with avatars, names, platform badges.
- "Add Account" per platform section.
- Long-press / right-click: "Switch to", "Log Out", "Remove Account".

**4. Chat List Screen — Server/Chat/Topic Layout**

The chat list follows a **Discord-inspired hierarchy** adapted for multi-platform messaging:

**Desktop layout (three-column):**
- **Column 1 — Server rail** (narrow, icon-only, left edge): vertical strip of circular platform icons (Telegram, Bale, Rubika) + per-platform account avatars. Each icon = one logged-in account. Clicking switches the active account. Unread dot badges on icons with new messages. "+" button at bottom to add an account.
- **Column 2 — Chat list / Topics** (medium width): shows the selected account's dialogs. For platforms with topics (Telegram supergroups with forum topics), the list is **two-level**: group name as a collapsible header, topic threads nested underneath with indent. Non-topic chats (DMs, regular groups, channels) are flat list items. Each tile: avatar, bold name, last message preview (single line, ellipsis), relative timestamp, unread badge, muted icon, lock icon, pinned indicator.
- **Column 3 — Chat view**: the active conversation.

**Mobile layout (single column with navigation):**
- Bottom nav: Chats, Search, Settings.
- Chats tab shows the active account's dialog list (same tiles as desktop column 2). Account switcher is a horizontal scrollable avatar strip at the top.
- Topic groups show as expandable list items — tap the group to expand/collapse its topics.

Common features:
- Pull-to-refresh (mobile), auto-refresh on update events from Go.
- Search bar at the top to filter chats by name or message content.
- Swipe-to-archive (mobile). Right-click context menu (desktop): Pin, Mute, Archive, Mark as Read, Delete.
- Tap/click to open the Chat/Conversation Screen.

**5. Chat / Conversation Screen**
- Header: back button (mobile), avatar, name, online status, **phone icon** (voice call, shown if `CALLS` capability), **video icon** (video call, shown if `CALLS` capability), encryption icon, three-dot menu. For group chats with active group calls: a "Join Call" chip in the header with participant count.
- Message list: `ListView.builder`, newest at bottom, auto-scroll on new message.
- Message bubbles:
  - Outgoing: right-aligned, `ColorScheme.primary` background, rounded + tail.
  - Incoming: left-aligned, `ColorScheme.surfaceVariant` background, rounded + tail.
  - Group chats: sender avatar + name above first bubble in a sequence.
  - Timestamp inside bubble, bottom-right, `labelSmall` style.
  - **Read receipts / seen status**:
    - Outgoing messages: single gray check (sent to server), double gray check (delivered to recipient), double blue check (read/seen by recipient).
    - In group chats: tap the checks to see a "Seen by" list — shows which members have read the message, with avatars and timestamps.
    - The core calls `MarkAsRead()` when the user scrolls a message into view. `GetReadState()` provides per-user read positions.
    - Platforms without read receipts (`READ_RECEIPTS` capability absent): show only single check (sent) — no delivered/read distinction.
  - Lock icon on encrypted messages. Red broken lock on failed decryption.
  - **Markdown rendering**: all message text is rendered as Markdown using the `flutter_markdown` package (or `markdown_widget`). Supported elements: **bold**, *italic*, ~~strikethrough~~, `inline code`, ```code blocks``` (with syntax highlighting via `flutter_highlight`), [links](url) (tappable, open in browser), > blockquotes, bullet/numbered lists, and headings. Markdown rendering must be fast enough to not drop frames when scrolling through hundreds of messages. Code blocks use a monospace font with a slightly different background surface color. Links are rendered in the accent color and are tappable.
  - **Media: download-to-view, play/display in-app.** All media is consumed inside the app — no launching external viewers for images, video, or audio.
    - **Images**: blurred thumbnail placeholder + file size + download button. Tap downloads with animated `CircularProgressIndicator`. Once downloaded: renders inline in the bubble (`Image.memory`, max 300px wide, aspect-ratio preserved). Tap image → fullscreen `InteractiveViewer` with pinch-to-zoom, swipe to dismiss. Multiple images in the same chat → swipeable gallery in fullscreen.
    - **Videos**: blurred poster frame + file size + download button + duration badge. Download with progress. Once downloaded: plays **inline in the bubble** via `media_kit` `Video` widget — play/pause/seek/volume/fullscreen/speed (1x/1.5x/2x). Tap to expand fullscreen with controls.
    - **GIFs / Animated images**: auto-download (small), auto-play inline muted and looping.
    - **Audio / Music files**: download-to-view. Once downloaded: in-app audio player via `media_kit` — play/pause, seek bar, duration, speed toggle. Shows filename, and artist/title if metadata is available (extracted in Go). **Does not open an external music player.** Background playback continues when navigating away.
    - **Voice messages**: auto-download (small). Waveform visualization + play/pause + duration + speed toggle. Plays in-app via `media_kit`.
    - **Round videos** (Telegram): `ClipOval` frame. Download-to-view with progress. Plays inline in circular mask after download.
    - **Stickers**: displayed inline at 200×200 logical pixels. Animated stickers play via Lottie or equivalent.
  - **File attachments** (non-media: PDFs, ZIPs, etc.): filename, size, download button → `LinearProgressIndicator` with speed + cancel → tap to open with system viewer (only non-media files go to external apps).
  - Base64 images: decoded in Go, rendered identically to regular images.
  - Stickers: 200×200 logical pixels inline.
  - Forwarded: "Forwarded from [name]" header.
  - **Replies**: quoted block inside the bubble showing the original sender's name + first line of the original message (or thumbnail if media). Tap the quoted block to scroll/jump to the original message with a highlight animation. When composing a reply, a preview bar appears above the text input showing the message being replied to, with a close button to cancel the reply.
  - Reactions: emoji chips with counts below bubble.
- Pinned message bar at top, tap to scroll.
- Bottom bar: attachment button, lock button, `TextField` (max 5 lines), send button (long-press for plaintext/encrypted override). Empty field → microphone button. Emoji picker.
- Swipe-to-reply (mobile). Long-press / right-click message actions: Reply, Forward, Copy, Edit, Delete, Pin, React, Select.
- Infinite scroll upward with loading spinner.

**6. File Upload / Download**
- Attachment bottom sheet (mobile) / dropdown (desktop): gallery, document, camera.
- Preview bar above input before sending.
- Upload: `LinearProgressIndicator` in bubble with percentage, speed, cancel.
- Download: same pattern. Tap completed file → open with system viewer.
- **Encrypted file transfer**: "Encrypt file" toggle in attachment menu (default on when chat is encrypted). File → zstd compress → AES-256-GCM encrypt → upload as `.enc`. Original filename/MIME inside ciphertext. Receiver: download → decrypt → decompress → save with original name. Toggle off per-file to send unencrypted.

**6a. Platform File Size Limits & Auto-Splitting**
- **Bale**: 19.5 MB limit (bot and user). Auto-split into `filename.part01`, `filename.part02`, etc. System caption per part. Auto-reassemble on receive. UI shows single file with aggregate progress.
- **Telegram**: 2 GB user mode, 50 MB bot mode. Same split logic if exceeded.
- **Rubika**: same pattern when limit is discovered.
- Splitting handled in Go (`utils/filesplit.go`), transparent to user.

**7. Message Search Screen**
- From chat header menu.
- Full-text search within current chat (Go handles the query, Dart renders results).
- Highlighted excerpts. Tap → scroll to message.

**8. Settings Screen**
- General: theme toggle, language, notifications, download folder.
- Proxy & Network: HTTP/SOCKS5 config, domain→IP override table, DNS fallback toggle.
- Encryption: as described above.
- Accounts: list with logout/remove.
- About: version, licenses.

**9. System Tray & Notifications**
- **Desktop**: system tray via `tray_manager` Flutter plugin (or `system_tray`). Unread badge. Right-click: "Open", "Mute All", "Quit". Close → minimize to tray (configurable).
- **Notifications**: `flutter_local_notifications` on desktop/mobile. Web: browser Notification API via Dart JS interop. Respect per-chat mute. Click → navigate to chat.
- Graceful degradation: if a plugin doesn't support the current platform, skip silently.

**10. Call Screen** (requires `CALLS` or `GROUP_CALLS` capability)

Accessible from:
- Phone icon button in chat header (1:1 calls).
- Group/channel header → "Join Call" (group calls).
- Incoming call notification overlay.

**1:1 Call UI:**
- Full-screen overlay with the contact's avatar (large, centered), name, and call duration timer.
- Buttons: Mute, Speaker, Video toggle, End Call (red, prominent).
- Video call: switches to showing the peer's video feed full-screen with the local camera in a small draggable PiP corner.
- Call state indicators: "Ringing...", "Connecting...", "Call in progress", "Call ended".
- On mobile: proximity sensor dims the screen. System audio routing (speaker/earpiece/bluetooth).

**Group Call UI:**
- Grid layout of participant tiles (avatar + name + speaking indicator).
- Active speaker highlighted with a glowing border.
- Participant list sidebar/bottom sheet showing all members with mute status.
- Controls: Mute/unmute self, toggle video, leave call, raise hand (if supported).
- Screen sharing (if supported by platform and device).

**Platform handling:**
- Telegram: full call support via gotd/td's call API (SRTP-based).
- Bale/Rubika: implement if the platform's API supports calls. If not, the call button is hidden (capability absent).
- On web: WebRTC-based calls if the platform supports it.

**11. Telegram-Specific Screens**
- Contacts, Saved Messages, Archived Chats, Chat Folders, Active Sessions, Channel/Group Info (member list, shared media gallery with tabs: photos, videos, files, links, audio).
- Only shown when active account is Telegram — Dart checks `capabilities` from the core.

---

## Test Infrastructure — Nothing Ships Without Tests

Every module, every core, every widget has tests. This is not optional. If it can break, it has a test that proves it works.

### Philosophy

- **ALL tests are real.** No fake tests. No struct-only checks. No mock-only flows pretending to be tests. If it says "test send message", it actually sends a message to a real server and verifies the response. If it says "test bot auth", it actually authenticates with a real bot token.
- **Every feature must be tested against the real API before it's marked done.** Before checking off any item in CHECKLIST.md, the test must have run successfully against the live service AND the user must confirm the result (e.g., "I see the message in Telegram", "the file arrived").
- **Ask the user for credentials.** When you need a bot token, API key, phone number, or OTP — ask. Don't skip, don't fake, don't mock the core flow. The user will provide what's needed.
- **Ask for user feedback after tests.** After running integration tests, ask the user to verify results on their end (check if the message arrived, check if the file is correct, etc.). Only mark the CHECKLIST item as done after user confirmation.
- **Credentials via environment variables.** Never hardcode tokens. Tests read from:
  - `TG_BOT_TOKEN` — Telegram bot token
  - `TG_API_ID`, `TG_API_HASH` — Telegram app credentials
  - `TG_PHONE` — for user mode
  - `TG_TEST_CHAT_ID` — a chat to send test messages to
  - `BALE_BOT_TOKEN`, `BALE_TEST_CHAT_ID` — Bale credentials
  - `RUBIKA_BOT_TOKEN`, `RUBIKA_TEST_CHAT_ID` — Rubika credentials
- **Interactive when needed.** If a test needs an OTP, prompt the user. If it needs confirmation, ask.
- **No feature is "done" without passing real tests + user feedback.** The workflow is: implement → write real test → ask for credentials → run test → ask user to verify → mark done in CHECKLIST.md.
- **Utils tests are the exception** — encryption, compression, file splitting, etc. test real algorithms on real data but don't need external credentials. These are still real tests (actual encrypt/decrypt, actual compress/decompress), just self-contained.
- **Delete test files after confirmation.** Once all tests in a file pass AND the user confirms, delete the test file. Don't keep old passing tests around. Write new tests only for untested features. Never re-run already-confirmed tests unless something changed.
- **Test each feature exactly once.** Don't repeat the same test in multiple runs. If it passed and user confirmed, it's done. Move on. Check CHECKLIST.md before writing tests to see what's already confirmed.
- **Table-driven tests in Go.** Every edge case is a row in a table, not a separate test function.
- **Golden tests for Dart widgets.** Key screens have golden snapshot tests.

### Go tests (`go test ./...`)

**Core tests (per core — ALL tests hit real APIs with real credentials):**

Every test below runs against the live platform. Ask the user for credentials before running. After each test group, ask the user to verify results on their end.

Auth (real credentials required):
- Authenticate as bot with real token → verify auth succeeds, bot can make API calls.
- Authenticate with intentionally bad token → verify auth fails, nothing is saved to vault.
- Authenticate as user with real phone + OTP (prompted) → verify auth succeeds.
- Logout → verify session is invalidated, subsequent API calls fail with ErrAuth.

Messages (real send/receive — user verifies on their device):
- SendMessage → actually sends to a real chat, user confirms it arrived.
- EditMessage → edit the sent message, user confirms the edit is visible.
- DeleteMessage → delete it, user confirms it's gone.
- ReplyToMessage → send a reply, user confirms the reply thread is correct.
- ForwardMessage → forward to another chat, user confirms it arrived with forward header.
- PinMessage / UnpinMessage → pin a message, user confirms the pin banner appears / disappears.
- ReactToMessage → add a reaction, user confirms the emoji shows up.

Dialogs & Organization (real API):
- GetDialogs → retrieve real dialog list, verify it contains known chats with correct metadata.
- CreateGroup → actually create a group with real members, user confirms it appears in their Telegram.
- CreateChannel → actually create a channel, user confirms it exists.
- CreateTopic → create a topic in a real forum supergroup, user confirms it appears.
- GetFolders / CreateFolder → create a real folder, verify it appears in the folder list.

Files (real upload/download):
- UploadFile → upload a real file to a real chat, user confirms the file arrived and is openable.
- DownloadFile → download a file from a real chat, verify the downloaded file matches the original.
- Upload progress callback → verify progress values are reported during real upload.

Read State (real API):
- MarkAsRead → mark messages as read in a real chat, user confirms read receipts appear.
- GetReadState → retrieve real read positions.

Calls (real when possible):
- StartCall / EndCall — if platform supports it, initiate a real call and end it.
- Bale calls: skip with note about geo-restriction from abroad. Test when access is available.

Real-time (real event stream):
- Send a message from another account → verify OnUpdate fires with the correct message.
- Edit a message from another account → verify edit update arrives.
- Ask user to send a message to the bot → verify the bot receives it via OnUpdate.

**Util tests (every util has its own `_test.go` — no exceptions):**
- `encryption_test.go`: ECDH key exchange handshake (both sides derive same shared secret), AES-256-GCM encrypt/decrypt round-trip, Argon2id manual password round-trip, `@@` prefix detection/absent handling, corrupt ciphertext → graceful error, key exchange with mismatched keys → fail, ECIES group key wrapping/unwrapping, group rekey on member removal → old key can't decrypt new messages.
- `compression_test.go`: Zstd round-trip on empty input, small input, large input (1MB+), already-compressed data, verify max compression level is used.
- `filesplit_test.go`: Split and reassemble at exact boundary (19.5MB, 50MB, 2GB), off-by-one at boundary, tiny file (no split needed), verify part naming (`*.part01`, `*.part02`), verify reassembled file is byte-identical to original.
- `msgsplit_test.go`: 4000-word boundary split, verify chunk count, single-word edge case, Unicode word boundaries (CJK, Arabic, emoji), empty string, exactly 4000 words (no split).
- `proxy_test.go`: SOCKS5 connectivity via mock proxy, HTTP proxy, DNS override hit (custom IP used), DNS override miss → automatic fallback to real DNS, proxy auth (username/password), proxy connection failure → meaningful error.
- `vault_test.go`: Create vault, store/retrieve all data types (credentials, keys, sessions, config, drafts), close and reopen with correct password, wrong password → fail with clear error, export/import round-trip (same password), export to new password → import with new password, concurrent read/write safety, corrupt vault file → graceful error.
- `markdown_test.go`: Parse and validate all Markdown elements (bold, italic, code, links, blockquotes, lists, headings), extract plain-text summary, edge cases (nested formatting, code blocks with backticks inside, malformed/unclosed tags, empty input).
- `storage_test.go`: Path resolution for each `GOOS` value (linux, windows, darwin, android), verify correct directory structure, verify no hardcoded paths.
- `config_test.go`: Load/save JSON settings round-trip, per-account config isolation, merge defaults with user overrides, invalid JSON → graceful error, missing config file → create with defaults.
- `base64img_test.go`: Encode image bytes → Base64 → decode → verify byte-identical, validate PNG/JPEG format detection, corrupt Base64 input ��� graceful error, empty input.

**End-to-end integration tests (real APIs, real encryption, real files):**
- Full encrypted message: compose → split (4000-word) → compress → encrypt → `@@` prefix → send to real chat → receive on other end → decrypt → decompress → reassemble → user confirms the decrypted message matches the original.
- Full encrypted file: file → compress → encrypt → split at platform limit → upload parts to real chat → download parts → reassemble → decrypt → decompress → verify byte-identical. User confirms the file arrived.
- Vault lifecycle: create vault → store real account sessions + keys → close → reopen with password → verify all data intact → export → import at new path → verify.
- Cross-core: send message via Telegram core, verify Bale core state is unaffected.

### Workflow: Implement → Test → Verify → Check Off

```
1. Implement the feature
2. Write the real test (uses live API)
3. Ask user for any credentials needed
4. Run the test
5. Ask user to verify on their device ("did you see the message?")
6. User confirms → mark item done in CHECKLIST.md
7. User reports issue → fix and repeat from step 4
```

**Never mark a CHECKLIST item as done without user confirmation that it works in the real world.**

### Dart tests (`flutter test`)

**Widget tests** (every screen and reusable widget gets a test file):
- `splash_test.dart`: vault password prompt, create vault flow, loading indicator, error display.
- `login_test.dart`: platform selector, mode toggle, phone→OTP→2FA sequential flow, API client dropdown, server selector, input validation errors, loading spinner, auth failure snackbar.
- `chat_list_test.dart`: server rail rendering, chat tiles with all badge variants (unread, muted, pinned, locked), topic hierarchy expand/collapse, search filtering, pull-to-refresh trigger.
- `chat_test.dart`: message list renders with mock data, auto-scroll on new message, infinite scroll upward loads older messages, header shows correct status/icons.
- `message_bubble_test.dart`: all variants — plaintext, markdown-rendered, encrypted (lock icon), failed decrypt (broken lock + warning), media placeholder (download-to-view), voice waveform, round video, forwarded header, reply quoted block, reactions.
- `markdown_body_test.dart`: bold, italic, strikethrough, inline code, code block with syntax highlighting, tappable links, blockquotes, lists, headings, malformed markdown renders gracefully.
- `lock_button_test.dart`: three mode states (off/ECDH/manual), bottom sheet opens, mode switching updates icon color.
- `media_player_test.dart`: video player controls (play/pause/seek/fullscreen/speed), audio player controls, voice message waveform playback.
- `download_indicator_test.dart`: circular and linear progress variants, percentage display, cancel button.
- `account_switcher_test.dart`: account list grouped by platform, add account button, switch/logout/remove actions.
- `call_screen_test.dart`: 1:1 call UI (avatar, timer, mute/speaker/video/end buttons, state indicators), group call UI (participant grid, speaking indicator, controls).
- `settings_test.dart`: theme toggle, proxy config fields, encryption settings, vault export/import buttons.
- `search_test.dart`: search input, result list with highlighted excerpts, tap navigates.
- `system_tray_test.dart`: desktop only — tray icon exists, right-click menu items, close-to-tray behavior. (Skipped on mobile/web.)
- `notifications_test.dart`: notification triggered on mock new message event, respects mute setting, click navigates to chat. (Platform-conditional.)

**Golden tests**: pixel-level snapshot tests for: splash, login, chat list (dark + light theme), chat conversation, message bubble variants, settings. Checked into repo, compared in CI.

**Bridge mock tests**: verify Dart correctly serializes requests and deserializes Go bridge responses for **every** bridge method — auth, send message, get dialogs, upload file, download file, encrypt, create group, start call, mark as read, logout, etc.

**State management tests**: account switching (verify active core changes), encryption mode toggling (verify lock state persists), theme switching (dark/light/system), chat list filtering by platform, folder selection.

### Running Tests

```bash
# All Go tests
go test ./go/...

# All Dart tests
cd dart && flutter test

# Full suite (add to scripts/test.sh)
go test ./go/... && cd dart && flutter test
```

The Nix flake's `devShell` includes all test dependencies. `nix develop` → tests work immediately.

---

## Dependencies

Only what's essential. Every dependency must be pure Go (no CGo) or a Flutter plugin with full platform support.

### Go (`go.mod`)

| Package | Purpose | Why this one |
|---|---|---|
| `github.com/gotd/td` | Telegram MTProto | Pure Go, no CGo, actively maintained, full MTProto coverage |
| `golang.org/x/crypto` | X25519, Argon2id, HKDF, AES-GCM, PBKDF2 | Go team's official crypto extensions |
| `github.com/klauspost/compress` | Zstd compression | Pure Go, fastest Go Zstd implementation |
| `golang.org/x/net` | SOCKS5 proxy dialer | Go team's official net extensions |
| `nhooyr.io/websocket` | WebSocket (Rubika) | Pure Go, works on all platforms including WASM |
| `go.etcd.io/bbolt` | Encrypted vault storage | Pure Go, single-file embedded KV, ACID, battle-tested |
| `github.com/disintegration/imaging` | Thumbnail generation | Pure Go image processing, no CGo |

No CGo dependencies in cores or utils. `go build` with `GOOS`/`GOARCH` alone must produce a working binary for every target.

Bale and Rubika protocols are implemented directly using `net/http` and `nhooyr.io/websocket`. No third-party wrapper libraries for these platforms.

### Dart (`pubspec.yaml`)

| Package | Purpose | Why this one |
|---|---|---|
| `media_kit` + `media_kit_video` | Video/audio playback | Only Flutter media package that covers Linux, Windows, macOS, Android, Web |
| `flutter_markdown` | Markdown message rendering | Official Flutter team package, fast, customizable |
| `flutter_highlight` | Syntax highlighting in code blocks | Works with flutter_markdown's code block builder |
| `system_tray` | Desktop system tray | Supports Linux, Windows, macOS |
| `flutter_local_notifications` | Notifications | Desktop + mobile. Web handled via JS interop |
| `path_provider` | Platform directories | Official Flutter plugin, all platforms |

Everything else is standard Flutter/Material — no additional dependencies unless absolutely required.

---

## Future Platforms (Planned)

The architecture is designed so that adding any of these is a matter of implementing the `Core` interface in a single Go file + one registration line in bridge.go:

| Platform | Protocol | Likely Approach | Notes |
|---|---|---|---|
| **Matrix** | HTTP REST + Server-Sent Events | `mautrix-go` or direct HTTP | Federated, E2EE via Olm/Megolm, rich feature set |
| **Discord** | HTTP REST + WebSocket Gateway | Direct implementation, study `discord.go` libs | Rich API, voice via WebSocket + UDP/WebRTC |
| **GitHub** | HTTP REST + GraphQL | `go-github` or direct HTTP | Notifications, issues, PR comments as "messages" — creative but useful |

Each of these will follow the exact same pattern:
1. Create `go/cores/<platform>.go` implementing `Core`.
2. Add one line to `bridge.go`.
3. Add platform icon/color to Dart's core registry map.
4. Write tests against the same `Core` interface template.
5. No other files change.

---

## Independence Guarantee

Every core and every util must be able to **exist and function in complete isolation**. This is the architectural invariant that makes everything else work:

- **Delete `telegram.go`** → the app starts without Telegram. No crash, no import error. Bale and Rubika work fine.
- **Delete `encryption.go`** → cores that don't need encryption still work. Cores that do will fail at runtime with a clear error, not at compile time with a mysterious import cycle.
- **Delete `bale.go` and `rubika.go`** → you have a Telegram-only client. Zero dead code.
- **Delete all cores** → the app starts, shows "No messaging platforms available. Add one in settings."
- **Take `utils/compression.go` and drop it into a completely different Go project** → it compiles and works. It has no knowledge of uniclient.

This is tested. The CI pipeline includes a build step that compiles each core in isolation (import only `base.go` + `utils/`) to verify no accidental cross-core dependencies.

---

This is the complete specification. Build in the stated order, verify each section with tests, and do not proceed to the next section until the current one passes. Track all progress in `CHECKLIST.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DarkReaperBoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
