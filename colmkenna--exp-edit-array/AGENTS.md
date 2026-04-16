# ck-editable-array Development Guide

This project builds a template-driven web component for editable arrays using **strict Test-Driven Development (TDD)**. All features were implemented following the Red-Green-Refactor cycle with comprehensive documentation.

## Core Architecture

### Component Model
- **Type**: Native Web Component (Custom Elements v1 + Shadow DOM)
- **Entry**: `src/components/ck-editable-array/ck-editable-array.ts` (1500+ lines, single file)
- **Pattern**: Template-driven rendering with data binding via custom attributes
- **State**: Immutable data flow - all operations clone data, preventing external mutations

### Key Binding Attributes
The component uses custom `data-*` attributes for declarative behavior:
- `data-bind="fieldName"` — Binds input/display to a field (supports dot notation: `person.name`)
- `data-action="add|save|cancel|toggle|delete|restore"` — Declares button actions
- `data-row="N"` — Row index marker (internal, set by component)
- `data-mode="display|edit"` — Current row mode (internal)
- `data-field-error="fieldName"` — Validation error message container
- `data-row-invalid="true"` — Marks row as having validation errors

### Data Flow
1. User provides two `<template>` elements: `slot="display"` and `slot="edit"`
2. Component clones templates for each row and binds data via `data-bind` attributes
3. All edits create deep clones; original data never mutates
4. Changes emit `datachanged` event with fresh cloned snapshot

### Modal Edit Mode
- Opt in by adding the `modal-edit` attribute or setting `el.modalEdit = true`.
- Display rows stay inline; the active edit template is rendered inside a built-in modal overlay (`part="modal"`) with a dialog surface (`part="modal-surface"`, `role="dialog"`, `aria-modal="true"`). The shared `hidden` class toggles visibility/`aria-hidden`.
- Save closes the modal and dispatches `datachanged`; Cancel closes the modal without emitting `datachanged` and restores the snapshot.
- Styling hooks: overlay uses classes `modal-overlay`/`modal-surface`; see `examples/demo-modal-edit.html` for a styled reference. Automated coverage lives in `tests/ck-editable-array/ck-editable-array.modal-edit.test.ts`.

## TDD Development Workflow

**MANDATORY**: Tests must be written before implementation code. No exceptions.

### Red-Green-Refactor Cycle
1. **RED**: Write failing test describing desired behavior
2. **GREEN**: Write minimal code to make test pass
3. **REFACTOR**: Clean up while keeping tests green
4. Document cycle in `docs/steps.md` with date, test names, and files touched

### Test Organization
All tests live in `tests/ck-editable-array/`:
- `ck-editable-array.step1.render.test.ts` — Basic rendering and data binding
- `ck-editable-array.step2.public-api.test.ts` — Schema, factories, shadow DOM structure
- `ck-editable-array.step3.lifecycle-styles.test.ts` — connectedCallback, style mirroring
- `ck-editable-array.step4.rendering-row-modes.test.ts` — Display/edit mode switching
- `ck-editable-array.step5.add-button.test.ts` — Add button, exclusive locking
- `ck-editable-array.step6.save-cancel.test.ts` — Save/cancel, toggle events
- `ck-editable-array.step7.validation.test.ts` — Schema-driven validation (2000+ lines)
- `ck-editable-array.step8.cloning.test.ts` — Deep cloning, immutability guarantees
- `ck-editable-array.modal-edit.test.ts` — Modal overlay edit mode (Step 9)
- `ck-editable-array.accessibility.test.ts` — ARIA attributes, keyboard navigation
- `ck-editable-array.security.test.ts` — XSS protection via `textContent`

Test utilities in `tests/test-utils.ts` provide helpers like `waitForRender()`, `getRow()`, `simulateInput()`.

### Running Tests
```powershell
npm test               # Run all tests
npm run test:watch     # Watch mode
npm run test:coverage  # Coverage report (target: high coverage)
```

## Documentation Standards

### Critical Files to Update
When adding features, update in this order:
1. **Write tests first** (see TDD workflow above)
2. `docs/steps.md` — Log each Red-Green-Refactor cycle with date, files touched
3. `docs/spec.md` — Formal specification with acceptance criteria matrix
4. `docs/README.md` — User-facing API reference (attributes, properties, events)
5. `docs/readme.technical.md` — Architecture diagrams, state flow, implementation notes
6. `examples/demo-*.html` — Working demos showcasing the new feature

### Checkpoint Pattern
After completing major steps, create `docs/checkpoint-YYYY-MM-DD-<feature>.md` summarizing:
- What was implemented
- Test coverage (e.g., "151 tests passing")
- Known limitations
- Next steps

## Validation System

The component supports JSON Schema-like validation via the `schema` property:

```javascript
el.schema = {
  required: ['name', 'email'],
  properties: {
    name: { minLength: 2 },
    email: { minLength: 5 }
  }
};
```

**Validation Behavior**:
- Runs on every input change (immediate feedback)
- Disables Save button when validation fails
- Sets `aria-invalid` on failed inputs
- Displays errors in `[data-field-error]` elements
- Adds `data-row-invalid` attribute to row wrapper

## Build & Development

### Commands
```powershell
npm run build        # Rollup build (UMD, ESM, minified)
npm run dev          # Watch mode
npm run serve        # Start http-server on :8080
npm run lint         # ESLint check
npm run lint:fix     # Auto-fix linting issues
npm run format       # Prettier format
```

### Build Artifacts
Rollup generates three bundles in `dist/ck-editable-array/`:
- `ck-editable-array.js` — UMD format
- `ck-editable-array.esm.js` — ES module
- `ck-editable-array.min.js` — Minified UMD

TypeScript declarations output to `dist/index.d.ts`.

## Coding Conventions

### TypeScript Patterns
- Use interfaces for data shapes (e.g., `InternalRowData`, `ValidationSchema`)
- Private properties prefixed with `_` (e.g., `_data`, `_schema`)
- Constants in SCREAMING_SNAKE_CASE as `static readonly` (e.g., `ATTR_DATA_BIND`)
- Explicit return types on public methods

### Style Management
Component mirrors light DOM styles into shadow DOM:
- Watch `<style>` elements in light DOM with MutationObserver
- Clone styles into shadow root on changes
- Use `::part(root|rows|add-button)` for external styling hooks
- Built-in `.hidden` class with `display: none !important`

### Event Contracts
Custom events follow strict contracts (see `docs/spec.md` Event Contracts section):
- `datachanged` — Fires on user edits (bubbles, composed, payload: `{ data: Array }`)
- `beforetogglemode` — Cancelable, fires before mode switch
- `aftertogglemode` — Fires after mode switch completes

## Common Pitfalls

1. **Never mutate `_data` directly** — Always clone with `cloneRow()` or `deepClone()`
2. **Don't dispatch `datachanged` on programmatic `data` set** — Only on user interactions
3. **Check `rowIndex` bounds** — Stale input events can reference deleted rows
4. **Set test expectations on cloned data** — `event.detail.data` is always a fresh clone
5. **Update all four docs files** — steps.md, spec.md, README.md, readme.technical.md

## Adding New Features

Follow this sequence:
1. Read `prompts/add-or-extend-webcomponent-feature-tdd.prompt.md` for detailed TDD workflow
2. Create new test file or extend existing step test (e.g., `step9.*.test.ts`)
3. Write RED test describing the feature
4. Implement GREEN code in `ck-editable-array.ts`
5. REFACTOR for clarity and consistency
6. Update all documentation (steps.md, spec.md, README.md, technical docs)
7. Add demo example to `examples/`
8. Run full test suite: `npm test`
9. Create checkpoint markdown file

## Project Philosophy

This codebase values:
- **Test coverage over speed** — No untested code ships
- **Incremental progress** — Small steps with clear documentation beats big leaps
- **Immutability** — Data cloning prevents subtle bugs
- **Accessibility** — ARIA attributes, keyboard support, error announcements are first-class
- **Developer experience** — Clear docs, working demos, predictable API

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ColmKenna) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
