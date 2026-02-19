# AGENTS.md
Guidance for coding agents working in this dotfiles repository.

## Repository Scope
- This repo tracks a small set of personal config entrypoints.
- Main tracked files:
  - `.zshrc`
  - `.config/tmux/tmux.conf`
  - `.config/aerospace/aerospace.toml`
  - `.gitmodules`
  - `.config/nvim` (git submodule pointer)
- Most files under `.config/tmux/plugins/*` are plugin checkouts and are ignored.
- `.config/nvim` is a separate git repo; treat it as a project boundary.

## Local Agent Rules Discovery
- `.cursor/rules/`: not found
- `.cursorrules`: not found
- `.github/copilot-instructions.md`: not found
- If these files are added later, follow them as highest-priority repo rules.

## Environment and Tooling
- Host environment is macOS-focused (Homebrew paths in `.zshrc`).
- Typical tools used for validation:
  - `zsh`
  - `tmux`
  - `aerospace`
  - `nvim`
  - `stylua` (for Neovim submodule Lua)
- There is no root `package.json`, `pyproject.toml`, `Makefile`, or `justfile`.
- Do not assume a single monorepo build command exists.

## Build / Lint / Test Commands
There is no unified root test suite. Validate by config type.

### Quick Sanity Sweep (repo root)
```bash
zsh -n .zshrc
tmux -f .config/tmux/tmux.conf -L dotfiles-check start-server
tmux -L dotfiles-check kill-server
```
- `zsh -n` is syntax-only.
- tmux commands parse config using an isolated socket name.

### Zsh Commands
- Lint/syntax check:
```bash
zsh -n .zshrc
```
- Single-test equivalent (one file):
```bash
zsh -n .zshrc
```
- Optional runtime smoke check:
```bash
zsh -ic 'source .zshrc; alias vim; echo OK'
```

### Tmux Commands
- Parse tmux config:
```bash
tmux -f .config/tmux/tmux.conf -L dotfiles-check start-server
tmux -L dotfiles-check kill-server
```
- Single-test equivalent:
```bash
tmux -f .config/tmux/tmux.conf -L dotfiles-check start-server
```
- Parse failures return non-zero and print line-level errors.

### AeroSpace Commands
- AeroSpace uses runtime reload validation (no dedicated local test suite).
- Reload after edits:
```bash
aerospace reload-config
```
- Single-test equivalent: change one binding/section and run reload.
- Confirm no error notification appears.

### Neovim Submodule Commands (`.config/nvim`)
- Run these from `.config/nvim`, not repo root.
- Format all Lua:
```bash
stylua .
```
- Lint/format-check all Lua:
```bash
stylua --check .
```
- Single-file check (single-test equivalent):
```bash
stylua --check lua/custom/plugins/copilot.lua
```
- Optional headless startup smoke test:
```bash
nvim --headless '+qa'
```

## Style Guidelines

## General Editing Principles
- Keep edits minimal and scoped to the target file.
- Preserve current structure and comment density.
- Avoid reformatting unrelated blocks.
- Prefer clarity over compact but cryptic one-liners.

## Imports / Module Loading (Lua in submodule)
- Prefer local requires near usage, e.g. `local x = require 'x'`.
- Use `pcall` for optional modules/extensions.
- Keep plugin spec structure consistent with existing files.

## Formatting Conventions
- Zsh:
  - Use 2-space indentation in block bodies.
  - Quote variable expansions unless unquoted behavior is required.
  - Guard optional `source` files with `if [ -f ... ]`.
- Tmux:
  - One directive per line.
  - Keep TPM initialization at end of file.
  - Group related keybindings together.
- AeroSpace TOML:
  - Keep table organization consistent (`[mode.main.binding]`, `[mode.service.binding]`).
  - Use existing keybinding naming style (`alt-h`, `alt-shift-h`).
- Lua (`.config/nvim/.stylua.toml`):
  - 2-space indentation
  - line width 160
  - Unix line endings
  - auto-prefer single quotes

## Types and Contracts
- Shell, tmux, and TOML are untyped; rely on defensive checks.
- Keep Lua annotations only where already used/valuable.
- Do not introduce new type tooling unless repo adopts it first.

## Naming Conventions
- Environment vars: uppercase (`EDITOR`, `PATH`, `GCLOUD_SDK_DIR`).
- Shell aliases/functions: lowercase and command-oriented.
- Tmux plugin options: `@plugin` / `@option` style.
- AeroSpace bindings: modifier-first (`alt-*`, `alt-shift-*`).
- Lua plugin files: lowercase descriptive names under `lua/custom/plugins/`.

## Error Handling and Reliability
- Shell:
  - Guard optional commands/files before use.
  - Prefer `command -v tool >/dev/null 2>&1` for optional binaries.
  - Avoid hard failures for optional integrations.
- Tmux:
  - Validate config through isolated socket parse before assuming success.
  - Be careful with escaping in `run-shell` strings.
- AeroSpace:
  - Apply small incremental edits, then reload immediately.
- Lua:
  - Fail soft for optional plugin integrations.

## Git and Submodule Boundaries
- Do not commit plugin vendor contents under `.config/tmux/plugins/*`.
- `.config/nvim` is a submodule:
  - Root repo tracks submodule pointer only.
  - Real content changes belong in the submodule repo commits.
- Keep root config edits and submodule pointer bumps logically separate.

## PR / Review Expectations for Agents
- Report exact validation commands that were run.
- If no automated tests exist, explicitly say manual validation was done.
- For config behavior changes, include concise before/after notes.
