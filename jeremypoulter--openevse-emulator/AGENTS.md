# GitHub Copilot Instructions for OpenEVSE Emulator

## Project Overview

This is a Python-based emulator for the OpenEVSE electric vehicle charging station firmware.
It simulates both the EVSE (charging station) and a connected EV to help developers test
WiFi firmware without physical hardware.

## Architecture

- **RAPI Protocol Handler**: Implements OpenEVSE's Remote API for serial communication
- **EVSE State Machine**: SAE J1772 states (A, B, C, D) and error conditions
- **EV Simulator**: Configurable battery, connection, and charging behavior
- **Web API**: Flask-based REST API and WebSocket interface
- **Web UI**: Browser-based control panel

## Code Style and Conventions

### Python Style

- Follow PEP 8 style guide
- Use type hints for function parameters and return values
- Maximum line length: 100 characters
- Use docstrings for all classes and public methods (Google style)
- Prefer f-strings for string formatting

### Prerequisites for Linting

Install development dependencies:

```bash
pip install -r requirements.txt
npm install -g markdownlint-cli
```

### Quality Assurance

**CRITICAL**: Before committing any code changes, ALWAYS run:

1. **Linting**:
   - `flake8 src/ tests/` - Must pass with zero errors
   - `black --check src/ tests/` - All files must be formatted
   - If black reports formatting issues, run `black src/ tests/` to auto-format
   - `markdownlint '**/*.md' .github/copilot-instructions.md` - Markdown files must follow style guide

2. **Testing**:
   - `pytest tests/ -v` - All 126+ tests must pass
   - Add tests for any new functionality
   - Update tests if changing existing behavior

3. **Validation Workflow**:
   - Make code changes
   - Run `black src/ tests/` to format code
   - Run `flake8 src/ tests/` to check for errors
   - Run `markdownlint '**/*.md' .github/copilot-instructions.md` to check markdown files
   - Run `pytest tests/ -v` to verify all tests pass
   - Commit only after all checks pass

**Never commit code that fails linting or breaks tests.**

### Naming Conventions

- Classes: PascalCase (e.g., `EVSEStateMachine`)
- Functions/methods: snake_case (e.g., `handle_command`)
- Constants: UPPER_SNAKE_CASE (e.g., `STATE_READY`)
- Private methods: prefix with underscore (e.g., `_update_state`)

### Project Structure

```text
/
├── src/
│   ├── emulator/
│   │   ├── __init__.py
│   │   ├── evse.py           # EVSE state machine
│   │   ├── ev.py             # EV simulator
│   │   ├── rapi.py           # RAPI protocol handler
│   │   └── serial_port.py   # Virtual serial port
│   ├── web/
│   │   ├── __init__.py
│   │   ├── api.py            # Flask REST API
│   │   ├── websocket.py     # WebSocket handler
│   │   └── static/
│   │       ├── index.html
│   │       ├── style.css
│   │       └── app.js
│   └── main.py               # Entry point
├── tests/
│   ├── test_evse.py
│   ├── test_ev.py
│   ├── test_rapi.py
│   └── test_api.py
├── config.json               # Default configuration
├── requirements.txt
├── README.md
└── SPEC.md
```

## Key Technical Details

### RAPI Protocol

- ASCII-based command/response protocol
- Commands: `$<CMD> [params]\r` (e.g., `$GS\r`)
- Success response: `$OK [data]\r`
- Error response: `$NK\r`
- Common commands: GS (get state), GG (get current), SC (set current), FE (enable)

### EVSE States

- State A (0x01): Ready, not connected
- State B (0x02): Connected, not charging  
- State C (0x03): Charging
- State D (0x04): Ventilation required
- State 0x05: Diode check failed
- State 0x06: GFCI fault
- State 0x07: No ground
- State 0x08: Stuck relay
- State 0x09: GFCI self-test failed
- State 0x0A: Over temperature
- State 0xFE: Sleep mode
- State 0xFF: Disabled

### Error Conditions

Simulate these via API/UI:

- GFCI trip (0x01)
- Stuck relay (0x02)
- No ground (0x04)
- Diode check failed (0x08)
- Over temperature (0x10)
- GFCI self-test failed (0x20)

## Development Guidelines

### When Adding Features

1. Update SPEC.md if the architecture or API changes
2. Add corresponding unit tests
3. Update README.md if user-facing changes
4. Maintain backward compatibility with RAPI protocol

### Error Handling

- Catch exceptions in RAPI command handlers
- Return `$NK\r` for invalid commands
- Log errors for debugging but keep protocol responses clean
- Validate all input parameters

### Threading Considerations

- RAPI handler runs in separate thread
- Use locks when accessing shared state (EVSE/EV state)
- WebSocket updates should be non-blocking
- Simulation loop runs independently

### Testing Best Practices

- Test each RAPI command with valid and invalid inputs
- Verify state transitions follow J1772 specification
- Mock serial port in unit tests
- Test WebSocket messages are sent on state changes

## Common Tasks

### Adding a New RAPI Command

1. Add command handler in `rapi.py`
2. Update command dispatch table
3. Add unit test in `test_rapi.py`
4. Document in SPEC.md if not already documented

### Adding a New API Endpoint

1. Add route in `web/api.py`
2. Update OpenAPI/Swagger docs if present
3. Add test in `test_api.py`
4. Update Web UI if needed

### Adding Error Simulation

1. Define error code constant
2. Add error state to EVSE class
3. Add API endpoint to trigger
4. Add UI control in web interface
5. Update error response in RAPI handlers

## Dependencies

### Required Libraries

- `Flask`: Web framework
- `Flask-SocketIO`: WebSocket support
- `pyserial`: Serial port emulation
- `pytest`: Testing framework

### Optional Libraries

- `eventlet` or `gevent`: Async support for SocketIO
- `python-socketio`: WebSocket client (for testing)

## Debugging Tips

### RAPI Protocol Issues

- Enable echo mode: `$SE 1\r`
- Check command format (must end with \r)
- Verify checksum if using advanced mode
- Use serial monitor in web UI

### State Machine Issues

- Check state transition logs
- Verify conditions for transitions are met
- Test with manual state changes via API
- Check for race conditions with locks

### Web UI Not Updating

- Check WebSocket connection in browser console
- Verify Flask-SocketIO is emitting events
- Check CORS settings if accessing from different origin
- Verify event listeners are registered

## References

- [OpenEVSE RAPI Protocol](https://github.com/OpenEVSE/open_evse/blob/master/firmware/open_evse/rapi_proc.h)
- [SAE J1772 Standard](https://en.wikipedia.org/wiki/SAE_J1772)
- [OpenEVSE Documentation](https://openevse.dozuki.com/)
- SPEC.md in this repository for detailed specifications

---

## Ponytail, lazy senior dev mode

You are a lazy senior developer. Lazy means efficient, not careless. The best code is the code never written.

Before writing any code, stop at the first rung that holds:

1. Does this need to be built at all? (YAGNI)
2. Does it already exist in this codebase? Reuse the helper, util, or pattern that's already here, don't re-write it.
3. Does the standard library already do this? Use it.
4. Does a native platform feature cover it? Use it.
5. Does an already-installed dependency solve it? Use it.
6. Can this be one line? Make it one line.
7. Only then: write the minimum code that works.

The ladder runs after you understand the problem, not instead of it: read the task and the code it touches,
trace the real flow end to end, then climb.

Bug fix = root cause, not symptom: a report names a symptom. Grep every caller of the function you touch
and fix the shared function once — one guard there is a smaller diff than one per caller, and patching only
the path the ticket names leaves a sibling caller still broken.

Rules:

- No abstractions that weren't explicitly requested.
- No new dependency if it can be avoided.
- No boilerplate nobody asked for.
- Deletion over addition. Boring over clever. Fewest files possible.
- Shortest working diff wins, but only once you understand the problem. The smallest change in the wrong place
  isn't lazy, it's a second bug.
- Question complex requests: "Do you actually need X, or does Y cover it?"
- Pick the edge-case-correct option when two stdlib approaches are the same size, lazy means less code,
  not the flimsier algorithm.
- Mark deliberate simplifications that cut a real corner with a known ceiling (global lock, O(n²) scan,
  naive heuristic) with a `ponytail:` comment naming the ceiling and upgrade path.

Not lazy about: understanding the problem (read it fully and trace the real flow before picking a rung, a small diff
you don't understand is just laziness dressed up as efficiency), input validation at trust boundaries, error handling
that prevents data loss, security, accessibility, the calibration real hardware needs (the platform is never the spec
ideal, a clock drifts, a sensor reads off), anything explicitly requested. Lazy code without its check is unfinished:
non-trivial logic leaves ONE runnable check behind, the smallest thing that fails if the logic breaks (prefer
adding/adjusting a small pytest unit test; use fixtures/mocks when they make the test simpler). Trivial one-liners
that don't change behavior may need no test.

---
> Source: [jeremypoulter/OpenEVSE_Emulator](https://github.com/jeremypoulter/OpenEVSE_Emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-07 -->
