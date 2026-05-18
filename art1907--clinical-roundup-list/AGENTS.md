# Copilot Instructions: Clinical Rounding Application

## Project Overview
Healthcare patient census/urology rounding platform. Single-page HTML app with Firebase Firestore real-time sync. Mobile-first adaptive UI, no build system. See compliance roadmap in `.kiro/specs/healthcare-compliance/{requirements,design,tasks}.md`.

## Architecture & Core Patterns

**Single File Structure**: All code in `clinical-rounding-adaptive.html` (vanilla JS, Tailwind CSS, no bundler).

**Data Flow**: 
1. Firebase Auth → `currentUser`
2. `onSnapshot` listeners populate `patients[]`, `onCallSchedule[]`, `globalSettings` 
3. UI renders via `renderUI()` → `renderMainTable()`, `renderCalendar()`, or `renderOnCallSchedule()` based on `currentTab`
4. User actions → modal form → `savePatient()` → Firestore doc create/update

**Key Global Objects**:
- `patients`: Array of records (room, name, dob, mrn, plan, status, findings, billing codes, etc.)
- `onCallSchedule`: Coverage assignments by date
- `globalSettings`: Default on-call provider and hospitals
- `Device`: Detects iOS/Android, tablet/phone, landscape/portrait
- `STATUS_COLORS`: Maps status strings to Tailwind + emoji (e.g., "In-Progress" → 🔵)
- `PROC_KEYWORDS`: Regex to identify surgical procedures for filtering

**Field Mapping Convention**: Form inputs use `id="f-{fieldname}"` (e.g., `f-room`, `f-findings-text`); mapped to camelCase in patient object (`f-supervisingMd` → `supervisingMd`).

## Development Workflows

**Local Testing**: Open HTML file in browser or serve via `python -m http.server`. Changes visible on reload.

**Firebase Setup**: Config injected via `window.__firebase_config` (JSON string) and `window.__initial_auth_token`. Offline mode uses local `patients[]` array.

**Critical Functions**:
- `openModal(id?)`: Opens patient entry modal; pre-fills from existing patient if editing
- `savePatient(e)`: Validates form, builds data object, creates/updates Firestore doc
- `setTab(name)`: Switches tabs (active/procedures/calendar/oncall/archive); calls `renderUI()`
- `renderMainTable()`: Renders patient table, highlights priority (🔴 STAT), shows status dropdown for procedures
- `generateHandoff()`: Creates formatted text handoff from selected patients (copy-to-clipboard)
- `startRealtimeListeners()`: Sets up collection listeners after auth

**Error Handling Pattern**: Try-catch + `console.error()` + `showToast()` (no silent failures). Example:
```javascript
try { 
  await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'patients', id), data);
  showToast("✓ Saved");
} catch (err) { 
  console.error("Save error:", err); 
  showToast("Error");
}
```

## Code Conventions

**No comments needed**: Names are self-explanatory (`isProc`, `isConnected`, `findingsCodes`).

**Inline events**: Use `onclick="window.functionName()"` in HTML; attach functions to `window` for global scope.

**Findings System** (complex pattern):
- Checkboxes (class `findings-checkbox`) store codes
- Hidden text inputs (class `findings-value`) store values keyed by code
- On checkbox change: show/hide corresponding value input; serialize both to `findingsCodes[]` and `findingsValues{}`
- Before save, convert hidden inputs to JSON: `findingsCodes` (comma-sep), `findingsValues` (stringified JSON)

**Tailwind-only styling**: Avoid custom CSS; use Tailwind utilities. Exception: `<style>` block for layout quirks (safe-area-inset, iOS -webkit- fixes).

## Compliance & Security

**Planned Compliance Features** (see `.kiro/specs/healthcare-compliance/`):
- Configurable modes: relaxed (current) → hipaa_strict (encrypted, audit logs, MFA) → sox_strict
- Field-level masking per role
- Audit logging for access/changes
- Encrypted exports to OneDrive/Google Drive
- 15-minute session timeout; MFA for financial data

**Current State**: Relaxed mode. When adding features, consider where compliance hooks will attach (e.g., mask sensitive fields, log access events, enforce role checks via RBAC layer).

**Security Validation**: Run appropriate security and code-quality checks before shipping.

## Common Modifications

**Add field to patient form**:
1. Add `<input id="f-{fieldname}">` in patient-modal
2. Add to `data` object in `savePatient()`
3. Add to field mapping in `openModal()` pre-fill logic
4. (Optional) display in `renderMainTable()`

**Add new tab**:
1. Nav button: `onclick="window.setTab('{tabname}')"`, `id="tab-{tabname}-btn"`
2. Content div: `id="{tabname}-section"`, class `hidden`
3. Render function: `render{TabName}()` called from `renderUI()`
4. Update `setTab()` visibility + styling logic

**Modify status dropdown**: Update `STATUS_COLORS` object + `<select>` options in modal.

**Firebase Path Structure**:
```
artifacts/{appId}/public/data/
  ├─ patients/{id}
  ├─ onCallSchedule/{id}
  └─ settings/global
```

---
> Source: [art1907/Clinical-Roundup-List](https://github.com/art1907/Clinical-Roundup-List) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
