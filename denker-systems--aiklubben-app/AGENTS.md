# Project Structure Rules - React Native Expo

## Activation

- **Mode**: Always On
- **Description**: Directory structure and file organization standards

---

## Root Directory Structure

### Standard Project Layout

```
aiklubben-app/
├── .expo/                    # Expo configuration (gitignored)
├── .windsurf/
│   └── rules/               # Cascade rules
├── assets/                   # Static assets (images, fonts)
│   ├── images/
│   └── fonts/
├── docs/                     # Documentation
├── src/                      # Source code
├── .env                      # Environment variables (gitignored)
├── .env.example              # Environment template
├── .eslintrc.cjs             # ESLint config
├── .gitignore
├── .prettierrc               # Prettier config
├── App.tsx                   # Entry point
├── app.json                  # Expo config
├── babel.config.js           # Babel config
├── metro.config.js           # Metro bundler config
├── package.json
├── tailwind.config.js        # NativeWind config
└── tsconfig.json             # TypeScript config
```

---

## Source Directory Structure

### /src Organization

```
src/
├── components/               # Reusable components
│   ├── ui/                   # Base UI components
│   │   ├── Button.tsx
│   │   ├── Text.tsx
│   │   ├── Card.tsx
│   │   ├── Input.tsx
│   │   └── index.ts          # Barrel export
│   ├── layout/               # Layout components
│   │   ├── ScreenLayout.tsx
│   │   ├── Header.tsx
│   │   └── index.ts
│   └── shared/               # Shared feature components
│       ├── EmptyState.tsx
│       ├── ErrorState.tsx
│       ├── LoadingState.tsx
│       └── index.ts
├── config/                   # App configuration
│   ├── supabase.ts           # Supabase client
│   ├── theme.ts              # Theme constants
│   └── index.ts
├── constants/                # App constants
│   ├── colors.ts
│   ├── spacing.ts
│   └── index.ts
├── contexts/                 # React Context providers
│   ├── AuthContext.tsx
│   ├── ThemeContext.tsx
│   └── index.ts
├── hooks/                    # Custom hooks
│   ├── useAuth.ts
│   ├── useFetch.ts
│   └── index.ts
├── lib/                      # Utilities and helpers
│   ├── api/                  # API client
│   ├── utils/                # Utility functions
│   ├── validation/           # Validation schemas
│   └── animations.ts         # Animation configs
├── navigation/               # Navigation setup
│   ├── AppNavigator.tsx
│   ├── AuthNavigator.tsx
│   └── types.ts
├── screens/                  # Screen components
│   ├── auth/                 # Auth screens
│   │   ├── LoginScreen.tsx
│   │   └── RegisterScreen.tsx
│   ├── courses/              # Course feature
│   │   ├── components/       # Feature-specific components
│   │   ├── hooks/            # Feature-specific hooks
│   │   ├── CourseListScreen.tsx
│   │   └── CourseDetailScreen.tsx
│   ├── lessons/              # Lesson feature
│   │   ├── components/
│   │   ├── steps/            # Lesson step components
│   │   └── LessonScreen.tsx
│   └── profile/              # Profile feature
│       └── ProfileScreen.tsx
├── services/                 # External service integrations
│   ├── auth.ts
│   └── api.ts
└── types/                    # TypeScript types
    ├── api.ts
    ├── navigation.ts
    └── index.ts
```

---

## File Naming Conventions

### Component Files

```
PascalCase.tsx              # React components
├── Button.tsx
├── UserCard.tsx
├── CourseDetailScreen.tsx
└── LessonPath.tsx
```

### Hook Files

```
camelCase.ts                # Custom hooks (with use prefix)
├── useAuth.ts
├── useFetch.ts
├── useCourses.ts
└── useForm.ts
```

### Utility Files

```
camelCase.ts                # Utilities and helpers
├── formatDate.ts
├── validation.ts
├── storage.ts
└── helpers.ts
```

### Type Files

```
camelCase.ts                # Type definitions
├── user.ts
├── course.ts
├── navigation.ts
└── api.ts
```

### Constant Files

```
camelCase.ts                # Constants
├── colors.ts
├── spacing.ts
├── routes.ts
└── config.ts
```

---

## Component Organization

### Feature-Based Structure

```
screens/
└── courses/
    ├── components/           # Feature-specific components
    │   ├── CourseCard.tsx
    │   ├── LessonNode.tsx
    │   ├── LessonPath.tsx
    │   └── index.ts
    ├── hooks/                # Feature-specific hooks
    │   ├── useCourse.ts
    │   ├── useLessons.ts
    │   └── index.ts
    ├── CourseListScreen.tsx
    ├── CourseDetailScreen.tsx
    └── index.ts              # Screen exports
```

### Shared vs Feature Components

```typescript
// SHARED: Used across multiple features
// Location: src/components/ui/ or src/components/shared/
// Examples: Button, Card, Text, LoadingState, EmptyState

// FEATURE: Used only within one feature
// Location: src/screens/[feature]/components/
// Examples: LessonNode, CourseCard, ProfileAvatar
```

---

## Import Aliases

### tsconfig.json Paths

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/screens/*": ["src/screens/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/lib/*": ["src/lib/*"],
      "@/types/*": ["src/types/*"],
      "@/contexts/*": ["src/contexts/*"],
      "@/config/*": ["src/config/*"],
      "@/constants/*": ["src/constants/*"],
      "@/services/*": ["src/services/*"],
      "@/navigation/*": ["src/navigation/*"]
    }
  }
}
```

### Import Order

```typescript
// 1. React
import React, { useState, useEffect } from 'react';

// 2. React Native
import { View, Text, StyleSheet } from 'react-native';

// 3. Third-party (alphabetical)
import { MotiView } from 'moti';
import * as Haptics from 'expo-haptics';

// 4. Navigation
import { useNavigation } from '@react-navigation/native';

// 5. Local - Components
import { Button, Card } from '@/components/ui';

// 6. Local - Hooks
import { useAuth, useCourse } from '@/hooks';

// 7. Local - Utils
import { formatDate } from '@/lib/utils';

// 8. Local - Types
import type { Course, User } from '@/types';

// 9. Local - Constants
import { COLORS } from '@/constants';

// 10. Relative imports (same feature)
import { LessonNode } from './components';
```

---

## Barrel Exports

### Index File Pattern

```typescript
// components/ui/index.ts
export { Button } from './Button';
export { Text } from './Text';
export { Card } from './Card';
export { Input } from './Input';
export { Badge } from './Badge';

// Re-export types if needed
export type { ButtonProps } from './Button';
export type { TextVariant } from './Text';
```

### When to Use Barrel Exports

```typescript
// USE barrel exports for:
// - UI components (src/components/ui/index.ts)
// - Shared components (src/components/shared/index.ts)
// - Hooks (src/hooks/index.ts)
// - Feature component folders

// DON'T use barrel exports for:
// - Screens (import directly)
// - Large utility collections (may cause bundle bloat)
// - Circular dependency risks
```

---

## Configuration Files

### Environment Variables

```bash
# .env.example - Template for environment variables
EXPO_PUBLIC_SUPABASE_URL=your_supabase_url
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
EXPO_PUBLIC_API_URL=https://api.example.com
```

### App Configuration

```typescript
// src/config/index.ts
export const config = {
  supabase: {
    url: process.env.EXPO_PUBLIC_SUPABASE_URL!,
    anonKey: process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!,
  },
  api: {
    baseUrl: process.env.EXPO_PUBLIC_API_URL!,
    timeout: 10000,
  },
  app: {
    name: 'AI Klubben',
    version: '1.0.0',
  },
} as const;
```

---

## Documentation Structure

### /docs Organization

```
docs/
├── README.md                 # Project overview
├── architecture/
│   ├── OVERVIEW.md           # Architecture overview
│   └── TECH_STACK.md         # Tech stack details
├── contributing/
│   └── CONTRIBUTING.md       # Contribution guidelines
└── development/
    └── WORKFLOW.md           # Development workflow
```

---

## Forbidden Structure Practices

1. **NEVER** put components directly in src/ root
2. **NEVER** mix component and utility files in same folder
3. **NEVER** use relative imports for shared modules
4. **NEVER** create deeply nested folder structures (max 4 levels)
5. **NEVER** put screens outside of src/screens/
6. **NEVER** skip barrel exports for component folders
7. **NEVER** hardcode paths without using aliases
8. **NEVER** put business logic in component files

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-26 -->
