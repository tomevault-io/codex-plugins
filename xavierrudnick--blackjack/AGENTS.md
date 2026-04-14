# Blackjack Monte Carlo Simulator - AI Coding Instructions

## Project Overview
A C++17 blackjack simulator for running Monte Carlo simulations to analyze card counting strategies and optimal playing deviations. Outputs statistical EV data per true count for strategy comparison.

## Architecture

### Core Components
- **Engine** ([include/core/Engine.h](include/core/Engine.h)): Main game loop handling blackjack rules, player actions, and hand evaluation
- **EngineBuilder**: Fluent builder pattern for configuring game rules (H17/S17, DAS, surrender, penetration)
- **FixedEngine** ([include/core/FixedEngine.h](include/core/FixedEngine.h)): Monte Carlo evaluation engine - runs forced actions and tracks EV by true count
- **Deck**: Supports multi-deck shoes with `Deck::createTestDeck()` for deterministic testing

### Strategy System
All counting strategies inherit from `CountingStrategy` ([include/strategy/CountingStrategy.h](include/strategy/CountingStrategy.h)):
- **Balanced**: HiLo, ZenCount, OmegaII, Mentor, R14, WongHalves, RAPC, RPC in `include/strategy/balanced/`
- **Unbalanced**: Ko, Red7, KissIII, UZenII, UstonSS in `include/strategy/unbalanced/`
- Strategies must implement: `getBetSize()`, `updateCount()`, `getTrueCount()`, `shouldAcceptInsurance()`, deviation methods

### Player System
- `Player` interface → `BotPlayer` (automated) / `HumanPlayer` (interactive)
- Players own a `CountingStrategy` via `std::unique_ptr`

### Event System
Singleton `EventBus` with observer pattern for optional console output during debugging.

## Build & Run

```bash
# Production build (optimized)
make

# Debug build
make CXXFLAGS="-std=c++17 -Wall -Wextra -g"

# Run simulation
./blackjack

# Run tests
./run_tests              # Core engine tests
./run_fixed_engine_tests # Monte Carlo tests
```

## Key Patterns

### Creating Engines
Always use `EngineBuilder` - never construct `Engine` directly:
```cpp
Engine engine = EngineBuilder()
    .setDeckSize(6)
    .setPenetrationThreshold(0.80)
    .withH17Rules(true)        // Dealer hits soft 17
    .allowDoubleAfterSplit(true)
    .enableMontiCarlo(true)    // For EV analysis
    .setActions({Action::Hit, Action::Stand})
    .setActionValues({{16, 10}, {15, 10}})  // player_total, dealer_upcard
    .build(&player);
```

### Adding New Counting Strategies
1. Create header in `include/strategy/balanced/` or `unbalanced/`
2. Implement source in `src/strategy/balanced/` or `unbalanced/`
3. Add to `createStrategies()` lambda in [src/main.cpp](src/main.cpp)
4. Makefile auto-discovers via `$(wildcard src/strategy/*.cpp ...)`

### Monte Carlo Simulations
- `runMontesims()` compares two actions across specified player/dealer value pairs
- Results saved to `stats/{Strategy}_{ActionSet}_{H17|S17}.csv`
- Use `analyze_deviations.py` to find optimal true count crossover points

### Test Structure
Tests use rigged decks for determinism:
```cpp
std::vector<Card> stack = {Card(Rank::Ten, Suit::Hearts), ...};
Deck riggedDeck = Deck::createTestDeck(stack);
```

## Code Conventions
- Headers in `include/`, sources in `src/`, mirrored directory structure
- Use `std::unique_ptr` for strategy ownership
- `GameConfig` struct holds all rule variations - modify via builder only
- Action enum: `Hit`, `Stand`, `Double`, `Split`, `Surrender`, `InsuranceAccept`, `InsuranceDecline`

## Data Flow
1. `main.cpp` → `setUpSimsH17()`/`setUpSimsS17()` configures scenarios
2. `runMontesims()` creates strategies and runs iterations
3. `Engine::runnerMonte()` → `FixedEngine::calculateEV()` records per-action stats
4. `FixedEngine::savetoCSVResults()` outputs CSV with EV per true count
5. Python scripts (`analyze_deviations.py`) post-process for deviation tables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XavierRudnick) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
