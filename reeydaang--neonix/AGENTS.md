# Neonix Codebase Guide for AI Agents

## Project Overview
**Neonix** is a full-stack chat/collaboration prototype with a **NestJS backend** (PostgreSQL + Prisma) and a **Next.js frontend** (React 19, Tailwind CSS). The system manages users, chat rooms, channels, and real-time messaging.

**Tech Stack:**
- **Backend:** NestJS 11, Prisma 7, JWT auth, PostgreSQL
- **Frontend:** Next.js 16, React 19, TypeScript, Tailwind CSS 4
- **Monorepo:** `/backend` and `/frontend` directories

---

## Architecture & Data Flow

### Core Components
1. **Auth Module** (`backend/src/modules/auth/`) - User registration, login, JWT tokens
2. **Chat Module** (`backend/src/modules/chat/`) - Rooms, channels, messages (hierarchical)
3. **Prisma Module** (`backend/src/modules/prisma/`) - Singleton database service

### Data Model (Prisma)
```
User (id, email, password, displayName, username)
  ↓
Message (who, text, time, me: boolean, createdAt)
  ↓
Channel (name) → Room (name, meta, badge)
```
- **Rooms** contain multiple **Channels**; **Channels** contain **Messages**
- `Message.me` flag indicates user-sent (frontend display toggle)
- Foreign keys use `onDelete: Cascade` for clean teardown

### Auth Flow
1. **Register/Login** → `POST /auth/{register|login}` → Returns `{ token, user }`
2. **Verification** → `GET /auth/me` with `Authorization: Bearer <token>` header
3. **JWT Secret:** `process.env.JWT_SECRET || "dev_secret_change_me"` (expires 7 days)
4. ⚠️ **Current limitation:** Plain-text password storage (development only)

### API Routes (Backend)
```
POST   /auth/register        - Create account
POST   /auth/login           - Authenticate
GET    /auth/me              - Verify token
GET    /rooms                - List all rooms + auto-seed if empty
GET    /rooms/:id/channels   - Channels in a room
GET    /rooms/:id/channels/:cid/messages - Messages in channel
POST   /rooms/:id/channels/:cid/messages  - Send message
```

### Frontend Integration (`src/lib/api.ts`)
- Single `http()` fetch wrapper with JSON headers
- API URL: `process.env.NEXT_PUBLIC_API_URL || "http://localhost:4000"`
- Organized by domain: `api.auth.*`, `api.chat.*`

### State Management (`src/state/`)
- **`auth.tsx`** - User login state, token persistence
- **`uiPrefs.tsx`** - UI theme/language preferences
- **`i18n.ts`** - Internationalization
- Providers wrap `AppShell` in root layout

---

## Key Workflows & Commands

### Backend Development
```bash
# Install & setup
cd backend && npm install

# Watch mode (for active development)
npm run start:dev      # Rebuilds on file change, localhost:3000

# Production build
npm run build

# Testing
npm run test           # Unit tests (*.spec.ts)
npm run test:watch     # Watch mode
npm run test:cov       # Coverage report
npm run test:e2e       # Integration tests (test/auth.int.e2e-spec.ts)

# Database
npx prisma migrate dev --name <name>  # Create & apply migration
npx prisma studio                      # UI for database inspection
npx prisma generate                    # Regenerate Prisma client

# Code quality
npm run lint --fix     # Fix ESLint + Prettier issues
npm run format         # Format with Prettier
```

### Frontend Development
```bash
cd frontend && npm install

npm run dev            # Dev server, hot reload, localhost:3000
npm run build          # Production build
npm run start          # Run production build locally
npm run lint           # ESLint check
```

### Database Seeding
- **Automatic:** `ChatService.ensureSeed()` runs on first `/rooms` call
- Creates 2 default rooms ("Neonix — Main", "Study Session") + 5 channels + 1 sample message
- Idempotent: checks `room.count()` before creating

---

## Project-Specific Patterns

### NestJS Conventions
- **Module per feature:** `auth/`, `chat/`, `prisma/` (controller + service + module + DTO)
- **Dependency injection:** Services inject `PrismaService` via constructor
- **DTO validation:** Use `class-validator` decorators (see `auth/dto.ts`)
- **Error handling:** Throw NestJS exceptions (`BadRequestException`, `UnauthorizedException`)

### Testing Strategy
- **Unit tests:** Mock Prisma, test service logic in isolation
- **Integration (E2E) tests:** Real app bootstrap, mock only Prisma at provider level, use `supertest` for HTTP
- **Pattern:** Reset mocks in `beforeEach()`, use `jest.fn()` for Prisma calls
- **Example:** `auth.service.spec.ts` (unit), `auth.int.e2e-spec.ts` (integration)

### DTOs & Validation
- Located in `modules/{feature}/dto.ts`
- Use `class-transformer` + `class-validator` decorators
- Example: `LoginDto` has `email` (string) and `password` (string)

### Prisma Patterns
- Always use `await` on Prisma queries
- Leverage `orderBy` for consistent results
- Use `createMany()` for bulk inserts (see `ensureSeed`)
- Define relations explicitly for eager loading

### Frontend Patterns
- **Page structure:** `app/{feature}/page.tsx` (route = `/feature`)
- **Component hierarchy:** `AppShell` → providers → page content
- **API calls:** Use `api.*` wrapper (handles JSON + errors)
- **Token storage:** Managed by `AuthProvider` context
- **Styling:** Tailwind CSS 4 + PostCSS (no component library)

---

## Critical Integration Points

### Client-Server Communication
- Backend defaults to `localhost:3000` (override via `.env`)
- Frontend expects `NEXT_PUBLIC_API_URL` env var for backend address
- All requests include `Content-Type: application/json`
- Auth token passed in `Authorization: Bearer <token>` header

### Environment Variables
**Backend (`.env`):**
```
JWT_SECRET=<your_key>
DATABASE_URL=postgresql://user:password@localhost:5432/neonix
```

**Frontend (`.env.local`):**
```
NEXT_PUBLIC_API_URL=http://localhost:3000  # or production URL
```

### Database Connection
- Prisma adapter: `@prisma/adapter-pg` (PostgreSQL)
- Client: `@prisma/client` (v7.1.0+)
- Ensure PostgreSQL 12+ is running before migrations

---

## Common Implementation Tasks

### Adding a New API Endpoint
1. **Define DTO** in `modules/{feature}/dto.ts`
2. **Add method to Service** (business logic, Prisma calls)
3. **Add route to Controller** with `@Post/@Get/@Patch` decorator
4. **Update frontend `api.ts`** with new HTTP method
5. **Write unit test** mocking Prisma, integration test with real app

### Adding a Database Model
1. **Update `prisma/schema.prisma`** (define model + relations)
2. **Create migration:** `npx prisma migrate dev --name <model_name>`
3. **Regenerate client:** `npx prisma generate`
4. **Seed if needed:** Add to `ChatService.ensureSeed()` or dedicated seed script
5. **Test migration rollback** (verify `down` behavior in migration file)

### Debugging Auth Issues
- Check `JWT_SECRET` matches frontend & backend
- Verify token expiration (7 days in code)
- Use `GET /auth/me` to validate token format
- Token format: `Bearer <jwt_string>` (case-sensitive)

---

## File Structure Reference
```
backend/src/
├── app.module.ts              # Root module imports (Prisma, Auth, Chat)
├── main.ts                    # Bootstrap
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts # Routes: /register, /login, /me
│   │   ├── auth.service.ts    # JWT sign, user lookup, validation
│   │   ├── auth.module.ts     # Module definition
│   │   ├── auth.service.spec.ts # Unit tests
│   │   └── dto.ts             # LoginDto, RegisterDto
│   ├── chat/
│   │   ├── chat.controller.ts # Routes: /rooms, /channels, /messages
│   │   ├── chat.service.ts    # ensureSeed(), Prisma queries
│   │   ├── chat.module.ts
│   │   ├── chat.service.spec.ts
│   ├── prisma/
│   │   ├── prisma.service.ts  # PrismaClient singleton
│   │   ├── prisma.module.ts   # Global module
└── test/
    ├── auth.int.e2e-spec.ts   # Integration tests with real app
    └── chat.int.e2e-spec.ts

frontend/src/
├── app/
│   ├── layout.tsx             # Root layout, providers
│   ├── page.tsx               # Home page
│   ├── auth/page.tsx          # Login/register
│   ├── chat/page.tsx          # Main chat interface
│   └── components/
│       ├── AppShell.tsx       # Navigation, layout wrapper
│       ├── Button.tsx, Topbar.tsx, Footer.tsx
│       └── icons/
├── lib/
│   └── api.ts                 # HTTP client wrapper
├── state/
│   ├── auth.tsx               # AuthProvider context
│   ├── uiPrefs.tsx            # Theme/UI settings
│   └── i18n.ts                # i18n configuration
└── styles/
    └── globals.css            # Tailwind setup
```

---

## Performance & Quality Notes
- **Coverage threshold:** Check `jest-e2e.json` & `package.json` for coverage goals
- **Linting:** ESLint 9 + Prettier (auto-format on commit recommended)
- **Build size:** Next.js 16 tree-shakes unused code; monitor bundle with `npm run build`
- **Database:** Index on `(roomId, channelId)` in `Message` model for fast channel queries

---

## When Stuck
1. **Tests fail?** Check mocks match actual Prisma API (`.findUnique`, `.create`, etc.)
2. **API 404?** Verify controller path + HTTP method + URL structure
3. **DB errors?** Run `npx prisma studio` to inspect schema; ensure migrations applied
4. **Frontend blank?** Check `NEXT_PUBLIC_API_URL` env var + network tab in DevTools
5. **Token issues?** Validate JWT format with `jwt.io`; confirm secret matches both sides

---
> Source: [ReEyDaAng/Neonix](https://github.com/ReEyDaAng/Neonix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
