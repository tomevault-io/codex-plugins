## Copilot Chat rules (read first)

These rules are binding for all Copilot Chat responses in this repo. Default to brevity and only expand when asked.

- After completing tasks, your feedback/response should be concise, summarize what you did in 1-4 sentences.

## Project Overview

QuickRite helps freelancers write better proposals for job postings on platforms like Upwork, Contra, and Fiverr. The core idea of quikRite is to help uers write better proposals by including relevant projects/case-studies, testimonials(social proof).

## Technology Stack

- **Framework**: Next.js 15 (App Router, React Server Components)
- **UI**: Shadcn UI + Tailwind CSS v4
- **Database**: PostgreSQL with Drizzle ORM
- **AI**: Vercel AI SDK with OpenAI
- **Auth**: BetterAuth (with magic links)
- **Forms**: React Hook Form + Zod validation
- **State**: TanStack Query for server state
- **Linting**: Biome (replaces ESLint/Prettier)

## Architecture & Data Flow

### Three-Layer Pattern (Critical)
The codebase follows a strict separation:

1. **Server Actions** (`src/actions/*-actions.ts`): All mutations go through server actions marked with `"use server"`. Always validate auth, ownership, and input with Zod schemas. Return structured responses: `{ data, error, success }`.

2. **Database Operations** (`src/lib/db/operations/*.ts`): Pure database logic with Drizzle ORM. No auth checks here—those belong in actions. Functions return typed entities or `PaginatedResult<T>`.

3. **Components**: Client components (`"use client"`) use TanStack Query hooks (`src/hooks/`) for data fetching and mutations. Server components fetch data directly from operations layer.

**Example**:
```typescript
// src/actions/project-actions.ts
export const createProject = async (data: NewProject) => {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session?.user) return { data: null, error: "Unauthorized", success: false };
  
  const project = await createProjectDb(data);
  return { data: project, error: null, success: true };
};
```

### AI Chat Architecture
- **Route**: `src/app/api/chat/route.ts` handles streaming with Vercel AI SDK
- **Prompts**: `src/lib/ai/prompts.ts` contains system prompts with tone variations
- **Tools**: `src/lib/ai/tools/*.ts` define AI function calling (e.g., `getTemplates`, `saveProposals`)
- **Conversations**: Stored in PostgreSQL with messages as JSON. Use `useChat` hook from `@ai-sdk/react` for UI.

**Tool Pattern**:
```typescript
export const getTemplates = (user: User) =>
  tool({
    description: "Retrieve the user's proposal templates.",
    inputSchema: z.object({}),
    execute: async () => getTemplatesByUserId(user.id),
  });
```

## Database & Schema Conventions

- **Schemas**: `src/lib/db/schema/*.ts` (one file per entity)
- **Export all**: Through `src/lib/db/schema/index.ts`
- **Timestamps**: All tables have `createdAt` and `updatedAt`
- **IDs**: Use UUIDs via `.defaultRandom()`
- **Relations**: Define in `schema/relations.ts` for Drizzle queries
- **Rich Text**: Store as HTML text in `text()` columns (from Tiptap editor)

**Migration workflow**:
```bash
npm run db:generate  # Generate migration from schema changes
npm run db:migrate   # Run migrations
npm run db:studio    # Open Drizzle Studio GUI
```

## UI/UX & Component Guidelines

- **Responsive by default**: Test mobile, tablet, desktop breakpoints
- **Async feedback**: Use `useTransition` for server actions, show loading states
- **Forms**: Always use Shadcn form components with `react-hook-form` + Zod
- **Data tables**: Use `@tanstack/react-table` (see `src/components/proposals/columns.tsx`)
- **Analytics**: Use Recharts for charts/stats
- **Error handling**: Use `toast` from `sonner` for user feedback

**Server Action Pattern in Components**:
```typescript
const [isPending, startTransition] = useTransition();

const onSubmit = (data) => {
  startTransition(async () => {
    const result = await createProject(data);
    if (result.success) {
      toast.success("Project created");
      queryClient.invalidateQueries(queryKeys.projects.all);
    } else {
      toast.error(result.error);
    }
  });
};
```

## Development Workflows

**Start dev server**:
```bash
npm run dev  # Uses Turbopack for fast HMR
```

**Code quality**:
```bash
npm run lint    # Check with Biome
npm run format  # Auto-fix with Biome
```

**Database**:
```bash
npm run db:push    # Push schema without migrations (dev only)
npm run db:seed    # Seed database (see src/lib/db/seed.ts)
```

## Coding Standards

- **No broad refactors**: Keep changes focused and small
- **TypeScript strict**: All new files must be `.ts` or `.tsx`
- **Imports**: Use `@/` alias for absolute imports from `src/`
- **Naming**: Descriptive names over comments (only comment "why", not "what")
- **Async/await**: No raw promises or `.then()` chains
- **Error handling**: Use custom `ChatSDKError` class for API errors (see `src/lib/error.ts`)
- **Query keys**: Centralized in `src/lib/query-keys.ts` for cache management

## Project-Specific Patterns

### Authentication
- BetterAuth config: `src/lib/auth.ts` with Drizzle adapter
- Client auth: `src/lib/auth-client.ts` for browser usage
- Session checks: `await auth.api.getSession({ headers: await headers() })`
- Magic links in dev log to console, production uses Resend (currently commented out)

### Rich Text Editing
- Tiptap editor for proposals/templates (see `src/components/chats/message-editor.tsx`)
- Store as HTML strings in database
- Custom extensions loaded: highlight, link, image, typography

### Pagination
- Standard: `PaginationParams { page, pageSize }` and `PaginatedResult<T>`
- Utility: `src/lib/db/operations/util.ts` for offset calculation and total pages

### Type Safety
- Generate Zod schemas from Drizzle: `createInsertSchema`, `createUpdateSchema`
- Custom types: `src/lib/types.ts` for AI messages, tools, and shared interfaces

---
> Source: [Rd-Trends/ai-proposals](https://github.com/Rd-Trends/ai-proposals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-30 -->
