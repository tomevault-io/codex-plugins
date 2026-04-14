# LLM Autopilot - AI Coding Agent Guide

## Project Overview
Self-healing autopilot system for LLM inference servers that maintains SLO latency (default 600ms TTFT) while maximizing GPU efficiency through dynamic resource adjustments.

**Current Version**: v0.5-α (Predictive Controller)
- **v0.4**: Reactive control with mode switching (baseline)
- **v0.5-α**: Predictive control with queue velocity, acceleration, TTFT forecasting, confidence gating, and emergency circuit breaker

## Architecture: Control Loop Pattern

**Four-component control system** (Models in `models.py`):
1. **Collector** (`collector.py`) - Polls `/metrics` endpoint every 1s, calculates queue velocity from deltas, exposes live metrics on `:8080/live_metrics`
2. **Controller** (`controller.py`) - Decision engine with mode switching (`safe`/`throughput_optimized`/`latency_optimized`), uses 3-metric stability window for smoothing
3. **Actuator** (`actuator.py`) - Applies decisions via `/configure` endpoint, tracks current config state
4. **FakeLLMServer** (`fake_llm_server.py`) - Simulation with auto-load generator creating realistic traffic bursts

**Orchestration**: `run_autopilot.py` -> `Autopilot` class manages lifecycle, decision loop runs every 2s (configurable), uses 3-metric moving average for TTFT smoothing.

## Key Data Flows

```
FakeLLMServer → /metrics → Collector → metrics_history (last 100) 
                                    ↓
                                Controller.make_decision(metrics)
                                    ↓
                                Actuator → /configure → FakeLLMServer.config
```

**Queue Velocity Calculation** (critical for pre-scaling):
```python
# collector.py line ~35
queue_velocity = (queue_delta) / time_delta  # requests/second
```

## Decision Logic Patterns

**Mode Switching** (`controller.py` lines 55-62):
- `throughput_optimized`: queue_depth > 50 AND avg_gpu < 85%
- `latency_optimized`: TTFT exceeds SLO
- `safe`: default stable state

**Action Triggers** (examples):
- `REDUCE_BATCH`: TTFT > SLO * 1.2 (hard breach)
- `ENABLE_SPECULATIVE_DECODE`: speculative_factor < 0.5 in latency mode
- `REBALANCE_LOAD`: memory_efficiency < 0.7 in throughput mode

## Critical Conventions

**Async/Await Everywhere**: All I/O uses `asyncio` + `aiohttp`. Never use blocking calls.

**Metrics Object Structure**: 11 fields including `queue_velocity` (computed), `timestamp`, standard latencies. Use `models.Metrics` dataclass.

**Config Boundaries**: 
- Batch size: 1-32 (actuator enforces, lines 115-118)
- GPU count: 1-8 (lines 120-123)
- Moving average window: 3 samples (stability_window)

**Decision Logging**: Dual-format logging:
- Human-readable: `decision_log.txt` (rotating, 5MB limit)
- Structured: `decision_log.jsonl` (one JSON per line with ISO timestamps)

## Development Workflows

**Run System**:
```bash
python run_autopilot.py  # Starts fake server + autopilot loop
```

**Testing**: 
```bash
pytest tests/test_run_autopilot.py  # Uses pytest-asyncio
```
Tests simulate load conditions by injecting fake metrics directly to controller.

**Monitor Live**:
```bash
curl http://localhost:8080/live_metrics  # Collector's HTTP server
curl http://localhost:8000/metrics       # Fake LLM server
```

**Adjust Server Behavior**:
```bash
curl -X POST http://localhost:8000/auto_load -d '{"enabled": false}'  # Disable traffic bursts
```

## Component-Specific Notes

**Collector**: 
- Maintains `metrics_history` (max 100), provides `get_moving_average()` and `get_trend()` helpers
- HTTP server runs on separate asyncio task (lines 138-147)

**FakeLLMServer**:
- Latency formula: `ttft = base(200) + batch^1.5 * 60 + queue * 40` (line 32)
- GPU saturation (>85%) multiplies latency by 1.4x (lines 59-61)
- Auto-load generates random bursts (5-20 requests) every 1-3s (lines 171-185)

**Actuator**:
- Always fetches config after applying changes to stay in sync
- Supports extra kwargs in `_update_config()` for future extensibility (rebalance, speculative_mode, overlap_mode)

## Gotchas

- **Queue velocity needs history**: First metrics poll has 0 velocity (no previous sample). Controller handles gracefully.
- **Shutdown order matters**: Stop collector → stop HTTP server → cleanup actuator → stop fake server (see `run_autopilot.py` lines 155-172)
- **Decision interval > poll interval**: Default 2s vs 1s to allow metric accumulation before deciding
- **Enum serialization**: Use `action.value` not `action` when logging (see `decision_logger.py` custom_serializer)

## Dependencies

Single external dep: `aiohttp==3.9.1` (handles all HTTP client/server needs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skreddi06) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
