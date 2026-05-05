# GitHub Copilot Instructions for Awesome Claude Plugins

This repository is a curated marketplace for Claude plugins, following the [Claude Plugin Marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces) specification.

## Repository Purpose

This is a plugin marketplace that provides a centralized location to discover and share Claude plugins. The repository contains:
- A marketplace metadata file (`marketplace.json`)
- Plugin definitions in the `plugins/` directory
- A plugin template for creating new plugins
- Validation scripts and CI/CD workflows

## Plugin Structure

All plugins must follow this exact directory structure:

```
plugins/your-plugin-name/
├── .claude-plugin/         # Plugin configuration directory (required)
│   └── plugin.json         # Plugin metadata in JSON format (required)
├── hooks/                  # Lifecycle hooks directory (required)
├── agents/                 # Agent definitions directory (required)
└── commands/               # Custom commands directory (required)
```

### Plugin Metadata Schema

The `.claude-plugin/plugin.json` file must contain these required fields:

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Clear description of what the plugin does",
  "author": {
    "name": "Author Name",
    "email": "author@example.com"
  }
}
```

## Naming Conventions

### Plugin Names
- Use lowercase letters only
- Separate words with hyphens (kebab-case)
- Be descriptive but concise
- Avoid special characters
- Examples: `weather-forecast`, `code-formatter`, `task-manager`

### Directory Names
- Plugin directory name must match the `name` field in `plugin.json`
- Example: If `name` is "example-calculator", directory must be `plugins/example-calculator/`

## Validation Requirements

### Before Any Changes
Always run the validation script before making changes:
```bash
./validate-plugins.sh
```

### Plugin Validation Checks
The validation script checks for:
- Valid JSON syntax in `plugin.json`
- Presence of all required fields (`name`, `version`, `description`, `author`)
- Semantic versioning format (X.Y.Z)
- Directory name matching plugin name
- Presence of required subdirectories (`hooks/`, `agents/`, `commands/`)

### CI/CD Validation
All pull requests are automatically validated via GitHub Actions (`.github/workflows/validate-plugins.yml`) which checks:
- Plugin JSON validity
- Marketplace.json validity
- No duplicate plugin names
- All marketplace references are valid

## Code Style and Best Practices

### JSON Files
- Use 2-space indentation
- Include proper newlines at end of file
- Validate JSON syntax before committing
- Keep descriptions clear and concise

### Shell Scripts
- Follow the existing bash script style in `validate-plugins.sh`
- Use proper error handling with `set -e`
- Include helpful error messages with emoji indicators (✅, ❌, ⚠️, 📦, 🔍)

### Documentation
- Use Markdown format for all documentation
- Keep documentation up to date with code changes
- Include code examples where helpful
- Reference the official Claude Code plugin examples: https://github.com/anthropics/claude-code/tree/main/plugins

## Contributing Workflow

When adding a new plugin:
1. Copy the `plugin-template/` directory to `plugins/your-plugin-name/`
2. Edit `.claude-plugin/plugin.json` with accurate metadata
3. Add components in `hooks/`, `agents/`, and `commands/` directories
4. Run `./validate-plugins.sh` to verify the plugin structure
5. Update `marketplace.json` to include the new plugin reference
6. Submit a pull request with a clear description

When modifying existing code:
1. Make minimal, focused changes
2. Test changes locally first
3. Ensure validation passes
4. Update relevant documentation
5. Follow existing code patterns and style

## Key Files

- `marketplace.json` - Marketplace metadata and list of available plugins
- `plugins/README.md` - Detailed plugin structure documentation
- `CONTRIBUTING.md` - Contribution guidelines
- `README.md` - Main repository documentation
- `validate-plugins.sh` - Plugin validation script
- `plugin-template/` - Template for creating new plugins

## Common Tasks

### Adding a New Plugin
1. Use the plugin template as a starting point
2. Ensure unique plugin name (check existing plugins)
3. Follow semantic versioning (start at 1.0.0)
4. Validate before submitting

### Updating Plugin Metadata
1. Increment version according to semantic versioning rules
2. Update relevant fields in `plugin.json`
3. Re-validate the plugin structure

### Modifying Validation Logic
1. Update `validate-plugins.sh` script
2. Test with existing plugins
3. Update CI/CD workflow if needed
4. Document any new validation rules

## Important Notes

- This repository structure is based on the official Claude Code plugin format
- Always maintain backward compatibility when making changes
- Keep the marketplace.json file synchronized with actual plugin directories
- Preserve the MIT License for all contributions
- Be respectful and follow the Code of Conduct outlined in CONTRIBUTING.md

## Testing

Before submitting changes:
1. Run `./validate-plugins.sh` to validate all plugins
2. Check that `marketplace.json` is valid JSON
3. Verify that all plugin references in `marketplace.json` point to valid files
4. Test that the plugin structure matches the documented schema

## Dependencies

This repository has minimal dependencies:
- `jq` - For JSON parsing and validation in shell scripts
- Bash shell for running validation scripts
- Git for version control

When making changes, avoid adding unnecessary dependencies to keep the repository lightweight and accessible.

---
> Source: [GiladShoham/awesome-claude-plugins](https://github.com/GiladShoham/awesome-claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-05 -->
