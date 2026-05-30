````instructions
# AI Coding Agent Instructions - CleubAutomation

## Introduction

**CleubAutomation (Lux Home Planner)** - Premium home automation planning & cost estimation platform. Users design smart homes room-by-room with real-time cost calculations and PDF proposals. Features role-based access, hybrid auth (Firebase + Supabase), and responsive UI.

**Your Mission:** Fix bugs systematically, implement features sustainably, maintain production-quality standards.

---

## Core Principles

### ✅ DO's

**1. Broad-to-Narrow Problem Solving**
- Understand the full structure before implementing
- Identify root causes, not just symptoms
- **Always check related files** - types, services, components may need updates
- One change ripples: if you modify a type, update all components using it

**2. Database Operations (ALWAYS refer to SUPABASE_NOTES.md)**
- NEVER use local Supabase CLI
- Execute SQL directly in Supabase Dashboard > SQL Editor
- Update `src/supabase/types.ts` after schema changes
- Test in browser immediately after DB changes

**3. Quality Over Shortcuts**
- Follow existing patterns (no `any` types)
- Clean, focused code > quick patches
- Test changes before completing
- Verify no console errors after implementation

**4. Holistic Thinking**
- If you change a type → update consuming components
- If you modify a service → update calling code
- If you add a route → update navigation links
- Files are interconnected—never work in isolation

**5. Concise Communication**
- Keep responses short (1-2 lines max for summaries)
- Focus on WHAT changed & WHY
- **STRICTLY DO NOT create ANY MD files** (summary, analysis, guides, documentation, etc.) **WITHOUT explicit user request**
- Update SUPABASE_NOTES.md only for DB schema changes (must be explicitly requested)

### ❌ DON'Ts

- ❌ Never modify Firebase files without explicit request
- ❌ Don't copy-paste components—extract shared logic
- ❌ Never use local Supabase CLI migrations
- ❌ Don't ignore RLS policies—all inserts/updates need user_id validation
- ❌ Don't leave broken code—fix all errors before completing
- ❌ Never create one-off components when a universal one exists
- ❌ **NEVER create MD files, summaries, analysis docs, or guides without explicit user request** - Work silently, no documentation unless asked

---

## Critical Architecture

### Hybrid Auth System (Firebase + Supabase)
- **Firebase Auth:** Email/Google OAuth authentication
- **Supabase DB:** User profiles, projects, inquiries, inventory
- **AuthContext** (`src/contexts/AuthContext.tsx`): Syncs both systems
- **Pattern:** Auth state change → fetch profile from Supabase → auto-create if missing

### Service Layer (3 Main Services)

**User Service** (`src/supabase/userService.ts`)
- Profile CRUD, project list fetching

**Project Service** (`src/supabase/projectService.ts`)
- Full project lifecycle: create, save rooms/requirements, update costs
- JSONB fields: `client_info`, `property_details`, `rooms`, `sections`
- **Pattern:** Every update validates user_id matches (RLS enforced)

**Admin Service** (`src/supabase/adminService.ts`)
- Inquiries, inventory, testimonials, settings
- Checks `is_admin: true` before mutations

### Data Structure

```typescript
Project {
  client_info: { name, email, phone, address },
  property_details: { type, size, budget },
  rooms: Array<{ id, name, type, appliances }>,
  sections: Array<{ id, name, items }>,
  status: 'draft' | 'in-progress' | 'completed'
}
```

### Page Flow
1. `/` → `/intake` → `/room-selection` → `/requirements` → `/final-review` → `/planner`
2. `/history` - User's projects
3. `/admin/*` - Admin dashboard (role-protected)

---

## Development Workflow

### Database Changes (Dashboard-First Approach)
1. Write SQL → Supabase Dashboard > SQL Editor
2. Create migration file: `supabase/migrations/###_description.sql`
3. Verify schema in Tables view
4. Update `src/supabase/types.ts` to match schema
5. Update services if needed
6. Test in browser

### Code Standards
- **Types:** snake_case (DB), camelCase (UI components)
- **Error handling:** `console.log('✅ action', 'message')` | `console.error('❌ Error:', error)`
- **Validation:** Use TypeScript strictly—no `any` unless absolutely necessary
- **Components:** UI primitives in `src/components/ui/`, features in `src/components/features/`

### Git Workflow
- Branch: `feature/name` or `fix/bug-name`
- Commit: `feat: desc` | `fix: desc` | `update: desc`
- Always test: `npm run dev`

---

## Key Patterns & File Reference

| Path | Purpose |
|------|---------|
| `src/supabase/config.ts` | Supabase client init |
| `src/supabase/types.ts` | DB schema types (auto-generated) |
| `src/supabase/{user,project,admin}Service.ts` | Database operations |
| `src/contexts/AuthContext.tsx` | Auth state management |
| `src/types/*.ts` | UI layer types |
| `SUPABASE_NOTES.md` | **Full DB documentation, RLS policies, operations** |
| `GIT_WORKFLOW.md` | Git standards |

---

## Common Tasks

### Adding Inventory Category
1. Insert in `adminService.ts` SQL
2. Update `src/constants/inventory.ts`
3. EstimatedCost auto-matches by category

### Extending Project Schema
1. Update SQL → Run in Dashboard
2. Modify `src/supabase/types.ts`
3. Add getter/setter in projectService if new field
4. Update consuming components
5. Use optional chaining (`?.`) for backward compatibility

### Creating Admin Feature
1. Add method in `src/supabase/adminService.ts`
2. Verify `is_admin: true` before mutations
3. Create UI in `src/components/admin/`
4. Add protected route in `src/pages/admin/`

---

## Important Gotchas

1. **RLS enforced** - All mutations must validate user_id or admin status
2. **JSONB nulls propagate** - Use `null` explicitly, test undefined values
3. **Timestamps are ISO strings** - Not JS Date objects
4. **Mobile-first responsive** - Test all features on mobile viewport
5. **Don't modify Firebase** (`src/firebase/`) without explicit request

---

## Resources

- **Supabase Dashboard:** https://supabase.com/dashboard/project/dalqnrkpjzlcklhqsoum
- **SUPABASE_NOTES.md:** Full schema, RLS policies, common SQL operations
- **README.md:** Features & deployment
- **GIT_WORKFLOW.md:** Git best practices

---

## Your Approach: Broad-to-Narrow

1. **Understand the structure** - Read related files, identify dependencies
2. **Plan systematically** - What files need changes? In what order?
3. **Implement holistically** - Make changes across all related files
4. **Verify thoroughly** - Test in browser, check for errors/console warnings
5. **Summarize concisely** - State what changed and why in 1-2 lines

**Trust your analysis. You're designed for this codebase.**

````

---
> Source: [Krishna9827/CleubAutomation](https://github.com/Krishna9827/CleubAutomation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-30 -->
