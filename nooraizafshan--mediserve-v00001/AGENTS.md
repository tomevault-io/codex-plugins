# MediServe Copilot Instructions

AI coding agents should understand these architecture patterns before implementation.

## Project Overview

**MediServe** is a medical report management system for hospitals/diagnostic centers. It automates workflow from lab uploads → consultant verification → departmental approvals → QR-validated certificate generation.

Tech Stack: Next.js 14+ App Router, TypeScript, MongoDB/Mongoose, Cloudinary, Tailwind CSS

## Critical Business Rule: Department-Based Lab Records

**NEVER use generic test arrays.** Lab tests MUST be stored as department-nested attributes:

```typescript
// ✓ CORRECT
departments.medicine.HCV = { value: "Positive", fileUrl: "..." }
departments.ophthalmology.LeftEyeVision = "6/6"

// ✗ WRONG - DO NOT USE
tests: [{ name: "HCV", value: "Positive" }]
```

This rule applies to:
- `src/models/LabRecord.ts` (schema design)
- `src/app/api/lab-record/**` (all API routes)
- `src/components/LabUploadForm.tsx` (form submission)
- `src/lib/services/LabRecordService.ts` (business logic)

## Architecture Layers

### 1. Models (`src/models/`)
- `LabRecord.ts`: Department-nested schema with 4 departments (medicine, ophthalmology, surgery, psychiatry)
- `caseModel.ts`: Medical case workflow (pending → payment-pending → lab-complete → approved)
- `userModel.ts`: Role-based access (admin, deo, consultant, dms, ms, it-support)

**Pattern**: Mongoose schemas with strict mode enabled, timestamps, and indexing.

### 2. Database Connection (`src/lib/mongooseConnect.ts`)
- Global cached connection for Next.js serverless functions
- Connection pooling (5-10 connections)
- Graceful shutdown on SIGINT

**Pattern**: Always use `await connectDB()` before model operations.

### 3. Services (`src/lib/services/LabRecordService.ts`)
- Abstraction layer between API routes and models
- Static methods for all CRUD operations
- ObjectId validation on all inputs
- Error handling with descriptive messages

**Pattern**: Use DTOs (Data Transfer Objects) for method parameters.

### 4. API Routes (`src/app/api/`)
- RESTful endpoints that delegate to services
- Input validation (file size, type, format)
- Consistent response format: `{ success, data, error }`
- Error logging with context

**Pattern**: Middleware-like approach for validation, then service call, then response.

### 5. Components (`src/components/`)
- Client-side forms (`'use client'` directive)
- State management with `useState` hooks
- Form validation before submission
- Toast/alert notifications for feedback

**Pattern**: Pass callbacks (`onSubmit`, `onError`) from parent pages.

## Core Workflows

### Workflow 1: Create & Upload Lab Record

```
1. Page calls LabRecordService.createLabRecord({ patientId, caseId, patientName })
   → Creates document with empty departments
   
2. User clicks LabUploadForm
   → Selects department + field + file
   → Calls POST /api/lab-record/upload
   
3. API route:
   a) Validate file (size, type)
   b) Upload to Cloudinary with upload_stream
   c) Update nested field: departments.${dept}.${field}.fileUrl = cloudinaryUrl
   d) Track file in uploadedFiles array
   e) Return updated document
```

### Workflow 2: Approve/Reject Lab Record

```
1. Consultant views lab record
2. Reviews all department fields
3. Calls LabRecordService.markAsApproved(recordId, consultantId, notes)
   → Sets status = "approved", approvedAt = now, approvedBy = consultantId
4. Case status updates to "approved"
5. DMS initiates certificate generation
```

## File Reference Guide

| File | Purpose | Key Functions |
|------|---------|---|
| `src/models/LabRecord.ts` | Mongoose schema | `updateDepartmentField()`, `addUploadedFile()`, `markAsApproved()` |
| `src/lib/mongooseConnect.ts` | DB connection | `connectDB()` |
| `src/lib/cloudinary.ts` | File upload | `uploadToCloudinary()`, `deleteFromCloudinary()` |
| `src/app/api/lab-record/upload/route.ts` | Upload endpoint | `POST /api/lab-record/upload` |
| `src/components/LabUploadForm.tsx` | Upload form UI | Form state, validation, API call |
| `src/lib/services/LabRecordService.ts` | Business logic | `createLabRecord()`, `updateDepartmentField()`, `markAsApproved()` |

## Common Patterns

### Pattern 1: Validate ObjectId
```typescript
if (!mongoose.Types.ObjectId.isValid(userId)) {
  throw new Error('Invalid user ID format');
}
```

### Pattern 2: Update Nested Field
```typescript
const updatePath = `departments.${department}.${fieldName}`;
await LabRecordModel.findByIdAndUpdate(
  id,
  { $set: { [updatePath]: value } },
  { new: true }
);
```

### Pattern 3: Fetch with Populate
```typescript
const record = await LabRecordModel.findById(id)
  .populate('patientId', 'name email')
  .populate('caseId', 'caseId status');
```

### Pattern 4: API Response
```typescript
return NextResponse.json({
  success: true,
  data: savedRecord.toObject()
}, { status: 200 });
```

## Department Field Mapping

Reference when implementing file uploads or field updates:

**Medicine** (15 fields):
- Tests with value+fileUrl: HBS, HCV, HIV, VDRL, TB, DM
- Tests with fileUrl only: ChestXray, ECG
- Vitals (numbers): Height, PulseRate, RespirationRate, Temperature
- Strings: BloodPressure, BloodGroup, InfectiousDiseaseScreening

**Ophthalmology** (3 fields):
- LeftEyeVision, RightEyeVision, FundusExam (all strings)

**Surgery** (4 fields):
- GeneralExam, HerniaCheck, DisabilityAssessment, SurgicalOpinion (all strings)

**Psychiatry** (2 fields):
- PsychologicalAssessment, MentalHealthOpinion (all strings)

## Environment Variables

Always validate in any API route that uses them:

```typescript
function validateEnvironment(): void {
  const required = ['MONGODB_URI', 'CLOUDINARY_API_KEY'];
  const missing = required.filter((env) => !process.env[env]);
  if (missing.length > 0) {
    throw new Error(`Missing: ${missing.join(', ')}`);
  }
}
```

## Code Style Standards

- TypeScript strict mode enabled in `tsconfig.json`
- Mongoose strict mode enabled: `{ strict: true }`
- JSDoc comments on exported functions
- Inline comments only for non-obvious logic
- No commented-out code
- Error messages specific to context
- Path aliases: `@/` → `src/`

## Testing Checklist

Before submitting code:
- [ ] All required ObjectIds are validated
- [ ] File uploads tested with various file types
- [ ] API errors include descriptive messages
- [ ] Database connection tested in new API route
- [ ] No hardcoded values or test data left
- [ ] Environment variables documented in `.env.example`
- [ ] TypeScript types exported from models

## Known Gotchas

1. **Cloudinary upload_stream**: Must end stream with `uploadStream.end(buffer)`
2. **Mongoose toObject()**: Call before returning from API to avoid circular references
3. **FormData parsing**: Use `const file = formData.get('file') as File` (type assertion needed)
4. **ObjectId conversion**: New ObjectId needed for Mongoose queries: `new mongoose.Types.ObjectId(id)`
5. **Strict mode**: Can't set undefined fields; use `{ $set: { field: value } }` syntax

## Folder Structure

```
src/
├── app/
│   ├── api/
│   │   ├── lab-record/upload/route.ts   ← File upload endpoint
│   │   ├── cases/route.ts
│   │   └── users/route.ts
│   ├── dashboard/                        ← Role-based dashboards
│   ├── auth/
│   └── page.tsx
├── components/
│   ├── forms/                            ← Form components
│   ├── ui/                               ← Reusable UI (badges, cards)
│   ├── common/                           ← Layout (Navbar, Sidebar)
│   └── LabUploadForm.tsx                 ← Lab form component
├── lib/
│   ├── mongooseConnect.ts                ← DB connection
│   ├── cloudinary.ts                     ← Cloudinary config
│   ├── services/
│   │   └── LabRecordService.ts           ← Business logic
│   └── schemas/                          ← Validation schemas
├── models/
│   ├── LabRecord.ts                      ← Lab schema
│   ├── caseModel.ts
│   ├── userModel.ts
│   └── index.ts
├── types/
│   └── index.ts                          ← Shared types (Case, User)
└── constants/
    ├── departmentTests.ts
    └── labTests.ts
```

## Building New Features

When adding features that touch lab records:

1. **Check**: Does it modify department fields? → Update schema in `LabRecord.ts`
2. **Check**: Does it hit API? → Add service method, then route
3. **Check**: Does it need UI? → Create form component with validation
4. **Check**: Does it affect workflow? → Update case status in `caseModel.ts`
5. **Check**: Are env vars needed? → Add to `.env.example` with docs
6. **Test**: API route in isolation with curl
7. **Test**: Component with mock API response
8. **Test**: Full flow end-to-end

## Performance Notes

- Lab records are indexed on `patientId`, `caseId`, `status`, `createdAt`
- Use pagination: always `limit` and `skip` for queries
- Cloudinary URLs are cached; no need for re-uploads
- Connection pooling handles concurrent requests

## References

- Lab system docs: `LAB_RECORD_GUIDE.md`
- **NEW Payment workflow docs: `PAYMENT_WORKFLOW.md`** ← READ THIS for payment implementation
- Schema reference: Check `ILabRecord` interface in `LabRecord.ts`
- Payment model: Check `PaymentData` interface in `src/models/paymentModel.ts`
- Cloudinary docs: See `uploadToCloudinary()` in `cloudinary.ts`
- Mongoose patterns: Review other routes in `api/` folder

## Payment & Challan Workflow (Feb 2026 Update)

**Complete payment-challan integration with database linking and case status gates.**

### Critical Rule: Payment Confirmation Gates Lab Entry
```typescript
// Lab entry ONLY available if:
case.paymentConfirmed === true OR case.status === "payment-received"
// Enforced in: /src/app/dashboard/deo/lab-records/page.tsx
```

### Workflow: Case → Challan → Payment → Confirmation → Lab Entry
```
1. Case created (status: pending)
2. Challan generated (POST /api/challans) → case.status: payment-pending
3. Payment created (POST /api/payments) → payment.status: pending
4. [CRITICAL] Payment confirmed (POST /api/payments/{id}/confirm)
   → payment.status: completed
   → case.paymentConfirmed: true, status: payment-received
   → challan.status: received
   ✓ Lab entry unlocked
```

### Key Components
- Model: `src/models/paymentModel.ts` (Mongoose schema + class)
- Service: `src/lib/services/paymentService.ts` (CRUD + confirmPayment)
- Routes: `src/app/api/payments/route.ts`, `/[id]/confirm/route.ts`
- Component: `src/components/PaymentForm.tsx` (fetch challan, submit payment)

### Database Links
```typescript
Case: { challanId, paymentId, paymentConfirmed, status }
Payment: { caseId, challanId, status, confirmedDate }
Challan: { caseId, totalAmount, status }
```

---
> Source: [nooraizafshan/MediServe_V00001](https://github.com/nooraizafshan/MediServe_V00001) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
