# CTDHUB Copilot Instructions

CTDHUB is a Web3 educational platform with AI-powered assessment tools, built on Next.js with Netlify Functions and Supabase. Focus on maintaining the sophisticated AI-driven questionnaire system and seamless user experience.

## Core Architecture

### Storage Strategy (Critical)
- **Dual storage pattern**: Always implement both Supabase and localStorage fallbacks
- Files like `lib/courses-storage.ts` show the pattern: check Supabase availability, fallback to local storage
- Use `lib/supabase.ts` null checks: `supabase ? await supabase.from() : fallbackFunction()`
- Test files (`test-*.js`) verify both storage methods work

### AI System - Binno Questionnaire
- **Adaptive questioning**: Uses previous answers to generate next questions via OpenAI
- **Session-based**: Each questionnaire gets unique session ID stored in localStorage
- **Interface pattern**: `lib/binno-questionnaire.ts` defines strict TypeScript interfaces
- **Three-phase flow**: Question generation → User response → Analysis/Report
- Functions: `binno-generate-question.ts` → user answers → `binno-final-analysis.js`

### Netlify Functions Architecture
- **TypeScript for new functions**, JavaScript for legacy analysis functions
- **Direct Supabase initialization** in functions (not using lib/supabase.ts)
- **Consistent error handling**: Always return { error: string } objects
- **ID generation**: Use `generateConsistentId()` pattern from `course-manager.ts`

## Development Patterns

### Component Structure
- **CTD theme system**: Use `ctd-bg`, `ctd-panel`, `ctd-text`, `ctd-yellow` classes
- **Card pattern**: `card hover:shadow-outline transition-shadow spotlight` classes
- **Error fallbacks**: Always include `onError` handlers for images with placeholder fallbacks
- **Responsive design**: Mobile-first with systematic breakpoint usage

### Data Flow
- **Course system**: `lib/courses.ts` → `course-manager.ts` function → Supabase tables
- **User profiles**: Wallet address as primary key, stored in `user_profiles` table
- **Video integration**: YouTube URL → extract ID → generate thumbnail with `ytThumb()`
- **Comments**: Linked to videos by `video_id`, user identified by `wallet_address`

### Testing Strategy
- **Integration tests**: `test-*.js` files test full user flows end-to-end
- **URL testing**: Verify both local (localhost:3000) and production URLs work
- **Function testing**: Direct function calls with proper payload structure
- **Status code validation**: 200 (success), 400/422 (expected errors), 500 (real failures)

## Key Conventions

### File Organization
- **Pages**: Follow Next.js structure, use `questionnaire.tsx` pattern for main flows
- **Components**: Single responsibility, always export default
- **Lib utilities**: Business logic separated from UI, typed interfaces
- **Functions**: One responsibility per function, consistent error responses

### Environment Handling
- **Build optimization**: `NETLIFY` env var changes Next.js output to 'export'
- **Image handling**: `unoptimized: true` for Netlify static builds
- **Function deployment**: TypeScript functions auto-compile for Netlify

### Critical Files to Understand
- `next.config.js`: Netlify-specific build configuration
- `netlify.toml`: Deployment settings and environment
- `lib/binno-questionnaire.ts`: Core AI system interfaces
- `netlify/functions/course-manager.ts`: Data persistence patterns
- `tailwind.config.js`: CTD theme system with custom animations

## Common Tasks

### Adding New AI Features
1. Define interfaces in `lib/binno-*.ts`
2. Create function in `netlify/functions/`
3. Add UI component following CTD theme
4. Create test file (`test-new-feature.js`)
5. Update fallback storage in lib utilities

### Course System Changes
1. Modify `lib/courses.ts` for business logic
2. Update `course-manager.ts` function for persistence
3. Check both Supabase schema and localStorage format
4. Test with `test-course-*.js` files

### Deployment
- **Development**: `npm run dev` (standard Next.js)
- **Netlify preview**: `npm run netlify:dev` (with functions)
- **Production build**: `npm run netlify:build` (exports static site)
- **Testing**: Run relevant `test-*.js` file against live URL

Remember: This platform prioritizes user experience and data persistence resilience. Always implement storage fallbacks and test the complete user journey.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChainTalkDailyHub)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ChainTalkDailyHub)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
