# GitHub Copilot Instructions for @sandeep-jaiswar/ui

## Project Overview

This is a professional Bloomberg Terminal-inspired React UI component library built with TypeScript, Tailwind CSS, and Turborepo. The library provides high-performance components optimized for financial trading applications with dark-first design, accessibility compliance, and enterprise-ready features.

## Architecture & Tech Stack

### **Monorepo Structure (Turborepo)**

```
@sandeep-jaiswar/ui/
├── packages/
│   ├── ui/                 # Core component library
│   ├── tokens/             # Design tokens package
│   ├── icons/              # Icon library
│   └── utils/              # Shared utilities
├── apps/
│   ├── docs/               # Storybook documentation
│   ├── playground/         # Development playground
│   └── examples/           # Usage examples
├── .github/
│   ├── workflows/          # CI/CD pipelines
│   └── copilot_instructions.md
└── tools/
    ├── eslint-config/      # Shared ESLint config
    └── tsconfig/           # Shared TypeScript configs
```

### **Technology Stack**

- **Framework**: React 18+ with TypeScript 5+
- **Styling**: Tailwind CSS with component classes
- **Build Tool**: Tsup for fast builds
- **Monorepo**: Turborepo for optimized builds
- **Testing**: Jest + React Testing Library
- **Documentation**: Storybook 7+
- **CI/CD**: GitHub Actions
- **Package Manager**: pnpm (required)

## Design System Principles

### **Bloomberg Terminal Aesthetic**

- **Dark-first design** with terminal black (#000000) primary background
- **High contrast colors** for optimal readability in trading environments
- **Monospace typography** for consistent financial data alignment
- **Professional interactions** with subtle animations and terminal-style glows

### **Color Palette**

```typescript
// Primary Colors
terminal-black: '#000000'        // Primary background
terminal-dark-gray: '#1a1a1a'    // Secondary surfaces
terminal-medium-gray: '#333333'   // Borders, dividers
terminal-light-gray: '#666666'    // Muted text, disabled states
terminal-white: '#ffffff'         // Primary text

// Financial Semantic Colors
terminal-blue: '#0068ff'          // Primary actions, focus states
terminal-green: '#4af6c3'         // Success, gains, buy actions
terminal-red: '#ff433d'           // Danger, losses, sell actions
terminal-orange: '#fb8b1e'        // Warnings, alerts
```

### **Typography System**

```typescript
// Font Families
font-terminal-mono: 'SF Mono', Monaco, Inconsolata, Consolas, monospace
font-terminal-sans: Inter, 'SF Pro Display', 'Helvetica Neue', sans-serif

// Size Scale (optimized for dense financial data)
text-xs: 10px / 14px    // Small labels, captions
text-sm: 12px / 16px    // Table data, secondary text
text-base: 14px / 20px  // Body text, buttons
text-lg: 16px / 22px    // Headings, important text
text-xl: 18px / 26px    // Large headings
```

### **Spacing System**

```typescript
// 4px base scale for precise layouts
spacing-1: 4px    spacing-6: 24px
spacing-2: 8px    spacing-8: 32px
spacing-3: 12px   spacing-10: 40px
spacing-4: 16px   spacing-12: 48px
spacing-5: 20px   spacing-16: 64px
```

## Component Development Guidelines

### **Component Structure**

Each component should follow this structure:

```
ComponentName/
├── index.ts                    # Public exports
├── ComponentName.tsx          # Main component
├── ComponentName.stories.tsx  # Storybook stories
├── ComponentName.test.tsx     # Unit tests
└── types.ts                   # TypeScript interfaces
```

### **Component Template**

```typescript
import React from 'react'
import { cn } from '@/utils/cn'
import { announceToScreenReader } from '@/utils/accessibility'

export interface ComponentProps {
  variant?: 'primary' | 'secondary' | 'danger' | 'success'
  size?: 'sm' | 'md' | 'lg'
  children?: React.ReactNode
  className?: string
  'data-testid'?: string
}

export const Component = React.forwardRef<HTMLElement, ComponentProps>(
  ({ variant = 'primary', size = 'md', className, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(
          'component-base',
          `component-${variant}`,
          `component-${size}`,
          className
        )}
        {...props}
      />
    )
  }
)

Component.displayName = 'Component'
```

### **CSS Classes Pattern**

Use Tailwind's `@apply` directive for component classes:

```css
@layer components {
  .component-base {
    @apply inline-flex items-center justify-center transition-all duration-150;
    @apply font-terminal-sans border-none cursor-pointer select-none;
  }

  .component-primary {
    @apply bg-terminal-blue text-terminal-white;
    @apply hover:bg-terminal-blue-dark hover:shadow-terminal-glow-blue;
  }
}
```

## Accessibility Requirements

### **WCAG 2.1 AA Compliance**

- Minimum 4.5:1 contrast ratio for normal text
- Minimum 3:1 contrast ratio for large text
- Focus indicators with 2px minimum thickness
- Screen reader announcements for dynamic content
- Keyboard navigation support

### **Financial-Specific A11y**

```typescript
// Price change announcements
announceFinancialChange("AAPL", 150.25, 148.8, "price");

// Trading alert announcements
announceTradingAlert("stop-loss", "Position closed at $145.30", "AAPL");

// Loading state announcements
announceToScreenReader("Processing order...", "polite");
```

## Testing Standards

### **Unit Tests** (Jest + RTL)

- **Coverage**: Minimum 90% line coverage
- **Accessibility**: Use jest-axe for automated a11y testing
- **User interactions**: Test keyboard navigation, mouse events
- **Props validation**: Test all prop combinations

### **Visual Testing** (Chromatic)

- **Component states**: All variants, sizes, states
- **Responsive behavior**: Mobile, tablet, desktop breakpoints
- **Dark theme**: All components in terminal theme

### **Integration Tests**

- **Form interactions**: Multi-component workflows
- **Real-time updates**: Data streaming, price changes
- **Trading scenarios**: Buy/sell workflows, portfolio updates

## Performance Requirements

### **Bundle Size**

- **Core library**: <200KB gzipped
- **Tree-shaking**: Components individually importable
- **Code splitting**: Async component loading support

### **Runtime Performance**

- **DataGrid**: Handle 10,000+ rows at 60fps
- **Real-time updates**: <16ms render time for price changes
- **Memory usage**: <100MB for typical trading dashboard

### **Loading Performance**

- **First paint**: <100ms for component rendering
- **Interactive**: <200ms for user interactions
- **Lighthouse score**: 90+ for performance

## Development Workflow

### **Branch Strategy**

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: Feature development
- `hotfix/*`: Critical fixes
- `release/*`: Release preparation

### **Commit Convention**

```
feat(button): add loading state with spinner
fix(datagrid): resolve virtualization scrolling issue
docs(readme): update installation instructions
test(badge): add accessibility tests
perf(table): optimize large dataset rendering
```

### **Pull Request Process**

1. Feature branch from `develop`
2. Implement component with tests and stories
3. Run local checks: `pnpm lint`, `pnpm test`, `pnpm build`
4. Create PR with component demo and breaking changes
5. Automated checks: tests, build, visual regression
6. Code review and approval required
7. Merge to `develop`, auto-deploy preview

### **Release Process**

1. Create release branch from `develop`
2. Update version and changelog
3. Final testing and validation
4. Merge to `main`
5. Automated: npm publish, GitHub release, docs deploy

## File Naming Conventions

### **Components**

- `PascalCase` for component files: `Button.tsx`, `DataGrid.tsx`
- `camelCase` for utilities: `formatCurrency.ts`, `announceToScreenReader.ts`
- `kebab-case` for config files: `tailwind.config.js`, `jest.config.js`

### **Imports**

```typescript
// Relative imports for same package
import { Button } from "./Button";

// Package imports
import { cn } from "@sandeep-jaiswar/ui/utils";
import { formatCurrency } from "@sandeep-jaiswar/ui/formatters";
```

## Documentation Standards

### **Component Documentation**

````typescript
/**
 * Professional button component for financial trading applications.
 *
 * Features Bloomberg Terminal-inspired styling with:
 * - High contrast colors optimized for terminal environments
 * - Financial semantic variants (success for buy, danger for sell)
 * - Screen reader announcements for trading actions
 * - Loading states with terminal-style spinners
 *
 * @example
 * ```tsx
 * <Button variant="success" onClick={handleBuyOrder} loading={isExecuting}>
 *   Buy 100 AAPL
 * </Button>
 * ```
 */
````

### **Storybook Stories**

- **Default**: Basic component usage
- **All Variants**: Show all visual variants
- **Interactive**: Demonstrate user interactions
- **Edge Cases**: Loading states, errors, empty states
- **Financial Examples**: Trading-specific use cases

## Code Quality Standards

### **TypeScript**

- Strict mode enabled
- No `any` types allowed
- Proper interface definitions
- Generic constraints where appropriate

### **ESLint Rules**

- React Hooks rules
- Accessibility linting (eslint-plugin-jsx-a11y)
- Import sorting
- Consistent formatting (Prettier)

### **Performance**

- Use `React.memo` for expensive components
- Implement `useMemo`/`useCallback` for heavy computations
- Prefer CSS animations over JavaScript
- Minimize re-renders with proper prop drilling

## Deployment & CI/CD

### **Automated Workflows**

- **PR Checks**: Tests, linting, build verification
- **Visual Regression**: Chromatic screenshot testing
- **Performance**: Bundle size monitoring
- **Security**: Dependency vulnerability scanning

### **Release Automation**

- **Version Bumping**: Semantic versioning (semver)
- **Changelog**: Auto-generated from commit messages
- **NPM Publishing**: Automated package publishing
- **Documentation**: Auto-deploy Storybook to Vercel

### **Quality Gates**

- All tests passing (unit + integration)
- 90%+ code coverage
- No linting errors
- Bundle size within limits
- Visual regression tests pass
- Accessibility audit pass

---

## Key Reminders for Copilot

1. **Always use our design tokens** from `tailwind.config.ts`
2. **Follow the component template** structure exactly
3. **Include accessibility features** in every component
4. **Write comprehensive tests** with good coverage
5. **Create meaningful Storybook stories** with financial examples
6. **Use our utility functions** (`cn`, `announceToScreenReader`)
7. **Follow Bloomberg Terminal aesthetic** consistently
8. **Optimize for performance** in financial data scenarios
9. **Document with JSDoc** and practical examples
10. **Consider screen readers** and keyboard navigation

This UI library should feel like a professional tool that major banks would confidently use in their trading applications. Every component should be enterprise-ready with proper error handling, accessibility, and performance optimization.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sandeep-jaiswar)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sandeep-jaiswar)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
