# NNS Telecom Management System - Copilot Instructions

## Project Overview

NNS is a Next.js-based telecom management platform for fiber optic line installations, inventory, and operational workflows. It manages the full lifecycle from planning and task assignment to completion, with AI-powered validation and suggestions.

## Architecture & File Structure

- **app/**: Next.js App Router pages (auth, inventory, invoices, tasks, reports, profile, welcome, etc.)
- **components/**: Reusable UI (layout, modals, tables, shadcn/ui)
- **contexts/**: React Context providers (auth, theme, notifications)
- **lib/**: Utilities (Prisma client, AI suggestions, report services)
- **hooks/**: Custom React hooks
- **styles/**: Tailwind CSS config and global styles

## Technology Stack

- Next.js 15 (App Router), React 19, TypeScript
- Tailwind CSS, Radix UI, shadcn/ui
- PostgreSQL via Prisma ORM, NextAuth for authentication
- React Context API for state
- React Hook Form + Zod for forms
- Lucide React for icons

## Key Domain Concepts

- **Distribution Points (DPs)**: Format `XX-XXXX-XXXX-XXX-0X`, regex-enforced, unique per line
- **Cable Measurements**: F1 (start-middle), G1 (middle-end), total = F1+G1, wastage ≤ 20% of total
- **Power Readings**: DP/inbox, ≥20 triggers warnings
- **Inventory**: Drums, rosettes, hardware, cable lengths

## Component & Data Patterns

- Use `PublicLayout` for landing/marketing pages, `DashboardLayout` for authenticated app
- Modals in `components/modals/`, tables in `components/tables/`, forms use React Hook Form + Zod
- Prisma client: `import { prisma } from "@/lib/prisma"`
- AI suggestions: `import { AIService } from "@/lib/ai-suggestions"`

## Database Integration

- Use singleton Prisma client (`lib/prisma.ts`)
- Key tables: `line_details`, `tasks`, `inventory_items`, `drum_tracking`, `invoices`, `profiles`
- Enforced rules: DP regex, wastage ≤ 20%, drum quantity checks, power warnings, serial validation

## Authentication

- Authentication uses NextAuth; see `AuthProvider` for session management and `app/api/auth/[...nextauth]` for providers
- Google OAuth is available via NextAuth: configure the Google provider in environment variables and NextAuth, with redirect URL `https://<your-domain>/auth/callback` (plus local `http://localhost:3000/auth/callback`)
- The `/auth/callback` page uses NextAuth to finalize the OAuth flow and redirects to the dashboard once complete

## Google Sheets Import (Admin/Moderator)

- API: `POST /api/import-lines` with `{ sheetUrl, sheetName, month, year }`
  - Supports dry-run with `{ ..., dryRun: true }` to preview deletions and insertions
- Validations:
  - Sheet name must contain `nns` (case-insensitive)
  - First row must match required headers (case-insensitive comparison)
  - Overwrite behavior: for the given month, existing `line_details` rows with phone numbers present in the import are deleted, then new rows inserted
- Mappings: Columns map to `line_details` fields; `Pole-5.6` -> `pole`, `Pole-6.7` -> `pole_67`, `Total` -> `total_cable`, etc.
- Status defaults to `completed` on import
- Auth: UI is gated by role, and the API enforces server-side role checks via NextAuth session (admin/moderator only)
- Dry-run response includes: totals, existingByPhone, sampleInserts
- Env vars:
  - `GOOGLE_SERVICE_ACCOUNT_EMAIL`
  - `GOOGLE_SERVICE_ACCOUNT_KEY` (PEM; escape newlines as `\n`)
- Access model: The Google Sheet must grant at least Viewer access to the service account email; public sheets may still require explicit sharing.

## Inventory Management

### Stock Flow

- **Adding Stock**: Inventory items are added/increased via:
  - Creating inventory invoices (primary method)
  - Manual editing of individual items (admin)

### Google Sheets Sync & Usage Tracking

- When syncing from Google Sheets, inventory is reduced based on hardware usage from imported lines
- **Monthly Inventory Usage** table (`monthly_inventory_usage`) tracks:
  - Total usage per item per month/year
  - Prevents duplicate deductions on multiple syncs
  - Only the **difference** between new and previous usage is deducted

### Key Tables

- `inventory_items`: Current stock levels, reorder thresholds
- `monthly_inventory_usage`: Tracks usage per item per month (unique constraint on item+month+year)
- `inventory_invoice_items`: Records stock additions from invoices
- `waste_tracking`: Manual waste entries

### API Endpoints

- `GET /api/inventory/usage?month=X&year=Y`: Get monthly usage summary
- `DELETE /api/inventory/usage`: Reset monthly usage (admin, restores inventory)
- `POST /api/inventory/usage` with `{action: "recalculate"}`: Recalculate all inventory from usage records

### Service Location

- `lib/inventory-usage-service.ts`: Core logic for usage tracking and inventory updates

## Workflows & Conventions

- Add new features by following modal/form/table patterns
- All forms require loading/error states and validation
- Inventory and drum updates must be consistent with line installations
- Use AI validation for DPs, wastage, power, and error detection
- Use TypeScript strict mode and Next.js conventions

## Development & Debugging

- Use `npm install --legacy-peer-deps` for React 19 compatibility
- Run dev server: `npm run dev`
- Test forms, calculations, and AI logic for new features
- For layout issues, ensure correct usage of layout components and file structure

## Best Practices

- Use semantic HTML, ARIA labels, and error boundaries
- Validate all inputs, enforce server-side authorization (NextAuth), sanitize user data, and apply database constraints via Prisma migrations
- Optimize queries and use loading states
- Document new domain rules and patterns in this file

---

If updating, merge with existing content and preserve any project-specific rules not covered above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MACM18) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
