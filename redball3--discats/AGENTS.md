# GitHub Copilot Context — discats

## What This Project Is

**discats** is a Scala 3 library for building Discord bots. It wraps the Discord Gateway (WebSocket) and REST API behind functional, effect-polymorphic algebras using the Typelevel ecosystem (Cats Effect, FS2, http4s, Circe).

## Tech Stack

| Layer        | Library                    |
|--------------|----------------------------|
| Effects      | Cats Effect 3 (`IO`, `Resource`, `Async`) |
| Streams      | FS2                        |
| HTTP         | http4s (Java 11 client)    |
| JSON         | Circe with generic derivation |
| Testing      | Weaver (`weaver-cats`)     |
| Scala        | 3.3.x (LTS)                |

## Key Conventions

- **Effect polymorphism**: all traits are parameterised by `F[_]: Async`. Never hardcode `IO` in library code.
- **Opaque types**: `Snowflake` (wraps `Long`) and `Intent` (wraps `Int`) are opaque to prevent accidental misuse. Keep new ID/bitfield types opaque too.
- **Fluent builders**: `ApplicationCommand` and `SlashCommandRouter` use a builder pattern — each method returns a new (copied) instance.
- **Resource lifecycle**: use `cats.effect.Resource` for anything that needs teardown (HTTP client, WebSocket connection).
- **ADT for events**: `GatewayEvent` is a sealed `enum`. Add new events as new cases — don't use strings.
- **Snake case JSON**: all Circe derivation uses `Configuration.default.withSnakeCaseMemberNames` to match Discord's API.
- **Rate limiting**: every REST call must go through `RateLimiter`, keyed by the route bucket string.

## Project Layout

```
src/main/scala/discats/
├── models/          # Pure data types (Snowflake, User, Message, Guild, Embed, ...)
├── gateway/         # WebSocket protocol (GatewayClient, GatewayEvent, Opcode, ...)
├── rest/            # HTTP client (RestClient, RateLimiter)
├── DiscordClient.scala       # Combined facade (REST + Gateway)
├── DiscordConfig.scala       # Bot token, intents, API version
└── SlashCommandRouter.scala  # Slash command dispatch

src/test/scala/discats/
├── LibrarySuite.scala         # Snowflake, payload, event, model codec tests
└── SlashCommandSuite.scala    # Interaction codec and router routing tests
```

## Common Patterns

### Consuming the event stream
```scala
client.events.evalMap {
  case GatewayEvent.MessageCreate(msg) => handleMessage(msg)
  case GatewayEvent.InteractionCreate(i) => router.dispatch(i, client)
  case _ => F.unit
}.compile.drain
```

### Adding a new REST endpoint
1. Add the abstract method to `RestClient[F[_]]` trait.
2. Implement it in the concrete class inside `RestClient.scala`, routing through `rateLimiter`.
3. Update `DiscordClient` if it needs to expose the method at the top level.

### Adding a new Gateway event
1. Add a new case to `GatewayEvent` enum in `GatewayEvent.scala`.
2. Add a matching decode case in `GatewayEvent.fromPayload` (or equivalent decoder).
3. Add a test case in `LibrarySuite.scala` / `GatewayEventSuite`.

### Adding a new model
1. Create a `final case class` in `models/`.
2. Derive `Encoder` / `Decoder` via Circe using `Configuration.default.withSnakeCaseMemberNames`.
3. Add a round-trip codec test in `LibrarySuite.scala`.

## Testing

```
sbt test
```

Tests use [Weaver](https://disneystreaming.github.io/weaver-test/). Key conventions:
- Test suites are `object`s (not `class`es) extending `weaver.SimpleIOSuite`.
- Pure tests use `pureTest("...")` and return `Expectations`.
- Effect tests use `test("...")` and return `IO[Expectations]`.
- Use `expect(condition)` for single assertions; combine with `and`: `expect(a) and expect(b)`.
- No real network calls in tests. Use stub/mock implementations (see `StubRestClient` in `SlashCommandSuite`) for anything requiring `RestClient`.

```scala
object MyModelSuite extends SimpleIOSuite:

  pureTest("encodes correctly") {
    val result = myModel.asJson
    expect(result == expectedJson)
  }

  test("effect-based test") {
    for
      ref    <- IO.ref(0)
      _      <- someOperation(ref)
      result <- ref.get
    yield expect(result == 1)
  }
```

## Things to Avoid

- Don't hardcode `IO` in the library — use `F[_]: Async`.
- Don't skip the rate limiter in REST calls.
- Don't add mutable state outside of `Ref` / `Deferred`.
- Don't use `null`; use `Option`.
- Don't extend `class` for test suites — use `object`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redball3)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/redball3)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
