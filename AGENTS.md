# AGENTS.md

Instructions for AI coding agents working in this Neovim configuration repository.

## Project Overview

A personal Neovim configuration based on [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim).
Single-file-first config (`init.lua`) with optional modular plugin files in `lua/kickstart/plugins/`
and `lua/custom/plugins/`. This is NOT a distribution -- it is one user's config.

## Build / Lint / Test Commands

There is no build step or test suite. This is a Neovim config, not a library.

| Task | Command |
|------|---------|
| **Format check** | `stylua --check .` |
| **Format fix** | `stylua .` |
| **Health check** | `:checkhealth kickstart` (inside Neovim) |
| **Plugin install/update** | `:Lazy` (inside Neovim) |
| **Verify config loads** | `nvim --headless "+q"` (exits 0 if no startup errors) |

CI runs `stylua --check .` on pull requests. All Lua files must pass before merging.

### External Tool Dependencies

These must be installed (Mason auto-installs them): `stylua`, `prettierd`, `eslint_d`.
System dependencies: `git`, `make`, `unzip`, `rg` (ripgrep).

## File Structure

```
init.lua                          # Main config (leader, options, keymaps, autocmds, plugins)
lua/kickstart/health.lua          # :checkhealth kickstart
lua/kickstart/plugins/            # Optional kickstart plugins (enable by uncommenting in init.lua)
  lint.lua, neo-tree.lua          # ENABLED
  autopairs, debug, gitsigns,     # disabled
  indent_line
lua/custom/plugins/init.lua       # User extension point (currently empty)
.stylua.toml                      # StyLua formatter config
lazy-lock.json                    # Plugin lock file (do not edit manually)
```

## Formatting (StyLua)

Configuration in `.stylua.toml`:
- **Indent:** 2 spaces (no tabs)
- **Column width:** 160
- **Quotes:** Single quotes (`'string'`)
- **Call parentheses:** None for single-string-argument calls
- **Line endings:** Unix (LF)

Always run `stylua .` after making changes. Files end with modeline: `-- vim: ts=2 sts=2 sw=2 et`

## Code Style Guidelines

### Require / Import Style

Omit parentheses for standalone single-string requires (StyLua enforces this):
```lua
local builtin = require 'telescope.builtin'
```
Use parentheses when chaining or indexing the result:
```lua
require('lazy').setup { ... }
require('lspconfig')[server_name].setup(server)
```

### Naming Conventions

- `snake_case` for all user-defined variables, functions, parameters, and table keys
- `kebab-case` for augroup names: `'kickstart-lsp-attach'`, `'kickstart-highlight-yank'`
- Short contextual names in callbacks: `client`, `builtin`, `map`, `dap`
- No camelCase in user code (only appears in external API references)

### Variables and Vim Options

- **Always use `local`** -- zero global assignments (only `vim.g.*` for Neovim globals)
- Assign `require` results to locals when used more than once
- `vim.o.*` for scalar options, `vim.opt.*` for table-like options, `vim.g.*` for global variables

### Keymap Definitions

Global keymaps use `vim.keymap.set` directly:
```lua
vim.keymap.set('n', '<Esc>', '<cmd>nohlsearch<CR>')
vim.keymap.set('n', '<leader>q', vim.diagnostic.setloclist, { desc = 'Open diagnostic [Q]uickfix list' })
```

Buffer-local keymaps (e.g., LSP) use a local `map` helper:
```lua
local map = function(keys, func, desc, mode)
  mode = mode or 'n'
  vim.keymap.set(mode, keys, func, { buffer = event.buf, desc = 'LSP: ' .. desc })
end
map('grn', vim.lsp.buf.rename, '[R]e[n]ame')
```

Description convention: `[X]` brackets denote mnemonics (e.g., `'[S]earch [H]elp'`).
LSP keymaps are prefixed with `'LSP: '`.

### Autocommands

Always use a named augroup with `{ clear = true }`:
```lua
vim.api.nvim_create_autocmd('TextYankPost', {
  desc = 'Highlight when yanking text',
  group = vim.api.nvim_create_augroup('kickstart-highlight-yank', { clear = true }),
  callback = function()
    vim.hl.on_yank()
  end,
})
```

### Plugin Specs (lazy.nvim)

Prefer `opts = {}` over `config = function() require(...).setup {} end` for simple configuration.
Use `config = function() ... end` only when imperative logic is needed (keymaps, extensions, etc.).
Plugin files in `lua/kickstart/plugins/` return either a single spec table or `{ { ... } }`.
Plugin files in `lua/custom/plugins/` are auto-imported when enabled in init.lua.

### LSP Server Configuration

Servers are declared in a `servers` table, processed by mason-lspconfig handlers.
Capabilities merged with: `vim.tbl_deep_extend('force', {}, capabilities, server.capabilities or {})`

### Error Handling

- `pcall` for non-critical operations (extension loading, optional requires)
- `error()` for fatal failures (bootstrap)
- Guard clauses with nil checks (`if client and client_supports_method(...)`)
- Neovim version compatibility: `(vim.uv or vim.loop)`, `vim.fn.has 'nvim-0.11' == 1`

### Common Lua Idioms

```lua
mode = mode or 'n'                              -- default values
vim.g.have_nerd_font and { ... } or {}           -- conditional tables
local ok, mod = pcall(require, 'module')          -- safe require
vim.tbl_deep_extend('force', {}, base, override)  -- config merging
vim.fn.executable 'make' == 1                     -- vimscript returns 0/1, not booleans
```

### Tables, Functions, Comments, and Strings

- Trailing commas on all multi-line table entries
- Single-line for short tables: `{ text = '+' }`, `{ clear = true }`
- Table arguments without outer parens (StyLua enforced): `setup { ... }`

### Function Definitions

- `local function name()` for named helpers inside callbacks
- `local name = function()` for module-level utilities
- Anonymous `function()` for inline callbacks

### Comments

- Section headers: `-- [[ Section Name ]]`
- Attention markers: `-- NOTE:`, `-- TIP:`, `-- WARN:`
- Plugin annotations inside opening brace: `{ -- Description of plugin`
- Help references: `-- See :help topic`
- Type annotations: `---@param name Type`, `---@return Type`
- Diagnostics: `---@diagnostic disable-next-line: rule-name`

### Strings

- Single quotes everywhere (StyLua enforced)
- `..` operator for simple concatenation
- `string.format` for template strings with `%s` placeholders
