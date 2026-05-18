# CRM + LMS Backend Copilot Instructions

## Project Overview
This is a **NestJS + Prisma** backend for a SaaS-ready **CRM + LMS system** for education centers. The system manages organizations, leads, students, courses, groups, enrollment, attendance, and payments with multi-tenant architecture.

**Key Stack:**
- Framework: NestJS (TypeScript)
- Database: PostgreSQL + Prisma ORM
- Auth: JWT (access + refresh tokens)
- RBAC: Role-based access control (SUPER_ADMIN, ADMIN, TEACHER, STUDENT)

---

## Architecture & Data Flow

### Multi-Tenancy Model (Critical)
- **Soft multi-tenancy**: All core entities have `organization_id` foreign key
- **Every query** must scope by `WHERE organization_id = currentOrganizationId`
- Enforce in services via JWT payload's `organization_id`
- Database Service: [src/database/database.service.ts](src/database/database.service.ts) extends PrismaClient with connection pooling

### Core Business Flow (Lead → Student → Enrollment)
```
Lead (status: NEW → CONTACTED → INTERESTED → CONVERTED/LOST)
  ↓ (on CONVERT)
User (created with password) + Student record
  ↓
Enrollment → Group (with teacher assignment)
  ↓
Track: Attendance, Progress, Exams
  ↓
Payment (income tracking per student)
```

### User Roles & Access Control
| Role | Scope | Permissions |
|------|-------|-------------|
| `SUPER_ADMIN` | Platform | Manage all orgs, users |
| `ADMIN` | Organization | Full control within org |
| `TEACHER` | Assigned Groups | View/mark attendance, track progress |
| `STUDENT` | Own Data | View enrollments, grades, attendance |

**RBAC Guard:** [src/components/auth/guards/roles.guard.ts](src/components/auth/guards/roles.guard.ts) - Uses `@Roles()` decorator

---

## Key File Structure & Patterns

### Authentication Flow
1. **Login** → [src/components/auth/auth.service.ts](src/components/auth/auth.service.ts):`login()` generates JWT tokens
2. **JWT Payload** ([src/libs/types/auth.ts](src/libs/types/auth.ts)): `{ sub, email, name, role, organization_id }`
3. **Guard Stack**: `JwtAuthGuard` + `RolesGuard` + `@CurrentUser()` decorator
4. **Token Refresh**: Refresh token stored in DB, rotated on each use

### Service Layer (Dependency Injection)
- Services depend on `DatabaseService` + `AuthService` 
- Example: [src/components/user/user.service.ts](src/components/user/user.service.ts)
- Always check `organization_id` from JWT before querying
- Use `BadRequestException`, `ForbiddenException`, `UnauthorizedException` from NestJS

### DTO Pattern (Validation)
- Location: `src/libs/dto/{feature}/`
- Use `class-validator` decorators (`@IsString()`, `@IsEmail()`, etc.)
- Response DTOs define the shape returned to clients (e.g., `User`, `Organ`)
- Request DTOs for input validation

### Database Schema Location
- **Prisma Schema**: [prisma/schema.prisma](prisma/schema.prisma)
- **Generated Types**: `generated/prisma/` (auto-generated from schema)
- **Key Models**: Organization, User, Student, Lead, Course, Group, Enrollment, Attendance, Progress, Exam, Test, Payment
- **Enums**: OrganizationStatus, UserRole, StudentStatus, LeadStatus, AttendanceStatus, CourseStatus

### Module Organization
- `src/components/`: Feature modules (auth, user, etc.)
- `src/database/`: Prisma service setup
- `src/libs/`: Shared DTOs, types, enums, guards, interceptors
- Each component has: `.module.ts` (imports/exports), `.service.ts` (business logic), `.controller.ts` (routes)

---

## Critical Developer Workflows

### Database Migrations
```bash
# After schema.prisma changes, always run:
npx prisma migrate dev --name "describe_change"
# This generates migration SQL and updates generated/ types
```

### Development Server
```bash
npm run start:dev        # Watch mode (recommended for development)
npm run start:debug      # Debug mode with inspector
npm run start            # Production build
```

### Code Quality
```bash
npm run lint             # ESLint with auto-fix
npm run format           # Prettier formatting
npm run build            # Compile TypeScript
npm run test             # Jest unit tests
npm run test:e2e         # E2E tests
```

---

## Project-Specific Conventions

### 1. Organization Scoping
**Pattern:** Every service method that queries data must verify user's organization:
```typescript
// ✅ Correct
const user = await this.database.user.findFirst({
  where: { 
    id: userId, 
    organization_id: jwtPayload.organization_id 
  }
});

// ❌ Wrong - Missing organization_id check
const user = await this.database.user.findUnique({ where: { id: userId } });
```

### 2. Password Handling
- Hash with `bcrypt` (salt rounds: 10) before storing
- Compare with `bcrypt.compare()` during login
- Never return password in response DTOs
- See [src/components/auth/auth.service.ts](src/components/auth/auth.service.ts)

### 3. Error Responses
Use NestJS exceptions with specific HTTP status codes:
- `BadRequestException` (400) - Validation, duplicate entity
- `UnauthorizedException` (401) - Auth failure, missing token
- `ForbiddenException` (403) - Role/org mismatch
- `InternalServerErrorException` (500) - Unexpected errors

### 4. Lead Conversion Workflow
Converting a Lead requires atomic creation:
1. Create User (hash password)
2. Create Student with same organization_id
3. Update Lead status to CONVERTED
- Handle in transaction to prevent partial creation

### 5. Environment Variables
Required for local development (check `.env` or ask team):
- `DATABASE_URL` - PostgreSQL connection string
- `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`
- `JWT_ACCESS_EXPIRES_IN` (default: 15m), `JWT_REFRESH_EXPIRES_IN` (default: 7d)

---

## Common Integration Points

### Adding a New Feature
1. **Update Schema** [prisma/schema.prisma](prisma/schema.prisma) → Add model + run migration
2. **Create DTOs** in `src/libs/dto/{feature}/`
3. **Create Service** in `src/components/{feature}/` → Inject DatabaseService
4. **Create Controller** → Use `@JwtAuthGuard()` + `@Roles()` guards
5. **Import in Module** → Add to ComponentsModule imports

### Database Queries with Organization Context
```typescript
// From JWT payload in request
const orgId = request.user.organization_id;

// All queries scoped by org
const data = await this.database.{model}.findMany({
  where: { organization_id: orgId, ... },
});
```

### API Response Format
All endpoints follow NestJS default (automatic JSON serialization):
- Success: Returns entity or array + status code
- Error: NestJS exception → JSON error object

---

## Known Patterns & Anti-Patterns

✅ **DO:**
- Scope all queries by organization_id
- Use UUIDs for primary keys (not auto-increment)
- Hash passwords before storage
- Include created_at/updated_at on data-bearing models
- Validate DTOs at controller entry point
- Use TypeScript strict mode
- Separate business logic (service) from HTTP layer (controller)

❌ **DON'T:**
- Query across organizations without org filter
- Return passwords in response DTOs
- Store plaintext passwords
- Mix database logic into controllers
- Use soft deletes inconsistently (decide per model)
- Skip JWT organization_id validation

---

## References
- **Cursor Rules**: [.cursor/rules/crmlmsrules/RULE.md](.cursor/rules/crmlmsrules/RULE.md) - Business domain rules
- **NestJS Docs**: https://docs.nestjs.com (guards, decorators, modules)
- **Prisma Docs**: https://www.prisma.io/docs (schema, migrations, queries)

---
> Source: [NWV-LMS/back-end](https://github.com/NWV-LMS/back-end) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
