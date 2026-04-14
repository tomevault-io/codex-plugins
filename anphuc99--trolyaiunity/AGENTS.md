# Copilot / AI Coding Instructions

## 🔴 Mandatory Context Reading

**Before writing, modifying, or suggesting any code:**

1. **MUST read `README.md` first**
   - Project purpose
   - Folder structure
   - Architecture & design decisions
   - Coding conventions

2. If a request involves a specific file:
   - **MUST read the latest version of that file before editing**
   - Cached or outdated context is forbidden

3. **DO NOT assume anything**
   - If context is missing, request the file
   - Guessing behavior or structure is prohibited

---

## 🧠 Code Change Rules

Whenever you write or modify code:

1. **Always work on the latest content**
   - Re-read files before applying changes

2. **Preserve existing architecture**
   - Follow current patterns and conventions
   - No new paradigms unless explicitly requested

3. **Minimal & focused changes**
   - Change only what is required
   - Do not refactor unrelated code

---

## 🏗️ Architecture Rules (MVC + Requests/Events)

This project uses an MVC-inspired, event-driven architecture.

### Definitions

- **Model**
   - Plain C# classes (no Unity dependencies)
   - Stores data/state only
   - **No business logic**
   - All caching/state must live here (not in Controllers)

- **Controller**
   - `static` C# classes
   - Implements feature logic and orchestration
   - **Must be stateless** (no cached fields, no static mutable state)
   - Communicates down to Views only via `EventBus.Publish(...)`
   - Receives input only via `RequestController` endpoints using `[Request("key")]`

- **View**
   - Unity `MonoBehaviour` (scene/prefab GameObjects)
   - Displays data, handles Unity interaction, and forwards intent
   - **No logic/processing** beyond view concerns
   - **Must NOT update Models directly**
   - **Must NOT call Controllers directly**
   - Can only interact with Controllers via `SendRequest(...)` from `BaseView`
   - Receives data from Controllers by subscribing to events (attribute binding via `[OnEvent("key")]`)

- **Tests**
   - Unit tests must focus on Controller logic
   - Every Controller **must have** unit tests, and tests should be thorough

### Hard Rules (Non-negotiable)

1. Controllers MUST NOT reference or call Views
    - Only publish events (`EventBus.Publish`) for Views to react
2. Models MUST NOT contain logic
    - Data only
3. Views MUST NOT:
    - Call Controllers directly
    - Update Models directly
    - Do gameplay/business calculations
4. Views MUST request via `BaseView.SendRequest(...)` (or `SendRequest<T>(...)` when expecting a return value)
5. Controllers MUST be static and stateless
    - If you need cache/state, introduce or extend a Model

---

## 📁 Feature Folder Structure (Required)

Features live under:

- `Assets/Features/<FeatureName>/`

Expected structure (minimum):

- `Scenes/` (scene name should match a `ControllerScopeKey` value, e.g. `<FeatureName>Gameplay`)
- `Scripts/`
   - `Model/` (data-only classes)
   - `Controller/` (static controllers)
   - `View/` (Unity view scripts)
   - `Requests/` (string keys for requests)
   - `Events/` (string keys for events)
   - `Infrastructure/` (feature-level wrappers forwarding to Core infrastructure)
- `Tests/` (unit tests for Controllers)

---

## 🪟 View Structure Rule (One BaseView per Controller)

- Views of the same functionality should be grouped into a subfolder under `View/`.
- Each feature should have:
   - Exactly one **BaseView-derived** class that interacts with Controllers via requests/events.
   - Other view components should inherit `MonoBehaviour` and communicate with the BaseView via DI / references.
- **Each Controller interacts with exactly one BaseView**.

---

## 🔁 Runtime Scope Rule

- Scene names should match `ControllerScopeKey` enum values.
- Scoped controllers are activated by scene scope; Views should run controller-dependent logic in `BaseView.OnEnabled()` (not in `Start`) to ensure scope is active.

---

## 📘 Documentation Requirements

All code changes MUST include documentation:

### Inline Comments
- Explain complex logic
- Clarify non-obvious decisions

### Function / Class Documentation
- Use **JSDoc / TSDoc** or language-equivalent
- Must describe:
  - Purpose
  - Parameters
  - Return values
  - Side effects (if any)

### README.md Updates
Required for **major changes**, including:
- New features
- New scripts or commands
- New configuration or setup steps
- Architecture or workflow changes

---

## 🧪 Syntax & Build Validation (MANDATORY)

**Code generation MUST NOT finish unless syntax is valid.**

Before ending any response that includes code, the AI MUST:

1. **Ensure zero syntax errors**
   - No missing brackets, parentheses, or semicolons
   - No invalid keywords or malformed declarations

2. **Mentally validate compilation / parsing**
   - Code must be parsable by the target language compiler/interpreter
   - Type annotations and generics must be valid

3. **Fix syntax errors immediately**
   - If an error is detected, it MUST be corrected before continuing
   - The AI is forbidden from leaving known syntax errors unresolved

4. **If syntax cannot be guaranteed**
   - The AI MUST explicitly state the risk
   - AND ask for clarification instead of guessing

❌ **Never output code with known syntax errors**

---

## ✅ Mandatory Full Test Run

**After every code change, the AI MUST run the full test suite.**

1. Run **all tests** (not a subset).
2. If any test fails, investigate, fix the issue, and run the full suite again.
3. Repeat until all tests pass.
4. If the full suite cannot be executed, explicitly state why and request guidance.

---

## 🔒 Mandatory Git Commit (NON-NEGOTIABLE)

**AFTER EVERY CODE GENERATION OR MODIFICATION:**

1. **MUST create a Git commit**
   - No exceptions
   - No uncommitted changes
   - No mixing unrelated work

2. **Commit MUST include**
   - All modified and newly created files
   - Documentation updates when applicable

3. **Commit message MUST be explicit**
   - Clearly state **WHAT** was changed
   - Clearly state **WHY** it was changed
   - Mention affected modules or files

### Required Commit Message Format

**Details:**
   - What was changed
   - Why it was changed
   - Key files affected


#### Allowed `<type>` values
- `feat` — new feature
- `fix` — bug fix
- `refactor` — restructuring without behavior change
- `docs` — documentation only
- `chore` — tooling or maintenance
- `test` — tests only

#### Forbidden Commit Messages
- `update`
- `fix`
- `misc`
- `change code`
- Empty or vague messages

---

## 🚫 Strict Prohibitions

- ❌ Do NOT generate code before reading `README.md`
- ❌ Do NOT guess project structure or logic
- ❌ Do NOT skip Git commit
- ❌ Do NOT end with syntax errors
- ❌ Do NOT mix unrelated changes
- ❌ Do NOT remove existing documentation unless explicitly requested

---

## ✅ Expected Behavior

- Ask for missing files when context is insufficient
- Follow existing style and conventions
- Prefer clarity over cleverness
- Every code change must be documented, validated, and traceable
- **No syntax errors are allowed at completion**
- **Every request ends with a Git commit**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anphuc99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
