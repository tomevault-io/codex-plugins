# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

House expense tracker — a React Native (Expo SDK 54) app with file-based routing via expo-router. Targets iOS, Android, and web. Dark theme throughout (`#121212` background, `#1E1E1E` surfaces, `#BB86FC` primary accent).

## Commands

```bash
npm install              # Install dependencies
npx expo start           # Start dev server (all platforms)
npx expo start --web     # Web only
npx expo lint            # Run ESLint
npx tsc --noEmit         # Type-check (ignore app-example/ errors — that's template boilerplate)
```

No test framework is configured.

## Architecture

### Routing
- `src/app/_layout.tsx` — Root layout wraps everything in `PaperProvider` with dark theme, renders `<Slot />`
- `src/app/(tabs)/_layout.tsx` — Tab navigator (Expenses + Reports tabs)
- `src/app/(tabs)/index.tsx` — Main expenses list page
- `src/app/(tabs)/reports.tsx` — Category charts with monthly filter

### Data Layer
- `src/server/constants.ts` — API base URL (`http://tracker.local/api/v1`)
- `src/server/expense/ExpenseService.ts` — Axios-based service class with CRUD methods
- `src/server/expense/Expense.ts` — Core data model: `id, amount, date, description, category, recipient, currency ('EUR'|'RON')`
- `src/server/expense/ExpensesWithMeta.ts` — Response wrapper with expenses array + currency sums
- No state management library — local `useState` in page components, service instantiated per-file

### Components (`src/components/`)
All components are plain React Native (no external component library except `react-native-paper` for `DatePickerModal` and `PaperProvider` theme). Modals use the `Modal + Pressable overlay + Pressable dialog` pattern to prevent click-through on web.

### Key Conventions
- Path alias: `@/*` maps to project root (use `@/src/...` for imports)
- Currency is always `'EUR' | 'RON'`
- Dark theme colors: background `#121212`, surface `#1E1E1E`, border `#333`, text `#E0E0E0`, muted `#9E9E9E`, primary `#BB86FC`, error `#CF6679`
- Responsive: `useWindowDimensions()` with `width >= 768` breakpoint for wide layouts
- Charts use `react-native-gifted-charts` (PieChart, BarChart)
- Date picker uses `react-native-paper-dates` with German locale (`de`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emung)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/emung)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
