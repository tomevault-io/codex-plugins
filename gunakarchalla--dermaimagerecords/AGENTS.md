## Purpose

This file tells AI coding agents how this repository is organized, how to run and change it safely, and which project-specific conventions to follow so changes stay consistent and build correctly.

## Quick Start

- **Install:** `npm install` (see [package.json](package.json#L1-L40)).
- **Run (dev):** `npm start` -> `expo start` (uses `expo-router` file-based routing).
- **Build on Android (dev client):** `npm run android` (runs `expo run:android`, creates a dev client).
- **Lint:** `npm run lint`.

## Big-picture architecture

- This is an Expo + React Native app using `expo-router` (file-based routing). The app entry is configured in `package.json` as `expo-router/entry`.
- UI and pages live under the `app/` directory. Layouts and route stacks are created using special filenames like [app/\_layout.tsx](app/_layout.tsx#L1-L40).
- Static assets (images) are in `assets/images/`.
- Platform-specific Android project files exist under `android/` for native builds.

## Project-specific patterns and conventions

- Routing: Create new screens by adding files under `app/`. A page must default-export a React component (see [app/index.tsx](app/index.tsx#L1-L40)).
- Layouts: Use `_layout.tsx` to provide shared wrappers, navigation stacks, and global CSS import (see [app/\_layout.tsx](app/_layout.tsx#L1-L20)).
- Styling: `nativewind` (Tailwind for RN) is used. Keep utility classes in components and import `global.css` in the root layout.
- Images: Prefer `expo-image` API (present in dependencies) over the plain `Image` where app-specific functionality is needed.
- TypeScript: Files are `.tsx`/`.ts`. Follow existing typing patterns and keep `@types/react` aligned with the project's `package.json`.

## Build / dev workflows (explicit)

- Local dev (Expo Metro):

```bash
npm install
npm start
```

- Run on an Android device/emulator (development build / dev client):

```bash
npm run android
```

- Reset starter project (moves starter code to `app-example`):

```bash
npm run reset-project
```

## Files to reference when making changes

- App entry & sample screen: [app/index.tsx](app/index.tsx#L1-L40)
- Root layout (global CSS + router stack): [app/\_layout.tsx](app/_layout.tsx#L1-L20)
- Project scripts and deps: [package.json](package.json#L1-L120)
- Tailwind + Nativewind config: `tailwind.config.js` and `nativewind-env.d.ts` (follow their conventions when adding utilities)
- Android native integration: `android/` (only modify if you intend to change native modules or rebuild dev clients)

## Code style and safety

- Keep changes small and focused. This repository uses TypeScript and Expo; avoid changing the router entry (`expo-router/entry`) or moving to a different routing system.
- Run `npm run lint` before opening a PR.
- When adding native dependencies, prefer Expo-compatible libraries and update `android/` only when necessary; document native changes in the PR.

## Examples (how to add a route)

- Add a screen at `app/profile.tsx` that default-exports a component. The router will expose it automatically as `/profile`.
- Use `_layout.tsx` when you need a shared Stack or header across nested routes.

## Where to ask questions

- If behavior isn't clear from the files above, start by inspecting `app/_layout.tsx` and the `app/` directory structure. If still unclear, ask the repo owner or open an issue with a short code pointer.

## Notes for AI agents

- Prefer minimal diffs that preserve current patterns (file-based routing, nativewind classes, TypeScript types).
- Run the project's scripts locally where feasible to validate changes. If you cannot run a native build, still ensure TypeScript and lint pass locally (`npm run lint`).
- If adding or changing packages, update `package.json` and ensure `npm install` works; document reasons for dependency additions in PR description.

If this file should merge with an existing `.github/copilot-instructions.md`, keep its high-value sections and add any missing concrete commands or file links from above.

Please review — tell me any unclear sections or other file patterns you want included.

---
> Source: [gunakarchalla/DermaImageRecords](https://github.com/gunakarchalla/DermaImageRecords) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
