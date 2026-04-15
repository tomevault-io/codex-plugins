# Project Guidelines

## Code Style
- Use C++20 and keep compatibility with the existing CMake configuration.
- Follow the current project style seen in src/: small focused classes, clear ownership with std::unique_ptr, and explicit error-path handling.
- Prefer extending existing modules under src/net/http and src/net/core instead of introducing parallel abstractions.

## Architecture
- Entry point is src/main.cpp, which starts hpserver.
- Event loop and connection lifecycle are in src/hpserver.cpp and src/hpserver.h.
- Core networking primitives live in src/net/core/.
- HTTP connection orchestration is in src/net/http/http_conn.h and src/net/http/http_conn.cpp, with protocol parsing and IO separated into http_request_parser and http_connection_io.
- Threading utilities are in src/util/.

## Build And Test
- Configure and build from workspace root:
  - cmake -S . -B build -G Ninja
  - cmake --build build
- Run all tests:
  - ctest --test-dir build --output-on-failure
- Run a specific test binary when needed:
  - ./build/tests/http_conn_test
  - ./build/tests/threadsafe_queue_test
  - ./build/tests/thread_pool_test

## Conventions
- Keep epoll IO handling and http_conn state mutation on the reactor thread unless a change explicitly introduces synchronized ownership.
- connections_ is indexed by client fd; preserve MAX_FD boundary checks and keep storage sized for direct fd indexing.
- Non-blocking accept loops should treat EAGAIN and EWOULDBLOCK as normal and retry on EINTR.
- Under EPOLLET, read and write paths should drain until EAGAIN/EWOULDBLOCK; treat EINTR as retry for syscalls that can be interrupted.
- Prefer test-first changes for parser and connection behavior; add or update tests in tests/http_conn_test.cpp for protocol edge cases.

## Documentation
- Use doc/http_conn.md for current http_conn behavior, state machine details, and parser/buffer limits.
- Use doc/proxy_agent_runbook.md for staged proxy implementation tasks and validation commands.
- Use doc/loop.md for event-loop flow and timer cleanup behavior.
- Use doc/http_conn_refactor_plan.md as historical context for refactor intent; confirm final behavior in src/ and tests/.
- Treat doc/agent.md as roadmap-level material; validate behavior against src/ and tests/ before implementing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kdb17wsl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
