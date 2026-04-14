# AI Coding Agent Instructions for Skill-Stacker

## Project Overview

**Skill-Stacker** is an AI-powered CV Builder built with Next.js 15, React 19, TypeScript, and MySQL. It allows users to create, edit, export (PDF/DOCX), and manage multiple CVs through an intuitive form-based interface with real-time preview.

### Architecture

- **Frontend**: Next.js app router with React components (Tailwind CSS, Lucide Icons)
- **Backend**: Next.js API routes (`/api/cv` and `/api/cv/[id]`)
- **Database**: MySQL with connection pooling (`mysql2/promise`)
- **Export**: `@react-pdf/renderer` (PDF) and `docx` library (DOCX)

## Essential Patterns

### 1. State Management & Component Hierarchy

**CVBuilderApp** (`app/components/CVBuilderApp.tsx`) is the root component managing all CV data. It uses React hooks for state:

```tsx
// Pattern: Local state for all CV sections
const [personal, setPersonal] = useState({fullName, title, phone, email, location, linkedin});
const [experiences, setExperiences] = useState([{id, company, role, period, details}]);
// ...similar for education, skills, certificates, references, competencies, additionalInfo
```

**Key convention**: Array items have an `id` field (generated via `Math.random().toString(36).substr(2, 9)`) for React keys and field updates.

### 2. Form Handlers Pattern

Forms follow a consistent pattern for array items:

```tsx
const addExperience = () => setExperiences(prev => [...prev, {id: generateId(), ...}]);
const updateExperience = (id, field, value) => 
  setExperiences(prev => prev.map(item => item.id === id ? {...item, [field]: value} : item));
const removeExperience = (id) => 
  setExperiences(prev => prev.filter(item => item.id !== id));
```

For single-value fields (profile, skills, competencies):
```tsx
const updateSkill = (idx, value) => setSkill(prev => prev.map((v, i) => i === idx ? value : v));
```

### 3. Database Integration

**saveCV()** creates a new CV with nested records:
- Inserts personal info into `cvs` table
- Uses transactions and cascading child table insertions (experiences, education, skills, etc.)

**updateCV()** deletes and re-inserts child records (no partial updates to nested items).

**Key limitation**: When updating a CV, all related records are deleted and recreated. Pass full arrays including items with `id` field stripped (see `handleSaveToDatabase` in CVBuilderApp.tsx).

### 4. API Response Format

All endpoints follow this pattern:

```ts
return NextResponse.json({
  success: true,          // boolean
  message: "...",         // optional
  cvId?: number,          // for POST/PUT
  cv?: CVData,           // for GET single
  cvs?: CVData[],        // for GET list
  error?: string         // on failure
}, { status: 200|201|400|404|500 });
```

**Error handling**: Wrap with try-catch, always return standardized JSON response.

### 5. TypeScript Types

Type definitions in `app/types/global.ts`:

```ts
type Experience = {id: string|number; company: string; role: string; period: string; details: string};
type PersonalInfo = {fullName, title, phone, email, location, linkedin};
```

**Convention**: Form component props take the data object + handler functions (update, add, remove), not the full state.

### 6. Export Module

The `exportModule.tsx` exports to PDF and DOCX:

```tsx
// PDF: Uses @react-pdf/renderer with CVDocument component
exportToPdf(props) → generates blob → saveAs(blob, filename)

// DOCX: Uses docx library to build document structure
exportToDocx(props) → creates Document + Packer → saveAs(blob, filename)
```

**Important**: Both functions accept `ExportFunctionProps` type which includes all CV data.

## Developer Workflows

### Starting Development

```bash
npm run dev              # Start Next.js dev server with Turbopack
npm run build           # Build with Turbopack
npm run lint            # Check with Biome
npm run format --write  # Format with Biome
```

### Database Setup

1. Ensure MySQL is running on `localhost:3306` (XAMPP/MySQL Server)
2. Create `.env.local` with:
   ```
   DB_HOST=localhost
   DB_USER=root
   DB_PASSWORD=
   DB_NAME=cvbuilder
   ```
3. Run test script: `npx ts-node scripts/test-database.ts`
4. Database tables auto-initialize on first API call via `initDb()`

### Adding a New CV Section

1. Create form component in `app/components/Forms/NewSectionForm.tsx` (follow pattern of ExperienceForm)
2. Add type definitions in `app/types/global.ts`
3. Add database table in `initDb()` with foreign key to `cvs(id)` and `ON DELETE CASCADE`
4. Add `saveCV()` insert logic in `app/lib/db.ts`
5. Add state, handlers, and form in `CVBuilderApp.tsx`
6. Connect form to `CVBuilderForm.tsx`
7. Update `exportModule.tsx` CVDocument component to render the section

## Project-Specific Conventions

### Naming

- **Components**: PascalCase (PersonalInfoForm, CVBuilderApp)
- **Functions**: camelCase (handleSaveToDatabase, updateExperience)
- **Database tables**: snake_case plural (cvs, experiences, competencies)
- **CSS**: Tailwind utility-first (no custom classes in components)

### Path Aliases

TypeScript `tsconfig.json` uses `@/*` → root for imports:
```tsx
import { CVBuilderApp } from "@/app/components/CVBuilderApp";
import type { ExportFunctionProps } from "@/app/types/global";
```

### Notification System

Use the `useNotification()` hook for user feedback:
```tsx
const {showSuccess, showError, showInfo} = useNotification();
showSuccess("Title", "Message");  // green notification
showError("Title", "Message");     // red notification
```

The hook uses `NotificationModal` component and state management.

### Key File Dependencies

- **CVBuilderApp.tsx**: Orchestrates entire app, imports all forms and utilities
- **db.ts**: All database operations, MySQL pool management
- **exportModule.tsx**: PDF/DOCX generation, imported by CVBuilderApp
- **global.ts**: All TypeScript interfaces, single source of truth for types
- **API routes**: Entry points for database operations, follow standardized response format

## Critical Developer Notes

1. **Database Transactions**: `updateCV()` uses transactions; ensure connections are released in finally blocks
2. **Form Data Cleanup**: When saving to DB, strip `id` field from array items (Next.js-specific IDs, not DB IDs)
3. **MySQL Pool**: Connection pool is singleton in `db.ts`, auto-creates on first `getPool()` call
4. **Client-Side State**: React state handles form validation; minimal error handling (improve as needed)
5. **No Incremental Updates**: Current DB design re-inserts all child records on CV update (inefficient for large CVs; consider refactoring)

## Testing Database

```bash
# Test database connectivity and schema
npx ts-node scripts/test-database.ts
```

Verify tables exist and connection works before deploying.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giiftmor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
