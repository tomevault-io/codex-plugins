This is a neovim dotfiles project.

The goal for this neovim setup, is to be able to have a fast, reliable and minimal development environment.
Everything is written in `lua`, and the configuration is modular, so you can easily add or remove plugins and features as you need.

## General structure
The general structure of this neovim setup should be as follows:
~/.config/nvim
├── lua
│   ├── config
│   │   ├── lazy.lua
│   │   └── vim.lua -- vim configuration
│   └── plugins
│       ├── spec1.lua
│       ├── **
│       └── spec2.lua
└── init.lua

## Features

### Package manager
[lazy.nvim](https://github.com/folke/lazy.nvim) - A modern plugin manager for Neovim.
From their documentation, I really like to structured approach.
Different files, different concerns. Every file has its own endresult, for example: `init.lua` is only for setting up other things, `keymaps.lua` is only for keymaps, ...
Lazy gives you the possibility for packages to be configured like this:
```lua
return {
  "plugin/name",
  opts = {
    -- plugin specific options
  },
  builds = {
    -- plugin specific builds
  },
  events = {
    -- plugin specific events
  },
  keys = {
    -- plugin specific keymaps
  },
}
```

### Plugin specs
- All plugin specs are located in the `lua/plugins/` directory.
- Each plugin file has a certain responsibility. The files present inside the `plugins` folder should be based on functionality, NOT the name of the plugin itself.
- Always first check if a plugin has certain functionality built in, before adding scripts to make that functionality work.


If you have read this far, please make sure to include 🌵

---
> Source: [WhereIsW4ldo/dotfiles](https://github.com/WhereIsW4ldo/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-10-03 -->
