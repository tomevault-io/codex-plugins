# Copilot Instructions for Pyxel Turn-Based Strategy Game

## Architecture Overview

This is a **turn-based strategy game** built with the Pyxel library using a **State Machine pattern** for game flow management. The core architecture centers around:

- **MapScene**: Main game scene that delegates behavior to states via `StateContext`
- **GameState**: Persistent game data (cities, roads, characters) with auto-save functionality
- **State Machine**: Handles turn flow through dedicated state classes in `map_states.py`

### Key Files & Dependencies
- `map.py` - Main scene with state machine integration
- `map_states.py` - All game state implementations (PlayerTurnState, EnemyTurnState, etc.)
- `map_state_machine.py` - State machine infrastructure and base classes
- `game_state.py` - Data model classes (City, Road, Player, Enemy) with serialization
- `game.py` - Scene base classes and global constants

## Coding Standards

follow flake8 and PEP 8 conventions.

## State Machine Pattern Implementation

The game flow is managed through distinct states inheriting from `MapGameState`:

```python
# State transition example from map_states.py
class PlayerTurnState(MapGameState):
    def update(self):
        # Handle player movement completion
        if player_movement_complete:
            self.transition_to(TransitionState(self.context))
```

### Critical State Flow
1. **PlayerTurnState** → **TransitionState** → **BattleSequenceState** (if battles) or **CutinState**
2. **CutinState** → **EnemySelectionState** → **EnemyTurnState** → **TransitionState**
3. Auto-save occurs in `TransitionState.enter()`

## Character System & AI

### AI Types (in Enemy class)
- `"aggressive"` - BFS pathfinding to nearest player
- `"patrol"` - Fixed route cycling through `patrol_city_ids`
- `"defensive"` - Move to farthest position from players
- `"random"` - Random connected city selection

### Character Movement Pattern
Characters use a target-based movement system:
```python
# Set movement target
character.target_x = city.x
character.target_city_id = city.id
character.is_moving = True

# Movement completion triggers state transitions
```

## Testing & Development Workflow

### Running Tests
```powershell
# Windows PowerShell (UTF-8 BOM encoded)
.\test.ps1 test           # Basic test run
.\test.ps1 test-coverage  # With coverage report
.\test.ps1 lint          # Code quality check
```

### Test Structure
- `test_game_state.py` - Comprehensive unit tests for all data classes
- Tests cover serialization, AI behavior, battle system, and state management
- Coverage target: 90%+ overall, 95%+ for core classes

## Project-Specific Conventions

### Coordinate System
- **World coordinates**: Absolute map positions (float)
- **Screen coordinates**: Relative to camera offset
- **Grid system**: 30x30 tiles, 16x16 pixels each

### Auto-Save Pattern
```python
# Every turn transition triggers auto-save
def enter(self):  # In TransitionState
    self.context.game_state.auto_save()
```

### State Machine Context Usage
```python
# Access shared game data through context
self.context.game_state.players
self.context.change_state(new_state)
self.context.set_camera_follow(target)
```

### Debug System
- Press `V` to cycle through 3 debug pages
- State history tracking in `StateContext`
- Real-time state visualization and AI behavior monitoring

## Integration Points

### SubScene Pattern
```python
# Battle integration example
self.context.set_sub_scene(BattleSubScene(battles))
# SubScene completion automatically returns to main state
```

### Resource Management
- Sprites loaded from `resources.pyxres`
- Game state persisted to `saves/game_state.json`
- Character sprites use indexed animation (2-frame cycles)

## Common Pitfalls

1. **State transitions**: Always use `self.transition_to()`, never direct assignment
2. **Camera follow**: Clear with `self.context.clear_camera_follow()` on state entry
3. **Movement completion**: Check `character.is_moving` before triggering transitions
4. **AI timer**: Reset in `EnemySelectionState` for proper 2-second think delay

## When Adding New Features

- **New AI types**: Add to Enemy class and implement decision logic in `EnemyTurnState`
- **New states**: Inherit from `MapGameState`, add enum to `MapStateType`
- **New characters**: Extend `Character` base class, update serialization methods
- **Map modifications**: Update `generate_30x30_map()` in MapScene

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/llatzhar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
