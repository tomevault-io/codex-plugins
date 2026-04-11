# Kennedy Management System - Development Guide

## System Overview
Multi-tenant SaaS platform for educational institutions (schools, tutoring centers). Each school operates as an isolated tenant with complete data separation enforced via Row Level Security (RLS) policies and application-level filtering.

**Stack:**
- Next.js 16.x (App Router with Turbopack)
- React 18
- TypeScript 5
- Supabase PostgreSQL with RLS
- Tailwind CSS 4.x with shadcn/ui
- SWR for client-side data fetching

**User Roles (hierarchical):** `owner > manager > receptionist`
- **owner** - Full access, user management, billing (future)
- **manager** - Add teachers/courses, approve payouts, view revenue
- **receptionist** - Add students, record attendance/payments

## Architecture - Three-Layer Service Pattern

```
Components → appDataService.js → databaseService.js → Supabase
     ↑              ↑                    ↑
  useAuth()    Service exports    getCurrentUserSchoolId()
```

**CRITICAL: Components must NEVER import Supabase directly.**

```javascript
// CORRECT - Components use appDataService
import { studentService } from "@/services/appDataService"
const students = await studentService.getAllStudents()

// WRONG - Never do this in components
import { supabase } from "@/lib/supabase"  // FORBIDDEN
```

**Multi-tenant security is automatic:**
- `databaseService.js` injects `school_id` into ALL queries via `getCurrentUserSchoolId()`
- Database RLS policies provide secondary enforcement
- SQL schemas in `/sql/` folder - DO NOT MODIFY without discussion

## Authentication Flow

**Owner Registration:** `/auth/create-school`
1. Create school via `supabase.rpc('create_school_for_owner', {...})` - bypasses RLS
2. Sign up user with `is_owner_signup: 'true'` in metadata
3. Database trigger creates profile automatically
4. Redirect to email confirmation → `/manager`

**Invited User:** `/auth/signup?token=X&email=Y`
1. Validate invitation token
2. Sign up with `invitation_token` in metadata
3. Trigger assigns role from invitation → role-based redirect

**Session Management:**
```typescript
// useSessionManager.ts - Centralized session with debouncing
const { sessionData, isValidating, refreshSession } = useSessionManager()

// AuthContext.tsx - Components use this
const { user, hasRole, canAccess, signIn, signOut } = useAuth()
```

## Required Patterns

**Page Protection:**
```tsx
// Wrap protected pages with AuthGuard
<AuthGuard requiredRoles={['owner', 'manager']}>
  <ManagerDashboard />
</AuthGuard>
```

**Role Checking:**
```typescript
const { hasRole, canAccess } = useAuth()
if (!hasRole(['owner', 'manager'])) return null
if (!canAccess('payments')) return <AccessDenied />
```

**Parallel Data Loading (see StudentsTab.tsx):**
```typescript
const [students, teachers, courses] = await Promise.all([
  studentService.getAllStudents(),
  teacherService.getAllTeachers(),
  courseService.getAllCourseInstances()
])
```

**Defensive Programming:**
```typescript
return data || []           // Never return null from services
user?.profile?.role         // Always use optional chaining
const items = data ?? []    // Nullish coalescing for defaults
```

## Project Structure

```
/app                    - Next.js App Router (role-based routing)
  /auth/*               - Authentication pages
  /manager/page.tsx     - Owner/Manager dashboard  
  /receptionist/page.tsx - Receptionist dashboard
  /student/[id]/        - Student detail pages
/components
  /auth/AuthGuard.tsx   - Route protection component
  /tabs/*               - Feature tab components (StudentsTab, etc.)
  /ui/*                 - shadcn/ui components
/services
  appDataService.js     - Public API (components import from here)
  databaseService.js    - Supabase operations with school_id injection
  authService.js        - Authentication (signIn, signUp, createSchool)
/contexts/AuthContext.tsx - Global auth state
/hooks/useSessionManager.ts - Session validation with debouncing
/utils/api-error-handler.ts - Standardized error handling
```

## Database Functions (use via RPC)

```typescript
// School creation (bypasses RLS for anonymous users)
supabase.rpc('create_school_for_owner', { p_school_name, p_school_address, p_school_phone, p_school_email })

// Session validation
supabase.rpc('get_current_user_session')  // Returns { authenticated, profile, school, permissions }

// Profile fallback creation
supabase.rpc('create_owner_profile_manual', { p_user_id, p_school_id, p_full_name, p_phone })
```

## Database Tables

**Auth Tables (`auth_schema.sql`):**
- `schools` - Tenant organizations
- `profiles` - User profiles (linked to auth.users)
- `invitations` - Token-based user invitations

**Business Tables (`business_schema.sql`):**
- `students` - Student records with academic info
- `teachers` - Teacher records with subjects/years
- `course_instances` - Courses with pricing, schedule, enrolled students
- `attendance` - Weekly attendance per student per course
- `student_payments` - Payment records (status: pending/paid/cancelled)
- `teacher_payouts` - Teacher payment records (status: pending/approved/paid)
- `revenue` - Revenue tracking
- `archive_requests` - Soft-delete workflow

## Payment Status Values

Teacher payouts use three statuses - treat `approved` AND `paid` as "completed":
```typescript
// Both are terminal paid states
const isPaid = status === 'paid' || status === 'approved'
```

## Commands

```powershell
npm install             # Install dependencies
npm run dev             # Development server (port 3000)
npm run build           # Production build (TS/ESLint errors ignored)
```

## Code Standards

- No emojis in code or commit messages
- Use `getErrorMessage()` from `/utils/api-error-handler.ts` for user-facing errors
- Conditional logging only: `const DEBUG = process.env.NODE_ENV === 'development'`
- All async operations wrapped in try/catch with graceful fallbacks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WRH-05)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/WRH-05)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
