# AI SYSTEM INSTRUCTIONS — BADIR PROJECT

This document defines **hard constraints and architectural ground truth** for any AI agent generating code, tests, or documentation for the Badir project.

The AI must remain strictly aligned with the existing system.
Deviation is considered an error.

---

## 1. Project Identity

Project name: **Badir**
Domain: Community & volunteer initiative platform
Primary goals: initiative discovery, participation, organization management, and social impact tracking.

---

## 2. Technology Stack (Fixed)

Frontend:

- Next.js 15 (App Router only)
- React 19
- TypeScript (strict)
- Tailwind CSS v4 (RTL + LTR support)
- Shadcn/UI + Radix UI

Backend & Infrastructure:

- Prisma ORM
- PostgreSQL
- Better Auth (JWT sessions)
- Supabase (file storage + RLS)

Validation & Content:

- Zod (input validation)
- TipTap (rich text)
- DOMPurify (HTML sanitization)

No alternative frameworks, ORMs, auth systems, or storage providers may be assumed or introduced.

---

## 3. Architectural Boundaries (Non-Negotiable)

### Data Flow

Client
→ Server Action / API Route / Server Component
→ `services/*`
→ Prisma
→ PostgreSQL

### Services Layer (Critical Rule)

- `/services` contains **classes with static methods**
- These methods:
  - Encapsulate **all Prisma model access**
  - Perform **business-level data operations**
  - Are **purely server-side**
- Services are invoked only from:
  - Server Actions
  - API routes
  - Server Components

❌ Prisma must never be accessed directly outside `services/`.

❌ Services must not contain:

- HTTP logic
- Request/response objects
- UI logic
- Client-side code

---

## 4. Folder Semantics (Must Match)

- `actions/` → Server Actions (orchestrate logic, auth checks)
- `services/` → Static service classes wrapping Prisma
- `schemas/` → Zod schemas only
- `lib/` → Infrastructure utilities (auth, Supabase, helpers)
- `components/` → UI only
- `app/` → Routing, layouts, pages
- `types/` → Shared TypeScript types

AI must not relocate responsibilities between folders.

---

## 5. Database & Prisma Rules

- Prisma schema is the **single source of truth**
- No inferred or fabricated fields, enums, or relations
- Enum values must match exactly
- All DB access must go through services
- No raw SQL unless already present in the codebase

Schema changes require migrations and must be explicit.

---

## 6. Authentication & Authorization

- Better Auth is the only auth system
- JWT sessions are authoritative
- Role-based access control must be enforced **server-side**
- Client checks are considered non-security measures

Auth logic must never be bypassed or simplified.

---

## 7. Security Constraints

AI output must assume:

- Untrusted user input
- Mandatory Zod validation
- Mandatory RBAC checks
- Mandatory HTML sanitization for rich text
- Supabase RLS enforcement

Forbidden patterns:

- Client-side authorization
- Hardcoded secrets
- Logging sensitive data
- Disabling validation for convenience

---

## 8. Internationalization & RTL

- Arabic (RTL) and English (LTR) are first-class
- No hardcoded user-visible strings
- Layouts must remain RTL-safe
- Direction-sensitive UI assumptions are invalid

---

## 9. UI Generation Rules

- Use Shadcn/UI and Radix primitives
- Tailwind utilities only
- Accessible by default
- Responsive layouts
- No custom UI frameworks or inline styles

---

## 10. TypeScript Discipline

- No `any`
- Explicit types where inference is unclear
- Use shared types from `/types`
- Code must compile under strict mode

---

## 11. Allowed AI Contributions

- Code generation aligned with existing patterns
- Refactoring without architectural change
- Test generation
- Documentation tied to current behavior

Forbidden:

- System redesign
- Technology substitution
- Speculative abstractions
- Introducing new global patterns

---

## 12. Governing Principle

The AI is an **implementation assistant**, not a decision-maker.

If a generated change alters:

- data flow
- trust boundaries
- ownership of responsibilities

Then the change is invalid by default.

End of instructions.

---
> Source: [algeriantechmakers/badir-core-web](https://github.com/algeriantechmakers/badir-core-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
