# Copilot Instructions

## Project Overview

This is an Expo Router + React Native app demonstrating NativeWind v4 dark mode implementation with manual theme toggling. It's designed as a companion to a Medium article showing practical dark mode patterns.

## Architecture & Key Patterns

### NativeWind Dark Mode Setup

- **Critical**: Dark mode uses `darkMode: 'class'` in `tailwind.config.js` for manual toggling (not system preference)
- Import `global.css` in `_layout.tsx` and each screen that uses Tailwind classes
- Use `useColorScheme()` from `nativewind` (not React Native's built-in hook) for theme control
- Pattern: `className="bg-white dark:bg-neutral-900"` for theme-aware styling

### File Structure Conventions

- `app/_layout.tsx`: Root layout with Stack navigation, imports global CSS
- `app/index.tsx` & `app/about.tsx`: Screen components following consistent dark mode pattern
- All screens import `'../global.css'` at the top (required for NativeWind)

### Development Workflow

- **Start**: `npm start` (Expo development server)
- **Platform-specific**: `npm run android`, `npm run ios`, `npm run web`
- **Metro config**: Uses `withNativeWind(config, { input: './global.css' })` wrapper

### Dark Mode Implementation Pattern

```tsx
const { colorScheme, toggleColorScheme } = useColorScheme();
const isDarkMode = colorScheme === 'dark';

// StatusBar adapts to theme
<StatusBar style={isDarkMode ? 'light' : 'dark'} />

// Switch component toggles theme
<Switch value={isDarkMode} onValueChange={handleSwitchChange} />
```

### Configuration Dependencies

- `babel.config.js`: Must include `'nativewind/babel'` preset and `jsxImportSource: 'nativewind'`
- `nativewind-env.d.ts`: Required TypeScript reference for NativeWind types
- `app.json`: Set `"userInterfaceStyle": "automatic"` to respect system theme initially

## Key Integration Points

- Expo Router for file-based navigation (`expo-router/entry` as main entry point)
- NativeWind v4 with Tailwind CSS classes in React Native components
- Expo StatusBar component that adapts to current theme state
- Manual theme switching via Switch component (not automatic system detection)

When adding new screens, always import global CSS and follow the established dark mode class pattern with neutral color scheme.

---
> Source: [carljohan-schnell/sso-login](https://github.com/carljohan-schnell/sso-login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
