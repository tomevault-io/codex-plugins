# Copilot Instructions for ABU Endowment Online & ABU PWA App

## Overview
This workspace contains two main projects:
- **abu-endowment-online**: Laravel backend for endowment management, donor workflows, and admin operations.
- **abu-pwa-app**: React PWA frontend for donor interactions, registration, and donation flows.

## Architecture & Data Flow
- **Backend (Laravel)**
  - API endpoints defined in `routes/api.php` and web routes in `routes/web.php`.
  - Uses Laravel Sanctum for authentication and device session management.
  - Key models: `Donor`, `Project`, `Faculty`, `Department`, with custom vision models for dropdowns.
  - Livewire components for admin dashboard features.
  - MessageApi integration for SMS/email notifications (`MessageApi/`).
  - Payment integration via `PaymentController` and `/payments/*` routes.
- **Frontend (React PWA)**
  - Main entry: `src/App.js` (uses React Router, AuthProvider context).
  - API communication via `src/services/api.js` (Axios, device fingerprint, token/session headers).
  - Verification and device session flows in `src/services/verificationService.js`.
  - Registration flows split by donor type (addressable, non-addressable, friends).

## Developer Workflows
- **Backend**
  - Start dev server: `php artisan serve`
  - Queue listener: `php artisan queue:listen --tries=1`
  - Run tests: `composer test` or `php artisan test`
  - Migrate DB: `php artisan migrate --graceful --ansi`
  - Multi-service dev: `composer run dev` (see `composer.json`)
- **Frontend**
  - Start dev server: `npm run dev`
  - Build: `npm run build`
  - Test: `npm test`

## Project-Specific Patterns
- **API Auth**: Device session and JWT token are both supported; see Axios interceptors in `api.js`.
- **Verification**: SMS/email verification flows are handled via `/api/verification/*` endpoints and device session logic.
- **Payments**: Payment initialization and verification via `/api/payments/*` endpoints; webhook route is public and CSRF-exempt.
- **Dropdown Data**: Faculty/department vision endpoints for dynamic dropdowns (`/api/faculty-vision`, `/api/department-vision`).
- **Admin**: Admin routes are prefixed and protected by role middleware; Livewire used for dashboard features.

## Conventions & Integration Points
- **Backend**: PSR-4 autoloading (`composer.json`), feature/unit tests in `tests/`, config in `config/`, environment in `.env`.
- **Frontend**: Contexts in `src/contexts/`, hooks in `src/hooks/`, services in `src/services/`, pages in `src/pages/`.
- **Cross-Component**: Device fingerprinting and session management are critical for donor flows.
- **External**: Twilio SDK for SMS, Maatwebsite Excel for imports, Livewire for interactive admin UI.

## Examples
- To add a new donor type, update both backend `/api/donor-types` and frontend registration pages.
- For new payment provider, extend `PaymentController` and update frontend service calls.
- For new admin dashboard widget, create a Livewire component in `app/Livewire/` and register route in `routes/web.php`.

---
For unclear or missing conventions, ask for clarification or review recent commits for evolving patterns.

---
> Source: [sdabolawal/abu-endowment](https://github.com/sdabolawal/abu-endowment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
