# Copilot Instructions for Quanta Admin Frontend

## Project Overview
This is a manufacturing resource planning (MRP) admin frontend built with Angular 19, using PrimeNG for UI components, NgRx for state management, and TailwindCSS with custom theming.

## Architecture Patterns

### 1. Standalone Components Architecture
- Uses Angular 19 standalone components pattern throughout
- Main layout is `MainLayoutComponent` in `src/app/shared/components/layout/main-layout/`
- Components are self-contained with their own imports array
- No traditional NgModules except for `CoreModule` (legacy structure)

### 2. NgRx State Management
- Centralized state configuration in `src/app/app.config.ts`
- State structure: `departmentCity`, `menu` reducers with corresponding effects
- Feature-based state in `src/app/features/[feature]/state/` directories
- Strict immutability and serialization checks enabled
- DevTools configured for development environment
- Pattern: Actions, Effects, Reducers, Selectors in dedicated folders with index.ts barrel exports

### 3. Layout System
- Complex multi-mode layout supporting: `static`, `overlay`, `slim-plus`, `slim`, `horizontal`, `reveal`, `drawer`
- Layout state managed via `LayoutService` with Angular signals
- Responsive design with mobile/desktop menu states
- Header, Sidebar, Footer, Breadcrumb components in dedicated directories

### 4. PrimeNG Integration
- Custom theme preset extending Material design in `src/app/core/theme/theme.config.ts`
- MRP brand colors defined: primary `#D62974` with full palette
- Dark/light mode support with `.app-dark` selector
- Input style set to 'filled', ripple effects enabled

## Key Development Patterns

### File Organization
```
src/app/
├── core/                    # Singleton services, interceptors, models
│   ├── enums/              # Business domain enums with labels
│   ├── interceptors/       # HTTP interceptors (loading, error)
│   ├── models/             # Shared interfaces
│   ├── services/           # Core services
│   └── state/              # NgRx store structure
├── features/               # Feature modules (empty, ready for lazy loading)
├── shared/                 # Reusable components and services
│   └── components/
│       ├── layout/         # Main layout system
│       └── ui/             # Reusable UI components
└── environments/           # Environment configurations
```

### Enum Pattern
Enums follow consistent pattern with companion label objects:
```typescript
export enum ProductionStatus {
    waiting = 'waiting',
    in_progress = 'in_progress'
}

export const PRODUCTION_STATUS_LABELS: Record<ProductionStatus, string> = {
    [ProductionStatus.waiting]: 'En Espera',
    [ProductionStatus.in_progress]: 'En Progreso'
};
```

### HTTP Interceptors
- `LoaderInterceptor`: Shows/hides global loader for all HTTP requests
- `HttpErrorInterceptor`: Centralized error handling

### Environment Configuration
- Multi-environment setup: development, qa, production
- Mock API support via `useMockApi` flag
- API base URL configuration per environment

## Development Workflow

### Commands
- `npm start` - Start development server (port 4200)
- `npm run build` - Production build
- `npm test` - Unit tests with Karma
- `ng generate component component-name` - Generate components

### Styling Approach
- TailwindCSS + PrimeFlex utility classes
- SCSS modules for component-specific styles
- Global layout styles in `src/assets/layout/`
- Custom CSS properties for theming

### State Management Guidelines
- Use NgRx for complex state (cross-component communication)
- Local component state with Angular signals for simple UI state
- Async operations handled through Effects with automatic toast notifications
- Strict typing for all actions and state interfaces
- Feature state follows pattern: `featureKey`, `FeatureState` interface, `ActionReducerMap`, Effects array
- Toast notifications integrated in Effects for success/error feedback

## Manufacturing Domain Context
This system manages production planning, material distribution, purchase orders, and production tracking. Key business entities include production orders, material reception, inventory management, and task scheduling.

---
> Source: [QPalliance-com/quanta-administrador-frontend](https://github.com/QPalliance-com/quanta-administrador-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
