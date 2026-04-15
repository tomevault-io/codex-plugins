# GitHub Copilot Instructions for AFC Stack

## Project Overview

AFC Stack is a production-ready full-stack monorepo template with an interactive CLI for customizable project generation. It combines Next.js 15, TypeScript, Drizzle ORM, WebSockets, and modern tooling.

## Core Architecture

### Monorepo Structure

- **Root**: Turborepo workspace with shared configs
- **apps/web**: Next.js 15 App Router frontend (TypeScript)
- **apps/ws**: Fastify WebSocket service (Bun runtime)
- **packages/db**: Shared Drizzle ORM schema and client
- **cli-templates/**: Template files for project generation

### Technology Stack

- **Frontend**: Next.js 15, React 18, Tailwind CSS
- **Backend**: Next.js API Routes, Fastify WebSocket
- **Database**: PostgreSQL with Drizzle ORM
- **Auth**: NextAuth v5 (beta)
- **Storage**: MinIO (dev), S3/UploadThing (prod)
- **Analytics**: PostHog Cloud
- **Rate Limiting**: Arcjet
- **Package Manager**: Bun (primary), supports pnpm/npm
- **Build Tool**: Turborepo, tsup for CLI

## Getting Started

### Initial Setup

```bash
# Clone and install dependencies
git clone https://github.com/arturict/afc-stack.git
cd afc-stack
bun install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Start Docker services (PostgreSQL, MinIO, etc.)
docker compose -f compose.dev.yml up -d

# Generate and run database migrations
bunx drizzle-kit generate
bunx drizzle-kit migrate
```

### Running the Project

```bash
# Start all services in development mode
bun run dev

# Start specific apps
bun run dev --filter=web        # Next.js app only
bun run dev --filter=ws         # WebSocket server only

# Build for production
bun run build

# Run production build
bun run start
```

### Available Scripts

- `bun run dev` - Start all apps in development mode
- `bun run build` - Build all packages and apps
- `bun run lint` - Lint all code
- `bun run lint:fix` - Auto-fix linting issues
- `bun run typecheck` - Type-check all TypeScript
- `bun run test` - Run all tests
- `bun run format` - Format code with Prettier
- `bun run clean` - Clean all build artifacts and dependencies
- `bun run create` - Interactive CLI to create new project
- `bun run add:websocket` - Add WebSocket support to existing project
- `bun run db:generate` - Generate database migrations
- `bun run db:migrate` - Run database migrations
- `bun run db:studio` - Open Drizzle Studio (database GUI)
- `bun run changeset` - Create a new changeset for versioning

### Project Structure

```
afc-stack/
├── apps/
│   ├── web/                    # Next.js frontend application
│   │   ├── src/
│   │   │   ├── app/           # App Router pages and API routes
│   │   │   │   ├── api/       # API endpoints
│   │   │   │   ├── (auth)/    # Auth-related pages
│   │   │   │   └── ...        # Other routes
│   │   │   ├── components/    # React components
│   │   │   ├── lib/           # Utility functions
│   │   │   ├── styles/        # Global styles
│   │   │   └── env.ts         # Environment variable validation
│   │   └── package.json
│   └── ws/                     # WebSocket server (optional)
│       ├── src/
│       │   └── server.ts      # Fastify WebSocket server
│       └── package.json
├── packages/
│   └── db/                     # Shared database package
│       ├── src/
│       │   ├── schema.ts      # Drizzle ORM schemas
│       │   └── index.ts       # Database client exports
│       └── package.json
├── cli-templates/              # Template files for CLI
│   ├── base/                  # Base template files
│   └── extras/                # Optional feature templates
├── .github/
│   ├── workflows/             # GitHub Actions workflows
│   ├── copilot-instructions.md # This file
│   └── ...                    # Other GitHub config
├── .env.example               # Environment variables template
├── compose.dev.yml            # Docker Compose for development
├── turbo.json                 # Turborepo configuration
├── tsconfig.base.json         # Base TypeScript config
└── package.json               # Root workspace config
```

## Common Workflows

### Adding a New Feature

1. **Create a branch**:

    ```bash
    git checkout -b feature/your-feature-name
    ```

2. **Make changes** following the code style guidelines

3. **Test your changes**:

    ```bash
    bun run lint        # Check for linting errors
    bun run typecheck   # Verify TypeScript types
    bun run test        # Run tests
    bun run build       # Ensure it builds
    ```

4. **Create a changeset** (for user-facing changes):

    ```bash
    bun run changeset
    # Select packages, version bump type, and write description
    ```

5. **Commit and push**:

    ```bash
    git add .
    git commit -m "feat: add your feature"
    git push origin feature/your-feature-name
    ```

6. **Create a Pull Request** on GitHub

### Adding a New API Route

1. Create file at `apps/web/src/app/api/[route-name]/route.ts`:

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import { db, tableName } from "@ac/db";
import arcjet from "@arcjet/next";

// Define request schema
const requestSchema = z.object({
    field: z.string().min(1)
});

// Apply rate limiting
export async function POST(req: Request) {
    // Rate limiting check
    const decision = await aj.protect(req);
    if (decision.isDenied()) {
        return NextResponse.json({ error: "Too many requests" }, { status: 429 });
    }

    try {
        // Parse and validate request body
        const body = await req.json();
        const parsed = requestSchema.safeParse(body);

        if (!parsed.success) {
            return NextResponse.json({ error: "Invalid input", details: parsed.error }, { status: 400 });
        }

        // Database operation
        const result = await db.insert(tableName).values(parsed.data).returning();

        return NextResponse.json({ data: result });
    } catch (error) {
        console.error("API error:", error);
        return NextResponse.json({ error: "Internal server error" }, { status: 500 });
    }
}
```

2. Test the endpoint locally
3. Add tests if applicable
4. Update documentation

### Adding a Database Table

1. **Update schema** in `packages/db/src/schema.ts`:

```typescript
import { pgTable, serial, text, timestamp } from "drizzle-orm/pg-core";

export const yourTable = pgTable("your_table", {
    id: serial("id").primaryKey(),
    name: text("name").notNull(),
    createdAt: timestamp("created_at").notNull().defaultNow(),
    updatedAt: timestamp("updated_at").notNull().defaultNow()
});
```

2. **Generate migration**:

    ```bash
    bun run db:generate
    ```

3. **Review the migration** in `packages/db/migrations/`

4. **Apply migration**:

    ```bash
    bun run db:migrate
    ```

5. **Update related queries** in your application

6. **Commit both schema and migration files**:
    ```bash
    git add packages/db/src/schema.ts packages/db/migrations/
    git commit -m "feat: add your_table schema"
    ```

### Adding a React Component

1. **Create component** in `apps/web/src/components/`:

```typescript
// apps/web/src/components/YourComponent.tsx
interface YourComponentProps {
    title: string;
    onAction?: () => void;
}

export function YourComponent({ title, onAction }: YourComponentProps) {
    return (
        <div className="p-4">
            <h2 className="text-xl font-bold">{title}</h2>
            {onAction && (
                <button onClick={onAction} className="mt-2 px-4 py-2 bg-blue-500">
                    Action
                </button>
            )}
        </div>
    );
}
```

2. **Use in pages** following Server/Client Component patterns

3. **Test responsiveness** and accessibility

4. **Update Storybook** if applicable (future)

## Code Style & Conventions

### TypeScript

- Use strict mode always
- Prefer interfaces over types for objects
- Use type inference when obvious
- Explicitly type function returns for public APIs
- Use `const` for all variables unless mutation needed

### React/Next.js

- Server Components by default
- Use `"use client"` only when necessary (hooks, events)
- File-based routing in `app/` directory
- API routes in `app/api/`
- Colocation of related files encouraged

### Naming Conventions

- **Files**: kebab-case for regular files, PascalCase for React components
- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Utilities**: camelCase (e.g., `formatDate.ts`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRIES`)
- **Database tables**: snake_case (e.g., `user_sessions`)

### Code Organization

```typescript
// Order of imports:
// 1. External dependencies
// 2. Internal packages (@ac/*)
// 3. Relative imports
// 4. Types

import { useState } from "react";
import { db, users } from "@ac/db";
import { formatDate } from "@/lib/utils";
import type { User } from "@/types";
```

## Development Guidelines

### Database (Drizzle ORM)

- Define schemas in `packages/db/src/schema.ts`
- Use relations for foreign keys
- Always create migrations with `drizzle-kit generate`
- Test migrations locally before deploying
- Use prepared statements for repeated queries

Example:

```typescript
// Good
export const users = pgTable("users", {
    id: serial("id").primaryKey(),
    email: text("email").notNull().unique(),
    createdAt: timestamp("created_at").notNull().defaultNow()
});
```

### API Routes

- Use Zod for validation
- Implement proper error handling
- Apply rate limiting with Arcjet
- Use Next.js request/response helpers
- Return consistent error shapes

Example:

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import arcjet from "@arcjet/next";

const schema = z.object({
    /* ... */
});

export async function POST(req: Request) {
    const decision = await aj.protect(req);
    if (decision.isDenied()) {
        return NextResponse.json({ error: "rate_limited" }, { status: 429 });
    }

    const body = await req.json();
    const parsed = schema.safeParse(body);
    if (!parsed.success) {
        return NextResponse.json({ error: "invalid_input" }, { status: 400 });
    }

    // ... handle request
}
```

### WebSocket Service

- Keep logic in `apps/ws/src/server.ts`
- Use Fastify plugins for modularity
- Broadcast events to all connected clients
- Handle connection/disconnection gracefully
- Log errors with Fastify logger

### Environment Variables

- Validate in `apps/web/src/env.ts` using Zod
- Always provide `.env.example`
- Never commit actual `.env` files
- Use `NEXT_PUBLIC_` prefix for client-side vars
- Use typed env access: `env.DATABASE_URL`

### Error Handling

- Use try-catch for async operations
- Log errors with context
- Return user-friendly messages
- Use appropriate HTTP status codes
- Handle edge cases explicitly

## CLI Development

### Interactive Prompts

- Use `@clack/prompts` for all user input
- Provide sensible defaults (marked with "recommended")
- Validate input immediately
- Show progress with spinners
- Handle cancellation gracefully

### Template System

- Base templates in `cli-templates/base/`
- Optional features in `cli-templates/extras/`
- Use conditional copying based on user choices
- Generate configs programmatically
- Preserve file permissions

### Build & Distribution

- Build with tsup: `bun run build:cli`
- Output to `dist/` (gitignored)
- Bundle all dependencies
- Use shebang for CLI entry
- Test in temp directory before release

## Testing Strategy

### Unit Tests

- Test utilities and helpers
- Mock external dependencies
- Use descriptive test names
- Aim for 80%+ coverage on critical paths

### Integration Tests

- Test API routes end-to-end
- Use test database
- Clean up after tests
- Test error scenarios

### E2E Tests (future)

- Use Playwright for browser tests
- Test critical user flows
- Run in CI pipeline

## Performance Optimization

### Frontend

- Use React Server Components by default
- Implement loading states
- Optimize images with Next.js Image
- Code-split large components
- Use `suspense` for data fetching

### Backend

- Use connection pooling (max 5 for Postgres)
- Implement caching where appropriate
- Optimize database queries
- Use indexes for frequent queries
- Monitor with PostHog

### Build

- Leverage Turborepo cache
- Optimize bundle size
- Use dynamic imports for large dependencies
- Configure proper `tsconfig` paths

## Deployment

### Docker

- Multi-stage builds for smaller images
- Use Alpine base images
- Run migrations before app start
- Health checks on all services
- Set proper environment variables

### CI/CD

- GitHub Actions for automation
- Run lint, typecheck, and tests
- Build Docker images on main branch
- Deploy via Coolify webhooks
- Auto-update dependencies with Dependabot

## Security Best Practices

- Validate all user input with Zod
- Use rate limiting on all public endpoints
- Sanitize database queries (Drizzle handles this)
- Use HTTPS in production
- Implement CSRF protection (NextAuth handles this)
- Regular dependency updates
- Never log sensitive data
- Use environment variables for secrets

## Common Patterns

### Server Action (Next.js 15)

```typescript
"use server";
import { db } from "@ac/db";

export async function createTodo(title: string) {
    const [todo] = await db.insert(todos).values({ title }).returning();
    return todo;
}
```

### WebSocket Broadcast

```typescript
// In apps/ws
const clients = new Set<WebSocket>();

// Broadcast helper
function broadcast(event: string, data: unknown) {
    const message = JSON.stringify({ type: event, payload: data });
    for (const client of clients) {
        client.send(message);
    }
}
```

### Type-Safe Environment

```typescript
import { z } from "zod";

export const env = z
    .object({
        DATABASE_URL: z.string().url()
        // ... other vars
    })
    .parse(process.env);
```

## Troubleshooting

### Common Issues

**Turborepo cache issues**: Run `bun run clean` and reinstall
**Database connection fails**: Check Docker compose is running with `docker compose -f compose.dev.yml ps`
**TypeScript errors in monorepo**: Verify `tsconfig.json` references and run `bun run typecheck`
**WebSocket not connecting**: Check CORS settings and firewall. Ensure WS service is running on correct port
**Build fails in CI**: Ensure all dependencies are in package.json and not just devDependencies
**Migrations fail**: Check database connection string in `.env` and ensure PostgreSQL is running
**Port already in use**: Kill process or change port in `.env` (default: 3000 for web, 3001 for ws)

### Debugging

**Next.js Application**:

- Use VS Code debug configuration (`.vscode/launch.json` included)
- Or use Chrome DevTools with `NODE_OPTIONS='--inspect' bun run dev`
- Check browser console for client-side errors
- Check terminal for server-side errors

**Database Queries**:

- Use Drizzle Studio: `bun run db:studio` (opens GUI on http://localhost:4983)
- Check query logs in application console
- Use `EXPLAIN` in direct SQL queries for performance analysis

**WebSocket Issues**:

- Check connection URL matches your configuration
- Verify CORS settings in `apps/ws/src/server.ts`
- Use browser DevTools Network tab to inspect WS connections
- Check server logs for connection/disconnection events

**Build Problems**:

- Clear Turborepo cache: `bun run clean`
- Clear Next.js cache: `rm -rf apps/web/.next`
- Reinstall dependencies: `rm -rf node_modules && bun install`
- Check for conflicting package versions

## Version Management

### Changesets

- Create changeset: `bunx changeset`
- Version bump: `bun run version`
- Choose semver level (major/minor/patch)
- Write meaningful changelog entries
- Commit changeset files

## When Making Changes

### Pre-Commit Checklist

1. **Check existing patterns** - Review similar code in the codebase for consistency
2. **Follow conventions** - Use established naming conventions and file structures
3. **Write tests** - Add tests for new functionality, update tests for changes
4. **Type safety** - Ensure TypeScript types are correct, avoid `any`
5. **Validate locally**:
    ```bash
    bun run lint          # Check linting
    bun run typecheck     # Verify types
    bun run test          # Run tests
    bun run build         # Ensure builds successfully
    ```
6. **Update documentation** - Update README, comments, or other docs as needed
7. **Create changeset** - For user-facing changes, create a changeset:
    ```bash
    bun run changeset
    ```

### Code Review Preparation

Before requesting a review:

- Self-review your changes in GitHub's "Files changed" view
- Ensure all tests pass in CI
- Add comments explaining complex logic
- Update the PR description with context and testing notes
- Check that commit messages follow conventional commit format
- Verify no sensitive data (API keys, passwords) is committed

### Commit Message Format

Follow conventional commits:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types:

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting)
- `refactor:` Code refactoring
- `perf:` Performance improvements
- `test:` Test changes
- `chore:` Build process or auxiliary tool changes
- `ci:` CI/CD changes

Examples:

```
feat(auth): add OAuth provider support
fix(api): handle null values in user endpoint
docs(readme): update installation instructions
refactor(db): optimize user query performance
```

## Repository Documentation

For more detailed information, refer to these documentation files:

### Core Documentation

- **[README.md](../README.md)** - Project overview and quick start guide
- **[SETUP.md](../SETUP.md)** - Detailed setup instructions and manual configuration
- **[CONTRIBUTING.md](../CONTRIBUTING.md)** - Contribution guidelines and code style

### Feature-Specific Guides

- **[CLI.md](../CLI.md)** - Interactive CLI usage and customization options
- **[WEBSOCKET.md](../WEBSOCKET.md)** - WebSocket service setup and usage
- **[DEPLOYMENT.md](../DEPLOYMENT.md)** - Production deployment guide (Docker, Coolify, etc.)

### GitHub Configuration

- **[.github/BRANCHING.md](.github/BRANCHING.md)** - Git workflow and branching strategy
- **[.github/CODE_QUALITY.md](.github/CODE_QUALITY.md)** - Code quality standards and tools
- **[.github/DX.md](.github/DX.md)** - Developer experience improvements tracking
- **[.github/SCRIPTS.md](.github/SCRIPTS.md)** - Available scripts and automation

### Other Resources

- **[CHANGELOG.md](../CHANGELOG.md)** - Version history and changes
- **[SECURITY.md](../SECURITY.md)** - Security policy and reporting vulnerabilities
- **[.changeset/README.md](../.changeset/README.md)** - Changesets workflow

## External Resources

- [Next.js Docs](https://nextjs.org/docs) - Next.js framework documentation
- [Drizzle ORM Docs](https://orm.drizzle.team) - Drizzle ORM query builder and migrations
- [Turborepo Docs](https://turbo.build/repo/docs) - Monorepo build system
- [Clack Prompts](https://github.com/natemoo-re/clack) - Interactive CLI prompts library
- [Changesets](https://github.com/changesets/changesets) - Version and changelog management
- [NextAuth.js](https://next-auth.js.org/) - Authentication library for Next.js
- [Tailwind CSS](https://tailwindcss.com/docs) - Utility-first CSS framework
- [PostHog](https://posthog.com/docs) - Product analytics platform
- [Arcjet](https://docs.arcjet.com/) - Rate limiting and security

## AI Pair Programming Tips

When working with AI assistants on this project:

1. **Be specific** about which app/package you're working in
2. **Mention the tech** being used (e.g., "in the Drizzle schema")
3. **Ask for explanations** of patterns you don't understand
4. **Request alternatives** if a solution doesn't fit the architecture
5. **Validate suggestions** against these guidelines before implementing
6. **Consider implications** on other parts of the monorepo

Remember: This is a template project. Changes should be generalizable and follow best practices for others to use.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arturict) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
