# Concord — Discord Clone (clone-disco)

## Project Overview
This is a full-featured Discord clone called "Concord" built as a monorepo. The AI coding agent must follow these instructions strictly when generating code.

## Tech Stack
- **Monorepo Tool:** Turborepo with npm workspaces
- **Frontend:** React 18, TypeScript 5, Vite 5, Tailwind CSS 3, Zustand 4, React Router 6, Socket.IO Client 4, Axios
- **Backend:** Node.js 20, Express 4, TypeScript 5, Prisma 5 ORM, Socket.IO 4, Passport.js
- **Database:** PostgreSQL 16
- **Cache:** Redis 7
- **Voice/Video:** mediasoup 3 (SFU architecture), WebRTC
- **Object Storage:** MinIO (S3-compatible)
- **Auth:** JWT (access + refresh tokens) + Passport.js (Google, GitHub OAuth)
- **Email:** Nodemailer
- **Validation:** Zod (shared between frontend and backend)
- **Containerization:** Docker + Docker Compose
- **Reverse Proxy:** Nginx (with WebSocket upgrade support)
- **CI/CD:** GitHub Actions

## Code Style Rules
- TypeScript strict mode everywhere (`"strict": true` in all tsconfig.json)
- All API responses MUST use this exact format:
  ```typescript
  { success: boolean; data: T | null; error: { code: string; message: string } | null }
  ```
- Use Zod for ALL request validation on both client and server
- Use cursor-based pagination for all list endpoints
- Every file MUST start with a JSDoc comment block explaining its purpose
- **NEVER** leave TODO, FIXME, placeholder, or `...` comments — every function must be fully implemented
- Use named exports only (no default exports)
- Use path aliases: `@/` maps to `src/` directory in both client and server
- All dates/timestamps in UTC
- Use async/await, never raw Promises with .then()

## Project Structure
```
/client    — React frontend application
/server    — Express backend application
/shared    — Shared TypeScript types, constants, and utilities
/nginx     — Nginx reverse proxy configuration
/database  — Database initialization scripts
```

## Database Conventions
- PostgreSQL with Prisma ORM
- IDs: cuid() (Prisma default)
- Table names: snake_case (via Prisma @@map)
- Column names: camelCase in Prisma, maps to snake_case in DB
- All models have createdAt and updatedAt timestamps
- Soft delete is NOT used — use hard delete with CASCADE

## Naming Conventions
- **Files:** camelCase.ts for utilities/services, PascalCase.tsx for React components
- **Folders:** camelCase
- **Database tables:** snake_case (via @@map)
- **API routes:** kebab-case (e.g., /api/friend-requests)
- **TypeScript types/interfaces:** PascalCase
- **Enums:** PascalCase with UPPER_SNAKE_CASE values
- **Constants:** UPPER_SNAKE_CASE
- **React components:** PascalCase
- **React hooks:** camelCase starting with "use"
- **Zustand stores:** camelCase ending with "Store"

## Error Handling
- Global error handler middleware on backend
- Custom error classes: AppError, NotFoundError, ForbiddenError, ValidationError, UnauthorizedError
- All async route handlers wrapped in try-catch or asyncHandler utility
- Frontend: Axios interceptor for 401 (auto token refresh), toast notifications for errors

## Security Requirements
- Passwords hashed with bcrypt (12 salt rounds)
- JWT access tokens: 15 min expiry
- JWT refresh tokens: 7 days expiry, stored in DB, rotated on use
- Rate limiting on all endpoints (Redis-backed)
- Input sanitization against XSS (DOMPurify on frontend, sanitize-html on backend)
- CORS configured for client origin only
- File upload: type whitelist (images, videos, audio, documents), max 25MB per file
- SQL injection protection via Prisma parameterized queries

## Permission System
- Bitwise permission flags (BigInt)
- Permission resolution order: server-level role permissions → channel-level overrides
- @everyone role as base, then role stack (highest position wins for allow), then member-specific overrides
- Channel permission overrides can ALLOW or DENY specific permissions per role or per member

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oncumuhammed)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/oncumuhammed)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
