## Additional Quality, Test & Codacy Enforcement Rules (Appended 2026-02-06)

- **Mandatory pre-commit reminder (must appear in every suggestion):**  
  "Before committing, run: npm run test:unit && npx playwright test tests/e2e --grep @smoke && codacy-analysis-cli analyze --upload"

- **Do not introduce regressions:**  
  - Never suggest code that would knowingly break existing passing unit tests, E2E tests, or trigger Codacy issues (security, duplication, complexity).  
  - Always verify tests locally before pushing.

- **Test-first workflow:**  
  - Propose failing test(s) first (Vitest for unit/integration; Playwright for E2E), then the minimal code required to make them pass.  
  - For logic changes, include a **"Tests to add"** section listing specific Vitest or Playwright specs to add (file name, brief description, and expected failing assertion).

- **When touching critical files (must be explicit):**  
  - If modifying `z80.mjs`, `ula.mjs`, `memory.mjs`, `frameBuffer.mjs`, `input.mjs`, or any rendering/input logic:  
    - Propose matching Vitest unit/integration tests and, if appropriate, Playwright E2E scenarios.  
    - Mention likely Codacy flags (e.g., cognitive complexity, magic numbers, security hotspots) and suggest mitigations.

- **Tools & scope guidance:**  
  - Prefer **Vitest** for isolated logic (Z80 opcodes, flags, ULA port reads with row mask, backfill).  
  - Use **Playwright E2E** for full-system behavior (boot success, keyboard → canvas text, © glyph visibility, status updates).  
  - End every suggestion with:  
    "Verify locally with: npm run test:unit && npx playwright test tests/e2e/boot*.spec.mjs tests/e2e/glyph*.spec.mjs && codacy-analysis-cli analyze --upload"

- **Commit & PR practices:**  
  - Suggest small, focused commits with clear messages, e.g.:  
    - `fix: ULA readPort row mask handling`  
    - `test: add Z80 ED prefix coverage`  
  - Before suggesting a commit, remind: "Run full unit suite + fast E2E subset + Codacy local scan locally first".  
  - For PRs to `main`: require passing GitHub Actions (unit + full E2E), SonarCloud quality gate, **and Codacy status** before merge.

- **Quality & static analysis priorities:**  
  - Prioritize coverage for low-coverage files: **`z80.mjs` (~49–53%), `ula.mjs` (~53%), `memory.mjs` (~54%)**.  
  - Flag potential SonarLint/SonarCloud/Codacy issues: high complexity, magic numbers, missing strict equality (`===`), security hotspots, duplicated code. Suggest concrete fixes (refactor, constants, smaller functions).

- **Hardware & domain-specific constraints:**  
  - Z80 behavior must match real hardware: flags (including undocumented bits 3/5), timing, prefix handling (DD/FD/ED/CB).  
  - ULA keyboard reads must use full 16-bit address (row mask = `addr >> 8`).  
  - Canvas rendering must be pixel-perfect (320×240 with borders) and correct **ink/paper/bright** mapping.  
  - Never break boot sequence or © glyph visibility.

- **Design & style:**  
  - Prefer testable patterns: pure functions, small methods, clear inputs/outputs.  
  - Keep suggestions concise, bulleted, and professional. Always include a **"Tests to add"** section for logic changes.

- **Final enforcement note:**  
  - All suggestions must explicitly mention potential Codacy and SonarCloud flags where relevant, include the mandatory pre-commit reminder (above), and end with the "Verify locally with: ..." line.

Always verify with unit tests, fast E2E subset, and Codacy local scan before commit.

## Additional Best Practices for .mjs, ES6+, Vitest & Playwright (Appended 2026-02-06)

Please refer to roo-code memory-bank instructions for more information.

- **.mjs / ESM style**
  - Use the `.mjs` extension for module files and always use ESM syntax (`import` / `export`); avoid `require()`.
  - Prefer named exports for clarity; use a single default export only where it simplifies the module's public API.
  - Use top-level `await` only when necessary and safe for consumers; modules are strict by default—no explicit `"use strict"` needed.
  - Keep module side-effects explicit and documented; avoid hidden global state.

- **ES6+ patterns**
  - Prefer `const` for immutable bindings and `let` for reassignable variables; avoid `var`.
  - Use arrow functions for concise callbacks and function expressions; prefer function declarations where hoisting or clarity is required.
  - Use object/array destructuring to improve readability and reduce noise.
  - Favor template literals for complex string construction and the spread/rest operators for argument/array handling.
  - Keep modules small and focused; prefer composition over large utility files.

- **Vitest best practices**
  - Organize tests with `describe`/`it` (or `test`) blocks; use `beforeEach`/`afterEach` for repeatable setup/teardown.
  - Name tests descriptively (e.g., "should handle ED prefix fallback correctly").
  - Use explicit matchers (`toBe`, `toEqual`, `toStrictEqual`, `toHaveLength`, etc.) and prefer `toStrictEqual` for deep structural assertions when appropriate.
  - Mock only what is necessary and scope mocks to the smallest possible area; prefer dependency injection where practical.
  - Use `it.each` for data-driven cases and keep tests isolated to avoid order dependencies.
  - Aim for clear coverage goals and add unit tests for low-coverage critical files (e.g., `z80.mjs`, `ula.mjs`, `memory.mjs`).
  - When proposing logic changes, include a **"Tests to add"** section with file name, brief description, and expected failing assertion.

- **Playwright E2E best practices**
  - Use `page.locator()` over raw selectors; prefer robust, semantic locators to reduce flakiness.
  - Always `await` actions and assertions (e.g., `await expect(locator).toBeVisible()`, `await expect(locator).toHaveText()`).
  - Use `test.describe` and fixtures for shared setup; capture traces and screenshots on failure to aid debugging.
  - Create a small, fast `@smoke` subset for pre-commit verification and broader `@regression` suites for CI.
  - Tag tests with `@smoke` / `@regression` and use `--grep` to run targeted subsets in CI and locally.

- **General code quality reminders**
  - Favor small, pure functions with clear inputs and outputs; minimize side-effects and global state.
  - Avoid magic numbers—extract named constants and document their meaning.
  - Use `===` / `!==` for comparisons and consistent naming conventions across the codebase.
  - Keep tests deterministic and independent; avoid network or file-system side effects in unit tests (use mocks/fakes instead).
  - Keep commits small, focused, and accompanied by relevant tests and a short changelog message.

- **Verification reminder**
  - Always verify with: `npm run test:unit && npx playwright test tests/e2e --grep @smoke`

---
> Source: [testudoq-org/zxspeccjs](https://github.com/testudoq-org/zxspeccjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
