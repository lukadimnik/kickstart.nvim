# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A personal Neovim configuration based on [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim). It is a single-file-first config (`init.lua`) with optional modular plugin files, not a distribution.

## Formatting

Lua files are formatted with **StyLua**. Configuration is in `.stylua.toml`:

- 2-space indentation (spaces, not tabs)
- 160 column width
- Single quotes preferred
- No call parentheses (e.g., `require 'module'` not `require('module')`)
- Unix line endings

Run: `stylua --check .` to verify, `stylua .` to fix.

## Architecture

### Entry Point

`init.lua` — the main configuration file. Contains (in order):

1. Leader key setup (`<space>`)
2. Vim options (`vim.o.*`)
3. Keymaps
4. Autocommands
5. lazy.nvim plugin manager bootstrap
6. `require('lazy').setup(...)` with all plugin specs inline

### Plugin Manager

Uses [lazy.nvim](https://github.com/folke/lazy.nvim). Plugin lock file is `lazy-lock.json`. Plugins are installed/updated via `:Lazy` inside Neovim.

### Plugin Organization

- **Inline in init.lua**: Most plugins are defined directly in the `lazy.setup()` call
- **`lua/kickstart/plugins/`**: Optional kickstart-provided plugins, enabled/disabled by uncommenting `require` lines near the bottom of `init.lua` (~line 990)
  - Currently enabled: `lint` (nvim-lint with eslint_d), `neo-tree` (file browser, toggle with `\`)
  - Available but disabled: `debug`, `indent_line`, `autopairs`, `gitsigns` (extended keymaps)
- **`lua/custom/plugins/`**: User's own plugins (currently empty `init.lua`). Enabled by uncommenting `{ import = 'custom.plugins' }` in `init.lua`

### Key Plugin Stack

- **LSP**: nvim-lspconfig + Mason (mason-org/mason.nvim) for auto-installing servers. Servers configured in `init.lua` `servers` table (~line 673). Currently: `ts_ls`, `lua_ls`
- **Completion**: blink.cmp (not nvim-cmp) with LuaSnip for snippets
- **Formatting**: conform.nvim — format-on-save enabled. Lua uses `stylua`, JS/TS/JSON/CSS/HTML use `prettierd`/`prettier`
- **Linting**: nvim-lint — JS/TS uses `eslint_d`
- **Fuzzy finder**: Telescope with fzf-native
- **Treesitter**: main branch, auto-install enabled
- **UI**: tokyonight-night colorscheme, which-key, mini.nvim (statusline, surround, ai textobjects), todo-comments, fidget.nvim (LSP progress)

### Mason-Installed Tools

Configured in `init.lua` via `mason-tool-installer`: `stylua`, `prettierd`, `eslint_d`, plus any servers in the `servers` table.

### Health Check

`lua/kickstart/health.lua` — run `:checkhealth kickstart` to verify Neovim version and external dependencies (git, make, unzip, rg).
