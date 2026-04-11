## Quick orientation

This React + Vite + TypeScript app uses shadcn-ui components and Tailwind CSS. Use pnpm (see `package.json: packageManager`).

- Dev: `pnpm i` then `pnpm run dev`
- Build: `pnpm run build`
- Preview: `pnpm run preview`
- Lint: `pnpm run lint`

Key entry points and folders
- `src/main.tsx` — app bootstrap (renders `<App />`).
- `src/App.tsx` — routes and providers (QueryClientProvider, TooltipProvider, AuthProvider).
- `src/components/ui/` — shadcn-ui components (imported as `@/components/ui/*`).
- `vite.config.ts` — alias `@` => `./src` (use this when resolving imports).

Auth and data flow (important to know)
- The app uses a localStorage-based pseudo-backend. Relevant keys:
  - `users`, `userRecords` — seeded user lists (Admin with msnv `1118` is protected).
  - `currentUserSession`, `sessionExpiry` — active session storage.
  - `rememberLogin`, `rememberedMsnv`, `rememberedUser` — remember-me support.
- Authentication is implemented in `src/contexts/AuthContext.tsx`:
  - `login(msnv, password, rememberMe)` validates against `userRecords` and sets `currentUser` from `users`.
  - `syncUserData()` ensures admin record exists and keeps `users`/`userRecords` in sync.
  - `refreshUserData()` reloads the user from localStorage when user data changes.

Routing & authorization patterns
- Routes are defined in `src/App.tsx`. Use `ProtectedRoute` for guarded pages.
- `src/components/ProtectedRoute.tsx` enforces:
  - Authentication (redirects to `/login` if not authenticated).
  - Role hierarchy checks (Admin > Duyệt/manager > User). See `checkRolePermission` in file.
  - Module-level permissions: `requiredModule` checks `currentUser.permissions[module].view`.

UI/component conventions
- Components from shadcn live at `src/components/ui/*` and are imported via `@/components/ui/...`.
- Use these UI primitives instead of recreating components (Button, Input, Card, Toaster, etc.).

Testing / debugging hints
- There are no automated tests in the repo; debug by running `pnpm run dev` and using browser console logs.
- Many components and auth functions log detailed messages (look for `console.log` in `AuthContext` and `ProtectedRoute`).

Project-specific conventions
- LocalStorage is treated as the source-of-truth for user and permission management — changing files that touch `users`/`userRecords` needs careful syncing.
- The project seeds a protected admin account (`msnv: 1118`) — do not remove or assume it can be deleted.

Where to look first when adding features
- For auth/permissions: `src/contexts/AuthContext.tsx`, `src/components/ProtectedRoute.tsx`, `src/pages/LoginPage.tsx`.
- For new UI pages: copy patterns from `src/pages/Index.tsx` and use components in `src/components/ui`.
- Data-heavy pages use React Query: see `QueryClientProvider` usage in `src/App.tsx`.

Small examples to copy
- Route that requires manager role: see `/quan-ly-danh-muc` in `src/App.tsx` (uses `requiredRole="manager"`).
- Module permission example: `/nhap-kho` uses `requiredModule="kho-tong"`.

If you update dependencies
- Use `pnpm add <pkg>` and avoid changing the package manager. After adding, run `pnpm i` and smoke-test `pnpm run dev`.

If anything is unclear, tell me which area (auth, routing, UI primitives, or build) and I'll expand examples or add snippets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/truongson501190-sys)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/truongson501190-sys)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
