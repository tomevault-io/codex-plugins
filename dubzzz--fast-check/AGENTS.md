# GitHub Copilot Instructions

## Pull Request Naming Convention

When creating or naming pull requests, follow the [gitmoji](https://gitmoji.dev/) specification:

### Format

- **For changes to the main `fast-check` package**: Use the format `emoji Description`
  - Example: `✨ Add new arbitrary for dates`
  - Example: `🐛 Fix edge case in integer shrinking`

- **For changes to other packages** (ava, vitest, jest, worker, poisoning, packaged): Use the format `emoji(package-name) Description`
  - Example: `👷(vitest) Add support for new vitest features`
  - Example: `🐛(jest) Fix compatibility with jest 29`
  - Example: `📝(ava) Update documentation for ava integration`

Limit the name of the PR to at most 50 characters for the Description part (that is, the text after the `emoji` or `emoji(package-name)` prefix). It forces the author to summarize the changes succinctly, making it easier for others to understand the main purpose of the PR at a glance.

### Common Gitmoji Examples

- ✨ Introduce new features
- 🐛 Fix a bug
- 📝 Add or update documentation
- ✅ Add or update tests
- 🏷️ Add or update types
- ⚡️ Improve performance
- 👷 Add or update CI/CD
- ♻️ Refactor code
- 🔧 Add or update configuration files
- 🎨 Improve structure/format of the code
- 🔥 Remove code or files
- 🚀 Deploy stuff
- 🔒️ Fix security or privacy issues
- ⬆️ Upgrade dependencies
- ⬇️ Downgrade dependencies
- 📦 Add or update compiled files or packages

For a complete list of gitmoji codes, see https://gitmoji.dev/

## Pull Request Template

When creating pull requests, always use the PR template located at `.github/PULL_REQUEST_TEMPLATE.md`. The template includes:

- **Description**: Provide a short description and link to related issues (if any)
- **Checklist**: Keep the entire checklist and all of its items; only check the boxes for the items that are fulfilled.
- **Advanced**: Fill in the Category and Impacts sections to help reviewers understand the changes

Make sure to fill out all sections of the template completely and accurately, and do not delete or reorder any sections or checklist items—only update their content and checkbox states.

---
> Source: [dubzzz/fast-check](https://github.com/dubzzz/fast-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
