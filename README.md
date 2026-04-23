<div align="center">

# ghostty · starship · yazi · lazygit

### an all-in-one macOS terminal toolchain for vibe coding with Claude Code

[![macOS](https://img.shields.io/badge/macOS-000000?style=for-the-badge&logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![shell zsh](https://img.shields.io/badge/shell-zsh-1a1a1a?style=for-the-badge&logo=gnu-bash&logoColor=white)](https://www.zsh.org/)
[![installer brew](https://img.shields.io/badge/installer-brew-fbb040?style=for-the-badge&logo=homebrew&logoColor=white)](https://brew.sh)
[![theme Catppuccin](https://img.shields.io/badge/theme-Catppuccin_Mocha-f5c2e7?style=for-the-badge)](https://catppuccin.com)
[![font Maple Mono](https://img.shields.io/badge/font-Maple_Mono_NF_CN-9cdcfe?style=for-the-badge)](https://github.com/subframe7536/maple-font)
[![agent executable](https://img.shields.io/badge/runbook-agent--executable-cba6f7?style=for-the-badge)](./INSTALL.md)

</div>

---

## why this combo

Claude Code runs in your terminal. If the terminal is sluggish, the prompt is noisy, and switching between edits, diffs, and files requires three apps, your flow state dies before the first tool use.

This repo is one runbook ([`INSTALL.md`](./INSTALL.md)) that installs a cohesive, opinionated stack:

- **[Ghostty](https://ghostty.org)** — GPU-accelerated terminal. Smooth while Claude streams tokens. Quake-style dropdown on `ctrl+backtick`, splits with `cmd+d`, transparent blur titlebar.
- **[Starship](https://starship.rs)** — prompt showing git state, language, and context at a glance — so you always know what Claude is about to operate on.
- **[Yazi](https://yazi-rs.github.io)** — async file manager with image and PDF previews. The bundled `y` shell function `cd`s your parent shell to wherever you exited.
- **[Lazygit](https://github.com/jesseduffield/lazygit)** — review every diff Claude makes before it hits `main`. Stage hunks, amend, rebase — all keyboard.

Plus the modern CLI bedrock — `fd`, `ripgrep`, `fzf`, `zoxide`, `jq`, `ffmpeg`, `poppler`, `imagemagick`, `sevenzip`, `resvg` — for search, preview, and transformation.

All wired to **Catppuccin Mocha** and **Maple Mono NF CN** (ligatures, Nerd Font icons, CJK support).

---

## what it looks like

```
╭─ ghostty ── ~/code/myapp ─────────────────────────────────────╮
│                                                                │
│   ~/code/myapp   main [!]   via  v20.11.0                 │
│  ❯ claude                                                      │
│                                                                │
│  ● Welcome to Claude Code!                                     │
│    /help for help · /status for your setup                     │
│                                                                │
│  > refactor the auth middleware to use jose                    │
│                                                                │
│  ⏺ Read(src/middleware/auth.ts)                                │
│  ⏺ Edit(src/middleware/auth.ts)                                │
│     └─  +34 / -18                                              │
│  ⏺ Bash(pnpm typecheck)                                        │
│     └─ ✓ no errors                                             │
│                                                                │
╰────────────────────────────────────────────────────────────────╯
```

> Real screenshots render far better than ASCII — drop your own here once the font and blur kick in.

---

## install

Hand the following instruction to your AI coding agent:

```
Fetch and follow instructions from https://raw.githubusercontent.com/ziyi-zhang-1130/ghostty-starship-yazi-lazygit/refs/heads/main/INSTALL.md, halt if any verification fails.
```

The runbook is idempotent, so manual execution is also supported: open [`INSTALL.md`](./INSTALL.md) and run each shell block top to bottom.

### prerequisites

- macOS
- [Homebrew](https://brew.sh)
- zsh (default on modern macOS)

### end-to-end smoke test

After install, this must exit 0 (the canonical version lives at the bottom of `INSTALL.md`):

```sh
set -e
test -d /Applications/Ghostty.app
command -v starship >/dev/null
command -v yazi >/dev/null
command -v lazygit >/dev/null
echo "OK"
```

---

## the stack at a glance

| tool | role | why it's in here |
|------|------|------------------|
| **Ghostty** | terminal emulator | GPU rendering, transparent titlebar, Quake-mode dropdown, native macOS |
| **Starship** | prompt | Rust-fast, context-rich, `catppuccin-powerline` preset out of the box |
| **Yazi** | file manager | async, image/PDF previews, `y` shell integration for seamless `cd` |
| **Lazygit** | git TUI | stage hunks, amend, rebase, resolve conflicts — the interface `git` should've shipped with |

---

## why it pairs well with Claude Code

- **Fast terminal → fast feedback.** When Claude streams a long diff or runs a noisy build, Ghostty doesn't choke — you stay in the loop.
- **Ambient context.** Starship's git/language segments mean you always know which branch Claude is editing and whether there are uncommitted changes before you say "ship it".
- **Agentic review surface.** Claude writes code; `lazygit` is where you read it. Hunk-level staging catches the occasional overreach before it commits.
- **File flow.** `yazi` + `zoxide` + `fzf` make jumping between the files Claude is touching near-zero-cost.
- **Long-session ergonomics.** Catppuccin Mocha + Maple Mono is a dark theme and font pairing you can stare at for hours without flinching.
- **Self-hosting install.** The install guide is itself agent-executable — your coding agent sets up its own environment. Meta, but it actually works.

---

## customize

The configs in `INSTALL.md` are opinionated but editable. Re-running the runbook won't overwrite files it already wrote — `CREATE` files (Ghostty, Starship) are skipped when present, and `APPEND-IF-MISSING` blocks (the `starship init` hook, `y` helper, and `lg` alias in `~/.zshrc`) are skipped when their marker is already in place. Edit freely; to let the runbook reinstall a file, delete it or remove the marker block.

See [`CLAUDE.md`](./CLAUDE.md) for the runbook's edit invariants and the three file-op primitives (CREATE, OVERWRITE-IF-MANAGED, APPEND-IF-MISSING).

---

<div align="center">

built for long sessions with Claude Code

</div>
