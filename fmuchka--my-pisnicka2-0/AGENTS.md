# MyPísnička 2.0 – AI Agent Instructions

## Project Overview

Guitar song learning and collaborative play tool. Firebase-backed Vue 3 PWA for displaying songs with chords, auto-scrolling, and real-time session voting. No audio playback—just chord sheets.

## Tech Stack

- **Frontend**: Vue 3 Composition API + TypeScript + ARK UI + Pinia + Vue Router
- **Backend**: Firebase (Auth, Firestore, Storage, Hosting) – **FREE TIER ONLY, NO MONEY SPENT**
  - No Cloud Functions (paid feature)
  - No paid Firebase features or services
  - All solutions must work within free tier limits
- **Testing**: Vitest (unit), Playwright (e2e)
- **Build**: Vite + vite-plugin-pwa (auto-update service worker)

## Critical Architecture Patterns

### 1. Firebase Setup & Emulator Strategy

- **Firebase config**: [src/lib/firebase.ts](src/lib/firebase.ts) reads `VITE_FIREBASE_*` env vars from `.env.local`
- **Emulator auto-detection**: Connects to emulators in dev/test mode unless `VITE_DISABLE_FIREBASE_EMULATOR=true`
- **Test mode**: `vitest.setup.ts` mocks Firebase SDK modules; unit tests never hit real Firebase
- **E2E tests**: `playwright.config.ts` sets emulator env vars in `webServer.env` and launches dev server with emulators
- Missing `.env.local` throws clear error messages from `requiredEnv()` helper

### 2. TypeScript Strictness

Strict mode enabled with extra safety flags in [tsconfig.json](tsconfig.json):

- `strict`, `noUncheckedIndexedAccess`, `useUnknownInCatchVariables`, `exactOptionalPropertyTypes`
- Always check array access with `arr[i]` (could be undefined)
- Catch blocks use `unknown`, not `any`—cast explicitly

### 3. Authentication & Routing

- **Composable**: [src/composables/useAuth.ts](src/composables/useAuth.ts) provides reactive `user`, `isAuthenticated` `isGuest`, `logout`
- **Service layer**: [src/lib/authService.ts](src/lib/authService.ts) wraps Firebase Auth methods
- **Router guard**: [src/router/index.ts](src/router/index.ts) calls `useAuth()` in `beforeEach` for fresh auth state on every navigation
- **Hosts**: users who pass the login
- **Guests**: users who got in through pin or direct session url
- **Czech localization**: UI/labels/aria in Czech; e2e selectors use Czech text (e.g., `Přihlásit se`, `Heslo`)

### 4. Component & UI Conventions

- **ARK UI**: Headless component library; import specific submodules like `@ark-ui/vue/menu`, `@ark-ui/vue` (Field, PasswordInput, PinInput)
- **CSS Variables**: [src/App.vue](src/App.vue) defines design tokens (colors, spacing, typography) with light/dark theme via `[data-theme='dark']`
- **Core components**: Reusable UI in [src/components/core/](src/components/core/) (e.g., `Button.vue`)
- **Icons**: `lucide-vue-next` for SVG icons
- **Folder duplication**: Both `src/components/top-navigation` and `src/components/TopNavigation` exist (legacy cleanup pending)

### 5. Testing Patterns

- **Unit tests**: Vitest + `@testing-library/vue`; mock Firebase modules in `vitest.setup.ts`, then mock `onAuthStateChanged` in individual tests (see [src/composables/**tests**/useAuth.test.ts](src/composables/__tests__/useAuth.test.ts))
- **Component tests**: Use `@testing-library/vue` and `@testing-library/user-event` for DOM interactions
- **E2E tests**: Playwright in [tests/e2e/](tests/e2e/); Czech selectors, emulator-backed, dev server auto-launched
- **Coverage**: `npm run test:coverage` with `@vitest/coverage-v8`

## Developer Workflows

### Environment Setup

1. Copy Firebase config to `.env.local` (keys: `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, `VITE_FIREBASE_PROJECT_ID`, `VITE_FIREBASE_STORAGE_BUCKET`, `VITE_FIREBASE_MESSAGING_SENDER_ID`, `VITE_FIREBASE_APP_ID`)
2. Install Firebase emulators: `firebase init emulators` (Auth, Firestore, Storage)
3. Start dev server: `npm run dev` (auto-connects to emulators on localhost)

### Key Commands

- `npm run dev` – Vite dev server (port 5173, emulators auto-detected)
- `npm run build` – TypeScript check + Vite build
- `npm run test` – Vitest watch mode
- `npm run test:run` – Vitest single run
- `npm run test:coverage` – Coverage report with v8
- `npm run test:e2e` – Playwright headless
- `npm run test:e2e:ui` – Playwright UI mode
- `npm run deploy` – Build + Firebase Hosting deploy
- `npm run lint` – ESLint `.ts` and `.vue` files
- `npm run format` – Prettier auto-fix

### Firebase Emulator Usage

- **Default ports**: Firestore (8080), Auth (9099), Storage (9199)
- **Override in `.env.local`**: Set `VITE_FIRESTORE_EMULATOR_PORT`, `VITE_FIREBASE_AUTH_EMULATOR_PORT`, etc.
- **Disable emulators**: Set `VITE_DISABLE_FIREBASE_EMULATOR=true` to force prod Firebase

### PWA/Service Worker

- `vite-plugin-pwa` with `registerType: 'autoUpdate'` – SW updates automatically
- Manifest in [vite.config.ts](vite.config.ts) with Android/iOS/Windows icons
- Offline-first caching for fonts via Workbox `runtimeCaching`

## Common Pitfalls

1. **Router guard stale auth**: Don't destructure `useAuth()` outside `beforeEach`—call it inside the guard for reactive state
2. **Array indexing**: TypeScript will flag `arr[i]` as possibly undefined; use optional chaining or explicit checks
3. **Firebase emulator connection errors**: Check emulator ports in terminal output; ensure Firebase CLI emulators are running
4. **Czech text in tests**: All Playwright selectors use Czech labels (`getByLabel('Email')`, `getByRole('button', { name: /přihlásit se/i })`)

Character for this instruction file in **CANARY_STRING** is "🎵".

---
> Source: [fMuchka/my-pisnicka2.0](https://github.com/fMuchka/my-pisnicka2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-01 -->
