## Versioning & Changelog

This project follows [Semantic Versioning](https://semver.org/) (SemVer) with version stored in the root `VERSION` file.

### When to Bump Version

**Before merging a PR**, determine the appropriate version bump:

- **MAJOR (X.0.0)**: Breaking changes that require user action or migration
  - Database schema changes that need manual intervention
  - API contract changes (removed/renamed endpoints, changed response structure)
  - Configuration changes requiring manual updates
  - Removal of features or significant behavior changes
  - Major architectural refactors affecting deployment

- **MINOR (0.X.0)**: New features or enhancements (backward-compatible)
  - New API endpoints or routes
  - New UI features or pages
  - New configuration options (with sensible defaults)
  - Database migrations that run automatically
  - Performance improvements with observable impact
  - New integrations or services

- **PATCH (0.0.X)**: Bug fixes, security patches, minor improvements
  - Bug fixes that don't change behavior for correct usage
  - Security vulnerability patches
  - Documentation updates
  - Dependency updates (non-breaking)
  - Code refactoring without functional changes
  - Performance optimizations (minor)
  - UI tweaks and styling adjustments

### Updating VERSION and CHANGELOG

**For every PR (and for every Copilot-implemented change intended to be deployed)**:

1. **Update `VERSION` file** with the new version number (X.Y.Z format only, no prefix)
2. **Update `CHANGELOG.md`**:
   - Add entries under `[Unreleased]` section using Keep a Changelog categories:
     - **Added** for new features
     - **Changed** for changes in existing functionality
     - **Deprecated** for soon-to-be removed features
     - **Removed** for now removed features
     - **Fixed** for any bug fixes
     - **Security** for vulnerability fixes
   - Use clear, user-facing language (not internal implementation details)
   - Include ticket/issue references when applicable

### Changelog Format

```markdown
## [Unreleased]

### Added
- New feature description (#123)

### Fixed
- Bug fix description (#456)

## [X.Y.Z] - YYYY-MM-DD

### Added
- Feature that was added
```

### Version Display

- Version is displayed in the public footer (all users can see it)
- Admins see an additional "Changelog" link to view full history at `/changelog`
- During deployment, `APP_VERSION` env var is injected from `VERSION` file
- Version is included in database backup filenames: `dsm_images_vX.Y.Z_timestamp.dump`
- Git tag `vX.Y.Z` is automatically created and pushed during deployment

### PR Checklist for Versioning

Before requesting review:
- [ ] VERSION file updated with appropriate bump (major/minor/patch)
- [ ] CHANGELOG.md updated under `[Unreleased]` section
- [ ] Changes categorized correctly (Added/Changed/Fixed/Security)
- [ ] Description is user-facing and clear
- [ ] Database migrations (if any) are documented in PR description

## README Maintenance

This is a public repo focused on helping QA teams get started with GitHub Copilot. The README and `docs/` folder must stay current and easy to navigate.

### Rules

- **Update on every change**: When adding a new agent, prompt, custom instruction, or tip, update all relevant locations:
  - `README.md` — add an entry to the appropriate table and update the repo structure section
  - `docs/` — add or update the corresponding doc page (e.g., `docs/agents.md`, `docs/prompts.md`)
- **Keep it scannable**: Use tables, short descriptions, and links to detail pages. The README is a landing page, not a deep-dive document.
- **Link to source files**: Always link directly to the `.agent.md`, `.prompt.md`, or instruction file so readers can grab what they need.
- **Maintain the repo structure block**: The ASCII tree in the README must reflect the current file layout.
- **Write for newcomers**: Assume readers are new to Copilot agents, prompts, and custom instructions. Keep language clear and jargon-free.

---
> Source: [aj-enns/ghcp-for-qa](https://github.com/aj-enns/ghcp-for-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
