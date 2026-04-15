# x402 VeChain Facilitator - Copilot Instructions

This is a TypeScript monorepo implementing the x402 payment protocol for VeChain blockchain. The project uses Turbo for build orchestration, pnpm for package management, and consists of three main applications: an API server (Hono), a dashboard (React Router), and a marketing website (Astro).

## Code Standards

### Required Before Each Commit
- Ensure TypeScript type checking passes: `pnpm typecheck` (if applicable)
- Build the project to verify no build errors: `pnpm build`
- Follow existing code style and conventions found in the codebase

### Development Flow
- **Install dependencies**: `pnpm install` (uses pnpm workspace)
- **Development**: `pnpm dev` (runs all apps in development mode using Turbo)
- **Build**: `pnpm build` (builds all apps using Turbo)
- **Type check**: `turbo check-types` (for apps that support it)
- **Lint**: `turbo lint` (for apps that support it)

### Technology Stack
- **Build System**: Turbo (monorepo build orchestration)
- **Package Manager**: pnpm 10.12.4 (workspace-based monorepo)
- **Language**: TypeScript 5.x
- **API Framework**: Hono (modern, fast web framework)
- **Frontend Frameworks**: 
  - Dashboard: React Router 7 with React 19
  - Web: Astro 5
- **Database**: PostgreSQL with Drizzle ORM
- **Blockchain**: VeChain SDK (@vechain/sdk-core, @vechain/sdk-network)
- **Validation**: Zod (schema validation)

## Repository Structure

```
/
├── apps/
│   ├── api/          # Hono-based x402 protocol API server
│   │   ├── src/
│   │   │   ├── config/       # Environment configuration
│   │   │   ├── db/           # Database schema and migrations (Drizzle ORM)
│   │   │   ├── middleware/   # Error handling, rate limiting
│   │   │   ├── routes/       # API route handlers
│   │   │   ├── schemas/      # Zod validation schemas
│   │   │   ├── services/     # Business logic (VeChain, payment verification)
│   │   │   ├── types/        # TypeScript type definitions
│   │   │   ├── dev.ts        # Development server entry point
│   │   │   └── index.ts      # Main application setup
│   │   └── README.md         # Detailed API documentation
│   ├── dashboard/    # React Router admin dashboard
│   │   └── README.md
│   └── web/          # Astro marketing website
│       └── README.md
├── package.json      # Root package.json with workspace scripts
├── pnpm-workspace.yaml  # pnpm workspace configuration
├── turbo.json        # Turbo build configuration
└── tsconfig.json     # Root TypeScript configuration
```

## Key Guidelines

### General Practices
1. **Follow TypeScript best practices**: Use strict type checking, avoid `any` types
2. **Maintain existing code structure**: Keep the monorepo organization intact
3. **Use workspace dependencies**: Reference local packages with `workspace:*`
4. **Write unit tests**: When adding new functionality (if test infrastructure exists)
5. **Document public APIs**: Add JSDoc comments for exported functions and types

### API Application (apps/api)
- Implements the **x402 payment protocol** for VeChain
- Uses **Hono** framework for routing and middleware
- **Zod schemas** for request/response validation (see `src/schemas/`)
- **Environment variables** are validated using Zod (see `src/config/env.ts`)
- Database operations use **Drizzle ORM** (schema in `src/db/`)
- VeChain blockchain interactions use **@vechain/sdk** (see `src/services/`)
- Follow the existing middleware pattern for error handling and rate limiting

### Environment Configuration
- Copy `.env.example` to `.env` before running the project
- All environment variables are validated at startup
- See `.env.example` for detailed documentation of each variable
- Never commit secrets (`.env` files are gitignored)
- Key variables:
  - `VECHAIN_NETWORK`: `testnet` or `mainnet`
  - `DATABASE_URL`: PostgreSQL connection string
  - `FEE_DELEGATION_PRIVATE_KEY`: Required if fee delegation is enabled
  - `JWT_SECRET`: For authentication (generate with `openssl rand -base64 32`)

### Database Management (apps/api)
- **Generate migrations**: `pnpm --filter api db:generate`
- **Run migrations**: `pnpm --filter api db:migrate`
- **Push schema changes**: `pnpm --filter api db:push`
- **Open Drizzle Studio**: `pnpm --filter api db:studio`
- Schema defined in `apps/api/src/db/schema.ts`

### x402 Protocol Implementation
- The API implements the x402 facilitator specification
- Core endpoints: `/verify`, `/settle`, `/supported`
- Uses CAIP-2 format for network identifiers (e.g., `eip155:100009` for VeChain testnet)
- Supports VeChain assets: VET, VTHO, VEUSD, B3TR
- See `apps/api/README.md` for detailed API documentation

### Running Individual Apps
- **API only**: `pnpm --filter api dev`
- **Dashboard only**: `pnpm --filter dashboard dev`
- **Web only**: `pnpm --filter web dev`
- **All apps**: `pnpm dev` (recommended for development)

### Building for Production
- **Build all**: `pnpm build`
- **Build specific app**: `pnpm --filter <app-name> build`
- Build outputs are configured in `turbo.json`

### Code Style
- Use **ES modules** (type: "module" in package.json)
- Prefer **async/await** over promise chains
- Use **arrow functions** for consistency
- Follow existing file naming conventions (kebab-case for files, PascalCase for types)
- Use **barrel exports** sparingly (prefer direct imports for better tree-shaking)

### Error Handling
- Use the existing error handling middleware pattern (see `apps/api/src/middleware/errorHandler.ts`)
- Return structured error responses with appropriate HTTP status codes
- Log errors appropriately for debugging

### Security Best Practices
- Never commit private keys or secrets to version control
- Validate all user inputs using Zod schemas
- Use rate limiting to prevent abuse (see `apps/api/src/middleware/rateLimiter.ts`)
- Sanitize error messages in production to avoid leaking sensitive information

## Deployment

### Vercel Deployment (API)
- Install Vercel CLI: `npm install -g vercel`
- Deploy: `pnpm --filter api install && vc deploy`
- Set environment variables in Vercel dashboard

### Prerequisites
- Node.js 20+
- pnpm 10.12.4 (use `npm install -g pnpm@10.12.4`)
- PostgreSQL database (for API)
- VeChain testnet/mainnet RPC access

## Important Notes
- This is a **monorepo**: always use `pnpm` commands from the root directory
- Use **Turbo** for parallel builds and caching
- Each app has its own README with specific documentation
- The API requires proper environment configuration to run
- VeChain network configuration determines which blockchain network is used

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ashutoshpw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
