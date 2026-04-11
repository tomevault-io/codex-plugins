# Copilot Instructions for bitchat-rs

## Project Overview
Rust implementation of the Bitchat protocol - a decentralized P2P mesh messaging system using Bluetooth Low Energy (BLE). This is a reference implementation compatible with the Swift version at https://github.com/permissionlesstech/bitchat.

**Primary Goals:**
- BLE-based mesh network with multi-hop message relay (max 7 hops TTL)
- Terminal UI (curses/TUI) with IRC-style commands (`/join`, `/msg`, `/quit`, `/nick`)
- Service discovery: scan for existing Bitchat services, create/advertise if none found
- Cross-compatible with Swift reference implementation (iOS/macOS app)

## Architecture

### Core Components
1. **BLE Service Layer** (`simplersble` crate)
   - Service UUID: `F47B5E2D-4A9E-4C5A-9B3F-8E1D2C3A4B5C` (mainnet) / `...4B5A` (testnet)
   - Characteristic UUID: `A1B2C3D4-E5F6-4A5B-8C9D-0E1F2A3B4C5D`
   - Both Central (scanner/client) and Peripheral (advertiser/server) roles
   - Dual role: act as central to connect to peers, peripheral to advertise service

2. **Binary Protocol** (see WHITEPAPER.md in reference)
   - Compact binary packet format optimized for BLE MTU constraints
   - Fixed header: version(1) + type(1) + TTL(1) + timestamp(8) + flags(1) + payload_length(2/4)
   - Variable fields: sender_id(8) + optional recipient_id(8) + payload + optional signature(64)
   - Message types: announce(0x01), message(0x02), leave(0x03), requestSync(0x04), noiseHandshake(0x10), noiseEncrypted(0x11), fragment(0x20), fileTransfer(0x22)

3. **Encryption** (Noise Protocol Framework)
   - Noise_XX_25519_ChaChaPoly_BLAKE2b pattern for peer sessions
   - X25519 ECDH key exchange for ephemeral session keys
   - Ed25519 for packet signatures (public messages)
   - Per-session encryption for direct messages

4. **TUI Interface** (ratatui or cursive)
   - 3-panel layout: sidebar (channels/users), main (messages), input box
   - IRC-style commands: `/help`, `/name <nick>`, `/dm <user>`, `/j #channel [pass]`, `/leave`, `/online`, `/block`, `/clear`, `/exit`
   - Visual indicators for unread messages, connection status

### Message Flow
1. **Outbound**: Encode → Sign (if public) → Encrypt (if DM) → Fragment (if >MTU) → Transmit via BLE
2. **Inbound**: Receive → Reassemble (if fragmented) → Decrypt (if encrypted) → Verify signature → Relay (TTL-1)
3. **Relay Logic**: K-of-N fanout, exclude ingress link (last-hop suppression), jitter scheduling

## Build & Development Workflow

### Essential Commands
```bash
# Build
cargo build --release

# Run with BLE service discovery
cargo run -- --scan-timeout 30 --advertise

# Run in client-only mode (no advertising)
cargo run -- --no-advertise

# Test BLE components
cargo test ble::

# Format before commits
cargo fmt && cargo clippy -- -D warnings

# Debug with verbose logging
RUST_LOG=bitchat_rs=trace,simplersble=debug cargo run
```

### Key Dependencies
- `simplersble` - BLE client/service (cross-platform wrapper)
- `ratatui` or `cursive` - Terminal UI framework
- `tokio` - Async runtime (for BLE event handling)
- `serde` + `bincode` - Binary protocol serialization
- `snow` - Noise Protocol Framework implementation
- `ed25519-dalek` - Signature verification
- `x25519-dalek` - ECDH key exchange
- `chacha20poly1305` - AEAD encryption

## Protocol Implementation Notes

### BLE Service Behavior
- **Discovery**: Scan for service UUID, connect if found within `--scan-timeout` seconds
- **Advertising**: If no service found AND `--advertise` flag set, become peripheral
- **Connection Management**: Maintain subscriptions to characteristic notifications, handle reconnects
- **MTU Handling**: Default 512 bytes, auto-fragment messages larger than `peripheral.maximumWriteValueLength - 42` (header overhead)

### Binary Packet Format (BitchatPacket)
```rust
struct BitchatPacket {
    version: u8,           // Always 1 (or 2 for extended payloads)
    msg_type: u8,          // MessageType enum
    ttl: u8,               // Hops remaining (typically 7)
    timestamp: u64,        // Milliseconds since epoch
    flags: u8,             // hasRecipient | hasSignature | hasRoute
    payload_length: u16,   // Or u32 for v2
    sender_id: [u8; 8],    // Truncated peer ID
    recipient_id: Option<[u8; 8]>,
    payload: Vec<u8>,
    signature: Option<[u8; 64]>,
    route: Option<Vec<[u8; 8]>>,
}
```

### Fragmentation Protocol
- Header: fragment_id(8) + index(2) + total(2) + original_type(1) + data
- Use fragment_id for deduplication (sender_id + timestamp hash)
- Reassemble in order, timeout incomplete fragments after 30s

### Announce Packet TLV
```rust
struct AnnouncementPacket {
    nickname: String,           // TLV 0x01
    noise_public_key: [u8; 32], // TLV 0x02 (X25519 static key)
    signing_public_key: [u8; 32], // TLV 0x03 (Ed25519)
    direct_neighbors: Vec<[u8; 8]>, // TLV 0x04 (optional)
}
```

### Noise Session Handshake
1. Initiator sends `noiseHandshake(0x10)` with ephemeral public key
2. Responder replies with own ephemeral + encrypted payload
3. Both derive shared secret, store session in `HashMap<PeerID, NoiseState>`
4. All subsequent DMs use `noiseEncrypted(0x11)` packets

## Code Conventions

### Error Handling
- Use `thiserror` for domain errors: `ProtocolError`, `BleError`, `CryptoError`
- Never panic on network/BLE errors - log and continue
- Use `anyhow::Result` for top-level functions, typed errors for libraries

### Async Patterns
- `tokio` runtime for BLE event loop and message queue
- Use `mpsc` channels for cross-thread communication (BLE callbacks → TUI updates)
- Spawn tasks for: BLE scanning, message relay, fragment timeout cleanup

### State Management
- `Arc<RwLock<AppState>>` for shared state (peers, messages, sessions)
- `PeerID` as key type (newtype wrapper around 8-byte hex string)
- Separate `HashMap` for: peers, noise_sessions, pending_fragments, message_dedupe

### Module Organization
```
src/
├── main.rs              # CLI args, TUI launch
├── ble/
│   ├── mod.rs           # BLE service wrapper
│   ├── service.rs       # Central + Peripheral roles
│   └── discovery.rs     # Scan/advertise logic
├── protocol/
│   ├── mod.rs
│   ├── packet.rs        # BitchatPacket encode/decode
│   ├── fragment.rs      # Fragmentation logic
│   └── tlv.rs           # TLV parsing (announce, file)
├── crypto/
│   ├── noise.rs         # Noise session management
│   └── signing.rs       # Ed25519 sign/verify
├── tui/
│   ├── mod.rs
│   ├── app.rs           # Main TUI app state
│   ├── input.rs         # Command parser
│   └── render.rs        # Layout rendering
└── types.rs             # PeerID, Message, Channel
```

## Testing Strategy

### Unit Tests
- Protocol encoding/decoding: test against Swift reference implementation's test vectors
- Fragmentation: verify reassembly with out-of-order delivery
- Noise handshake: test session establishment and encryption round-trip

### Integration Tests
- Mock BLE service using `simplersble` test helpers
- Simulate 3-node mesh: verify message relay and TTL decrement
- Test announce broadcast and peer discovery

### Compatibility Tests
- Cross-test with Swift iOS app: send/receive messages between Rust<->Swift
- Verify binary packet format matches byte-for-byte

## Debugging Tips
```bash
# Enable BLE tracing
RUST_LOG=simplersble=trace cargo run

# Dump raw packets
RUST_LOG=bitchat_rs::protocol=trace cargo run 2>&1 | grep "📦"

# Test without BLE (mock transport)
cargo test --features mock-ble

# Check packet encoding
cargo run --bin packet-inspector < message.bin
```

## Critical Reference Points

### Swift Reference Implementation
- **BLE Service**: `bitchat/Services/BLE/BLEService.swift` (lines 23-3662)
- **Packet Format**: `bitchat/Models/BitchatPacket.swift` + `bitchat/Protocols/BinaryProtocol.swift`
- **Noise Encryption**: `bitchat/Noise/NoiseProtocol.swift`
- **Message Types**: `bitchat/Protocols/BitchatProtocol.swift` (lines 77-104)

### TUI Reference (not protocol-compatible)
- **bitchat-tui**: https://github.com/vaibhav-mattoo/bitchat-tui (UI patterns only, not protocol)

## Common Pitfalls
1. **MTU Calculation**: Always subtract 42 bytes (L2CAP + ATT + packet header) from max write length
2. **TTL Decrement**: Decrement before relay, NOT on receive (prevents double-hop)
3. **Signature Verification**: Public messages MUST be signed with Ed25519; verify before relay
4. **Ingress Suppression**: Never relay back to the link a packet arrived on (use `ingressByMessageID` map)
5. **Fragment Timeouts**: Clean up incomplete fragments after 30s to prevent memory bloat
6. **Announce Throttling**: Min 5s between announces to avoid flooding mesh

---
*Last updated: November 17, 2025*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wise0wl)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/wise0wl)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
