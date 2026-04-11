# Trading Application Workspace Instructions

This workspace contains a desktop algorithmic trading application that replicates TradingView Pine Script strategy logic with MetaTrader 5 integration.

## Project Overview
- **Language**: Python 3.10+
- **UI Framework**: PySide6
- **Broker Integration**: MetaTrader 5
- **Architecture**: Clean Architecture with separated concerns

## Core Modules
- `market_data_service.py` - Fetches OHLC data from MT5
- `indicator_engine.py` - Computes EMA50, EMA200, ATR14
- `pattern_engine.py` - Detects Double Bottom patterns
- `strategy_engine.py` - Evaluates entry/exit conditions
- `risk_engine.py` - Calculates position sizing
- `execution_engine.py` - Sends orders to MT5
- `state_manager.py` - Tracks positions and cooldowns
- `logger.py` - Comprehensive logging

## Trading Strategy
- **Instrument**: XAUUSD
- **Timeframe**: 1H
- **Mode**: LONG ONLY (no short trades)
- **Pattern**: Double Bottom detection
- **Risk Management**: Fixed % per trade

## Development Rules
- No intrabar execution (bar-close values only)
- No repainting logic
- Logic must match Pine Script specification exactly
- Prefer clarity over performance optimizations
- All trading logic stays in engine modules, NOT in UI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mutafchievAlex)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mutafchievAlex)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
