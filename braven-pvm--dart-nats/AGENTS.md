# nats_dart Project Constitution

**Last Updated**: February 23, 2026  
**Applies To**: All development in `nats_dart` package

---

## 1. Core Principles (Non-Negotiable)

### Pure Dart
- **Every line of code must be valid Dart.** No platform-specific logic except in transport implementations.
- Platform differences are handled **exclusively** via conditional imports in `transport_factory.dart` 
- All protocol logic, parsing, JetStream, and KV APIs are **100% identical** across all platforms
- Test all protocol code without native dependencies (tests use stubs, not platform code)

### Test-Driven Design (TDD)
- **Tests exist BEFORE implementation**. Add test skeletons (`test.skip()`) before the implementation
- Every public method has at least one test
- Parser, encoder, NUID generator, and protocol logic are unit-tested in isolation
- Integration tests exercise real server connections (Docker NATS)
- Minimum coverage target: 80% for core protocol; 60% for platform-specific code

### SOLID Principles
- **Single Responsibility**: Each class has one reason to change
- **Open/Closed**: Extend via subclassing/interfaces, not modification
- **Liskov Substitution**: Implementations are true to their contracts
- **Interface Segregation**: No client forced to depend on methods it doesn't use
- **Dependency Inversion**: Depend on abstractions, not concretions

---

## 2. Code Organization & Patterns

### Directory Layout
```
lib/src/
├── transport/           # I/O abstraction (TCP, WebSocket)
│   ├── transport.dart           # Abstract interface
│   ├── transport_factory.dart   # Conditional import selector
│   ├── tcp_transport.dart       # dart:io implementation
│   └── websocket_transport.dart # web_socket_channel implementation
├── protocol/            # Wire protocol (codec, parsing)
│   ├── parser.dart      # Stateful MSG/HMSG/INFO parser
│   ├── encoder.dart     # HPUB/PUB/SUB/CONNECT encoder
│   ├── message.dart     # Protocol message model
│   └── nuid.dart        # Unique ID generator
├── client/              # High-level NATS API
│   ├── connection.dart  # NatsConnection (pub/sub, request/reply)
│   ├── subscription.dart
│   └── options.dart     # ConnectOptions & enums
├── jetstream/           # JetStream-specific APIs
│   ├── jetstream.dart
│   ├── stream_manager.dart
│   ├── consumer_manager.dart
│   ├── pull_consumer.dart
│   └── js_msg.dart
└── kv/                  # KeyValue store layer
    ├── kv.dart
    └── kv_entry.dart
```

### Abstraction Strategy
- **Transport** is the only location where platform logic lives
- All other code uses the `Transport` interface—no imports of `dart:io` elsewhere
- Conditional imports (`if (dart.library.io)` / `if (dart.library.html)`) **only** in `transport_factory.dart`
- Protocol parser/encoder work identically whether reading from TCP or WebSocket

### SOLID Application

#### S — Single Responsibility
```dart
// ✅ Good: Parser only parses
class NatsParser {
  Stream<NatsMessage> get messages;
  void addBytes(Uint8List data);
}

// ❌ Bad: Parser does auth AND parsing
class NatsParser {
  Future<void> authenticate() { /* ... */ }
  void parse() { /* ... */ }
}
```

#### O — Open/Closed
```dart
// ✅ Good: Extend Transport, don't modify it
abstract class Transport { /* interface */ }
class TcpTransport implements Transport { /* implementation */ }
class WebSocketTransport implements Transport { /* implementation */ }

// ❌ Bad: Add platform code directly to Transport
class Transport {
  if (kIsWeb) { /* WebSocket code */ }
  if (!kIsWeb) { /* TCP code */ }
}
```

#### L — Liskov Substitution
```dart
// ✅ Good: All implementations can replace Transport
NatsConnection(Transport transport) {
  // Works with TcpTransport, WebSocketTransport, or mock
}

// ❌ Bad: Implementation violates interface contract
class TcpTransport implements Transport {
  @override
  Future<void> write(Uint8List data) {
    throw UnimplementedError(); // Violates contract!
  }
}
```

#### I — Interface Segregation
```dart
// ✅ Good: Subscription interface only has what's needed
abstract class Subscription {
  Stream<NatsMessage> get messages;
}

// ❌ Bad: Forces implementation of unrelated methods
abstract class Subscription {
  Future<void> reconnect(); // Not all subscriptions need this
}
```

#### D — Dependency Inversion
```dart
// ✅ Good: Connection depends on Transport abstraction
class NatsConnection {
  final Transport _transport;
  NatsConnection(this._transport);
}

// ❌ Bad: Connection depends on concrete TCP implementation
class NatsConnection {
  final TcpTransport _transport = TcpTransport();
}
```

---

## 3. Test Requirements

### Unit Tests (before implementation)
```dart
// test/unit/parser_test.dart
void main() {
  test('parse MSG without reply-to', () {
    final parser = NatsParser();
    parser.addBytes(Uint8List.fromList('MSG subject 1 5\r\nHello\r\n'.codeUnits));
    final msg = parser.messages.first;
    
    expect(msg.subject, 'subject');
    expect(msg.payload, [72, 101, 108, 108, 111]); // 'Hello'
  });

  test('parse HMSG with headers and status code', () { /* ... */ });
  test('handle partial frames', () { /* ... */ });
  test('throw on invalid format', () { /* ... */ });
}
```

### Integration Tests (run against Docker NATS)
```dart
// test/integration/connection_test.dart
void main() {
  late NatsConnection nc;
  
  setUpAll(() async {
    nc = await NatsConnection.connect('nats://localhost:4222');
  });
  
  tearDownAll(() => nc.close());
  
  test('pub/sub round-trip', () async { /* ... */ });
  test('reconnect on disconnect', () async { /* ... */ });
  test('jetstream publish with dedup', () async { /* ... */ });
}
```

### Test Checklist
- [ ] Parser tests (MSG, HMSG, INFO, PING, +OK, -ERR, status codes)
- [ ] Encoder tests (PUB, HPUB, SUB, UNSUB, CONNECT)
- [ ] NUID uniqueness and format validation
- [ ] Transport interface contracts (no platform-specific tests)
- [ ] Connection lifecycle (connect, disconnect, reconnect)
- [ ] Pub/sub and request/reply
- [ ] Queue groups and wildcards
- [ ] Authentication modes (token, user/pass, JWT, NKey)
- [ ] JetStream (streams, consumers, pull fetch, ordered consumers)
- [ ] KeyValue (put, get, delete, watch)
- [ ] Error handling and edge cases

---

## 4. Implementation Guidelines

### Protocol Parser & Encoder (lib/src/protocol/)
- Parser must be **stateful** (handle multi-frame messages)
- Encoder output must be **byte-perfect** (HPUB counting exact)
- Both must be **platform-agnostic** (no IO dependencies)
- Reference [NATS Protocol Spec](https://docs.nats.io/reference/reference-protocols/nats-protocol)

### Transport Implementations (lib/src/transport/)
- **Only place** where `dart:io` and `web_socket_channel` live
- Implement `Transport` interface (connect, write, listen, close)
- Both TCP and WebSocket have identical error handling contracts
- No protocol parsing in transports—parser is independent

### Connection & Auth (lib/src/client/)
- Reconnection logic with subscription replay
- Auth: decouple auth method from connection logic
- NUID injected (testable)
- Parser injected (testable—can mock)

### JetStream APIs (lib/src/jetstream/, lib/src/kv/)
- Built on pub/sub primitives (composition, not inheritance)
- PullConsumer: fetch batches, consume streams
- OrderedConsumer: auto-recreate on sequence gaps
- KeyValue: watch pushes entries, put/get idempotent

---

## 5. Code Style

### Naming
- Classes: `PascalCase` (e.g., `NatsConnection`)
- Methods/variables: `camelCase`
- Constants: `camelCase` (Dart convention, not `SCREAMING_SNAKE_CASE`)
- Private: prefix with `_`

### Formatting
```bash
dart format lib test
dart analyze lib test  # Check linter rules
```

### Import Order
```dart
import 'dart:async';
import 'dart:convert';
import 'dart:typed_data';

import 'package:nats_dart/src/...';

import '../other_module/file.dart';
```

### Error Handling
- Don't silently fail
- Throw with descriptive messages: `throw StateError('...')`
- Don't catch and ignore—let caller decide
- Log errors, don't swallow them

### Documentation
- Public APIs have doc comments (`///`)
- Complex logic has inline comments explaining **why**, not **what**
- Example in doc comments for non-obvious methods

```dart
/// Publish a message to a subject.
///
/// For JetStream, use HPUB with [headers] to include context.
/// Returns a [Future] that completes when server acknowledges.
Future<void> publish(String subject, Uint8List data, {
  String? replyTo,
  Map<String, String>? headers,
}) async { /* ... */ }
```

---

## 6. CI/CD & Quality

### Before Pushing
```bash
dart pub get
dart format lib test example
dart analyze lib test example    # Must pass
dart test test/unit              # All unit tests pass (no skipped)
dart test test/integration       # Integration tests (if server available)
```

### Test Coverage
```bash
dart test --coverage=coverage
```

### Commit Messages
- Format: `<type>(<scope>): <description>`
- Types: `feat`, `fix`, `refactor`, `test`, `docs`
- Example: `feat(jetstream): implement pull consumer fetch()`

---

## 7. Architecture Decision Records (ADRs)

For significant decisions (new module, redesign), document in `docs/adr/`:

```markdown
# ADR-001: Conditional Transport Imports

## Status
Accepted

## Context
Dart has platform differences (TCP vs WebSocket for browsers)

## Decision
Use `if (dart.library.io)` conditional imports in `transport_factory.dart`

## Consequences
- Protocol code is 100% identical across platforms
- Only transport layer is platform-specific
- Smaller build size (platform code excluded at compile time)
```

---

## 8. Dependency Philosophy

**Minimal and vetted:**
- `web_socket_channel`: Only WebSocket dependency (required for Flutter Web)
- Dev: `lints`, `mockito`, `test`
- No JSON serialization libraries (use `dart:convert`)
- No async/concurrency libraries beyond `dart:async`

**Rationale:** NATS is a protocol layer. Keep it lightweight and portable.

---

## 9. Quick Checklist for All PRs

- [ ] Code is Pure Dart (no platform logic outside transport/)
- [ ] Tests exist BEFORE implementation
- [ ] All tests pass (`dart test`)
- [ ] No linter warnings (`dart analyze`)
- [ ] Code is formatted (`dart format`)
- [ ] SOLID principles respected (S, O, L, I, D)
- [ ] Public APIs documented
- [ ] No breaking changes without discussion
- [ ] Commit message follows convention
- [ ] Git history is clean (squash if needed)

---

## 10. References

- [Architecture Reference](../docs/nats_dart_architecture_reference.md)
- [NATS Protocol Spec](https://docs.nats.io/reference/reference-protocols/nats-protocol)
- [Dart Style Guide](https://dart.dev/guides/language/effective-dart)
- [Dart Testing Guide](https://dart.dev/guides/testing)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)

---

**Questions?** Refer to the architecture reference or open a discussion.  
**Something missing?** Update this document and commit it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/braven-pvm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
