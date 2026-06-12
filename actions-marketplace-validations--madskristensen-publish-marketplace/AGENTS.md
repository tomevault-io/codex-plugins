# Copilot Instructions

This repository is a GitHub Action. Follow these conventions when proposing changes.

## Versioning & releases

Use SemVer with a `v` prefix for all tags and releases.

### Tagging

- **Immutable release tags**: `vMAJOR.MINOR.PATCH` (e.g. `v2.0.0`, `v2.1.0`, `v2.1.1`). Always 3 parts, always `v`-prefixed. Never reuse or move these.
- **Moving major tag**: `vMAJOR` (e.g. `v2`) must always point to the latest release within that major version. Force-push it on every new release:
  ```
  git tag -f v2 v2.1.0
  git push origin v2 --force
  ```
- **Bump MAJOR only on breaking changes** — renaming or removing an input, changing required behavior, or anything that would break an existing consumer's workflow. Otherwise bump MINOR (new feature) or PATCH (fix).
- Do **not** create two-part tags like `v1.1` or bare-number tags like `2`. They are not valid SemVer.

### GitHub Releases

- Create a GitHub Release for **every** immutable tag (`v2.1.0`), not for the moving major tag (`v2`).
- Release `name` must match the tag name exactly (e.g. tag `v2.1.0` → release title `v2.1.0`).
- Mark the newest release as **Latest**.
- Use auto-generated release notes from PRs (Releases → Draft new release → Generate release notes), or write a short summary plus a usage snippet.

### Consumer guidance

Users should reference the action as either:
- `@v2` — auto-updates within the v2 major (recommended for most users)
- `@v2.1.0` — fully pinned (recommended for security-sensitive workflows)

## Commit messages

Use the Conventional Commits format:

- First line: `type(scope): short summary` in the present tense.
- Valid types: `feat`, `fix`, `test`, `docs`, `refactor`, `chore`, `style`, `perf`.
- Example: `feat(action): pin VsixPublisher.exe via NuGet`
- Keep the first line under 72 characters.
- Leave the second line blank.
- From the third line, add a more detailed description explaining the reason for the change. Use bullet points for multiple points.

---
> Source: [actions-marketplace-validations/madskristensen_publish-marketplace](https://github.com/actions-marketplace-validations/madskristensen_publish-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-12 -->
