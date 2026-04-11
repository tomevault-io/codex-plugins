<!-- Copilot instructions for the `contas-residenciais` Angular app -->
# Quick guide for AI coding agents

This repository is a lightweight Angular 21 standalone-components app (no NgModule). The goal of these notes is to help an AI agent be productive quickly with edits, routing, and component creation.

- **Entrypoint & bootstrap**: Application is bootstrapped with `bootstrapApplication` in `src/main.ts`. Configuration is provided by `src/app/app.config.ts` (uses `provideRouter` and `provideBrowserGlobalErrorListeners`).

- **Routing & lazy components**: Routes are declared in `src/app/app.routes.ts`. Routes use the `loadComponent` lazy-loading pattern, e.g.:

  ```ts
  { path: 'login', loadComponent: () => import('../app/pages/login/login').then(m => m.Login) }
  ```

  - The authenticated area is behind the `authGuard` and mounted under path `app`. See `src/app/core/auth/auth.guard.ts` for the guard signature (exported `authGuard`).
  - Add new pages by creating `src/app/pages/<name>/<name>.ts` and adding a `loadComponent` entry to `app.routes.ts`.

- **Layout & shell**: The main protected layout is `src/app/core/layout/shell/shell.ts` (loaded as a route component). The app uses child routes to populate the shell.

- **Component style & conventions**:
  - Components are standalone classes exported directly (no NgModule). Use `templateUrl`/`styleUrl` (SCSS) same as existing files.
  - Keep styles in SCSS; default global styles live in `src/styles.scss`.

- **Dev / build / test commands** (from `package.json`):
  - `npm start` → runs `ng serve` (development server)
  - `npm run build` → `ng build` (production build)
  - `npm run watch` → `ng build --watch --configuration development`
  - `npm test` → `ng test` (uses the Angular builder + Vitest configured via devDependencies)

- **Project layout highlights** (use these paths when making changes):
  - `src/main.ts` — app bootstrap
  - `src/app/app.config.ts` — application providers
  - `src/app/app.routes.ts` — routing + lazy component examples
  - `src/app/core/auth/auth.guard.ts` — auth guard implementation
  - `src/app/core/layout/shell/shell.ts` — shell layout for protected routes
  - `src/app/pages/*` — page components (login, register, dashboard, itens, relatorio-*)

- **Common patterns to follow when editing/adding code**:
  - Prefer `loadComponent` lazy import style for routes to match repo conventions.
  - Export component classes using the same naming pattern used by peers (e.g., `export class Login {}`) and file name parity `<name>.ts`.
  - Do not introduce NgModule usage—this app leverages standalone components and `ApplicationConfig`.

- **Testing & quick verification**:
  - After code changes, run `npm start` to verify dev server runs and navigation works.
  - Use `npm test` to run unit tests; if adding tests, follow Vitest conventions used by the Angular test builder.

- **Edge cases / gotchas**:
  - Routes expect the exported component symbol name to match the import `.then(m => m.Name)` expression — preserve exported symbol names.
  - The `authGuard` is used as a function provider; match the existing guard signature when modifying it.

If anything above is unclear or you want examples for creating a new page or guard, say which file(s) you'd like and I will add a short, copy-pasteable snippet.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alebarre)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alebarre)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
