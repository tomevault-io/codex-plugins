<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# VDay Game - 2D Sprites in 3D Space

This project is a 2D game with 3D movement capabilities, similar to Stardew Valley's top-down perspective.

## Project Architecture

- **Game Engine**: Three.js for 3D rendering and physics
- **Build Tool**: Vite for fast development and building
- **Rendering**: 2D sprites in 3D space with proper depth sorting
- **Camera**: Top-down perspective with slight angle for depth perception

## Code Guidelines

1. **Sprite Management**: Use the SpriteManager class for all sprite operations
2. **Player Movement**: 3D movement with 2D sprite representation
3. **Input Handling**: WASD movement, Space for jump, Shift for run
4. **Animation**: Simple scale-based animations for character states
5. **Depth Sorting**: Sprites are sorted by distance from camera for proper layering

## Key Components

- `Game.js` - Main game loop and scene management
- `Player.js` - Player character with physics and animation
- `SpriteManager.js` - Handles sprite creation and depth sorting
- `InputManager.js` - Keyboard and mouse input processing

## Development Notes

- Sprites are created using Canvas for placeholder graphics
- Physics includes gravity, jumping, and ground collision
- Camera follows player with smooth interpolation
- UI shows player position and FPS counter

When making changes, ensure:
- Proper sprite depth sorting is maintained
- Physics calculations are frame-rate independent
- Input handling is responsive and smooth
- Animation states transition correctly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jacqueb-1337) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
