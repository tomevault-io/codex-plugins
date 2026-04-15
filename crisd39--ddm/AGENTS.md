# DesformatConcentrator - AI Coding Agent Instructions

## Project Overview
Qt6 C++17 backend server (DDM - Display Data Manager) for a naval radar tactical display system. Processes commands from QML frontend, binary data from hardware concentrator (DCL), and interactive console commands. Uses UDP/LocalIPC for transport.

## Architecture - The Big Picture

### Three Input Sources, One State
```
Frontend (QML) ──JSON via ITransport──┐
Console (stdin) ──CommandDispatcher───┼──► CommandContext (shared state)
DCL Hardware ────Binary via ITransport┘     • std::deque<CursorEntity> cursors
                                             • std::deque<Track> tracks
                                             • double centerX, centerY
```

**Critical insight**: `CommandContext` (in `src/model/commandContext.h`) is the single source of truth. All controllers modify it, encoder reads from it. Thread-safe via Qt event loop - no mutexes needed.

### Message Routing Pattern
`MessageRouter` (`src/controller/messagerouter.cpp`) is the entry point for ALL network messages:
- Starts with `{` → `JsonCommandHandler` (frontend commands)
- High 0xFF density → `DclConcController` (binary hardware data)

**Key architectural decision**: Detection is O(1) for JSON (90% of traffic), so routing adds negligible overhead.

### JSON Command Processing Pipeline
```
JsonCommandHandler::processJsonCommand()
  ├─► Parse JSON → extract "command" + "args"
  ├─► Route via m_commandMap (QMap<QString, std::function>)
  │    └─► Example: "create_line" → LineCommandHandler::createLine()
  ├─► Handler validates (JsonValidator), executes, returns QByteArray
  └─► Send response via ITransport::send()
```

**Pattern to extend**: Add to `JsonCommandHandler::initializeCommandMap()`:
```cpp
m_commandMap["new_command"] = [this](const QJsonObject& args) {
    return m_newHandler->executeCommand(args);
};
```

### Console Command Pattern
Separate from JSON pipeline. Uses Command Pattern with `ICommand` interface:
```cpp
class MyCommand : public ICommand {
    QString getName() const override { return "mycommand"; }
    CommandResult execute(const CommandInvocation& inv, CommandContext& ctx) const override {
        // Direct manipulation of ctx
        ctx.emplaceCursorFront(...);
        ctx.out << "Success\n";
        return {true, ""};
    }
};
// Register in main.cpp:
registry->registerCommand(QSharedPointer<ICommand>(new MyCommand()));
```

## Critical Data Types & Conventions

### qfloat16 for Network Precision
`CursorEntity` uses `QPair<qfloat16, qfloat16>` for coordinates and `qfloat16` for angles/lengths. **Why**: Matches hardware protocol precision (16-bit floats reduce bandwidth). Convert from double→qfloat16 on input, maintain as qfloat16 in entities.

### std::deque for Entity Storage
`CommandContext` uses `std::deque<CursorEntity>` and `std::deque<Track>`, not `QList`. **Why**: Front insertion is O(1) (`emplaceCursorFront`), and no reallocation invalidates iterators during iteration.

### QSharedPointer for Commands
`CommandRegistry` stores `QSharedPointer<ICommand>`, not raw pointers. Qt's shared pointer with signal/slot integration.

### Header-Only Classes
`CommandRegistry` is header-only (in `.h` file). Pattern used for simple template-like classes.

## Transport Layer (ITransport Abstraction)

Two implementations switchable via `Configuration::instance().useLocalIpc`:
- **LocalIPC** (`LocalIpcClient`): QLocalSocket, for same-machine frontend (default: `useLocalIpc = true`)
- **UDP** (`UdpClientAdapter` wrapping `clientSocket`): For network-distributed systems

**Factory pattern**: `makeTransport(TransportKind::Udp/LocalIpc, opts, parent)` in `transportFactory.cpp`.

**Key**: Controllers depend on `ITransport*` interface, never concrete types. Signals: `messageReceived(QByteArray)`, methods: `send(QByteArray)`, `start()`.

## JSON Response Architecture (SOLID Applied)

### Separation of Concerns
- **JsonValidator** (`src/controller/json/validators/`): Validates input fields, returns `ValidationResult` struct
- **LineCommandHandler**: Business logic (create/delete cursors)
- **JsonSerializer**: Entity → JSON (`serializeLine(CursorEntity)`)
- **JsonResponseBuilder**: Constructs standardized responses (`buildSuccessResponse()`, `buildErrorResponse()`)

**Never** mix validation/serialization/response building in handler. Each class has one job.

### Standard Response Format
```json
{
  "status": "success" | "error",
  "command": "create_line",
  "args": { /* command-specific data */ }
}
```

Error responses include `error_code`, `message`, optional `details` object. See `JsonResponseBuilder` methods.

## Binary Protocol (DCL Concentrator)

`ConcDecoder` (`src/model/decoders/concDecoder.cpp`) decodes packed binary from hardware. Emits signals like:
- `newOverlay(QString)` - overlay mode changed
- `newQEK(QString)` - QEK button state
- `newRange(double)` - range value
- `newRollingBall(QPair<int,int>)` - trackball deltas

**Encoding**: `encoderLPD` (`lpdEncoder.h/cpp`) builds outgoing binary messages from `CommandContext` every 40ms (timer in `main.cpp`).

**Pattern**: Decoder emits signals → Connected in `main.cpp` to handlers (`OverlayHandler`, `OBMHandler`, etc.).

## Threading Model

**Main thread**: Event loop, timers, all controllers, network I/O (Qt's async sockets)
**Separate thread**: `StdinReader` (blocking `QTextStream::readLine()` on stdin)

`StdinReader` lives in `ioThread`, emits `lineRead(QString)` signal cross-thread to `CommandDispatcher` on main thread. Thread-safe via Qt's queued connections.

**Rule**: Never block main thread. Use `QTimer` for periodic tasks (encoder send every 40ms).

## Build & Development Workflow

### Build with qmake
```bash
qmake DDM.pro
make
# or on Windows with MinGW:
mingw32-make
```

Output: `build/Desktop_Qt_6_7_3_MinGW_64_bit-Debug/debug/DDM.exe` (or `DDM` on Linux)

### Run & Test
```bash
# Run backend:
./DDM

# In another terminal, send JSON command (UDP mode):
echo '{"command":"create_line","args":{"azimut":45.0,"length":100.0}}' | nc -u localhost 6340

# Console commands (in DDM stdin):
> addCursor 45.0 100.0
> listcursors
> deletecursors 2
> exit
```

### Configuration
`Configuration::instance()` singleton in `src/model/utils/configuration.h`:
- `useLocalIpc = true` (default): LocalIPC transport, matches `siag_ddm` name
- Ports: `PORT_MASTER = 6340` (UDP), `DA_LPD = 0x00` (device addresses)
- Change in code, no config file currently

**Important**: All constants (ports, addresses, device codes, magic numbers) should be declared in `configuration.h` using `#define` macros. Never hardcode values in implementation files.

### Adding New Files
1. Create `.h` and `.cpp` in appropriate `src/` subdirectory
2. Add to `DDM.pro` under `HEADERS +=` and `SOURCES +=`
3. Add directory to `INCLUDEPATH +=` if new folder
4. Run `qmake` again before `make`

### Documentation Requirements
**CRITICAL**: Keep `docs/README_BACKEND_ARCHITECTURE.md` updated at all times:
- When adding new commands (JSON or console), document them in the API section
- When adding new handlers, update architecture diagrams
- When changing message routing, update flow diagrams
- When modifying data structures, update class descriptions
- Include code examples for new patterns

## Common Patterns & Idioms

### Emplacing Entities (Prefer emplace over add)
```cpp
// PREFER (no extra copy):
ctx.emplaceCursorFront(coords, azimut, length, type, cursorId, true);

// AVOID (creates temp then copies):
CursorEntity temp(...);
ctx.addCursorFront(temp);
```

### Validation Before Execution
**Every** handler must validate inputs before modifying `CommandContext`:
```cpp
ValidationResult validation = JsonValidator::validateNumericField(
    args, "azimut", 0.0, 359.9, true  // required=true
);
if (!validation.isValid) {
    return JsonResponseBuilder::buildValidationErrorResponse(
        "create_line", validation.fieldName, 
        validation.fieldValue, validation.expectedRange
    );
}
```

### Signal/Slot Naming Convention
- Signals: `newOverlay`, `lineCreated`, `quitRequested` (noun/past tense)
- Slots: `onMessageReceived`, `onDatagram`, `onLine` (on-prefix for event handlers)

### Forward Declarations
Extensively used to break circular dependencies. See `messagerouter.h`:
```cpp
class DclConcController;  // Forward declaration
class JsonCommandHandler; // Forward declaration
```
Include full header only in `.cpp` file.

### ANSI Color Output (Console)
`#define ANSI_YELLOW "\x1b[33m"` in `configuration.h`, enabled on Windows via `enableAnsiColorsOnWindows()` in `main.cpp`. Use `ctx.out << ANSI_YELLOW << "text" << ANSI_RESET`.

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/main.cpp` | Orchestrates initialization: creates all controllers, connects signals, starts threads |
| `src/model/commandContext.h` | Shared application state (header-only with inline methods) |
| `src/controller/messagerouter.h/cpp` | Routes incoming network messages to appropriate handler |
| `src/controller/json/jsoncommandhandler.h/cpp` | JSON command dispatcher, maintains `m_commandMap` |
| `src/controller/handlers/linecommandhandler.h/cpp` | CRUD operations for lines/cursors |
| `src/controller/json/jsonresponsebuilder.h/cpp` | Standardized JSON response factory |
| `src/controller/json/validators/jsonvalidator.h/cpp` | Input validation with structured error reporting |
| `src/model/decoders/concDecoder.h/cpp` | Binary protocol decoder for DCL concentrator |
| `src/model/decoders/lpdEncoder.h/cpp` | Binary protocol encoder (outgoing to hardware) |
| `src/model/network/transportFactory.h/cpp` | Factory for ITransport implementations |
| `docs/README_BACKEND_ARCHITECTURE.md` | Comprehensive architecture documentation with diagrams |
| `docs/README_MESSAGE_ROUTER.md` | Frontend counterpart architecture |

## What NOT To Do

❌ Don't bypass `MessageRouter` - it's the single entry point for network data
❌ Don't modify `CommandContext` from multiple threads - use signals to main thread
❌ Don't hardcode IPs/ports/magic numbers - **always declare constants in `configuration.h`**
❌ Don't mix validation and business logic in handlers - use `JsonValidator`
❌ Don't create JSON manually with `QString::arg()` - use `JsonResponseBuilder`
❌ Don't forget to register new commands in `JsonCommandHandler::initializeCommandMap()` or `CommandRegistry`
❌ Don't use `QList` for entities - use `std::deque` (front insertion performance)
❌ **Don't skip updating `docs/README_BACKEND_ARCHITECTURE.md`** - it must reflect current architecture

## Frontend Integration Notes

Frontend (separate QML project) sends commands like `{"command":"create_line","args":{...}}` via same `ITransport`. Response format documented in `README_BACKEND_ARCHITECTURE.md` § "API de Comandos JSON".

Frontend has mirrored classes: `JsonCommandBuilder` (constructs requests), `JsonResponseParser` (parses our responses). Both sides communicate via standardized JSON protocol.

## Debugging Tips

- Enable verbose logging: Add `qDebug()` statements, output goes to console
- Watch message routing: Add debug in `MessageRouter::onMessageReceived()`
- Inspect context state: Print `ctx.cursors.size()`, iterate and log entities
- Test JSON commands: Use `nc` (netcat) or `curl` to send raw JSON to UDP port
- Console commands are immediate: Type `listcursors` to see current state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CrisD39) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
