<!-- IMPORTANT, IGNORING WILL THROW ERRORS -->
ALL DIALOG RECEIVE children PROP, that acts as trigger for the dialog (unless specified otherwise or valid reason not to)
WE ARE USING ZOD V4, NOT ZOD V3 SO MAKE SURE TO USE ZOD V4 SYNTAX (BREAKING CHANGES FROM V3)
DO NOT EDIT ESLINT.CONFIG FILE AT ANY COST, WITHOUT ASKING

most standalone components should not receive any on* events, so it can be called from a server component, server component cant pass unserializable props

use #file:action-button.tsx  instead of a custom confirmation dialog for actions that mutate data/ delete etc

🚨 STRICT CODING RULES — NON-NEGOTIABLE 🚨

❌ as IS BANNED (unsafe casting)
❌ any IS BANNED (type safety killer)
❌ ! (non-null assertion) IS BANNED (dangerous assumptions)
❌ // @ts-ignore IS BANNED (silencing the compiler = silencing safety)
❌ // @ts-expect-error IS BANNED (lazy excuse for broken code)
❌ Function TYPE IS BANNED (too vague — define proper signatures)
❌ eval() IS BANNED (security risk, unnecessary)
❌ MAGIC NUMBERS ARE BANNED (use well-named constants instead)

⚠️ RULE: If you feel the urge to bypass the type system → you're probably doing something wrong. Stop. Rethink. Refactor.


# 🚨 CODEBASE STRUCTURE & RULES 🚨

## 📂 Project Structure

* **`features/`** — each feature lives here

  * **`components/`** → React components for this feature
  * **`schemas/`** → Zod schemas (+ inferred types when needed)
  * **`server/`** → server-only functions (not actions)

    * 🔒 only callable from server code
    * usually for fetching data
  * **`actions/`** → safe client-callable actions

    * update or create resources from client components
  * **`search-params/`** → manages URL search params with `nuqs`

* **`services/`** → for services that are **not tied to a specific feature**

* **`shared/`** → reusable, shared features across the app


cache
if the calling page uses changed resource, purgecache with immediate mode.
for other cache purges, use swr mode as it is not access by caller page
prefer swr mode unless caller page uses the changed resource

---

## ⚡ Actions vs Server Functions

* ✅ **Actions**

  * live in `features/<name>/actions`
  * created + called using `safe-action-client` from `/lib/safe-action`
  * called from **client**

* ✅ **Server Functions**

  * live in `features/<name>/server`
  * created using safe action (for validation & auth)
  * ❌ NOT called via `safe-action-client`
  * only used in **server code**

---

## 🔒 Safe Action Rules

* All **actions** and **server functions** must use **Zod validation**
* Must pass through **auth middleware** when required
* No bypassing validation or type safety

---

## 🎨 UI Rules

* ✅ Prefer **ShadCN components** over custom ones
* ✅ Use only **ShadCN color classnames**
* ❌ No arbitrary Tailwind classes

---

## 📏 General Coding Rules

* 🚫 **`as` is banned**
* 🚫 **`any` is banned**
* 🚫 **`!` (non-null assertion) is banned**
* 🚫 **`@ts-ignore` / `@ts-expect-error` are banned**
* 🚫 **Function** type is banned (define proper signatures)
* 🚫 **Magic numbers** are banned (use named constants)
* 🚫 **Eval** is banned
* ✅ Always follow `/eslint.config.mjs`
prefer early return pattern to reduce nesting

---

⚠️ **Bottom Line:**
Every line of code must be **type-safe, validated, consistent, and compliant**.

# If you see a violation of these rules, call it out immediately.
# No exceptions. No excuses.
# Your vigilance keeps our codebase safe and robust!

⚙️ After Changing Code

please use ur internal get_errors tool, dont lint or build untill ur internal get_errors tool returns that no problems found. ur get_errors checker is more efficient

use ur get_errors as many times as needed until no problems found. it is free and unlimited, and it is very fast. you should love it.

Run lint for changed files
pnpm lint


Ensure your modified files comply with all linting and safety rules.

Run lint for the entire codebase, (after get_errors returns no problems found)
Once the changed files pass, run lint across the entire project to confirm nothing else is broken.

Build the codebase
get_errors for the workspace and then if no errors,
pnpm build


Verify that the project compiles successfully and behaves as expected.

🧱 Component and File Rules

page.tsx must always be a server component

❌ Never add "use client" at the top of a page.tsx.

✅ If you need client-side behavior, create a minimal client component inside the components/ directory and import it into the page.

⚡ Always minimize the number of client components — prefer server components for performance, security, and type safety.

Server imports are forbidden in client components

❌ Do not import anything from /server into a file containing "use client".

❌ Do not import "server-only" in a client component.

Either action will trigger build-time errors in Next.js.

🗄️ Database Schema Rule

❌ Do not change the database schema without explicit approval.

✅ If a schema update is required, request approval first before making modifications.

🧾 Action Logging Rule

All calls to logAction must follow this strict, non-vague message format:

[Action] [EntityType] '[EntityName]' [Details]

✅ Example
Create Tenant 'John Doe' with email 'john.doe@example.com'

✅ Format Breakdown
Part	Required	Description
[Action]	✅	The specific operation performed (e.g., Create, Update, Delete).
[EntityType]	✅	The entity affected (e.g., Tenant, Policy, Claim).
'[EntityName]'	✅	The entity's identifying name or label, wrapped in quotes.
[Details]	✅	A clear, specific description of what was done — must never be vague. Include diffs, context, or relevant data.
⚠️ Important Notes

Vague or incomplete log messages are strictly prohibited.
Each logAction entry must make sense on its own without requiring metadata.
✅ Example of a clear message:

Update Policy 'Comprehensive Cover' — changed status from 'Pending' to 'Approved'


❌ Example of a vague message (not allowed):

Updated policy


Users cannot view metadata, so the action message must contain all relevant context (what changed, who or what was affected, and how).

Metadata is only for internal tracking and should never be relied upon to understand the event.

Consistent, descriptive logging enables:

Accurate auditing

Easier searching and filtering

Clear visibility into all system activities


DO NOT EDIT ESLINT.CONFIG FILE AT ANY COST, WITHOUT ASKING

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gerkim62)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gerkim62)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
