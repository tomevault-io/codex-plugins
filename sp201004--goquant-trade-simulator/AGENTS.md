# Copilot Instructions for GoQuant Trade Simulator

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a high-performance trade simulator for cryptocurrency markets that processes real-time L2 orderbook data from OKX exchange via WebSocket connections.

## Code Style Guidelines
- Use Python type hints for all function parameters and return values
- Follow PEP 8 style guidelines
- Use dataclasses for structured data
- Implement proper error handling and logging
- Use async/await for WebSocket and I/O operations

## Financial Modeling Context
- Implement Almgren-Chriss model for market impact calculation
- Use linear/quantile regression for slippage estimation
- Apply logistic regression for maker/taker proportion prediction
- Calculate transaction costs including fees, slippage, and market impact

## Performance Requirements
- Process market data faster than stream reception
- Minimize latency in data processing pipeline
- Use efficient data structures (numpy arrays, pandas DataFrames)
- Implement proper memory management

## WebSocket Implementation
- Connect to: wss://ws.gomarket-cpp.goquant.io/ws/l2-orderbook/okx/BTC-USDT-SWAP
- Handle reconnection and error scenarios
- Process L2 orderbook data in real-time
- Update UI components with each tick

## UI Framework
- Use modern web technologies (HTML5, CSS3, JavaScript)
- Implement real-time updates without page refresh
- Create responsive design for different screen sizes
- Display input parameters and calculated outputs clearly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sp201004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
