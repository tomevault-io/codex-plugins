# Copilot Code Review Guidelines

---
This file configures GitHub Copilot Code Review according to GitHub’s official guidance: [Use Copilot Code Review](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review/use-code-review).

## Project Context

**Language:** Ruby ~> 4
**Framework:** [trailblazer](https://trailblazer.to/)
**Testing:** [minitest](https://docs.seattlerb.org/minitest/) following [BetterSpecs](https://evenbetterspecs.github.io/) guidelines

---

## Purpose & Scope

Focused inline review guidance for Ruby pull requests—style, tests, performance, and tone. (CI setup, contributor workflows, and documentation policies belong in separate docs.)

---

## 1. Style Guides

> ⚠️ **Be constructive:** Frame style feedback as suggestions (e.g., “Consider using…”).

All standard rules follow the [Shopify Ruby Style Guide](https://ruby-style-guide.shopify.dev/); only team-specific overrides are listed below:
- Indentation and line length enforced by RuboCop per project config.
- Use guard clauses, and explicit receivers.
- Prefer Railway Oriented Programming (ROP) for error handling and flow.
- Leverage dry-rb conventions and Trailblazer patterns for clarity and maintainability.
- Keep methods short. 1 line is fine. 5 is a lot. At 7, you should consider extracting a method.

---

## 2. Tone and Delivery

> 🤝 **Be succinct:** Don't overwhelm with too many minor issues; prioritize key improvements.

- Use phrasing like “Consider extracting…” or “You might simplify…” rather than absolutes.
- Keep comments concise—2–3 sentences max.

---

## 3. Conventions & Readability

> ✨ **Keep it clear:** Propose naming and structure that clarify intent.

- **SOLID principles:** Single responsibility; inject dependencies via initializers.
- **Descriptive names:** Recommend clear class, method, and variable names.
- **DRY:** Suggest extracting duplicate logic into modules or service objects.
- **Method size:** Flag methods >= 8 lines; suggest splitting into helpers.
- Ensure files have EOF newline.
- Flag trailing white space in code and comments.
- Always ensure rubocop runs cleanly.

---

## 4. Testing and Coverage

> 🧪 **Be thorough:** Highlight gaps in meaningful test coverage.

- Flag missing or outdated specs for changed behaviors (unit, integration, request).
- Suggest tests for edge cases: error conditions, invalid inputs, boundary values, and potential `nil` handling or pointer exceptions. Follow best practices from the [BetterSpecs style guide](https://evenbetterspecs.github.io/).
- Encourage descriptive `context` blocks and example names per [BetterSpecs](https://evenbetterspecs.github.io/).
- Flag missing specs when logic is updated but no tests were added or modified
---

## 5. Performance and Security

> 🚀 **Guard quality:** Call out patterns that may hurt performance or security.

- Detect potential N+1 queries; suggest `includes` or batching.
- Attempt to identify O(n^2) or O(n log n) algorithms; recommend more efficient alternatives.
- Flag raw SQL, unsanitized interpolation, or other patterns that could introduce potential SQL injection vulnerabilities; recommend parameter binding and Sequel-based alternatives. Build these into a model, avoid raw sequel unless absolutely necessary.
- Identify unescaped user input in route methods

---

## 6. Commit Message and PR title guidelines

> 📝 **Be clear:** Ensure commit messages and PR titles reflect changes accurately.

- We use Conventional Commits. Ensure commit messages follow the format: `<type>(<scope>): <description>`.
- We use Semantic PR titles as well, ensure PR titles also match the conventional commit format.

## 7. Balancing Feedback

> ⚖️ **Be selective:** Focus on the highest-impact issues.

- Limit critical suggestions to the top 3–5 items per review.
- Combine minor style tweaks into a single comment when possible.

---

## 8. Quick Reference

- **Max comment length:** 2–3 sentences.
- **Severity tags:** `[Major]`, `[Medium]`, `[Minor]`.
- **Mandatory header:** `Enable frozen-string-literal`.
- **Top issues:** 3–5 critical comments.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rubyists) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
