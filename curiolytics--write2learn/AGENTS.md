# Write2Learn - AI Coding Agent Instructions

## Architecture Overview

Write2Learn is a Next.js 15 (App Router) app for English writing improvement through journaling, vocabulary, and roleplay. Built with TypeScript, Supabase, and Tailwind CSS.

## Coding style
Delete all code that unused, and unneccasry. For example when adjusting existing code, delete the old code instead of commenting it out, never turn them into fallbacks or leave them as dead code.

### Key Service Patterns
- **Feature-first organization**: Code organized by domain (`auth/`, `journal/`, `vocab/`) not by tech layer
- **Singleton services**: Each service exports a single instance (`vocabularyService`, `journalService`)
- **Service boundaries**: Services handle API/DB, stores manage state, hooks contain component logic

### Authentication & Middleware
- Uses `middleware.ts` with session caching to redirect unauthenticated users
- `AuthProvider` context wraps app with user state and onboarding status
- API routes use `authenticateUser()` from `utils/api-helpers.ts` for auth validation
- Protected routes redirect to `/onboarding` if `profiles.onboarding_completed` is false

### API Route Patterns
All API routes follow this structure:
```typescript
export async function POST(request: Request) {
  try {
    const user = await authenticateUser();
    const { field } = await parseRequestBody<Type>(request);
    const result = await service.method(user.id, data);
    return createSuccessResponse(result, 'Success message');
  } catch (error) {
    return handleApiError(error);
  }
}
```

### Database Schema (Supabase)
- `profiles` - user info with `onboarding_completed` flag
- `journals` - entries with optional template linking
- `vocabulary` - user vocab with FSRS spaced repetition
- `journal_templates` - reusable writing prompts
- Auto-generated types in `types/database.types.ts`

### UI Components
- Radix UI + Tailwind with custom variants using `class-variance-authority`
- `components/ui/` for reusable primitives (Button, Dialog, etc.)
- Feature components in `components/{feature}/`
- Use `cn()` utility from `utils/ui.ts` for conditional classes

### State Management
- Zustand stores in `stores/` (auth, user profile)
- React Context for auth state (`AuthProvider`)
- Server state via direct API calls in services, not React Query

### Development Commands
- `npm run dev` - start dev server with Turbopack
- `npm run build` - production build with Turbopack
- `npm run run:fsrs` - run FSRS spaced repetition tests

### Critical Patterns
1. **Service instantiation**: Always export singleton instances (`export const serviceName = new ServiceClass()`)
2. **File naming**: Use kebab-case for components, PascalCase for TypeScript types
3. **Error handling**: API routes must use `handleApiError()`, never throw raw errors
4. **Auth flow**: Check `onboarding_completed` in middleware, redirect accordingly
5. **Markdown editing**: Uses custom markdown editor with task list support, not TipTap for journal content

### Integration Points
- Supabase client creation via `@supabase/auth-helpers-nextjs` 
SUPABASE_PROJECT_ID=eqhldzwiymtcyxyxezos

- FSRS library (`ts-fsrs`) for vocabulary spaced repetition
- Dynamic imports for client-only components (markdown editor)

When implementing new features, follow the established service → API route → component pattern and maintain the feature-based directory structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CurioLytics)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/CurioLytics)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
