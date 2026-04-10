# Copilot Instructions for concurrency-problems

## Project Overview

This repository contains educational C/C++ implementations of classic concurrency problems and a production-grade stream processing system. All projects focus on **thread safety, lock-free algorithms, and high-throughput concurrent systems**.

## Repository Structure

| Directory | Language | Purpose |
|-----------|----------|---------|
| `stream-processing/` | C++17 | Kafka-like message broker with topics, partitions, and persistent storage |
| `stream-processing-deploy/` | C++/Docker | Production deployment + Coinbase market data bridges |
| `linkedlist/` | C11 | Lock-free sorted linked list using CAS operations |
| `dining-philosophers-semaphore/` | C11 | Classic synchronization problem with custom semaphores |

## Build Commands

```bash
# Stream Processing (C++17, CMake)
cd stream-processing/build && cmake .. && make -j$(nproc)
./test_basic && ./test_broker && ./test_network  # Run tests

# Lock-Free List (C11)
cd linkedlist && make && make run  # Performance test
make stress  # 60-second stress test

# Dining Philosophers (C11)
cd dining-philosophers-semaphore && make && make run
```

## Architecture Patterns

### Stream Processing Core (`stream-processing/`)
- **Broker → Topic → Partition → LogSegment** hierarchy
- Messages routed by key hash to partitions (see `include/hash.h`)
- Thread-safe with `std::shared_mutex` for read-heavy workloads
- Binary wire protocol in `include/network/protocol.h`
- Tests use inline `#define TEST(name)` macro framework (no external deps)

### Lock-Free Patterns (`linkedlist/`)
- Custom `cas_ptr()` and `cas_bool()` wrappers around C11 `_Atomic`
- Two-phase deletion: logical mark → physical removal during traversal
- Sentinel head/tail nodes to simplify boundary conditions

### Semaphore Pattern (`dining-philosophers-semaphore/`)
- Custom semaphore built from `pthread_mutex_t` + `pthread_cond_t`
- Room semaphore limits concurrent fork-grabbers (N-1 of N philosophers)

## Code Conventions

- **C code**: C11 standard, `_Atomic` types, explicit `atomic_init()/atomic_store()/atomic_load()`
- **C++ code**: C++17, `std::shared_ptr<Message>` aliased as `MessagePtr`, RAII throughout
- **Headers**: Use `#pragma once`, namespace `stream` for C++
- **Error handling**: Return codes in C, success booleans/optional results in C++
- **No external test frameworks**: Custom `ASSERT()` macros, `TEST()` definitions

## Key Files to Reference

| When working on... | Reference these files |
|--------------------|----------------------|
| Message serialization | `include/message.h`, `tests/test_serialization.cpp` |
| Thread-safe queues | `include/thread_safe_queue.h` (header-only, MPMC) |
| Network protocol | `include/network/protocol.h`, `src/network/tcp_server.cpp` |
| Lock-free algorithms | `linkedlist/lock_free_list.h`, CAS pattern examples |
| Docker deployment | `stream-processing-deploy/docker-compose.yml`, `Dockerfile` |

## Testing Patterns

- Each `stream-processing/tests/test_*.cpp` is a standalone executable
- Run individual tests: `./build/test_<name>` (e.g., `test_broker`, `test_network`)
- C projects use `make valgrind` for memory leak detection
- Use `-DCMAKE_BUILD_TYPE=Tsan` for ThreadSanitizer, `-DCMAKE_BUILD_TYPE=Asan` for AddressSanitizer

## Docker Deployment

```bash
cd stream-processing-deploy
docker-compose build
docker-compose up -d                    # Basic server on port 9092
docker-compose --profile coinbase up -d  # With Coinbase WebSocket bridge
```

## Common Tasks

**Adding a new message type**: Update `include/message.h`, add serialization in `src/core/message.cpp`, test in `tests/test_serialization.cpp`

**Adding a broker feature**: Implement in `src/core/broker.cpp`, expose in `include/broker.h`, test in `tests/test_broker.cpp`

**Modifying lock-free list**: Update both `lock_free_list.c` and `lock_free_list.h`, verify with `make valgrind` and `make stress`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Siyam-A-S)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Siyam-A-S)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
