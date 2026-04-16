# Copilot Instructions for dynaform

## Project Overview

**dynaform** is a TypeScript library for generating React forms from declarative YAML/JSON schemas. It targets non-programmers (business analysts, product managers) who need to author complex data-entry interfaces without writing JavaScript or React code.

## Architecture

The library follows a **headless, schema-first, tri-layered architecture**:

```
YAML/JSON Schema (DSL)
        │
        ▼
┌─────────────────────┐
│   Schema Compiler   │  Parses YAML → generates Zod schema + dependency graph (DAG)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  State Management   │  React Hook Form (uncontrolled refs, O(1) renders)
│  + Validation       │  + zodResolver for runtime-generated Zod schemas
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Component Registry │  Headless; maps DSL widget strings to user-provided React components
└─────────────────────┘
```

### Core Modules

1. **Schema Compiler** — Ingests YAML DSL, outputs:
   - A dynamically constructed Zod validation object (never static, always runtime-generated via schema factory pattern)
   - A dependency graph (DAG via topological sort) to prevent circular render loops

2. **Form Engine** — Uses `<FormProvider>` from React Hook Form to provide form state context; avoids prop drilling

3. **AST Rule Engine** — Uses `json-logic-js` for safe, sandboxed evaluation of `visibleIf` / inter-field dependency rules (never `eval()`)

4. **Component Registry** — Maps abstract DSL widget names (e.g. `select`, `multiselect`) to concrete React components registered by the consumer app; makes the library UI-framework agnostic

### DSL Field Types → Zod → UI

| DSL `type` | Zod primitive | Notes |
|---|---|---|
| `text` | `z.string()` | Supports `minLength`, `maxLength`, regex |
| `integer` | `z.number().int()` | Rejects floats |
| `decimal` | `z.number()` | `multipleOf: 0.01` for currency |
| `bool` | `z.boolean()` | Use `z.preprocess()` to coerce HTML checkbox strings |
| `datetime` | `z.string().datetime()` or `z.coerce.date()` | Prefer ISO 8601 strings for serialization |
| `lists` | `z.array(...)` | Pairs with `useFieldArray`; supports `minItems`, `maxItems`, `uniqueItems` |

### Key Patterns

**Dynamic Zod schema generation** — Schemas are never static constants. A schema factory function reconstructs the Zod object when form state changes (e.g., hiding a field wraps it in `z.optional()` or removes it entirely).

**Option filtering via AST** — To exclude a value from a listbox based on another field's value, attach a `logic.visibleIf` JsonLogic rule to each option. The rendering component uses `useWatch` on the source field and evaluates the AST per option. The validation schema is NOT mutated — only the presentation layer filters the options array.

**DAG enforcement** — When compiling the schema, traverse all AST rules to build the dependency graph and perform a topological sort. Cyclic dependencies must be detected and rejected at compile time to prevent infinite render loops.

**`useWatch` for targeted subscriptions** — Use `useWatch` (not `watch`) to subscribe only to specific fields in dependent components. This avoids global form re-renders.

**Symmetrical validation** — The same YAML schema can be parsed in a Node.js backend to reconstruct the identical Zod schema, enabling server-side validation without duplicating validation logic.

## YAML DSL Structure

```yaml
form:
  id: form_id
  fields:
    - name: fieldName
      type: text | integer | decimal | bool | datetime | lists
      ui:
        label: "Human Label"
        widget: input | select | multiselect | checkbox | textarea | datepicker
      validation:
        required: true
        minLength: 2
      options:                    # for select/multiselect
        - value: option_key
          label: Display Label
          logic:
            visibleIf: {"!=": [{"var": "otherField"}, "some_value"]}  # JsonLogic AST
```

For multi-step workflows, the root contains `steps` (array) instead of `fields`, with JsonLogic routing rules evaluated at each step boundary.

## Technology Stack

- **React Hook Form** — form state (uncontrolled refs); use `<FormProvider>` + `zodResolver`
- **Zod** — runtime validation; use `zod-from-json-schema` or programmatic API for runtime schema generation
- **json-logic-js** — AST evaluation for `visibleIf` rules and option filtering
- **YAML parser** — ingests schema DSL (e.g. `js-yaml`)
- **TypeScript** — all types should be inferred from Zod schemas; avoid redundant type declarations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stormsw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
