# HRM System - AI Coding Agent Instructions

## Project Overview

React + Vite HRM (Human Resource Management) system with role-based access control (Superadmin, HR, Employee). Uses localStorage for mock data persistence with no backend integration.

## Architecture & Data Flow

### Context Provider Hierarchy (see `src/main.jsx`)

```
MockDataProvider (outer) → AuthProvider (inner) → App
```

This order is critical: `AuthContext` depends on `MockDataProvider` for user data.

### State Management Pattern

**All application state lives in React Context + localStorage**:

- `src/context/MockData.jsx` - Master data store (users, departments, attendance, leaves)
- `src/context/AuthContext.jsx` - Authentication state (current user session)
- `src/services/api.js` - Dummy API integration for initial data population
- localStorage keys: `hrm_users`, `hrm_departments`, `hrm_attendance`, `hrm_leaves`, `hrm_current_user`

**Key principle**: Every data mutation must update both state AND localStorage to persist changes.

### Dummy API Integration

System uses external APIs to populate initial data (see `src/services/api.js`):

- **RandomUser API** (`https://randomuser.me/api/`) - Generates realistic employee profiles
- **Data Transformation**: API users transformed to HRM format with roles, departments, and employee IDs
- **Auto-Generation**: 20 users (2 HR, 18 employees), 30 days of attendance, and leave requests created on first load
- **Caching**: Data fetched once and stored in localStorage; clear localStorage to refresh from API

Pattern for API initialization:

```jsx
const dummyData = await initializeDummyData();
localStorage.setItem("hrm_users", JSON.stringify(dummyData.users));
```

### Role-Based Access Control

Three roles with distinct capabilities (see `src/components/Sidebar.jsx` for navigation logic):

- **superadmin**: Create HR accounts (`/superadmin/create-hr`), view payroll, approve leaves
- **hr**: Manage employees, attendance tracking, approve leaves, process payroll
- **employee**: Request leave, view personal dashboard

Route protection implemented via `src/components/ProtectedRoute.jsx` with `allowedRoles` prop.

## Key Development Patterns

### Data Mutation Pattern

Always follow this sequence when modifying data:

```jsx
const newData = [...existingData, newItem];
setState(newData);
localStorage.setItem("hrm_key", JSON.stringify(newData));
```

Example: `src/context/MockData.jsx` - `addUser`, `addAttendance`, `updateLeaveStatus` functions.

### Form Handling Convention

Pages follow this structure (see `src/pages/Employees.jsx`, `src/pages/superadmin/CreateHR.jsx`):

1. Form state in component
2. `handleChange` updates state via `e.target.name`/`e.target.value`
3. `handleSubmit` calls context function to persist
4. Modal toggles via boolean state (`showModal`)
5. Success message with 3-second auto-hide

### Nepali Date Integration

`nepali-date-converter` library used for BS/AD date conversion (see `src/pages/Employees.jsx`):

- Auto-calculate Nepali date (BS) from Gregorian date (AD)
- Calculate age from date of birth
- Pattern: `useEffect` watches `dobAD`, updates `dobBS` automatically

### Component Structure

- **Layout**: `src/layouts/MainLayout.jsx` - Fixed sidebar (pl-64) + header (pt-16) + content area
- **Protected Pages**: Wrapped in `<ProtectedRoute>` + `<MainLayout>` (see `src/App.jsx`)
- **Styling**: Tailwind utility classes, rounded-2xl cards, shadow-sm borders, slate color palette

## Development Workflow

### Running the App

```powershell
npm run dev          # Start dev server (Vite HMR on http://localhost:5173)
npm run build        # Production build
npm run preview      # Preview production build
npm run lint         # ESLint check
```

### Testing Credentials

Default superadmin (always available):

- Email: `admin@hrm.com`
- Password: `admin`

All dummy API users:

- Password: `password123`
- HR users: `hr.firstname.lastname@hrm.com`
- Employees: Use email from RandomUser API

**Reset Data**: Clear browser localStorage or use DevTools to remove `hrm_*` keys to reload fresh data from API.

### Adding New Features

**New Page with Role Protection**:

1. Create component in `src/pages/`
2. Add route in `src/App.jsx` with `<ProtectedRoute allowedRoles={['role1', 'role2']}>`
3. Update `getMenuItems()` in `src/components/Sidebar.jsx` for navigation

**New Data Entity**:

1. Add state + localStorage key in `MockDataProvider`
2. Create helper functions (add/update/delete pattern)
3. Initialize with `useEffect` on mount
4. Export via context value

## Project-Specific Conventions

### Naming Conventions

- Component files: PascalCase (`CreateHR.jsx`)
- Context hooks: `use` prefix (`useAuth`, `useMockData`)
- ID generation: Template literals with timestamp (`EMP${Date.now().toString().slice(-4)}`)
- localStorage keys: `hrm_` prefix

### Icon Library

Uses `lucide-react` - import specific icons as components:

```jsx
import { Home, Users, Calendar } from "lucide-react";
```

### Conditional Rendering Pattern

Ternary for simple toggles, early returns for role checks:

```jsx
if (!user) return <Navigate to="/login" replace />;
```

### Styling Conventions

- Primary color: blue-600
- Background: slate-50
- Cards: white bg, rounded-2xl, shadow-sm, border-slate-100
- Buttons: rounded-xl with shadow-lg and color/30 opacity shadow
- Transitions: transition-all or transition-colors duration-200/300

## External Dependencies

- **react-router-dom v7**: Client-side routing (BrowserRouter, Routes, Route, Navigate)
- **lucide-react**: Icon library (replaces FontAwesome/Heroicons)
- **nepali-date-converter**: BS/AD date conversion for Nepali calendar support
- **Tailwind CSS v3**: Utility-first styling (config: `tailwind.config.js`)

## Critical Files to Reference

- `src/context/MockData.jsx` - Data schema and all CRUD operations
- `src/services/api.js` - Dummy API integration and data transformation logic
- `src/App.jsx` - Complete routing structure with role requirements
- `src/components/Sidebar.jsx` - Role-based navigation logic
- `src/components/ProtectedRoute.jsx` - Authorization guard implementation
- `src/components/LoadingSpinner.jsx` - Loading indicator during API data fetch
- `src/pages/Employees.jsx` - Complex form with date conversion example

## Known Limitations

- No backend API - localStorage only (data lost on clear cache)
- Passwords stored in plain text (mock authentication only)
- Attendance check-out doesn't update existing records (demo limitation)
- No TypeScript - JSX files with PropTypes omitted
- External API dependency (RandomUser API) - requires internet connection for initial load

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alamnurain786)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Alamnurain786)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
