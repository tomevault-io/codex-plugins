# ShiftSync Japan – Copilot Instructions

## Dev Commands

```bash
npx expo start          # Start dev server (choose iOS/Android/Web from menu)
npx expo start --ios    # Launch directly on iOS simulator
npx expo start --android
```

No test suite or linter is configured. TypeScript checking is implicit via the editor/Expo bundler.

When adding dependencies, always use `npx expo install <package>` (not `npm install`) to get Expo-compatible versions.

## Architecture Overview

**Stack:** React Native (Expo Router v6) + NativeWind (Tailwind) + Supabase + TanStack Query v5.

### Navigation & Auth Flow (`app/_layout.tsx`)
The root layout handles all auth routing imperatively:
1. No session → redirect to `/` (login)
2. Session + no `organization_id` on profile → redirect to `/onboarding`
3. Session + org → redirect to `/(tabs)`

Deep links capture `orgId` param and persist it to AsyncStorage as `@pending_org_id` for the onboarding join flow.

### Data Layer Pattern
Prefer putting Supabase queries in custom hooks under `hooks/` instead of calling `supabase` directly from components. Some existing screens still call `supabase` directly (tracked in issue #9); when touching them, prefer using the existing hook or creating a new one to migrate the logic out.

- **Reads:** `useQuery` with descriptive `queryKey` arrays, e.g. `['schedule', { allUsers }]`
- **Writes:** `useMutation` + `queryClient.invalidateQueries` immediately after for optimistic-feel UX
- **Offline writes:** `useOfflineQueue` queues expense uploads in AsyncStorage when offline, retries on reconnect via NetInfo
- **Persistence:** `PersistQueryClientProvider` + `createAsyncStoragePersister` caches query results across app restarts (24h gcTime, 5min staleTime)

### Role System
`useUserRole()` from `hooks/useUserRole.ts` returns `{ role, loading, isAdmin }`. Role is fetched from `profiles.role` ('admin' | 'staff'). Admin-only features (create/edit/delete schedule, approve expenses, view all-staff data) gate on `isAdmin`.

### i18n
`useTranslation()` from `hooks/useTranslation.ts` returns `{ t, locale, changeLanguage }`. Language resolution order:
1. AsyncStorage (`user_language_preference`)
2. `profiles.preferred_language` in DB
3. `expo-localization` system locale fallback

All new UI strings must use `t('translation_key')`. Keys are typed via `TranslationKey` from `lib/translations.ts`. Add new keys to **both** `en` and `ja` objects. Avoid introducing new hardcoded strings; existing legacy hardcoded strings can be migrated to `t(...)` over time (tracked in issue #12).

### Feedback / Toasts
Prefer `useToast()` from `context/ToastContext.tsx` for transient feedback (success, error, info). Use `Alert.alert()` only for blocking confirmations (e.g. destructive actions like delete). Avoid `Alert.alert()` for simple error/success messages.
```ts
const { showToast } = useToast();
showToast("Saved!", "success"); // types: "success" | "error" | "info"
```

### Supabase Multi-Tenancy
All data is scoped to `organization_id`. Queries that fetch org-wide data first look up the current user's `organization_id` from `profiles`, then filter by it. RLS policies must also enforce this. When adding a new table, ensure it has an `organization_id` column with an RLS policy: `USING (organization_id = (SELECT organization_id FROM profiles WHERE id = auth.uid()))`.

### Edge Functions (`supabase/functions/`)
- `translate-message` – Translates chat messages via Groq, triggered by a DB trigger on message insert
- `push-notification` – Sends Expo push notifications using tokens stored in `profiles.expo_push_token`

## Key Conventions

### Styling
- NativeWind with Tailwind utility classes. Brand colors: `bg-brand-red` (`#D9381E`), `bg-brand-dark`, `bg-brand-gray`.
- Tablet responsiveness uses NativeWind breakpoints: `md:w-[23%]` etc. (`supportsTablet: true` is set in `app.json`).
- `global.css` imports Tailwind base; `nativewind-env.d.ts` provides type support.

### Types
All shared data models are in `types/index.ts`. Add new DB entities there. Supabase join results (e.g. `profiles?`) are typed as optional fields on the parent interface.

### Sentry
`app/_layout.tsx` wraps the root with `Sentry.wrap(Layout)`. DSN is read from `EXPO_PUBLIC_SENTRY_DSN`. In dev, Spotlight is enabled (`enableSpotlight: __DEV__`).

### Environment Variables
Prefix with `EXPO_PUBLIC_` for client-side access. Required: `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY`. Optional: `EXPO_PUBLIC_SENTRY_DSN`.

### Reanimated Compatibility
`react-native-reanimated` is pinned to `~4.1.1` (Expo 54 compatible). `react-native-worklets` must remain installed — it's required by the Reanimated v4 Babel plugin. Do not upgrade reanimated without running `npx expo install`.

---
> Source: [matiifernandez/ShiftSync-Japan](https://github.com/matiifernandez/ShiftSync-Japan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
