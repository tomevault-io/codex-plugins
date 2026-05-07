# GitHub Copilot Instructions for Scrum Guide Expansion Pack

> **📖 Main AI Guide**: For comprehensive AI assistant instructions, see [agents.md](../agents.md)

## Quick Reference

This is a **Hugo-based static website** (v0.146.0+) hosted on **Azure Static Web Apps**. The site provides guidance for applying Scrum to complex work, AI, and adaptive strategy.

**Live Sites:**
- **Production**: [scrumexpansion.org](https://scrumexpansion.org)
- **Preview**: [agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net](https://agreeable-island-0c966e810-preview.centralus.6.azurestaticapps.net/)

## 🚨 Critical Information

### ⚠️ Primary Editing Location

**CRITICAL**: Most edits should be made to `site/content/` only. Other areas require deep knowledge of:
- Hugo static site generator and Hugo Modules
- Azure Static Web Apps deployment pipeline
- Bootstrap 5 and responsive design
- Multilingual i18n architecture

**Safe Editing Zones:**
- ✅ `site/content/` - Content files (Markdown)
- ✅ `site/data/contributions/` - Contributor attribution (YAML)
- ✅ `site/i18n/` - Translation strings (with guidance)

**Restricted Zones (Advanced Knowledge Required):**
- ⚠️ `site/layouts/` - Hugo templates (requires Hugo module understanding)
- ⚠️ `site/static/` - Assets and CSS (requires Bootstrap 5 knowledge)
- ⚠️ `.github/workflows/` - GitHub Actions (requires CI/CD expertise)
- ⚠️ `scripts/` - Automation scripts (requires PowerShell expertise)
- ⚠️ `site/hugo.yaml` - Configuration (requires Hugo expertise)

### Deployment Model

**IMPORTANT**: Understand the deployment workflow before making suggestions:

1. **Fork/Branch** → Make changes → Test locally
2. **Create PR** → Automatic deployment to PR-specific test site (`...{PullRequestId}.centralus.6.azurestaticapps.net`)
3. **Merge to Main** → Automatic deployment to **Preview** environment
4. **Create GitHub Release** with version tag → Automatic deployment to **Production**

**Production Deployment**: Use semantic versioning for releases:
- **Patch** (v1.0.1): Typo fixes, tiny changes
- **Minor** (v1.1.0): New sections, content additions
- **Major** (v2.0.0): Complete document revamp, breaking changes

### Hugo Version Requirement

- **Minimum**: Hugo Extended v0.146.0+
- **Hugo Modules**: Site uses [HugoGuides module](https://github.com/nkdAgility/HugoGuides/) for base templates
- **Local templates**: Only overrides and site-specific templates in `layouts/`

### Key File Locations

```
site/
├── content/         # Markdown content
├── layouts/         # Local template overrides only
│   ├── index.html  # Homepage override
│   ├── categories/ # Category templates
│   ├── creators/   # Creator templates (legacy)
│   ├── _partials/  # Local partial overrides
│   └── _markup/    # Render hooks
│   
│   # Base templates from Hugo module (NOT in local layouts/):
│   # - baseof.html, home.html, single.html, list.html
│   # Provided by: github.com/nkdAgility/HugoGuides/module
│
├── static/         # CSS, images, assets
├── i18n/           # Translations
└── hugo.yaml       # Main configuration (includes module imports)
```

## 📚 Documentation References

For detailed information, always reference these documents:

| Topic | Document | Purpose |
|-------|----------|---------|
| **AI Guide** | [agents.md](../agents.md) | Complete AI assistant instructions |
| **Getting Started** | [docs/getting-started.md](../docs/getting-started.md) | Setup and installation |
| **Development** | [docs/development.md](../docs/development.md) | Development workflows, Hugo template system |
| **Deployment** | [docs/deployment.md](../docs/deployment.md) | Deployment process and environments |
| **Content** | [docs/content-management.md](../docs/content-management.md) | Content creation, academic references |
| **Contributing** | [docs/contributing.md](../docs/contributing.md) | Contribution guidelines, PR process |
| **Translations** | [docs/translations.md](../docs/translations.md) | i18n process, language support |
| **Troubleshooting** | [docs/troubleshooting.md](../docs/troubleshooting.md) | Common issues and solutions |

## 🎯 Quick Guidelines

### Content Writing Style

- **Formal reference style**, not blog/article
- Professional, neutral, instructional tone
- Academic references with footnotes: `[^1]`, `[^2]`
- High information density, no filler
- Suitable for Scrum practitioners seeking depth

### References Format

Always use academic format:
```markdown
[^1]: Author (Year) 'Article title', *Journal*, Volume(Issue), pp. pages.
```
See [content-management.md](../docs/content-management.md) for complete reference formatting guide.

### Hugo Templates

- Use `_partials/` not `partials/`
- Use `_shortcodes/` not `shortcodes/`
- Homepage is `home.html` not `index.html`
- Templates in root `layouts/` not `layouts/_default/`

### Styling

- **Bootstrap 5** for all UI components
- Custom CSS in `site/static/css/style.css`
- Mobile-first responsive design
- Dark theme: Primary `#135289`, Cards `#353535`

### Internationalization

- All content must support multiple languages
- Use PowerShell script: `.\scripts\Create-TranslationTemplate.ps1`
- Test all language versions

## 🔄 GitHub Actions Workflows

The repository uses automated workflows for CI/CD and maintenance:

- **main.yaml**: Primary build and deployment workflow (all environments)
- **close-pr.yaml**: Automatically cleans up PR test environments
- **docs-to-wiki.yml**: Syncs `/docs` folder to GitHub Wiki
- **copilot-setup-steps.yml**: Development environment setup (manual)
- **discussion-tag-external-edits.yml**: Tags external contributor discussions
- **stale-issues.yml**: Closes stale duplicate issues (daily)
- **⛔ azure-static-web-apps-*.yml**: LEGACY - DO NOT USE

See [agents.md](../agents.md#-github-actions-workflows) for complete workflow details.

## ⚠️ Critical Warnings

### DO NOT

- ❌ Edit files outside `site/content/` unless user explicitly understands technical implications
- ❌ Edit Hugo templates without understanding the Hugo Modules architecture
- ❌ Modify workflows without CI/CD expertise
- ❌ Edit or run `.github/workflows/azure-static-web-apps-agreeable-island-0c966e810.yml` (legacy)
- ❌ Deploy to production without proper version tagging via GitHub Release
- ❌ Use Hugo versions below v0.146.0
- ❌ Skip multilingual support for content changes
- ❌ Use casual/blog style writing
- ❌ Forget academic references for claims

### ALWAYS

- ✅ Read [agents.md](../agents.md) for complete AI instructions
- ✅ Reference appropriate documentation before changes
- ✅ Test locally: `hugo server -D --source site`
- ✅ Follow semantic versioning for releases
- ✅ Maintain multilingual support
- ✅ Use academic reference format
- ✅ Follow PR process

## 🔧 Common Commands

```powershell
# Local development
hugo server -D --source site

# Production build
hugo --environment production --minify --source site

# Create translation template
.\scripts\Create-TranslationTemplate.ps1 -LanguageCode "de" -LanguageName "German"

# Check Hugo version
hugo version
```

## 🌐 Technology Stack

- **Hugo Extended** v0.146.0+ (static site generator)
- **Bootstrap 5** (CSS framework)
- **Azure Static Web Apps** (hosting)
- **GitHub Actions** (CI/CD)
- **PowerShell 7+** (automation)

## 📖 For Complete Information

See the comprehensive [AI Agent Guide (agents.md)](../agents.md) for:
- Detailed deployment model explanation
- Complete version numbering strategy
- Task-specific workflows
- Template system migration guide
- Common tasks and troubleshooting
- Security considerations

---

**Remember**: Accuracy and consistency over speed. Reference [agents.md](../agents.md) and documentation before making changes.

---
> Source: [ScrumGuides/ScrumGuide-ExpansionPack](https://github.com/ScrumGuides/ScrumGuide-ExpansionPack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
