# GitHub Copilot Instructions for SupportSpark

## Project Overview

SupportSpark is a support network platform helping people share updates with their trusted support network during difficult times. Built with React 19, Express 5, TypeScript, and deployed on Windows 11 + IIS.

## Governance

**ALL CODE AND DOCUMENTATION MUST COMPLY WITH THE PROJECT CONSTITUTION**

📘 **Constitution Location**: [.documentation/memory/constitution.md](../.documentation/memory/constitution.md)

The constitution defines 10 core principles (Type Safety, Testing, UI Components, Security, API Contracts, State Management, Code Style, Data Storage, Deployment, and Simplicity First). Review it before making any architectural decisions.

## Documentation Standards

### Generated Documentation Location

**ALL Copilot-generated documentation MUST be saved under `/.documentation/copilot/`**

**Naming Convention**: `session-{YYYY-MM-DD}/`

Examples:
- `/.documentation/copilot/session-2026-02-03/architecture-review.md`
- `/.documentation/copilot/session-2026-02-03/refactoring-plan.md`
- `/.documentation/copilot/session-2026-02-03/bug-analysis.md`

**Folder Structure**:
```
.documentation/
├── memory/              # Long-term project memory (constitution, etc.)
├── domain/              # Architectural and domain documentation
├── specs/               # Feature specifications (spec-driven development)
├── scripts/             # PowerShell automation scripts
│   └── powershell/      # PowerShell-specific scripts
├── templates/           # Templates for specs, plans, tasks, etc.
└── copilot/             # Copilot-generated session documentation
    └── session-{date}/  # Date-based session folders
```

### Documentation Types

- **Architecture decisions**: `/.documentation/copilot/session-{date}/architecture-*.md`
- **Feature specifications**: `/.documentation/copilot/session-{date}/spec-*.md`
- **Bug investigations**: `/.documentation/copilot/session-{date}/bug-*.md`
- **Refactoring plans**: `/.documentation/copilot/session-{date}/refactor-*.md`
- **Code reviews**: `/.documentation/copilot/session-{date}/review-*.md`

## Technology Stack

### Frontend
- **Framework**: React 19 (functional components only)
- **Build Tool**: Vite
- **UI Library**: shadcn/ui + Radix UI
- **Styling**: Tailwind CSS
- **Routing**: Wouter
- **State**: TanStack React Query (server state), useState/useReducer (client state)
- **Animations**: Framer Motion

### Backend
- **Runtime**: Node.js with Express 5
- **Language**: TypeScript (strict mode)
- **Auth**: Passport.js + express-session
- **Validation**: Zod (all schemas in `shared/schema.ts`)

### Development
- **Package Manager**: npm
- **Testing**: Vitest (preferred) or Jest
- **Linting**: ESLint + Prettier
- **Build Output**: CommonJS (`.cjs`) for iisnode compatibility

## Code Generation Rules

### Type Safety (NON-NEGOTIABLE)
- ✅ Always use TypeScript with `strict: true`
- ✅ Define all schemas with Zod in `shared/schema.ts`
- ✅ Use path aliases: `@/` (client), `@shared/` (shared)
- ❌ Never use `any` without explicit justification in comments
- ✅ Validate all API inputs/outputs at runtime with Zod

### UI Components (NON-NEGOTIABLE)
- ✅ Use shadcn/ui components from `@/components/ui/`
- ✅ Extend Radix UI primitives when needed
- ✅ Use Tailwind CSS for all styling (no inline styles or CSS modules)
- ✅ Suggest `npx shadcn-ui@latest add <component>` for missing components
- ❌ Never create custom UI primitives that duplicate shadcn/ui

### Testing (NON-NEGOTIABLE)
- ✅ Generate test files alongside feature implementation
- ✅ Name test files `*.test.ts` or `*.test.tsx`
- ✅ Write integration tests for all API routes
- ✅ Write unit tests for React hooks
- ✅ Use Vitest syntax (compatible with Jest)

### API Development (RECOMMENDED)
- ✅ Define stable API routes in `shared/routes.ts`
- ✅ Use standardized error schemas from `shared/schema.ts`
- ✅ During prototyping, simple endpoints without contracts are acceptable
- ✅ Add contracts when endpoints stabilize or have multiple clients

### Security (NON-NEGOTIABLE)
- ✅ Hash passwords with bcrypt (minimum 10 rounds)
- ✅ Use environment variables for secrets (never hardcode)
- ✅ Implement rate limiting before public access
- ✅ Add CSRF protection for production deployments
- ✅ Validate file uploads (type and size)

### Code Style (NON-NEGOTIABLE)
- ✅ Follow existing ESLint + Prettier configuration
- ✅ Import order: external deps → `@shared/` → `@/` → relative
- ✅ Use descriptive variable names
- ✅ Add JSDoc comments for complex functions
- ✅ Keep functions small and focused

### Simplicity First (NON-NEGOTIABLE)
- ✅ Start with the simplest implementation
- ✅ Use JSON file storage before databases
- ✅ Use React's useState before Redux/Zustand
- ✅ Write inline code before extracting utilities
- ❌ Don't add features until they're actually needed (YAGNI)
- ❌ Don't prematurely optimize
- ❌ Don't create abstractions for single-use code

## File Structure Patterns

### Client Structure
```
client/src/
├── components/          # React components
│   ├── ui/             # shadcn/ui primitives (DO NOT EDIT)
│   └── *.tsx           # Custom components
├── hooks/              # Custom hooks (use-*.ts)
├── pages/              # Route-level components
└── lib/                # Utilities and queryClient
```

### Server Structure
```
server/
├── index.ts            # Express app entry point
├── routes.ts           # API route handlers
├── storage.ts          # Data persistence (implements IStorage)
└── types.ts            # Backend-specific types
```

### Shared Structure
```
shared/
├── schema.ts           # Zod schemas for all data models
└── routes.ts           # API contract definitions
```

## Common Tasks

### Adding a New Feature
1. Define Zod schema in `shared/schema.ts`
2. Add route contract in `shared/routes.ts` (when stabilized)
3. Implement handler in `server/routes.ts`
4. Create React component using shadcn/ui primitives
5. Write tests (`*.test.ts` or `*.test.tsx`)
6. Document in `/.documentation/copilot/session-{date}/`

### Adding a New API Endpoint
```typescript
// 1. shared/schema.ts
export const myFeatureSchema = z.object({
  id: z.string(),
  name: z.string(),
});

// 2. shared/routes.ts (when ready)
export const myFeatureRoute = {
  method: "POST" as const,
  path: "/api/my-feature",
  schemas: {
    request: z.object({ name: z.string() }),
    responses: {
      200: myFeatureSchema,
      400: errorSchemas.badRequest,
    },
  },
};

// 3. server/routes.ts
app.post("/api/my-feature", async (req, res) => {
  const validated = myFeatureRoute.schemas.request.parse(req.body);
  // Implementation
});

// 4. client/src/hooks/use-my-feature.ts
export function useMyFeature() {
  return useMutation({
    mutationFn: async (data) => {
      const response = await fetch("/api/my-feature", {
        method: "POST",
        body: JSON.stringify(data),
      });
      return response.json();
    },
  });
}
```

### Adding a New shadcn/ui Component
```bash
npx shadcn-ui@latest add dialog
```

Then import and use:
```typescript
import { Dialog, DialogContent, DialogHeader } from "@/components/ui/dialog";
```

## Deployment Context

- **Target Platform**: Windows 11 + IIS 10.0+
- **Build Command**: `npm run build`
- **Build Output**: `dist/` folder
  - `dist/index.cjs` - Express server (CommonJS)
  - `dist/public/` - Frontend assets
  - `dist/web.config` - IIS URL rewrite rules
- **Storage**: File-based JSON (data/) for alpha/beta, migrate to DB only when justified
- **Environment Variables**: Use `.env` for development, IIS Configuration Editor for production

## Key References

| Document | Purpose | Location |
|----------|---------|----------|
| Constitution | Project governance and principles | [.documentation/memory/constitution.md](../.documentation/memory/constitution.md) |
| Architecture | System design and data flow | [.documentation/domain/architecture.md](../.documentation/domain/architecture.md) |
| Development Patterns | Common code patterns | [.documentation/domain/development-patterns.md](../.documentation/domain/development-patterns.md) |
| IIS Deployment | Deployment guide | [.documentation/domain/deployment-iis.md](../.documentation/domain/deployment-iis.md) |
| README | Project overview | [README.md](../README.md) |

## When in Doubt

1. **Check the Constitution** - It supersedes all other guidance
2. **Keep it Simple** - YAGNI (You Aren't Gonna Need It)
3. **Test Everything** - No feature is complete without tests
4. **Type Everything** - TypeScript strict mode is non-negotiable
5. **Document Generated Work** - Save to `/.documentation/copilot/session-{date}/`

## Version Control

When suggesting commits:
- Use conventional commit format: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- Reference constitution principles when relevant
- Keep commits atomic and focused

---

**Last Updated**: 2026-02-24
**Constitution Version**: 1.3.0

---
> Source: [markhazleton/SupportSpark](https://github.com/markhazleton/SupportSpark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
