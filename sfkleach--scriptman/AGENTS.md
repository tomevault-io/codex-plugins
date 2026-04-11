# Scriptman coding guidelines

Scriptman is a command-line tool that manages single-file scripts downloaded
from GitHub repositories by wrapping them with native executables.

## Collaboration Guidelines

When providing technical assistance:

- **Be objective and critical**: Focus on technical correctness over agreeability
- **Challenge assumptions**: If code has clear technical flaws, point them out directly
- **Prioritize correctness**: Don't compromise on proper implementation to avoid disagreement
- **Think through implications**: Consider how users will actually use features in practice
- **Be direct about problems**: If something is wrong or will cause user confusion, say so clearly

The goal is to build robust, well-designed software, not to avoid technical disagreements.


## Collaboration Guidelines

- When testing the behaviour of a binary, such as scriptman, always use 
  `go run ./cmd/scriptman` rather than `./scriptman` directly. 
  This ensures we are always testing the latest code rather than an out-of-date 
  compiled binary. (Unless we are deliberately testing an out-of-date binary).
- Do not create artefacts within the repository folder structure
  - EXCEPT in folders starting with an underscore, such as  `_build/`.
- Design decisions should be documents in the `docs/decisions/` folder using
  the established template. However you do not need to precisely follow the template.
  In some cases there were not multiple options considered so the pros-and-cons section may be omitted
  and we simply document the reasoning behind the decision.

## Programming Guidelines

- Comments should be proper sentences, with correct grammar and punctuation,
  including the use of capitalization and periods.
  - EXCEPT for comments that are simply single words or short phrases
    such as `// TODO: ...` or `// Deprecated` or bullet-points.
  - OR for comments that introduce a list. The introductory line will typically
    not be a complete sentence and will finish with `:` or `:-`.
- Where defensive checks are added, include a comment explaining why they are
  appropriate (not necessary, since defensive checks are not necessary).

## Programming Style Guidelines

For projects we own, including this one, we adopt the following single, uniform, good practice for our own projects and work entirely cross-platform with no use of "smart" defaults (e.g. Git's autocrlf).

- I prefer LF to CRLF/CR line endings in source code files and documentation files.
- I prefer text files to use new-line (LF) as a terminator rather than a separator
  i.e. newlines at the end of non-empty files, including on Windows.
- And lines should not have trailing whitespace EXCEPT in Markdown files where
  trailing whitespace indicates a line break. In those cases, use a single space
  at the end of the line to indicate a line break.
- We use 120 as the maximum line-length and not 80 characters. The detailed guideline
  is that the length first-to-last non-whitespace character should be 80 characters
  and that an additional 40 characters of indentation is allowed.
- Indentation in source files should use spaces only, no tabs EXCEPT in Golang or 
  Makefiles where tabs are effectively required.
- Use 4 spaces per indentation level EXCEPT when working in YAML/JSON files where 2 spaces per indentation level is more practical owning to higher nesting levels.
- UTF-8 encoding should be used for all text files EXCEPT when working with compilers/interpreters that do not support UTF-8.

## Developer documentation guidelines

- Use Unix-style paths (forward slashes) in code and documentation, even on Windows.
- Use Markdown for documentation files wherever possible with the .md file extension.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sfkleach)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sfkleach)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
