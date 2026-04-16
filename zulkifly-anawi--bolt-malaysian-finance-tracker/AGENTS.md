# GitHub Copilot Instructions - Malaysian Financial Tracker

> Repository-wide instructions for AI assistants working on this project.
> This file provides context, conventions, and guidelines for generating accurate, consistent code.

## Project Overview

**Malaysian Financial Tracker** is a comprehensive financial tracking platform built specifically for Malaysians. It helps users manage investments (ASB, EPF, Tabung Haji), savings, and financial goals with beautiful visualizations and smart insights.

### Business Context
- **Target Users**: Malaysian individuals managing personal finances
- **Core Domain**: Financial tracking for Malaysian investment products
- **Key Products Supported**:
  - ASB (Amanah Saham Bumiputera) - Unit trust with annual dividends
  - EPF (Employees Provident Fund) - Retirement savings with Account 1/Account 2 split
  - Tabung Haji - Hajj savings with hibah (Islamic dividend)
  - Savings accounts, fixed deposits, unit trusts, stocks, crypto

### Privacy Principles
- **Manual Tracking Only**: We never connect to actual bank accounts
- **Zero Credentials**: We never ask for bank passwords
- **User Data Ownership**: Users can export all their data anytime
- **Supabase Only**: Financial data never leaves the Supabase infrastructure

---

## Tech Stack

### Frontend
- **Framework**: React 18.3.1 with TypeScript 5.5.3
- **Build Tool**: Vite 5.4.2
- **Styling**: Tailwind CSS with custom glassmorphism design
- **Icons**: Lucide React (only use icons from this library)
- **State Management**: React Context API (AuthContext)

### Backend
- **Database**: PostgreSQL via Supabase
- **Authentication**: Supabase Auth (Email/Password only)
- **Security**: Row Level Security (RLS) policies on all tables
- **Real-time**: Supabase real-time subscriptions (where applicable)

### Development
- **Package Manager**: npm
- **Linting**: ESLint with TypeScript rules
- **Type Checking**: `npm run typecheck`
- **Dev Server**: `npm run dev` (port 5173)

---

## Architecture Patterns

### Component Organization
```
src/components/
├── accounts/        # Account CRUD and display
├── admin/           # Admin panel (restricted to is_admin users)
│   └── pages/       # Admin sub-pages
├── common/          # Shared UI components (buttons, cards, modals)
├── engagement/      # Achievements, notifications, insights, export
├── goals/           # Goal CRUD and progress tracking
├── help/            # FAQ, onboarding, help system
├── investments/     # ASB, EPF, Tabung Haji calculators
└── [Component].tsx  # Top-level shared components
```

### State Management
- **AuthContext**: Global authentication state, user profile, `isAdmin` flag
- **Local State**: Component-level state with `useState`, `useEffect`
- **No Redux**: Keep state management simple with Context + hooks

### Data Flow
1. **Supabase Client** (`src/lib/supabase.ts`) - Single instance
2. **Services** (`src/services/`) - Business logic and API calls
3. **Components** - UI rendering and user interaction
4. **Types** (`src/types/database.ts`) - Shared TypeScript interfaces

---

## Code Conventions

### TypeScript
```typescript
// ✅ DO: Use explicit types
interface AccountProps {
  account: Account;
  onUpdate: (account: Account) => void;
}

// ❌ DON'T: Use 'any' type
function process(data: any) { ... }

// ✅ DO: Use type imports
import type { Profile, Account, Goal } from '../types/database';

// ✅ DO: Define interfaces for component props
export const AccountCard = ({ account, onUpdate }: AccountProps) => { ... }
```

### React Components
```typescript
// ✅ DO: Use functional components with TypeScript
export const MyComponent = ({ prop1, prop2 }: MyComponentProps) => {
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    // Effect logic
  }, [dependencies]);
  
  return (
    <div className="...">
      {/* JSX */}
    </div>
  );
};

// ❌ DON'T: Use class components
class MyComponent extends React.Component { ... }
```

### Naming Conventions
- **Components**: PascalCase (`AccountCard.tsx`, `GoalProgress.tsx`)
- **Hooks**: camelCase with `use` prefix (`useConfig.ts`)
- **Services**: camelCase with `Service` suffix (`adminConfigService.ts`)
- **Utils**: camelCase (`formatters.ts`, `validation.ts`)
- **Types**: PascalCase for interfaces (`Profile`, `Account`, `Goal`)

### File Structure
```typescript
// Component file structure
import { useState, useEffect } from 'react';
import { IconName } from 'lucide-react';
import { supabase } from '../lib/supabase';
import type { TypeName } from '../types/database';

interface ComponentProps {
  // Props definition
}

export const ComponentName = ({ props }: ComponentProps) => {
  // State declarations
  // Effects
  // Event handlers
  // Return JSX
};
```

---

## Styling Guidelines

### Tailwind CSS
```jsx
// ✅ DO: Use Tailwind utility classes
<div className="bg-white/10 backdrop-blur-sm rounded-2xl p-6 border border-white/20">

// ✅ DO: Use glassmorphism pattern (project standard)
<div className="glass-card"> {/* Custom class in index.css */}

// ❌ DON'T: Use inline styles
<div style={{ backgroundColor: 'rgba(255,255,255,0.1)' }}>

// ❌ DON'T: Create new CSS files
```

### Color Palette
- **Background**: Dark theme with glass-morphic overlays
- **Primary Gradient**: `from-cyan-500 to-blue-600`
- **Success**: `text-green-400`, `bg-green-500/20`
- **Warning**: `text-yellow-400`, `bg-yellow-500/20`
- **Error**: `text-red-400`, `bg-red-500/20`
- **Text Primary**: `text-white`
- **Text Secondary**: `text-white/70`
- **Borders**: `border-white/20`

### Responsive Design
```jsx
// ✅ DO: Mobile-first responsive
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// ✅ DO: Use responsive padding
<div className="p-4 md:p-6 lg:p-8">
```

---

## Database Conventions

### Table Naming
- Use `snake_case` for table and column names
- Tables: `profiles`, `accounts`, `goals`, `account_goals`
- Admin tables: `admin_config_*`, `admin_audit_log`

### Required Patterns

#### RLS Policies
```sql
-- Every user data table MUST have RLS policies
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;

-- User can only access own data
CREATE POLICY "Users can view own data"
ON table_name FOR SELECT
TO authenticated
USING (user_id = auth.uid());

-- Admin can view all (use is_admin() function)
CREATE POLICY "Admins can view all"
ON table_name FOR SELECT
TO authenticated
USING (is_admin());
```

#### Migration Format
```sql
/*
  # Migration Title

  ## Purpose
  Brief description

  ## Changes
  - Change 1
  - Change 2
*/

-- Section 1
CREATE TABLE IF NOT EXISTS ...

-- Section 2
CREATE POLICY IF NOT EXISTS ...
```

### Key Functions
- `is_admin()` - Check if current user is admin (dual-source: profiles.is_admin OR admin_authorized_emails)
- `auth.uid()` - Get current user's UUID
- `auth.email()` - Get current user's email

---

## Malaysian Financial Context

### ASB (Amanah Saham Bumiputera)
- **Unit Price**: Always RM 1.00 (enforced in code)
- **Dividend Rate**: 5.00% - 7.75% historical range
- **Maximum Investment**: RM 300,000 (for calculations)
- **Tracking**: Balance = Units × RM 1.00

### EPF (Employees Provident Fund)
- **Account Split**: 70% Account 1, 30% Account 2
- **Employee Rate**: 11% (default), 4% (age 60+)
- **Employer Rate**: 13% (standard), 12% (age 60+)
- **Schemes**: Conventional and Syariah
- **Dividend Rates**: Vary by scheme and account type

### Tabung Haji
- **Purpose**: Hajj pilgrimage savings
- **Hibah**: Islamic dividend (not guaranteed, but historically consistent)
- **Goal Types**: Hajj or Umrah

### Currency
- **Always use RM** (Malaysian Ringgit)
- **Format**: `RM X,XXX.XX` or `formatCurrency()` utility
- **No currency conversion** - single currency app

---

## Testing & Validation

### Before Committing
```bash
# Type checking
npm run typecheck

# Linting
npm run lint

# Build verification
npm run build
```

### Common Errors to Avoid
1. **Missing RLS policies** - Every table needs them
2. **Using `any` type** - Always define proper types
3. **Hardcoded credentials** - Use environment variables
4. **Missing error handling** - Always handle Supabase errors
5. **Infinite recursion in RLS** - Use SECURITY DEFINER functions

---

## Security Requirements

### Authentication
- All API calls must be authenticated (use `supabase.auth.getUser()`)
- Admin routes must check `isAdmin` from AuthContext
- Never expose service role key to client

### Data Access
- Users can only access their own data (enforced by RLS)
- Admins can view all data (via `is_admin()` function)
- Audit logging for admin actions

### Sensitive Operations
```typescript
// ✅ DO: Check admin status before admin operations
const { isAdmin } = useAuth();
if (!isAdmin) {
  return <Navigate to="/" />;
}

// ❌ DON'T: Trust client-side checks alone
// RLS policies are the real security layer
```

---

## OpenSpec Integration

This project uses OpenSpec for spec-driven development.

### When to Create Proposals
- Adding new features or functionality
- Making breaking changes (API, schema)
- Changing architecture patterns
- Security-related changes

### OpenSpec Commands
```bash
openspec list           # List active changes
openspec list --specs   # List specifications
openspec validate       # Validate changes
```

### Spec Location
- Specifications: `openspec/specs/[capability]/spec.md`
- Changes: `openspec/changes/[change-id]/`
- Project config: `openspec/project.md`

---

## Commit Message Format

Follow conventional commits:
```
type(scope): subject

body (optional)

footer (optional)
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style (no logic change)
- `refactor`: Code refactoring
- `test`: Tests
- `chore`: Maintenance

### Examples
```
feat(accounts): add ASB dividend projection calculator
fix(goals): correct progress calculation for multi-account goals
docs(readme): update installation instructions
refactor(admin): extract user management to separate component
```

---

## Quick Reference

### Import Patterns
```typescript
// React
import { useState, useEffect, useCallback } from 'react';

// Icons (Lucide only)
import { Wallet, Target, TrendingUp, Plus, X } from 'lucide-react';

// Supabase
import { supabase } from '../lib/supabase';

// Types
import type { Profile, Account, Goal } from '../types/database';

// Context
import { useAuth } from '../contexts/AuthContext';

// Utils
import { formatCurrency, formatDate } from '../utils/formatters';
```

### Common UI Patterns
```jsx
// Loading state
{loading ? (
  <div className="text-center py-8 text-white/50">Loading...</div>
) : (
  // Content
)}

// Error handling
{error && (
  <div className="bg-red-500/20 border border-red-500/50 text-red-200 p-4 rounded-lg">
    {error}
  </div>
)}

// Empty state
{items.length === 0 && (
  <div className="text-center py-8 text-white/50">
    No items found. Create your first one!
  </div>
)}
```

### Supabase Query Pattern
```typescript
const loadData = async () => {
  setLoading(true);
  try {
    const { data, error } = await supabase
      .from('table_name')
      .select('*')
      .eq('user_id', user?.id)
      .order('created_at', { ascending: false });

    if (error) throw error;
    setData(data || []);
  } catch (err) {
    console.error('Error loading data:', err);
    setError('Failed to load data');
  } finally {
    setLoading(false);
  }
};
```

---

## Files to Reference

For deeper context, consult these files:
- `README.md` - Project overview and features
- `CONTRIBUTING.md` - Contribution guidelines
- `MIGRATION_GUIDELINES.md` - Database migration best practices
- `openspec/AGENTS.md` - OpenSpec workflow instructions
- `src/types/database.ts` - TypeScript type definitions
- `openspec/specs/*/spec.md` - Feature specifications

---

*Last updated: November 2025*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zulkifly-anawi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
