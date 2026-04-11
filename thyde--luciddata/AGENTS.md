# LucidData - GitHub Copilot Instructions

## Project Overview

**LucidData** is a privacy-first personal data bank MVP that empowers users to own, control, and share their data on their terms. Built with Next.js 15, Prisma, and Supabase Auth, it emphasizes user data sovereignty, transparency through immutable audit trails, and data portability through open standards.

**Core Philosophy:**
- Treat user data as property, not product
- Transparency through comprehensive audit logging
- Granular consent management with time-bound permissions
- End-to-end encryption with user-controlled keys (future)
- Data portability via open standards (JSON-LD, VerifiableCredential, FHIR)

**Current State:** MVP with infrastructure and security utilities production-ready. API routes currently use mock data; ready for database integration.

---

## 📚 Documentation Map

### Critical Patterns (Always Reference Before Implementing)
- **[Security Patterns](.github/copilot/critical/security-patterns.md)** - Encryption, authentication, audit logging (Phase 2)
- **[API Route Patterns](.github/copilot/critical/api-patterns.md)** - Standard structure, validation, error handling (Phase 2)
- **[Database Schema](.github/copilot/critical/database-schema.md)** - Prisma models, relationships (Phase 2)

### Common Patterns (Frequently Referenced)
- **[Component Patterns](.github/copilot/common/component-patterns.md)** - Server/Client components, shadcn/ui, layouts (Phase 3)
- **[Code Examples](.github/copilot/common/code-examples.md)** - Complete CRUD flows for vault, consent (Phase 3)
- **[Validation Patterns](.github/copilot/common/validation-patterns.md)** - Zod schemas and usage (Phase 3)

### Reference Documentation (Occasionally Referenced)
- **[Coding Conventions](.github/copilot/reference/coding-conventions.md)** - Naming, imports, file organization (Phase 3)
- **[Tech Stack Reference](.github/copilot/reference/tech-stack.md)** - Libraries, versions, configuration (Phase 3)
- **[Development Workflow](QUICKSTART.md)** - Daily commands, troubleshooting
- **[Initial Setup](SETUP.md)** - One-time configuration, service URLs, architecture
- **[Project Overview](README.md)** - Vision, roadmap, high-level architecture

**How to use this map:**
1. Check Core Security Patterns (below) for critical rules that apply to ALL features
2. Read relevant pattern file from Critical Patterns before implementing features
3. Use Common Patterns for templates and examples
4. Reference other docs as needed for setup, commands, or conventions

---

## Core Security Patterns

**These rules apply to ALL features. Failure to follow results in security vulnerabilities.**

### Encryption (AES-256-GCM)
- **Always encrypt** vault data before storing using `lib/crypto/encryption.ts`
- **Never log** decrypted data in errors, logs, or responses
- **Use `getMasterKey()`** only - never access `process.env.ENCRYPTION_KEY` directly
- **Store IV format:** `"hexIv:hexAuthTag"` in the `iv` field (colon-separated hex strings)

### Audit Logging (Hash Chains)
- **Create audit log** for ALL vault/consent operations (create, read, update, delete, grant, revoke)
- **Include `previousHash`** to maintain immutable chain integrity
- **Use `lib/crypto/hashing.ts`** - `createAuditHash()` for hash generation
- **Never modify** existing audit log entries (immutable by design)

### Authentication (Supabase)
- **Always validate** user with `lib/supabase/server.ts` in API routes
- **Always check** `user` exists before proceeding (never assume authentication)
- **Filter all queries** by `userId` - never trust client-provided IDs
- **Use correct Supabase client:** server for API routes, client for Client Components

**For detailed code examples, see:** [Security Patterns documentation](.github/copilot/critical/security-patterns.md) (Phase 2)

---

## Tech Stack

**For comprehensive stack overview, see [README.md](README.md). Critical versions and configuration notes below.**

### Core Framework
- **Next.js 15.1.3** with App Router (React 19) - Path alias: `@/*` maps to project root
- **TypeScript 5** with strict mode enabled
- **Tailwind CSS 3.4.1** for styling

### Database & ORM
- **PostgreSQL** via Supabase
- **Prisma 6.19.1** (downgraded from 7.x for stability) - Use `npx dotenv -e .env.local -- npx prisma <command>`

### Authentication
- **Supabase Auth** (`@supabase/supabase-js ^2.89.0`, `@supabase/ssr ^0.8.0`)
- Server-side session management with cookies
- Use `@/lib/supabase/server` (await `createClient()`) in API routes and Server Components
- Use `@/lib/supabase/client` in Client Components

### UI & Components
- **shadcn/ui** with Radix UI primitives, **Lucide React 0.562.0** icons
- Use `cn()` utility for className merging (`clsx` + `tailwind-merge`)

### Forms & Validation
- **React Hook Form 7.70.0** with **Zod 4.3.5** - All inputs MUST be validated
- Use `.parse()` in API routes (throws), `.safeParse()` in forms (returns result)

### Security
- Node.js native `crypto` with custom **AES-256-GCM** encryption, **SHA-256** hashing
- Utilities: `lib/crypto/encryption.ts`, `lib/crypto/hashing.ts`

---

## Database Schema

### User (`users`)
```prisma
model User {
  id                String   @id @default(uuid())
  email             String   @unique
  encryptionKeyHash String?  // For future user-controlled encryption
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  
  vaultData    VaultData[]
  consents     Consent[]
  auditLogs    AuditLog[]
  exportRequests ExportRequest[]
}
```
**Note:** Synced with Supabase Auth users

### VaultData (`vault_data`)
```prisma
model VaultData {
  id            String   @id @default(uuid())
  userId        String
  encryptedData String   // AES-256-GCM encrypted JSON
  iv            String   // Format: "iv:authTag" (hex strings)
  keyId         String   @default("master-key-1")
  label         String
  category      VaultCategory // personal, health, financial, credentials, other
  tags          String[]
  schemaType    String?  // e.g., "Person", "MedicalRecord"
  schemaVersion String?  // For JSON-LD/FHIR portability
  expiresAt     DateTime?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  consents  Consent[]
  auditLogs AuditLog[]
}
```
**Encryption:** Field-level encryption on `encryptedData`. Metadata (`label`, `category`, `tags`) unencrypted for queryability.

### Consent (`consents`)
```prisma
model Consent {
  id            String      @id @default(uuid())
  userId        String
  vaultDataId   String?
  grantedTo     String      // Organization/service name
  purpose       String
  permissions   Permission[] // read, export, verify
  status        ConsentStatus // active, revoked, expired
  startDate     DateTime
  endDate       DateTime?
  revokedAt     DateTime?
  revokedBy     String?
  revokedReason String?
  consentText   String
  termsVersion  String
  ipAddress     String?
  userAgent     String?
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
  
  user       User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  vaultData  VaultData?  @relation(fields: [vaultDataId], references: [id], onDelete: SetNull)
  auditLogs  AuditLog[]
}
```
**Time-bound permissions:** Use `startDate`, `endDate` for temporal access control.

### AuditLog (`audit_logs`)
```prisma
model AuditLog {
  id            String   @id @default(uuid())
  userId        String
  vaultDataId   String?
  consentId     String?
  eventType     EventType // data_created, data_accessed, consent_granted, etc.
  action        String
  actorId       String
  actorType     ActorType // user, system, service
  currentHash   String   // SHA-256 hash of current log entry
  previousHash  String?  // SHA-256 hash of previous log entry (hash chain)
  ipAddress     String?
  userAgent     String?
  metadata      Json?
  success       Boolean  @default(true)
  errorMessage  String?
  timestamp     DateTime @default(now())
  
  user      User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  vaultData VaultData? @relation(fields: [vaultDataId], references: [id], onDelete: SetNull)
  consent   Consent?   @relation(fields: [consentId], references: [id], onDelete: SetNull)
}
```
**Hash chain:** Each entry's `currentHash` includes `previousHash`, creating tamper-evident trail. Verify integrity with `verifyHashChain()` from `lib/crypto/hashing.ts`.

### ExportRequest (`export_requests`)
```prisma
model ExportRequest {
  id               String        @id @default(uuid())
  userId           String
  format           ExportFormat  // json_ld, verifiable_credential, csv
  status           ExportStatus  // pending, processing, completed, failed
  includeCategories VaultCategory[]
  includeAuditLog  Boolean      @default(false)
  fileUrl          String?
  expiresAt        DateTime?
  errorMessage     String?
  requestedAt      DateTime     @default(now())
  completedAt      DateTime?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

---

## Security & Encryption Requirements

**For detailed security patterns including encryption, audit logging, and authentication with code examples, see:** [Security Patterns documentation](.github/copilot/critical/security-patterns.md)

Core rules are shown in [Core Security Patterns](#core-security-patterns) above.

---

## API Route Patterns

**For complete API route patterns with code examples, see:** [API Route Patterns documentation](.github/copilot/critical/api-patterns.md)

Key patterns: GET/POST/PATCH/DELETE, authentication, validation, error handling, pagination, filtering

---

## Component Structure & Patterns

### Server vs Client Components
**Default to Server Components:**
```typescript
// app/(dashboard)/vault/page.tsx
// No 'use client' directive = Server Component
export default async function VaultPage() {
  // Can directly access database, auth, etc.
  return <div>Content</div>;
}
```

**Use Client Components only when needed:**
```typescript
'use client'; // Required for useState, useEffect, event handlers, etc.
import { useState } from 'react';

export function VaultForm() {
  const [data, setData] = useState('');
  // Interactive component
}
```

### shadcn/ui Component Pattern
```typescript
// components/ui/button.tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input hover:bg-accent",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3",
        lg: "h-11 px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
```

### Layout Pattern
```typescript
// app/(dashboard)/layout.tsx
import { createClient } from '@/lib/supabase/server';
import { redirect } from 'next/navigation';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  
  if (!user) {
    redirect('/login');
  }
  
  return (
    <div>
      <nav>{/* Navigation */}</nav>
      <main>{children}</main>
    </div>
  );
}
```

**Rules:**
- Use Server Components by default
- Add `'use client'` only for interactivity (state, effects, event handlers)
- Use `cn()` utility for merging classNames
- Follow shadcn/ui patterns for new UI components
- Validate auth in layouts for protected routes

---

## Zod Validation Schemas

**Zod validation patterns are documented in:** [API Route Patterns](.github/copilot/critical/api-patterns.md#zod-validation)

Summary: Use schemas from `lib/validations/`, `.parse()` in API routes, `.safeParse()` in forms.

---

## Development Workflow

**For daily development commands and troubleshooting, see [QUICKSTART.md](QUICKSTART.md).**

**For service URLs, environment setup, and initial configuration, see [SETUP.md](SETUP.md).**

### Quick Reference
- **Start dev server:** `npm run dev` (http://localhost:3000)
- **Supabase Studio:** http://127.0.0.1:54323 (database admin)
- **Prisma commands:** Must use `npx dotenv -e .env.local -- npx prisma <command>`

### Critical Environment Variables
- `ENCRYPTION_KEY` - 256-bit AES key (base64-encoded) - **Never change in production**
- `DATABASE_URL` - PostgreSQL connection string
- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Supabase config

**⚠️ CRITICAL:** Never commit `.env.local` to git. Changing `ENCRYPTION_KEY` makes existing data unreadable.

---

## Coding Conventions

### Naming Conventions
- **Files**: kebab-case (`vault-data.ts`, `encryption.ts`)
- **Components**: PascalCase (`VaultPage`, `Button`, `AuditLogTable`)
- **Functions**: camelCase (`createClient`, `generateKey`, `verifyHashChain`)
- **Types/Interfaces**: PascalCase (`VaultDataInput`, `User`, `ConsentStatus`)
- **Constants**: UPPER_SNAKE_CASE (`ALGORITHM`, `KEY_LENGTH`, `IV_LENGTH`)
- **Database**: snake_case (`vault_data`, `audit_logs`, `created_at`)
- **API Routes**: kebab-case, RESTful (`/api/vault`, `/api/consent/[id]`)

### File Organization
```
app/
  (auth)/          # Route group: auth pages
  (dashboard)/     # Route group: protected pages
  api/             # API routes
components/
  ui/              # shadcn/ui components
  vault/           # Feature-specific components
  consent/
  audit/
lib/
  crypto/          # Encryption & hashing utilities
  db/              # Prisma client & queries
  supabase/        # Supabase clients
  validations/     # Zod schemas
  hooks/           # Custom React hooks
prisma/
  schema.prisma    # Database schema
  seed.ts          # Seed data
types/             # Shared TypeScript types
```

### Import Order
```typescript
// 1. React/Next.js
import { useState } from 'react';
import { redirect } from 'next/navigation';

// 2. External libraries
import { z } from 'zod';
import { useForm } from 'react-hook-form';

// 3. Internal utilities (@ alias)
import { createClient } from '@/lib/supabase/server';
import { prisma } from '@/lib/db/prisma';
import { encrypt, decrypt } from '@/lib/crypto/encryption';

// 4. Components (@ alias)
import { Button } from '@/components/ui/button';
import { VaultCard } from '@/components/vault/vault-card';

// 5. Types (@ alias)
import type { VaultDataInput } from '@/lib/validations/vault';
```

### TypeScript Rules
- Enable strict mode (already configured)
- Always type function parameters and return values
- Use `type` for object shapes, `interface` for extensible contracts
- Prefer `const` over `let`
- Use optional chaining (`?.`) and nullish coalescing (`??`)

---

## Common Patterns & Examples

### Complete Vault Data Creation Flow
```typescript
// app/api/vault/route.ts
import { createClient } from '@/lib/supabase/server';
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/db/prisma';
import { vaultDataSchema } from '@/lib/validations/vault';
import { encrypt, getMasterKey } from '@/lib/crypto/encryption';
import { createAuditHash } from '@/lib/crypto/hashing';
import { z } from 'zod';

export async function POST(request: Request) {
  try {
    // 1. Authenticate
    const supabase = await createClient();
    const { data: { user } } = await supabase.auth.getUser();
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }
    
    // 2. Validate input
    const body = await request.json();
    const validated = vaultDataSchema.parse(body);
    
    // 3. Encrypt data
    const masterKey = getMasterKey();
    const { encrypted, iv, authTag } = encrypt(
      JSON.stringify(validated.data),
      masterKey
    );
    
    // 4. Store in database
    const vaultEntry = await prisma.vaultData.create({
      data: {
        userId: user.id,
        encryptedData: encrypted,
        iv: `${iv}:${authTag}`,
        label: validated.label,
        category: validated.category,
        tags: validated.tags,
        schemaType: validated.schemaType,
        schemaVersion: validated.schemaVersion,
        expiresAt: validated.expiresAt,
      },
    });
    
    // 5. Create audit log with hash chain
    const previousLog = await prisma.auditLog.findFirst({
      where: { userId: user.id },
      orderBy: { timestamp: 'desc' },
      select: { currentHash: true },
    });
    
    const currentHash = createAuditHash(
      previousLog?.currentHash ?? null,
      {
        userId: user.id,
        eventType: 'data_created',
        action: `Created vault entry: ${validated.label}`,
        actorId: user.id,
        actorType: 'user',
        timestamp: new Date(),
      }
    );
    
    await prisma.auditLog.create({
      data: {
        userId: user.id,
        vaultDataId: vaultEntry.id,
        eventType: 'data_created',
        action: `Created vault entry: ${validated.label}`,
        actorId: user.id,
        actorType: 'user',
        currentHash,
        previousHash: previousLog?.currentHash ?? null,
      },
    });
    
    // 6. Return response
    return NextResponse.json(vaultEntry, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    console.error('Error creating vault entry:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Reading & Decrypting Vault Data
```typescript
// app/api/vault/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // Fetch with user ownership check
  const vaultEntry = await prisma.vaultData.findUnique({
    where: {
      id: params.id,
      userId: user.id, // Security: ensure user owns this data
    },
  });
  
  if (!vaultEntry) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }
  
  // Decrypt data
  const [ivHex, authTagHex] = vaultEntry.iv.split(':');
  const masterKey = getMasterKey();
  const decrypted = decrypt(
    vaultEntry.encryptedData,
    masterKey,
    ivHex,
    authTagHex
  );
  const data = JSON.parse(decrypted);
  
  // Create audit log for data access
  const previousLog = await prisma.auditLog.findFirst({
    where: { userId: user.id },
    orderBy: { timestamp: 'desc' },
    select: { currentHash: true },
  });
  
  const currentHash = createAuditHash(
    previousLog?.currentHash ?? null,
    {
      userId: user.id,
      eventType: 'data_accessed',
      action: `Accessed vault entry: ${vaultEntry.label}`,
      actorId: user.id,
      actorType: 'user',
      timestamp: new Date(),
    }
  );
  
  await prisma.auditLog.create({
    data: {
      userId: user.id,
      vaultDataId: vaultEntry.id,
      eventType: 'data_accessed',
      action: `Accessed vault entry: ${vaultEntry.label}`,
      actorId: user.id,
      actorType: 'user',
      currentHash,
      previousHash: previousLog?.currentHash ?? null,
    },
  });
  
  // Return decrypted data with metadata
  return NextResponse.json({
    ...vaultEntry,
    data, // Decrypted data
  });
}
```

---

## Testing & Demo Data

**For test user credentials and seeded data details, see [SETUP.md](SETUP.md#database-seeding).**

**Quick reference:** Test user: `demo@lucid.dev`, seed with `npx dotenv -e .env.local -- npx prisma db seed`

---

## Documentation References

### Project Documentation
- **[README.md](README.md)** - Project vision, roadmap, comprehensive tech stack overview
- **[SETUP.md](SETUP.md)** - Initial setup, architecture notes, service URLs, troubleshooting
- **[QUICKSTART.md](QUICKSTART.md)** - Daily development workflow, commands, common tasks

### Copilot Instructions (Modular)
- **[Documentation Map](#-documentation-map)** - Navigation to all pattern files (this document)
- **[Core Security Patterns](#core-security-patterns)** - Critical rules for encryption, audit, auth (this document)
- **Critical Patterns** - `.github/copilot/critical/` (Phase 2: security-patterns.md, api-patterns.md, database-schema.md)
- **Common Patterns** - `.github/copilot/common/` (Phase 3: component-patterns.md, code-examples.md, validation-patterns.md)
- **Reference** - `.github/copilot/reference/` (Phase 3: coding-conventions.md, tech-stack.md)

### Code Reference
- **Prisma Schema** - [prisma/schema.prisma](prisma/schema.prisma) for complete data models
- **Security Utilities** - [lib/crypto/encryption.ts](lib/crypto/encryption.ts), [lib/crypto/hashing.ts](lib/crypto/hashing.ts)
- **Validations** - [lib/validations/](lib/validations/) for Zod schemas (vault.ts, consent.ts, user.ts)

---

## Important Architectural Notes

1. **Current MVP State**: API routes use mock data. Infrastructure (encryption, hashing, auth, database schema) is production-ready and awaiting integration.

2. **Security-First**:
   - Always encrypt vault data before storing
   - Always create audit logs for sensitive operations
   - Always validate auth before allowing access
   - Never expose decrypted data in logs or errors

3. **Hash Chain Integrity**: Audit logs form an immutable chain. Any modification breaks the chain and is detectable via `verifyHashChain()`.

4. **User Data Ownership**: All queries must filter by `userId`. Never allow users to access others' data.

5. **Prisma Version**: Using 6.19.1 (downgraded from 7.x) for stability. Standard `schema.prisma` format (no config file needed).

6. **Future Enhancements**:
   - User-controlled encryption (PBKDF2 utilities ready in `lib/crypto/encryption.ts`)
   - TanStack Query integration for client-side data fetching
   - Database query abstraction layer (`lib/db/queries/` currently empty)
   - Production password hashing (switch from SHA-256 to bcrypt/argon2)

---

## Quick Reference Checklist

When implementing new features:

- [ ] Validate all inputs with Zod schemas
- [ ] Authenticate user with Supabase in API routes
- [ ] Encrypt sensitive data with AES-256-GCM before storing
- [ ] Create audit log with hash chain for sensitive operations
- [ ] Filter database queries by `userId` for security
- [ ] Use Server Components by default, add `'use client'` only when needed
- [ ] Follow naming conventions (PascalCase components, camelCase functions, kebab-case files)
- [ ] Handle errors gracefully with appropriate HTTP status codes
- [ ] Use `cn()` utility for className merging in components
- [ ] Return appropriate HTTP status codes (200, 201, 400, 401, 404, 500)

---

**Last Updated**: January 4, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thyde)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/thyde)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
