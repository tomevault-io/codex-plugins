# VideoShare - Secure Video Sharing Platform

VideoShare is a modern, secure video sharing platform built with Next.js 15, Clerk authentication, PostgreSQL, and Google Cloud Storage. The application provides secure video upload, watermarking, time-limited sharing, and download prevention capabilities.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Package Management
**ALWAYS use pnpm** as the package manager (project has pnpm-lock.yaml):
```bash
npm install -g pnpm  # Install pnpm globally first
pnpm install  # Takes ~3 minutes
```
**Do not use npm install** - always use pnpm to maintain lock file consistency and ensure proper script execution.

### Build and Development
- **Install dependencies**: `pnpm install` (3 minutes)
- **TypeScript check**: `npx tsc --noEmit` (9 seconds)
- **Linting**: `pnpm run lint` (4.5 seconds) - always passes, no warnings
- **Build**: `pnpm run build` (20-40 seconds compilation + environment validation)
  - **NEVER CANCEL**: Set timeout to 10+ minutes minimum
  - Build compiles successfully but fails during static export without valid environment variables
  - Requires valid Clerk keys to complete fully
- **Development server**: `pnpm run dev` (3.2 seconds startup)
  - Server starts on http://localhost:3000
  - Uses Turbopack for fast development builds
  - Will show "Publishable key not valid" error without proper Clerk configuration

### Database Operations
- **Schema validation**: `pnpm run db:check` (0.5 seconds) - always passes
- **Schema generation**: `pnpm run db:generate` (requires DATABASE_URL environment variable)
- **Database operations**: All db:* commands require proper PostgreSQL connection
- **Test database**: `pnpm run test:db` (1 second) - validates schema only

### Environment Setup
Copy environment template and configure:
```bash
cp env-template.txt .env.local
# Fill in all required values from env-template.txt
```

Required environment variables for full functionality:
- Clerk authentication keys (NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY, CLERK_SECRET_KEY)
- Database URL (DATABASE_URL for PostgreSQL)
- Google Cloud Storage credentials
- Supabase credentials

## Validation Requirements

### Pre-Commit Validation
**ALWAYS** perform these checks before committing and pushing any changes:

1. **Lint check**: `pnpm run lint` (4.5 seconds, never fails)
2. **TypeScript validation**: `npx tsc --noEmit` (9 seconds)
3. **Build validation**: `pnpm run build` (20-40s compilation, set timeout to 10+ minutes)
4. **Database schema check**: `pnpm run db:check` (0.5 seconds) - if working with database changes

**Never commit without running these validation steps.** The build step is especially critical as it catches compilation errors and ensures the application can be deployed.

### Build Validation
- **Build validation**: The application compiles successfully but requires valid environment variables for full build completion
- **Environment testing**: Without valid credentials, dev server will start but show runtime errors (expected behavior)

### Manual Testing Scenarios
After making changes, validate these core workflows:

1. **Authentication Flow**: 
   - Access sign-in/sign-up pages
   - Test Clerk authentication integration
   - Verify protected route access

2. **Video Upload Workflow**:
   - Navigate to upload page (/upload)
   - Test file dropzone functionality
   - Verify upload progress tracking
   - Confirm database metadata storage

3. **Video Management**:
   - Dashboard video grid display
   - Search, filter, and sort functionality
   - Video metadata editing
   - Share link generation

4. **Video Viewing**:
   - Secure video streaming
   - Share link access validation
   - Download prevention features
   - View limit tracking

### Expected Behavior
- **Development server**: Starts successfully but shows "Publishable key not valid" error without real Clerk keys
- **Build process**: Compiles TypeScript and JSX successfully, fails during static export without valid environment
- **Database commands**: Schema operations work, connection operations require real database

## Common Tasks

### Quick Start for Development
```bash
# If pnpm not available
npm install -g pnpm

# Install dependencies
pnpm install  # 3 minutes

# Validate code quality
pnpm run lint  # 4.5 seconds

# Check TypeScript
npx tsc --noEmit  # 9 seconds

# Start development
pnpm run dev  # 3.2 seconds startup
```

### Available Scripts
**Always use pnpm** for running scripts:
- `pnpm run dev` - Start development server (3.2s startup)
- `pnpm run build` - Production build (20-40s compilation, needs env vars for completion)
- `pnpm run start` - Start production server
- `pnpm run lint` - ESLint validation (4.5s, always passes)
- `pnpm run db:check` - Validate database schema (0.5s)
- `pnpm run db:generate` - Generate migrations
- `pnpm run db:push` - Push schema to database
- `pnpm run db:studio` - Open Drizzle Studio
- `pnpm run test:db` - Validate database schema (1s)

### Project Structure
```
src/
├── app/                    # Next.js App Router
│   ├── (protected)/       # Protected routes (dashboard, upload, watermark)
│   ├── api/               # API routes
│   ├── watch/             # Video viewing
│   └── shared-videos/     # Shared video access
├── components/            # React components
│   ├── dashboard/         # Dashboard-specific components
│   ├── upload/            # Upload functionality
│   ├── video/             # Video player and controls
│   └── ui/                # Base UI components (Radix UI)
├── hooks/                 # Custom React hooks
└── lib/                   # Utilities and database
    └── db/                # Database schema and queries
```

### Key Technologies
- **Frontend**: Next.js 15 with App Router, TypeScript, Tailwind CSS, Radix UI
- **Authentication**: Clerk (OAuth and email/password)
- **Database**: PostgreSQL with Drizzle ORM
- **Storage**: Google Cloud Storage for video files
- **Development**: Turbopack for fast builds

### Architecture Notes
- **Security-first**: Multiple layers of download prevention
- **Secure sharing**: Time-limited tokens with view restrictions
- **Watermarking**: Custom overlay system for brand protection
- **Streaming**: Secure proxy streaming (no direct GCS URLs)
- **Responsive**: Mobile-first design with Tailwind CSS

### Limitations and Known Issues
- **No test suite**: Project does not include Jest, Playwright, or other testing frameworks
- **Environment dependent**: Many features require external service configuration
- **Build dependency**: Full build requires valid Clerk authentication keys
- **Database required**: Most functionality needs PostgreSQL connection

### CI/CD Considerations
- **Pre-commit validation**: Always run lint, TypeScript check, and build before committing
- Lint check always passes (4.5 seconds)
- Build requires environment variables for completion
- No automated tests to run
- Database migrations need separate execution

### Troubleshooting
- **"Publishable key not valid"**: Expected with dummy environment variables
- **Build fails**: Ensure all environment variables are properly configured
- **pnpm command not found**: Install pnpm globally with `npm install -g pnpm`
- **Database errors**: Verify DATABASE_URL and PostgreSQL connection

Always validate your changes by running the development server and testing the specific functionality you modified. The application starts reliably but requires proper environment configuration for full functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bhaveshsinghal95182)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/bhaveshsinghal95182)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
