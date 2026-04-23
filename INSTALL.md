# Tool Setup (macOS)

> **Audience:** AI agents executing this autonomously.
> Every step ships with a verification command. If any verification fails,
> halt and surface the error — do not proceed to the next step.

## 0. Preconditions

Run these checks before anything else. If any fails, stop.

```sh
# macOS
test "$(sw_vers -productName)" = "macOS" || { echo "not macOS"; exit 1; }

# Homebrew
command -v brew >/dev/null || { echo "Homebrew not installed — install from https://brew.sh first"; exit 1; }

# zsh (expected login shell)
echo "$SHELL" | grep -q zsh || echo "warning: login shell is not zsh; zsh-specific config below may not apply"

# ~/.zshrc exists (create empty if not)
test -f ~/.zshrc || touch ~/.zshrc

# ~/.config exists
mkdir -p ~/.config
```

## Conventions

Every tool section has five parts:

- **Purpose** — one line of context.
- **Install** — idempotent; `brew install` / `brew install --cask` is a no-op when a formula is already present at a satisfactory version.
- **Verify install** — must exit 0. Non-zero means the install step failed.
- **Configure** — explicit file operation, one of:
  - **CREATE** — write file only if it does not exist.
  - **OVERWRITE-IF-MANAGED** — write file if absent, or if its first line matches the managed marker. Otherwise halt and report "existing file differs from managed version".
  - **APPEND-IF-MISSING** — append a block wrapped in `# >>> <name> >>>` / `# <<< <name> <<<` markers; skip if the opening marker is already present.
- **Verify config** — must exit 0.

---

## 1. Ghostty

### Purpose
Terminal emulator.

### Install

```sh
brew install --cask ghostty
brew install --cask font-maple-mono-nf-cn
```

### Verify install

```sh
test -d "/Applications/Ghostty.app" || { echo "ghostty not installed"; exit 1; }
```

### Configure

- **File:** `~/.config/ghostty/config`
- **Operation:** CREATE (skip if already present)
- **Upstream source (for humans):** https://gist.github.com/zhangchitc/7dead7c1b517390e061e07759ed80277

Apply with:

```sh
mkdir -p ~/.config/ghostty
CONFIG=~/.config/ghostty/config
if [ -f "$CONFIG" ]; then
  echo "ghostty config already exists at $CONFIG — skipping"
else
cat > "$CONFIG" <<'GHOSTTY_CONFIG_EOF'
# managed-by: ~/install.md
# ============================================
# Ghostty Terminal - Complete Configuration
# ============================================
# File: ~/.config/ghostty/config
# Reload: Cmd+Shift+, (macOS)
# View options: ghostty +show-config --default --docs

# --- Typography ---
font-family = "Maple Mono NF CN"
font-size = 14
font-thicken = true
adjust-cell-height = 2

# --- Theme and Colors ---
# Catppuccin with automatic light/dark switching
theme = Catppuccin Mocha

# --- Window and Appearance ---
background-opacity = 0.85
background-blur-radius = 30
macos-titlebar-style = transparent
window-padding-x = 10
window-padding-y = 8
window-save-state = always
window-theme = auto

# --- Cursor ---
cursor-style = bar
cursor-style-blink = true
cursor-opacity = 0.8

# --- Mouse ---
mouse-hide-while-typing = true
copy-on-select = clipboard

# --- Quick Terminal (Quake-style dropdown) ---
quick-terminal-position = top
quick-terminal-screen = mouse
quick-terminal-autohide = true
quick-terminal-animation-duration = 0.15

# --- Security ---
clipboard-paste-protection = true
clipboard-paste-bracketed-safe = true

# --- Shell Integration ---
shell-integration = detect

# --- Keybindings ---
# Tabs
keybind = cmd+t=new_tab
keybind = cmd+shift+left=previous_tab
keybind = cmd+shift+right=next_tab
keybind = cmd+w=close_surface

# Splits
keybind = cmd+d=new_split:right
keybind = cmd+shift+d=new_split:down
keybind = cmd+alt+left=goto_split:left
keybind = cmd+alt+right=goto_split:right
keybind = cmd+alt+up=goto_split:top
keybind = cmd+alt+down=goto_split:bottom

# Font size
keybind = cmd+plus=increase_font_size:1
keybind = cmd+minus=decrease_font_size:1
keybind = cmd+zero=reset_font_size

# Quick terminal global hotkey
keybind = global:ctrl+grave_accent=toggle_quick_terminal

# Splits management
keybind = cmd+shift+e=equalize_splits
keybind = cmd+shift+f=toggle_split_zoom

# Reload config
keybind = cmd+shift+comma=reload_config

# --- Performance ---
# Generous scrollback (25MB)
scrollback-limit = 25000000
GHOSTTY_CONFIG_EOF
fi
```

### Verify config

```sh
test -f ~/.config/ghostty/config
```

---

## 2. Starship

### Purpose
Cross-shell prompt.

### Install

```sh
brew install starship
```

### Verify install

```sh
command -v starship >/dev/null
```

### Configure

Two files.

**2a. Preset** — `~/.config/starship.toml`

- **Operation:** CREATE (skip if already present)

```sh
test -f ~/.config/starship.toml || starship preset catppuccin-powerline -o ~/.config/starship.toml
```

**2b. Shell hook** — `~/.zshrc`

- **Operation:** APPEND-IF-MISSING
- **Marker:** `# >>> starship init >>>`
- Initializes starship only under Ghostty (which ships the required Nerd Font for the Catppuccin Powerline preset); other terminals fall back to the plain zsh prompt so missing glyphs don't render as tofu.

```sh
if ! grep -qF '# >>> starship init >>>' ~/.zshrc; then
  cat >> ~/.zshrc <<'STARSHIP_EOF'

# >>> starship init >>>
if [[ "$TERM_PROGRAM" == "ghostty" ]]; then
	eval "$(starship init zsh)"
fi
# <<< starship init <<<
STARSHIP_EOF
fi
```

### Verify config

```sh
test -f ~/.config/starship.toml && grep -qF '# >>> starship init >>>' ~/.zshrc
```

---

## 3. Yazi

### Purpose
Terminal file manager (plus companion CLIs used by its previewers and plugin system).

### Install

```sh
brew update
brew install yazi ffmpeg-full sevenzip jq poppler fd ripgrep fzf zoxide resvg imagemagick-full font-symbols-only-nerd-font
brew link ffmpeg-full imagemagick-full -f --overwrite
```

### Verify install

```sh
command -v yazi >/dev/null && command -v ya >/dev/null
```

### Configure

**Shell helper** — `~/.zshrc`

- **Operation:** APPEND-IF-MISSING
- **Marker:** `# >>> yazi cwd helper >>>`
- Adds a `y` function that launches yazi and, on exit, `cd`s the parent shell into yazi's last working directory.

```sh
if ! grep -qF '# >>> yazi cwd helper >>>' ~/.zshrc; then
  cat >> ~/.zshrc <<'YAZI_EOF'

# >>> yazi cwd helper >>>
function y() {
	local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
	command yazi "$@" --cwd-file="$tmp"
	IFS= read -r -d '' cwd < "$tmp"
	[ "$cwd" != "$PWD" ] && [ -d "$cwd" ] && builtin cd -- "$cwd"
	rm -f -- "$tmp"
}
# <<< yazi cwd helper <<<
YAZI_EOF
fi
```

### Verify config

```sh
grep -qF '# >>> yazi cwd helper >>>' ~/.zshrc
```

---

## 4. Lazygit

### Purpose
Terminal UI for git.

### Install

```sh
brew install lazygit
```

### Verify install

```sh
command -v lazygit >/dev/null
```

### Configure

**Shell alias** — `~/.zshrc`

- **Operation:** APPEND-IF-MISSING
- **Marker:** `# >>> lazygit alias >>>`
- Adds an `lg` alias for `lazygit`.

```sh
if ! grep -qF '# >>> lazygit alias >>>' ~/.zshrc; then
  cat >> ~/.zshrc <<'LAZYGIT_EOF'

# >>> lazygit alias >>>
alias lg='lazygit'
# <<< lazygit alias <<<
LAZYGIT_EOF
fi
```

### Verify config

```sh
grep -qF '# >>> lazygit alias >>>' ~/.zshrc
```

---

## End-to-end verification

After completing all sections, the following block must exit 0:

```sh
set -e
test -d /Applications/Ghostty.app
test -f ~/.config/ghostty/config
command -v starship >/dev/null
test -f ~/.config/starship.toml
grep -qF '# >>> starship init >>>' ~/.zshrc
command -v yazi >/dev/null && command -v ya >/dev/null
grep -qF '# >>> yazi cwd helper >>>' ~/.zshrc
command -v lazygit >/dev/null
grep -qF '# >>> lazygit alias >>>' ~/.zshrc
echo "OK: all four tools installed and configured"
```

Post-install, the user must open a new zsh session (or `source ~/.zshrc`) for the starship prompt and `y` function to take effect, and launch Ghostty once for its config to load.
