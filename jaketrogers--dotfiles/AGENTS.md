# Copilot Instructions for dotfiles

## Project Overview

This is a personal dotfiles manager supporting two installation modes:
- **Minimal** (`./install.sh`): For devcontainers - installs curl, less, tmux, git, gpg, vim, zsh, omz
- **Full** (`./install.sh full`): For workstations - adds bat, fd, ripgrep, fzf, neovim

## Architecture Pattern

Each tool follows a **directory-per-tool** structure under the repo root:
```
<tool>/
├── install.sh     # Optional: Custom installation (skips OS package manager)
├── configure.sh   # Optional: Symlinks configs and sets up plugins
└── <config-files> # Tool-specific configuration files
```

The orchestration flow is: `install.sh` → `bin/dotfiles <tool>` → `<tool>/install.sh` (if exists) → `<tool>/configure.sh` (if exists)

## Key Conventions

### Script Patterns
- All bash scripts use `#!/usr/bin/env bash` with `set -e`
- Use `$DOTFILES_LOCATION` for paths back to this repo
- Use `$INSTALL_MODE` to check for `'full'` vs minimal install
- Use `$elevate` variable for sudo (empty when running as root)
- Symlink configs using `ln -sf` to home directories (`~/.config/`, `~/`, etc.)

### Adding a New Tool
1. Create `<tool>/` directory with `install.sh` and/or `configure.sh`
2. Add `./bin/dotfiles <tool>` call in root `install.sh` (conditionally under `full` if needed)
3. Handle package name → command name mapping in [bin/dotfiles](bin/dotfiles) if they differ (see `neovim`→`nvim`, `ripgrep`→`rg`)

### Zsh/Oh-My-Zsh Structure
Custom zsh files live in [omz/](omz/) and are symlinked to `$ZSH_CUSTOM`:
- `aliases.zsh` - Shell aliases
- `functions.zsh` - Function autoloads
- `functions/` - Individual function files (sourced on-demand)
- `completions/` - Completion functions (prefixed with `_`)
- `env.zsh`, `variables.zsh` - Environment setup

### Neovim Configuration
Uses lazy.nvim plugin manager with structure in [neovim/lazy-nvim/](neovim/lazy-nvim/):
- Entry point: `init.lua` → loads `jake.core` and `jake.lazy`
- Plugins defined in `lua/jake/plugins/*.lua`
- LSP configs in `lua/jake/plugins/lsp/`

### Git Hooks
Pre-commit framework hooks are templated in [git/git-template/hooks/](git/git-template/hooks/) and linked to `~/.git-template/hooks`

## Testing Changes

```bash
# Run full installation
./install.sh full

# Test individual tool
./bin/dotfiles <tool-name>
```

## Changelog & Versioning

- Follow [Conventional Commits](https://www.conventionalcommits.org/) format
- Prefix scope with tool name: `fix(neovim):`, `feat(omz):`
- See [CHANGELOG.md](CHANGELOG.md) for version history and patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JakeTRogers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
