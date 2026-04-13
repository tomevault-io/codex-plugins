# GitHub Copilot Instructions for JSRekordFXBridge

This document provides context and guidance for GitHub Copilot when working on the JSRekordFXBridge project.

## 🎯 Project Overview

JSRekordFXBridge is a professional DJ lighting control system built with TypeScript that synchronizes Philips Hue Entertainment lights and DMX fixtures with music. The project follows enterprise-grade development practices using Domain-Driven Design (DDD), clean architecture principles, and dependency injection with InversifyJS.

**Key Technologies:**
- TypeScript 5.2+ with strict mode
- InversifyJS for dependency injection
- Node.js >= 18.0.0
- YAML for configuration and effect definitions
- Hardware integrations: Philips Hue, DMX, MIDI (JZZ)

## 🏛️ Architecture Principles

### Clean Architecture Layers

The project strictly follows Domain-Driven Design with clear separation of concerns:

1. **Domain Layer** (`src/domain/`): Pure business logic and entities with no external dependencies
   - Interfaces and contracts for all services
   - Domain entities and value objects
   - Business rules and invariants

2. **Application Layer** (`src/application/`): Use cases and orchestration
   - Depends only on Domain layer
   - Implements business workflows
   - Coordinates between domain services

3. **Infrastructure Layer** (`src/infrastructure/`): External integrations and hardware
   - Depends on Domain and Application layers
   - Implements domain interfaces for real and mock hardware
   - File system, databases, external APIs

4. **Interface Layer** (`src/interfaces/`): User interfaces
   - Depends on Application layer
   - CLI, REST APIs, WebSocket interfaces

### Dependency Flow

Dependencies must flow inward:
```
Interfaces → Application → Domain
         ↘    ↓         ↗
         Infrastructure
```

**Never import from outer layers into inner layers.**

## 📝 Coding Standards

### TypeScript Guidelines

1. **Strict Mode**: Always use TypeScript strict mode (already configured in `tsconfig.json`)
2. **No `any` types**: Use proper typing or `unknown` if type is truly unknown
3. **Explicit return types**: Always specify return types for functions
4. **Import extensions**: Use `.js` extensions in imports (ES modules requirement)
   ```typescript
   import { Effect } from './effect.js';  // ✅ Correct
   import { Effect } from './effect';     // ❌ Wrong
   ```

### Dependency Injection

1. **Use decorators**: `@injectable()` for classes, `@inject()` for dependencies
2. **Constructor injection**: Always inject dependencies through constructor
3. **Interface-based**: Inject interfaces, not concrete implementations
4. **Type constants**: Use `TYPES` from `src/types/infrastructure/di-container.ts`

Example:
```typescript
import { injectable, inject } from 'inversify';
import { TYPES } from '../types/infrastructure/di-container.js';
import type { ILightController } from '../domain/lighting/light-controller.js';

@injectable()
export class MyService {
  constructor(
    @inject(TYPES.LightController) private lightController: ILightController
  ) {}
}
```

### File Organization

1. **Domain files**: Contain only interfaces and types (no implementations)
2. **Infrastructure files**: Contain concrete implementations of domain interfaces
3. **One class per file**: Each file should contain a single exported class/interface
4. **Naming conventions**:
   - Interfaces: `ILightController`, `EffectRepository` (descriptive names)
   - Implementations: `HueLightController`, `FileEffectRepository` (prefix with tech/approach)
   - Files: kebab-case matching class name: `hue-light-controller.ts`

## 🎨 Effect System

Effects are defined in YAML files in the `effects/` directory:

```yaml
name: "strobo"
description: "Classic strobe effect"
tags: ["strobe", "energy"]
steps:
  - duration_ms: 50
    commands:
      - type: "set_all"
        color: [255, 255, 255]
        intensity: 1.0
  - duration_ms: 50
    commands:
      - type: "set_all"
        color: [0, 0, 0]
        intensity: 0.0
```

**Effect System Architecture:**
- Domain: `EffectEngine`, `EffectExecutor`, `EffectRepository` interfaces
- Application: `EffectEngineService` (orchestration and business logic)
- Infrastructure: `FileEffectRepository` (YAML parsing), `HardwareEffectExecutor` (hardware control)

## 🔧 Configuration

Configuration is managed via YAML files with environment variable substitution:

1. **Primary config**: `config/default-config.yaml`
2. **Environment variables**: `.env` file (use `.env.example` as template)
3. **Environment substitution**: Use `${VAR_NAME}` syntax in YAML
4. **Validation**: Configuration service validates required fields on startup

## 🎭 Demo Mode

The project includes comprehensive mock implementations for hardware-free development:

- `MockLightController`: Simulates Philips Hue lights
- `MockDMXController`: Simulates DMX fixtures
- `MockMIDIController`: Simulates MIDI controllers

**Running demo mode**: `npm run demo` or set `DEMO_MODE=true` environment variable

## 🧪 Testing Strategy

1. **Build before testing**: Run `npm run build` to ensure TypeScript compilation succeeds
2. **Manual testing**: Use `npm run demo` or `npm run demo-cli` for interactive testing
3. **Type checking**: Run `npm run type-check` to verify types without building
4. **Test files**: Located in `tests/` directory, run with `node tests/test-*.js`

## 🚀 Development Workflow

### Adding New Features

1. **Start with domain**: Define interfaces in `src/domain/`
2. **Add application logic**: Implement use cases in `src/application/`
3. **Create infrastructure**: Add real and mock implementations in `src/infrastructure/`
4. **Register in DI**: Update `src/infrastructure/di/container.ts`
5. **Add interface**: Create user-facing interface in `src/interfaces/`

### Adding New Hardware Integration

1. Create domain interface in appropriate domain subfolder
2. Create real implementation in `src/infrastructure/`
3. Create mock implementation for demo mode
4. Register both in DI container with conditional binding based on `DEMO_MODE`
5. Update configuration schema and YAML files
6. Document in README.md

### Adding New Effects

1. Create YAML file in `effects/` directory
2. Use existing effect structure (name, description, tags, steps, commands)
3. Test using: `await bridge.triggerEffect('effect_name')` in demo mode
4. Effects auto-reload when files change

## 📦 Build System

- **Build**: `npm run build` - Compiles TypeScript to `dist/`
- **Watch mode**: `npm run build:watch` - Continuous compilation
- **Development**: `npm run dev` - Run with tsx (no compilation)
- **Production**: `npm run build && npm start`

**Output**: `dist/` directory (excluded from git)

## 🔍 Common Patterns

### Creating a New Service

```typescript
// 1. Define domain interface
// src/domain/my-feature/my-service.ts
export interface MyService {
  doSomething(param: string): Promise<void>;
}

// 2. Implement in infrastructure
// src/infrastructure/my-feature/my-service-impl.ts
import { injectable, inject } from 'inversify';
import { TYPES } from '../../types/infrastructure/di-container.js';
import type { MyService } from '../../domain/my-feature/my-service.js';

@injectable()
export class MyServiceImpl implements MyService {
  constructor(
    @inject(TYPES.SomeDependency) private dependency: SomeDependency
  ) {}

  async doSomething(param: string): Promise<void> {
    // Implementation
  }
}

// 3. Register in DI container
// src/infrastructure/di/container.ts
container.bind<MyService>(TYPES.MyService).to(MyServiceImpl);
```

### Async/Await Convention

Always use async/await, never callbacks or raw promises:

```typescript
// ✅ Good
async doWork(): Promise<void> {
  const result = await this.dependency.fetchData();
  await this.process(result);
}

// ❌ Avoid
doWork(): Promise<void> {
  return this.dependency.fetchData()
    .then(result => this.process(result));
}
```

## ⚠️ Common Pitfalls

1. **Don't mix layers**: Infrastructure should not know about application layer implementation details
2. **Don't use relative imports across layers**: Use absolute imports from layer roots
3. **Don't forget `.js` extensions**: ES modules require explicit extensions in imports
4. **Don't use `any`**: Use proper types or `unknown` with type guards
5. **Don't forget decorators**: All DI classes need `@injectable()` decorator
6. **Don't bypass DI**: Always use constructor injection, never create instances directly

## 📚 Key Files to Reference

- **Architecture overview**: `README.md` (lines 19-100)
- **Type definitions**: `src/types/` directory structure documented in `src/types/README.md`
- **DI setup**: `src/infrastructure/di/container.ts`
- **Effect examples**: `effects/*.yaml`
- **Configuration example**: `config/default-config.yaml`
- **Development roadmap**: `docs/next-steps.md`

## 🎵 Project Status

**Current State**: v3.0 - Fully functional with TypeScript architecture, DI system, and demo mode

**Active Development Areas**:
- Real hardware integration (Philips Hue Entertainment API with DTLS)
- Audio beat detection (aubiojs integration)
- Web-based control interface
- Advanced show system with cue management

## 💡 Tips for Copilot

- When creating new domain interfaces, keep them pure and focused
- Always provide both real and mock implementations for hardware integrations
- Use existing patterns from `src/infrastructure/lighting/` as reference for new hardware
- Check `TYPES` constants before adding new ones to avoid duplication
- Reference existing effect definitions in `effects/` when creating new ones
- Follow the established error handling patterns (console.warn for recoverable errors)
- Use the global `bridge` object in demo mode for interactive testing
- Leverage InversifyJS container resolution instead of manual dependency wiring

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tamaygz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tamaygz)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
