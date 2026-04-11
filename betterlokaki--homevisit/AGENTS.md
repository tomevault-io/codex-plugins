# GitHub Copilot Instructions for HomeVisit

## Status Terminology & Merged Statuses

**CRITICAL: Use consistent terminology throughout the codebase:**

### Core Status Types

- **`coverStatus`** (formerly `updateStatus`): Represents the coverage status from external service

  - Type: `CoverStatus = "Full" | "Partial" | "No"`
  - Indicates whether new coverage data came in today
  - **Always use `coverStatus` - never `updateStatus` or `updatedStatus`**

- **`seenStatus`**: Represents the visit status of a site

  - Type: `SeenStatus = "Seen" | "Partial" | "Not Seen"`
  - Indicates whether the site has been visited/inspected

- **`mergedStatus`**: The combined result of `coverStatus` + `seenStatus`
  - Type: `MergedStatus = "Seen" | "Partial Cover" | "Partial Seen" | "Not Seen" | "Not Cover"`
  - Used for display and business logic decisions

### Merged Status Logic

Merged statuses combine coverage availability with visit status. Understanding them is critical:

#### No Coverage (Red) 🔴

- **Meaning**: Cannot see the site because **no new coverage came in today**
- **Cover Status**: `coverStatus = "No"`
- **Visit Status**: Always `seenStatus = "Not Seen"` (cannot visit without coverage)
- **Merged Status**: `"Not Cover"`
- **Display**: Red badge with "אין איסוף" (No Collection)
- **Note**: Currently stored as `"No"` in the database, but understand it means "No Coverage"

#### Partial Coverage (Yellow) 🟡

- **Meaning**: Only **partial coverage came in today**, so sites can only be partially seen
- **Cover Status**: `coverStatus = "Partial"`
- **Visit Status**: Can be either:
  - `seenStatus = "Partial"` → **Merged**: `"Partial Seen"` (Yellow - partially visited)
  - `seenStatus = "Not Seen"` → **Merged**: `"Partial Cover"` (Yellow - waiting for visit)
- **Display**: Yellow badge indicating partial state

#### Full Coverage (Green) 🟢

- **Meaning**: **Full coverage came in today**, allowing complete site inspection
- **Cover Status**: `coverStatus = "Full"`
- **Visit Status**: Can be:
  - `seenStatus = "Seen"` → **Merged**: `"Seen"` (Green - fully visited)
  - `seenStatus = "Partial"` → **Merged**: `"Partial Seen"` (Yellow - partially visited)
  - `seenStatus = "Not Seen"` → **Merged**: `"Not Seen"` (Yellow - waiting for visit)
- **Display**: Green when fully seen, Yellow when waiting or partially done

### Why Merge Statuses?

Merging `coverStatus` and `seenStatus` is important because:

1. **Business Logic**: Determines what actions are available (e.g., can't visit without coverage)
2. **UI Display**: Shows the correct color and text based on both coverage and visit state
3. **User Experience**: Users need to understand both "can I visit?" (coverage) and "have I visited?" (seen)

### Implementation Notes

- Always use `coverStatus` in new code (never `updateStatus` or `updatedStatus`)
- When displaying status, use `mergedStatus` for the final display value
- The `mergedStatusCalculator.ts` service handles the merging logic
- Frontend components should use `getUpdatedStatusDisplay(coverStatus, seenStatus)` for display

## Code Structure Rules

- **One type/class per file** - Never define multiple types or classes in a single file
- **Max 100 lines per file** - Split larger files into focused modules
- **Single responsibility** - Each class handles one logic only
- **Dependency injection** - Never instantiate dependencies inside classes, inject from outside
- **Throw first, catch later** - Services throw errors, only catch at the highest layer (controllers/schedulers)

## Frontend Conventions

- **RTL Layout**: All components use `dir="rtl"` - Hebrew is the primary language
- **No px units**: Use `%`, `rem`, or Tailwind relative classes only (never use px)
- **Generic components**: Build components as reusable/generic as possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betterlokaki)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/betterlokaki)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
