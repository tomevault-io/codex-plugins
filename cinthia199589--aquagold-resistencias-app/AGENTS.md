# Copilot Instructions for Aquagold Resistencias App

## Project Overview
- **Type:** Next.js app for managing shrimp resistance tests
- **Core integrations:** Firebase Firestore, Firebase Storage, Microsoft OneDrive, Azure AD (MSAL)
- **Main flows:** Test creation, real-time data entry, photo uploads, Excel report generation, daily reporting

## Architecture & Key Files
- `app/` — Next.js pages and layouts
  - `dashboard/` — Main dashboard for active tests
  - `layout.tsx`, `page.tsx` — App shell and login
- `lib/` — Service modules
  - `firebase.ts` — Firebase config
  - `firestoreService.ts` — Firestore CRUD
  - `graphService.ts` — OneDrive/Graph API
  - `excelExport.ts` — Excel report generation
  - `types.ts` — Shared TypeScript types
  - `utils.ts` — Utilities
- `components/` — UI components (e.g., `SearchBar.tsx`, `DailyReportModal.tsx`)
- `public/` — Static assets

## Data & External Services
- **Firestore:** Stores resistance test records (`resistance_tests` collection) - TEXT DATA ONLY
- **OneDrive:** Stores Excel reports and photos in `Aquagold_Resistencias/`
- **Azure AD:** Handles authentication via MSAL
- **Photos:** Saved ONLY to OneDrive (not Firebase Storage). URLs stored in Firestore.

## Developer Workflows
- **Install:** `npm install`
- **Run dev server:** `npm run dev` (http://localhost:3000)
- **Environment:** Set secrets in `.env.local` (see README)
- **Firestore/Storage rules:** Require authenticated access (see README for rules)
- **Testing:** No explicit test runner detected; add tests in `app/dashboard/tests/` if needed
- **Debugging:** Use browser console and check Firestore/Storage/OneDrive permissions

## Patterns & Conventions
- **Components:** PascalCase (`ResistanceTestList`)
- **Functions:** camelCase (`handleSubmit`)
- **Constants:** UPPER_SNAKE_CASE (`APP_ROOT_FOLDER`)
- **Types:** PascalCase (`ResistanceTest`)
- **Files:** kebab-case (`firestore-service.ts`)
- **Data flow:** UI → lib/services → Firestore/OneDrive
- **Auto-save:** Data is saved in real-time to Firestore
- **Excel export:** Triggered on test completion or daily report

## Integration Points
- **Firebase:** See `lib/firebase.ts` and `lib/firestoreService.ts`
- **OneDrive:** See `lib/graphService.ts` and `lib/excelExport.ts`
- **Authentication:** See MSAL config in `.env.local` and login flow in `app/page.tsx`

## Troubleshooting
- Permission errors: Check Firestore/Storage rules and Azure AD setup
- Excel/report issues: Ensure all required fields are filled
- Photo upload issues: Check both Firebase and OneDrive permissions

## Example: Creating a New Test
- Creates Firestore record, OneDrive folder, initializes samples

## Example: Completing a Test
- Marks test as completed, generates Excel, saves to OneDrive, removes from dashboard

---
**For more details, see `README.md`.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cinthia199589)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cinthia199589)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
