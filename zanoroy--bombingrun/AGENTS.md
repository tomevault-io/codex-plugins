# Copilot Instructions for Bombing Run Project

## Project Overview
**Project Name:** Bombing Run  
**Type:** 2D Aerial Combat Strategy Game  
**View Perspective:** Top-Down 2D  
**Language:** C++  
**Platform:** Cross-platform (Linux/Windows)  
**Current Phase:** Phase 3 - Bomber Aircraft System  
**Project Start:** 3 January 2026

## Project Context

### Core Architecture
- **View Perspective:** Top-Down 2D (bird's eye view of battlefield)
- **Design Pattern:** Object-oriented with self-contained objects
- **Extensibility:** Designed for future expansion of weapons, bombs, and targets
- **Graphics:** 2D sprite-based rendering from above
- **Coordinate System:** Standard 2D canvas with origin at top-left
- **Map Resolution:** Standard map is 1000x600 pixels
- **Sprite Scaling:** All sprites scale proportionally to map size
- **Target Performance:** 60 FPS

### Key Game Mechanics
1. **Aircraft System:** Bombers (3 HP) and Fighter Jets (4 HP, BattleGround only)
2. **Bomb Types:** 7 standard types (100lb-8000lb) + Airstrike special weapon
3. **Maps:** City (1.0x scale) and BattleGround (0.25x scale)
4. **Combat:** Multi-hit damage system, bullet physics, predictive targeting
5. **Destruction:** Permanent craters, destructible buildings, runway damage

### Technical Specifications

#### Bomb Configuration Table
| Type     | Max Bombs | Crater Size | Target Radius | Damage | Speed |
|----------|-----------|-------------|---------------|--------|-------|
| 100lb    | 10        | 15px        | 60px          | 1      | 6.0   |
| 250lb    | 8         | 20px        | 75px          | 2      | 5.5   |
| 500lb    | 6         | 25px        | 90px          | 3      | 5.0   |
| 1000lb   | 3         | 40px        | 120px         | 5      | 4.0   |
| 2000lb   | 1         | 80px        | 90px          | 10     | 0.8   |
| 4000lb   | 1         | 160px       | 60px          | 20     | 0.3   |
| 8000lb   | 1         | 320px       | 60px          | 40     | 0.2   |
| Airstrike| 15/plane  | 15px        | 90px          | 1      | 5.0   |

#### Key Game Values
- **Bomber Health:** 3 hits to destroy
- **Fighter Jet Health:** 4 hits to destroy
- **Fighter Jet Ammo:** 579 bullets
- **Fighter Jet Accuracy:** 80%
- **Runway Health:** 1000 HP (regenerates gradually)
- **Building-Runway Link:** Each destroyed building reduces runway max health by 10 HP

### Code Structure Guidelines

#### Core Classes to Implement
1. **Game Controller:** Main game loop and state management
2. **Aircraft Manager:** Handle bomber and fighter jet systems
3. **Weapon System:** Bomb and bullet mechanics
4. **Map Generator:** Procedural building and base generation
5. **Collision System:** Detect impacts and interactions
6. **UI Manager:** Handle interface and HUD updates
7. **Rendering System:** Draw all visual elements with scaling

#### Base Class Hierarchy
```cpp
// Example structure (adjust as needed)
GameObject (base)
├── Aircraft (abstract)
│   ├── Bomber
│   └── FighterJet
├── Weapon (abstract)
│   ├── Bomb (abstract)
│   │   ├── StandardBomb (100lb-8000lb)
│   │   └── Airstrike
│   └── Bullet
└── Structure (abstract)
    ├── Building (abstract)
    │   ├── CivilianBuilding
    │   └── MilitaryBuilding
    └── Runway
```

#### Data Structures
- **Aircraft Arrays:** Store all active planes and fighters
- **Projectile Arrays:** Track bombs and bullets in flight
- **Building Arrays:** Manage destructible structures
- **Effect Arrays:** Handle explosions and visual effects

### Design Patterns to Use
- **Object Pooling:** For frequently created/destroyed objects (bullets, explosions)
- **Component Pattern:** For game objects with modular behaviors
- **State Pattern:** For game states (Menu, Playing, GameOver)
- **Observer Pattern:** For event handling (explosions, damage, victory)

### Coding Standards

#### Style Guidelines
- Use modern C++ (C++17 or later)
- RAII for resource management
- Smart pointers for dynamic memory
- Const-correctness throughout
- Descriptive variable names (avoid abbreviations)
- Comment complex algorithms and game logic

#### Performance Considerations
- Efficient collision detection algorithms
- Object pooling for frequently created items
- Optimized rendering with culling for off-screen elements
- Avoid dynamic allocation in game loop
- Profile regularly to identify bottlenecks

### File Organization
```
BombingRun/
├── src/
│   ├── core/           # Game loop, state management
│   ├── entities/       # Aircraft, weapons, buildings
│   ├── systems/        # Collision, rendering, physics
│   ├── ui/             # Menu, HUD, interface
│   ├── maps/           # Map generation and management
│   └── utils/          # Helper functions, math
├── assets/
│   ├── sprites/        # Aircraft, building, effect sprites
│   ├── audio/          # Sound effects and music
│   └── textures/       # Background textures
├── include/            # Header files
├── tests/              # Unit tests
└── docs/               # Additional documentation
```

### Current Development Status
- **Active Phase:** Phase 3 - Bomber Aircraft System
- **Next Steps:** Create Bomber class, implement spawning system, add movement patterns and health
- **Blockers:** None

### Build & Run Information
- **Project Root:** `/home/nightcoder/Projects/BombingRun`
- **Build Directory:** `/home/nightcoder/Projects/BombingRun/build`
- **Executable Location:** `/home/nightcoder/Projects/BombingRun/build/bin/BombingRun`
- **Build Command:** `cd /home/nightcoder/Projects/BombingRun/build && make`
- **Run Command:** `cd /home/nightcoder/Projects/BombingRun && build/bin/BombingRun`

### Important Reminders for AI
1. **Top-Down View:** All game elements viewed from above (bird's eye perspective)
2. **Sprite-based:** All visual elements should support sprite swapping for customization
3. **Extensibility:** Design for easy addition of new weapons, bombs, and targets
4. **Cross-platform:** Code must compile and run on both Linux and Windows
5. **Self-contained Objects:** Each object should manage its own state and behavior
6. **Map Independence:** Each map is a separate "level" that can be selected
7. **Ground Destruction:** Ground base damage affects vehicle movement (pathfinding consideration)

### Key References
- **Main Specification:** `Bombing Run Game Specification.md`
- **Phase Tracking:** `Development-Phases.md`
- **Project Repository:** `/home/nightcoder/Projects/BombingRun`

### Decision Log
- **Graphics Library:** SDL2 (selected 3 Jan 2026 - better Linux support, hardware acceleration)
- **Audio Library:** SDL_mixer (complements SDL2)
- **Build System:** CMake (cross-platform)
- **Version Control:** Git

### Testing Strategy
- Unit tests for core mechanics (collision, damage, physics)
- Integration tests for system interactions
- Cross-platform testing on Linux and Windows
- Performance profiling at each phase completion
- Gameplay balance testing during Phase 13

### Common Patterns in This Project

#### Scaling System
- Standard Map Resolution: 1000x600 pixels
- City Map: 1.0x scale (standard)
- BattleGround Map: 0.25x scale (4x zoom out)
- All coordinates and sizes must scale proportionally
- All sprites scale proportionally to map size

#### Multi-hit Damage System
- Track hit count per entity
- Visual feedback on damage (color changes: normal → orange → red)
- Smoke effects for damaged units
- Destroy when hit count reaches max health

#### Input Bindings
- **Mouse Click:** Drop bombs at cursor position
- **Number Keys (1-7):** Select bomb types
- **A Key:** Deploy airstrike
- **Spacebar:** Deploy bomber with current bomb type

---

**Last Updated:** 3 January 2026  
**Update Frequency:** Update this file when making major architectural decisions or completing phases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zanoroy)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Zanoroy)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
