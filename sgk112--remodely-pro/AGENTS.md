# Copilot Instructions for Remodely.AI

## Architecture Overview

This is a **production-ready Next.js 14 App Router** marketplace connecting homeowners with verified r### File Structure Conventions
```
app/
├── dashboard/[customer|contractor|admin]/  # Role-based dashboards
├── api/[user|quotes|contractors|scrape]/   # RESTful API routes
├── api/[ai|voice|google-agent]/            # AI & voice service routes
├── auth/                                   # NextAuth pages
├── voice-consultation/                     # Voice consultation interface
└── [contractors|search|quote]/             # Public marketplace pages

lib/
├── scrapers/          # 9 automated data collection modules
├── validation.ts      # Zod schema definitions
├── multi-ai-service.ts # AI routing & orchestration
├── smartMatching.ts   # AI-powered contractor matching
├── advancedLocationService.ts # Singleton location service
└── [service].ts       # Business logic layer

scripts/               # Production utilities
test-*.js             # 40+ test files for all systems
```s. The platform features **role-based authentication**, **automated contractor acquisition**, **AI-powered matching**, and **voice consultation capabilities**.

### Key Data Flow

## Critical Systems

### AI & Voice Integration
The platform's **key differentiator** is its intelligent contractor-customer matching:
```bash
# Voice system testing
node test-voice-comprehensive.js         # Complete voice pipeline test
node test-google-cloud-agents.js         # Google AI agents testing
node test-elevenlabs-integration.js      # Voice synthesis testing

# AI services testing
node test-ai-services.js                 # Multi-AI routing tests
node test-smart-matching.js              # Matching algorithm tests
```

**AI Service Stack**:

### Contractor Scraping Architecture
Automated contractor acquisition system with **9 scraper types**:
```bash
# Main scraping commands
node test-scraping.js                    # Test all scrapers
node test-single-scraper.js [scraper]    # Test specific scraper
node populate-contractors.js             # Production data import
./system-status-check.sh                 # Health monitoring
node scripts/roc-converter.js            # Arizona ROC license integration
```

**Scraper Categories** (`lib/scrapers/`):

### Database Schema Pattern
Hub-and-spoke around `User` model with **role-specific profiles**:
```prisma
User (userType: CUSTOMER|CONTRACTOR|ADMIN)
├── Contractor? (business profile + scraped data + ROC verification)
├── Customer? (personal profile + AI preferences)
├── Quote[] (bidirectional communication + AI matching scores)
├── Booking[] + Payment[] + Review[]
└── VoiceConsultation[] + AIInteraction[] + Message[]
```

**Critical Schema Details**:

## Development Workflows

### Essential Commands
```bash
# Database operations (ALWAYS in sequence)
npm run db:generate && npm run db:push && npm run db:seed

# Scraping operations
npm run seed:production        # Production contractor import
npm run import:contractors     # Bulk contractor processing
npm run deploy:full           # Full production deployment

# Development
npm run dev                   # Development with hot reload
npm run dev:port              # Run on port 3001 (alternative)
npm run build:analyze         # Bundle analysis
npm run db:studio            # Prisma database browser
```

### Quick Setup & Test Data
```bash
# One-command development setup
./setup-dev.sh               # Validates environment, creates test users, displays URLs

# Test user creation
node create-test-users.js     # Creates customer, contractor, admin accounts
node dev-scripts/setup-profiles.js  # Ensures proper profile relationships

# ROC (Arizona contractor license) tools
npm run roc:seed              # Import Arizona ROC contractor data
npm run roc:stats             # View ROC integration statistics
npm run roc:clear             # Clear ROC data
```

**Test Credentials (created by setup)**:

### Testing Patterns
Use project's **extensive test suite** (40+ test files):
```bash
# Authentication & registration flow
node test-registration-flow.js
node test-phone-verification.js

# AI & voice system validation
node test-voice-comprehensive.js          # End-to-end voice testing
node test-google-cloud-agents.js          # AI agent functionality
node test-ai-services.js                  # Multi-AI routing
node test-enhanced-voice.js               # Voice quality testing

# Scraping system validation  
node test-arizona-scrapers.js
node test-authenticated-scraping.js
node test-roc-integration.js              # Arizona ROC validation

# Location & maps integration
node test-location-service.js
node test-google-maps.js
node test-apple-maps.js                   # Apple Maps integration
```

## Code Patterns & Integration Points

### Authentication Architecture

### API Route Architecture

### Service Layer Pattern
All business logic in `lib/` with consistent patterns:

### Environment Configuration
**Three environment tiers**:
```bash
.env.example           # Template with all required keys
.env.production        # Production configuration
.env.scraping.example  # Scraping-specific credentials
```

**Required integrations**: Google Maps API, Twilio, Stripe, Cloudinary, database (PostgreSQL in production), ElevenLabs, Google Cloud AI

**Development ports**: Main app runs on `:3001` (not :3000), health check endpoint at `/api/health`

### File Structure Conventions
```
app/
├── dashboard/[customer|contractor|admin]/  # Role-specific dashboards
├── api/[user|quotes|contractors|scrape]/   # RESTful API routes
├── api/[ai|voice|google-agent]/            # AI & voice service routes
├── auth/                                   # NextAuth pages
├── voice-consultation/                     # Voice consultation interface
└── [contractors|search|quote]/             # Public marketplace pages

lib/
├── scrapers/          # 27 automated data collection modules
├── validation.ts      # Zod schema definitions
├── multi-ai-service.ts # AI routing & orchestration
├── smartMatching.ts   # AI-powered contractor matching
└── [service].ts       # Business logic layer

scripts/               # Production utilities
test-*.js             # 40+ test files for all systems
```

## Production Deployment Patterns

### Build Process
```bash
npm run build:render    # Render.com deployment
npm run build:clean     # Clean build with cache clearing
```

### Monitoring & Health Checks
Use built-in monitoring scripts:
```bash
./system-status-check.sh           # System health overview
node check-contractors.js          # Verify contractor data
node check-database.js             # Database connectivity
```

## Development Guidelines


### Critical Code Patterns

**Location Distance Calculation**:
```typescript
const locationService = AdvancedLocationService.getInstance()
const distance = locationService.calculateDistance(userPos, contractorPos)
```

**Specialty Parsing** (handles JSON strings or arrays):
```typescript
const getSpecialties = (specialties: string | string[]) => {
  if (Array.isArray(specialties)) return specialties
  if (typeof specialties === 'string') {
    try { return JSON.parse(specialties) }
    catch { return specialties.split(',').map(s => s.trim()) }
  }
  return []
}
```

**Authentication Middleware Pattern**: Routes are protected via `middleware.ts` - public paths explicitly listed, all others require auth.

The project prioritizes **automated data acquisition** and **production reliability** - maintain these patterns when adding features.

# Remodely.AI Copilot Instructions



## Architecture & Data Flow
- **Next.js 14 App Router** monolith, transitioning to microservices (see `microservices/README.md`)
- **Role-based dashboards**: `app/dashboard/[customer|contractor|admin]/`
- **RESTful APIs**: `app/api/[user|quotes|contractors|scrape]/`, plus AI endpoints (`app/api/ai`, `app/api/voice`, etc.)
- **Business logic**: All core logic in `lib/` (singleton patterns, Zod validation, service orchestration)
- **Automated contractor acquisition**: 27+ scrapers in `lib/scrapers/`, with ROC license verification
- **AI/Voice**: ElevenLabs, Twilio, Google Cloud AI, multi-AI routing (`lib/multi-ai-service.ts`)
- **Location**: Google Maps, Apple Maps, singleton `AdvancedLocationService`
- **Database**: Prisma, hub-and-spoke schema around `User` (see below)

## Key Patterns & Conventions
- **Absolute imports**: Always use `@/lib/`, `@/components/`
- **Role-based access**: Never bypass User → profile relationships; use `middleware.ts` for route protection
- **Validation**: All forms use Zod schemas from `lib/validation.ts`
- **Environment**: Strictly typed variables; see `.env.example` for required keys
- **Location services**: Always use `AdvancedLocationService.getInstance()`
- **JSON fields**: Parse with try/catch for string/array inputs (see specialty parsing example below)
- **Component images**: Use `ImageService.getContractorProfileImage()` for fallbacks

## Essential Workflows
- **Setup**: `./setup-dev.sh` (creates test users, validates env)
- **Database**: `npm run db:generate && npm run db:push && npm run db:seed` (run in sequence)
- **Development**: `npm run dev` (hot reload), `npm run dev:port` (port 3001)
- **Testing**: Node scripts in `test-*.js` (40+ files); see README for test suite details
- **Scraping**: `node test-scraping.js`, `node test-single-scraper.js [scraper]`, `npm run seed:production`
- **Deployment**: `npm run build:render` (Render.com), `npm run build:clean` (cache clear)
- **Monitoring**: `./system-status-check.sh`, `node check-contractors.js`, `node check-database.js`

## Database Schema (Prisma)
User (userType: CUSTOMER|CONTRACTOR|ADMIN)
├── Contractor? (business profile + scraped data + ROC verification)
├── Customer? (personal profile + AI preferences)
├── Quote[] (bidirectional communication + AI matching scores)
├── Booking[] + Payment[] + Review[]
└── VoiceConsultation[] + AIInteraction[] + Message[]

## Integration Points
- **Authentication**: Custom credentials (`lib/auth.ts`), Twilio phone verification (`lib/twilio.ts`)
- **AI/Voice**: ElevenLabs, Google Cloud, Twilio, multi-AI routing (`lib/multi-ai-service.ts`)
- **Location**: Google Maps, Apple Maps, singleton pattern
- **Scraping**: Arizona ROC, manufacturer networks, business directories, public sources

## Critical Code Examples
**Location Distance Calculation**
```typescript
const locationService = AdvancedLocationService.getInstance()
const distance = locationService.calculateDistance(userPos, contractorPos)
```
**Specialty Parsing**
```typescript
const getSpecialties = (specialties: string | string[]) => {
  if (Array.isArray(specialties)) return specialties
  if (typeof specialties === 'string') {
    try { return JSON.parse(specialties) }
    catch { return specialties.split(',').map(s => s.trim()) }
  }
  return []
}
```
**Authentication Middleware**
Routes protected via `middleware.ts`; public paths listed, all others require auth.

## Microservices Transition (2025+)
- See `microservices/README.md` and service templates for migration details
- Each service (auth, users, quotes, contractors, voice, matching, scrapers, location, web, admin) is independently deployable
- Service-to-service communication via HTTP APIs; see example in `microservices/services/README.md`

## Troubleshooting & Support
- If stuck, run `npm run validate:config`, check `CONFIGURATION_CHECKLIST.md`, clear `.next` cache, restart dev server
- For unclear patterns, review `README.md` and service-level docs in `microservices/services/*/README.md`

---
**Feedback requested:** If any section is unclear or missing, please specify so it can be improved for future AI agents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SGK112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
