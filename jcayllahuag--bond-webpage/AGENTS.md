# Git Commit Standards

When generating commit messages, you MUST strictly follow the Conventional Commits specification (v1.0.0).

## Format
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

## Types
- **feat**: A new feature (e.g., adding the Scroll Dial to the Home page).
- **fix**: A bug fix.
- **docs**: Documentation only changes.
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc).
- **refactor**: A code change that neither fixes a bug nor adds a feature.
- **perf**: A code change that improves performance (e.g., optimizing Astro images).
- **test**: Adding missing tests or correcting existing tests.
- **chore**: Changes to the build process or auxiliary tools and libraries.

## Rules
1. Use the imperative, present tense: "change" not "changed" nor "changes".
2. The description must be in lowercase.
3. Do not end the description with a period.
4. Keep the first line under 50 characters.
5. If there is a breaking change, add a `!` after the type/scope or "BREAKING CHANGE:" in the footer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JCayllahuaG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
