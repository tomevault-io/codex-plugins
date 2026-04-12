
<rule>
name: conventional_git_commit_message
description: Enforce commit-message style when AI generates commit messages
when: "generating git commit message"
then:
  - type: "instruction"
    value: |
      Use the following structure for commit messages:

      <type>[optional scope]: <short description>

      [optional blank line + detailed body explaining what/why changed]

      [optional blank line + footer(s): e.g. "BREAKING CHANGE: ..." or "Closes #123"]

      - Allowed types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
      - Use lowercase for the type.
      - If you include a scope, wrap in parentheses immediately after the type (no spaces), e.g. `fix(api): ...`
      - Keep the subject (description) concise — ideally under 50–60 characters.
      - Use imperative, present-tense style in the subject (e.g. “add login support”, “fix crash on startup”).
      - If there are multiple points, put them in bullet list in the body.
      - If the changes break compatibility or change behavior in a non-backward-compatible way, mark as breaking: either with `!` after type/scope, e.g. `feat(db)!: migrate schema`, **or** include a `BREAKING CHANGE:` footer.
      - If the commit resolves or closes an issue, add `Closes #<issue-number>` in the footer.

</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndyHaas)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/AndyHaas)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
