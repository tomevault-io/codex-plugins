# Battery Visualizer - AI Agent Instructions

## Project Overview
A React TypeScript application for calculating and visualizing 18650 battery pack configurations with 3D modeling capabilities. Built with Vite, Three.js, and Tailwind CSS.

## Architecture & Data Flow

### Core Components
- **App.tsx**: Root component managing page navigation between calculator and 3D modeler
- **CalculatorPage**: Dual calculator interface (general pack + e-bike estimator) 
- **ModelerPage**: 3D visualization with export capabilities (STL/SVG)
- **ThreeScene**: Complex Three.js component using instanced meshes for performance

### State Management Pattern
Uses React hooks with callback-based navigation. Key pattern: `PackConfig` object flows from calculator to modeler via props, not global state.

```typescript
// Core data structure - used throughout app
interface PackConfig {
  series: number;
  parallel: number; 
  totalCells: number;
  actualVoltage: number;
  actualCapacity: number;
  totalEnergy: number;
}
```

### Performance Considerations
- **BatteryGrid**: Renders max 200 cells to prevent DOM overload
- **ThreeScene**: Uses THREE.InstancedMesh for thousands of battery cells
- Lazy building pattern with loading states in ModelerPage

## Development Workflows

### Local Development
```bash
pnpm install
# Set GEMINI_API_KEY in .env.local (though app works without it)
pnpm dev  # Vite dev server
```

### Build & Deploy
```bash
pnpm build    # Vite production build
pnpm preview  # Test production build locally
```

## Project-Specific Patterns

### Three.js Integration
- **Memory Management**: Manual disposal of geometries/materials in useEffect cleanup
- **Performance**: InstancedMesh pattern for repeated objects (battery cells)
- **Scene Building**: Async pattern with `onBuildComplete` callback to manage loading states

### Type Safety
- Enum-based constants: `CellType`, `RangeUnit`, `RidingStyle`
- Dimension lookup pattern: `CELL_DIMENSIONS` record for cell specifications
- Strict typing on all props and state

### Component Composition
- **Card Pattern**: `CalculatorCard` wrapper for consistent styling
- **Input Pattern**: `InputGroup` with built-in unit switching capability
- **Icon System**: SVG components in `components/icons/` following consistent naming

### Export Functionality
- **STL Export**: Uses three/examples/jsm/exporters/STLExporter for 3D printing
- **SVG Export**: Custom SVG generation for 2D layouts with dimensions
- File download pattern using Blob URLs and temporary anchor elements

## Key Files & Dependencies

### Critical Files
- `types.ts`: All TypeScript interfaces and enums
- `ThreeScene.tsx`: Most complex component - handles 3D rendering and performance
- `CalculatorPage.tsx`: Business logic for battery calculations

### External Dependencies
- **three**: 3D graphics library with OrbitControls and STLExporter
- **react 19.1.1**: Latest React with modern hooks patterns
- **vite**: Build tool with TypeScript support

### Configuration Files
- `vite.config.ts`: Custom config for environment variables (GEMINI_API_KEY)
- `tsconfig.json`: TypeScript configuration for modern React

## Common Tasks

### Adding New Cell Types
1. Update `CellType` enum in `types.ts`
2. Add dimensions to `CELL_DIMENSIONS` in `ModelerPage.tsx`
3. Test with both 2D grid and 3D visualization

### Modifying Calculations
- E-bike calculations: Update consumption constants in `CalculatorPage.tsx`
- Pack calculations: Logic in `calculatePack` callback
- Both use `useEffect` with dependency arrays for auto-recalculation

### Performance Optimization
- Check cell count limits in `BatteryGrid` (MAX_CELLS_TO_RENDER)
- Monitor Three.js memory usage in `ThreeScene` cleanup
- Consider InstancedMesh patterns for new repeated 3D objects

---
> Source: [raushanraja/lithium-pack-designer](https://github.com/raushanraja/lithium-pack-designer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-11 -->
