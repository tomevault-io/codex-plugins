---
### Commit Message Generation
When asked to generate a commit message for staged changes, your goal is to create a message that is clean, logical, and tells the story of the change. Adhere to the following process:

**1. Create the Subject Line:**
- Analyze the changes to determine the primary intent.
- Write a concise subject line in the **Conventional Commits** format: `<type>(<scope>): <subject>`.
- The subject must be in the imperative mood (e.g., "add", "fix", "refactor") and under 50 characters.

**2. Write the High-Level Summary (Body):**
- After a single blank line, write one or two sentences describing the core problem that was solved or the main goal of the feature. This is the "why" behind the change.

**3. Detail the Most Impactful Changes (Body):**
- Do NOT simply list every file that was changed. Instead, analyze the diff and identify the 2-4 most significant **logical changes** or improvements.
- Describe these changes as bullet points. Focus on the concepts, not just the code.
- *Good Example:* "- Abstract user validation into a dedicated service class."
- *Bad Example:* "- Modified auth_service.py and user_api.py."

**4. Add the Footer (Conditional):**
- After another blank line, add a footer **if and only if** a relevant GitHub Issue number is mentioned in our chat or is clearly associated with the branch name.
- If an issue number is available, use the format: `Refs: #<issue-number>`.
- **If no issue number is known, omit the footer entirely.** Do not use placeholders.
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/travismi87) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
