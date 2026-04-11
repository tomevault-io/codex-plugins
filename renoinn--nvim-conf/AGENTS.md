# Copilot Instructions for nvim_conf

A Neovim configuration repository using **Lazy.nvim** for plugin management, with a focus on LSP, mini.nvim, and Snacks.nvim.

## Architecture Overview

### Plugin Manager & Entry Point
- **Lazy.nvim**: Plugin manager and configuration loader
  - `init.lua` bootstraps Lazy and imports three plugin spec directories:
    - `plugins/` - General plugins
    - `plugins/lsp/` - LSP and completion plugins
    - `plugins/ui/` - UI-specific plugins
  - Also requires `options.lua`, `keymaps.lua`, and `luals.lua` for configuration

### Core Organization
```
lua/
├── options.lua         # Vim options (editor settings, clipboard sync, etc.)
├── keymaps.lua         # ALL keybindings except plugin-specific ones
├── luals.lua           # Lua Language Server configuration for Vim API
└── plugins/            # Lazy.nvim plugin specifications
    ├── snacks.lua      # Dashboard, picker, notifier, explorer
    ├── mini.lua        # mini.nvim bundled modules (ai, surround, pairs, comment, etc.)
    ├── sidekick.lua    # Sidekick AI integration
    ├── clue.lua        # Which-key style command hints
    ├── gitsigns.lua    # Git signs in gutter
    ├── guess-indent.lua # Auto-detect indentation
    ├── lsp/
    │   ├── nvim-lspconfig.lua      # LSP server setup (Mason + lspconfig)
    │   ├── blink-cmp.lua           # Completion engine
    │   ├── treesitter.lua          # Syntax highlighting + text objects
    │   └── conform.lua             # Code formatting
    └── ui/
        ├── heirline.lua            # Status line + tab bar
        ├── colorscheme.lua         # Kanagawa theme
        └── mini.icons.lua          # Icon configuration
```

### Key Design Principles
1. **Keymap Centralization**: All keybindings (except plugin-specific setup) live in `keymaps.lua` to prevent scattering. Exception: plugin-specific keybindings are set within plugin configuration files using `vim.keymap.set()` or Snacks/mini built-in keybinding options.

2. **Plugin Priority & Lazy Loading**:
   - Snacks.nvim: `priority = 1000, lazy = false` (loads first, always active)
   - Most others: lazy-loaded on first use or specific events

3. **Snacks vs mini.nvim**: When functionality overlaps, **Snacks.nvim** takes priority for UI features (Dashboard, Picker, Notifier). mini.nvim handles editing operations (Surround, Pairs, Comment).

## Code Conventions

### Lua Style
- **Formatter**: Stylua (`column_width = 120`, `indent_width = 2`, spaces, double quotes preferred)
- **LSP Configuration**: Included in `.luarc.json` (note: configuration is disabled or external; main Lua LS setup is in `luals.lua`)
- All type annotations use LuaLS comment syntax (`---@type`, `---@param`, etc.)

### Plugin Configuration Pattern
Each plugin file returns a Lazy spec:
```lua
return {
  {
    "plugin/name",
    opts = { ... },        -- Lazy will automatically call setup(opts) or init(opts)
    config = function() end,
    keys = { ... },        -- Lazy keybindings
    -- or use vim.keymap.set() in config function
  }
}
```

### Import Structure
Lazy specs use `{ import = "plugins" }` pattern. To add a new plugin:
1. Create file in `lua/plugins/` (or `lua/plugins/lsp/` or `lua/plugins/ui/`)
2. Return a Lazy plugin spec
3. Lazy auto-discovers it

### Keybinding Helpers
`keymaps.lua` includes helper functions like `nmap()` for common keymap patterns. Use these for consistency.

## Build, Test & Lint

### Formatting
```bash
# Format Lua files with Stylua
stylua lua/
```

No build, test, or CI pipeline currently configured. This is a configuration repository, not a package.

## Important Notes

- **Dependencies**: Ensure Neovim 0.9+ is installed (for Lazy.nvim compatibility)
- **Mason**: Manages LSP servers, formatters, and linters. Run `:Mason` inside Neovim to manage them
- **Treesitter**: Auto-installs parsers on demand; ensure `git` and a C compiler are available
- **Terminal Mode**: `<Esc><Esc>` to exit terminal mode (defined in keymaps.lua)
- **Clipboard**: Synced with OS by default (scheduled after `UiEnter`)
- **Undo**: Persistent across sessions (enabled by default in options.lua)

## Related Files
- `.stylua.toml` - Stylua formatter configuration
- `.gitignore` - Excludes tags, test.sh, nvim directory, spell files, and more
- `lazy-lock.json` - Locked plugin versions (commit-pinned)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/renoinn)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/renoinn)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
