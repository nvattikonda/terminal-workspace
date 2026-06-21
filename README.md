# Kitty + Ghostty + tmux Developer Environment

## Overview

Cross-platform terminal workspace for macOS and Linux based on

* Kitty or Ghostty
* tmux
* Oh My tmux
* Bash

Goals

* Cross-platform consistency
* Session persistence
* Beautiful fonts and themes
* Excellent clipboard integration
* SSH-friendly
* Minimal complexity
* Incremental extensibility

---

# Architecture

Primary workflow

```text
Terminal OS Window
    ↓
tmux session (main)
    ↓
tmux windows
    ↓
tmux panes
    ↓
bash + tools
```

Supported terminal options

```text
Kitty
  or
Ghostty
    ↓
tmux
    ↓
bash + tools
```

Responsibilities

| Layer         | Responsibility                           |
|---------------|------------------------------------------|
| Kitty/Ghostty | Rendering, fonts, clipboard, terminal UI |
| tmux          | Sessions and workspace management        |
| Bash          | Command execution                        |
| Tools         | Editor, git, docker, kubectl, etc.       |

---

# Multiple Environment Example

```text
Terminal OS Window
├── Terminal Split / Window
│      ↓
│    tmux session (dev)
│      ↓
│    tmux windows
│      ↓
│    tmux panes
│
└── Terminal Split / Window
       ↓
     tmux session (prod)
       ↓
     tmux windows
       ↓
     tmux panes
```

Terminal-native windows and splits should be used sparingly. Most workspace organization should remain inside tmux.

---

# Installation

## macOS

```bash
brew install kitty ghostty tmux bat lazygit
brew install --cask font-jetbrains-mono-nerd-font
```

Install Oh My tmux

```bash
git clone https://github.com/gpakosz/.tmux.git
ln -s -f ~/.tmux/.tmux.conf ~/.tmux.conf
cp ~/.tmux/.tmux.conf.local ~/
```

---

## Linux

```bash
sudo apt install kitty tmux bat unzip
```

Ghostty installation depends on distribution/package availability. Use the official Ghostty installation instructions
for your OS.

---

## Install JetBrains Mono Nerd Font on Linux

Download:

```bash
curl -LO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.zip
```

Create font directory:

```bash
mkdir -p ~/.local/share/fonts
```

Extract

```bash
unzip JetBrainsMono.zip -d ~/.local/share/fonts/JetBrainsMono
```

Refresh font cache

```bash
fc-cache -fv
```

Verify

```bash
fc-list | grep JetBrainsMono
```

---

# Kitty Configuration

Location

```text
~/.config/kitty/kitty.conf
```

```conf
#
# Font configuration
# JetBrains Mono Nerd Font provides excellent readability and icon support.
# Ensure the font is installed on the system.
#
font_family JetBrainsMono Nerd Font

#
# Font size in points.
#
font_size 15

#
# Slight transparency for a modern appearance.
# Set to 1.0 for a fully opaque background.
#
background_opacity 0.95

#
# Use a thin beam cursor similar to modern editors.
#
cursor_shape beam

#
# Disable terminal bell sounds.
#
enable_audio_bell no

#
# Do not prompt when closing the Kitty window.
# Set to 1 if you prefer confirmation dialogs.
#
confirm_os_window_close 0

#
# Display tabs at the top.
#
tab_bar_edge top

#
# Enable shell integration for improved command tracking and terminal features.
#
shell_integration enabled

#
# Automatically copy mouse selections to the system clipboard.
#
copy_on_select yes

#
# Copy selected text to the system clipboard.
#
map ctrl+shift+c copy_to_clipboard

#
# Paste from the system clipboard.
#
map ctrl+shift+v paste_from_clipboard

#
# Automatically attach to tmux session "main".
# If the session doesn't exist, create it.
# This allows Kitty to own tmux while leaving bash and other terminals unaffected.
#
shell tmux new-session -A -s main

#
# Navigate between neighboring Kitty windows (splits).
# Primarily useful when occasionally using Kitty splits
# (for example dev/prod or local/remote environments).
#
map ctrl+shift+left neighboring_window left
map ctrl+shift+right neighboring_window right
map ctrl+shift+up neighboring_window up
map ctrl+shift+down neighboring_window down

#
# Create a vertical split within the current Kitty tab.
#
map ctrl+shift+enter launch --location=vsplit

#
# Create a horizontal split within the current Kitty tab.
#
map ctrl+shift+- launch --location=hsplit

#
# Reload kitty.conf without restarting Kitty.
# Useful after making configuration changes.
#
map ctrl+shift+f5 load_config_file
```

---

# Ghostty Configuration

Location:

```text
~/.config/ghostty/config
```

```conf
# --- Appearance & Fonts ---
theme = "TokyoNight"
font-family = "JetBrainsMono Nerd Font"
font-size = 15

# --- Window Layout Tweaks ---
window-padding-x = 8
window-padding-y = 8
window-padding-balance = true

# --- Custom Keybindings ---
# Easy copy and paste shortcuts.
keybind = ctrl+shift+c=copy_to_clipboard
keybind = ctrl+shift+v=paste_from_clipboard

# --- tmux Integration ---
# Automatically create or reconnect to tmux session "main" on startup.
# Disable this if you prefer to start tmux manually.
command = tmux new-session -A -s main
```

Validate Ghostty config

```bash
ghostty +show-config
```

List Ghostty fonts

```bash
ghostty +list-fonts
```

List Ghostty keybindings

```bash
ghostty +list-keybinds
```

---

## tmux Keybinding Customizations

Oh My tmux ships with default pane split bindings

```text
prefix -  → horizontal split (top/bottom)
prefix _  → vertical split (left/right)
```

Since `_` requires using the Shift key, a more ergonomic configuration is to use:

- `prefix v` → vertical split (left/right panes)
- `prefix s` → horizontal split (top/bottom panes)

### Update `~/.tmux.conf.local`

Add the following to

```text
~/.tmux.conf.local
```

```tmux
#
# Pane splits
# Override Oh My tmux defaults.
# Use easy-to-type keys without requiring Shift.
#

# Remove existing bindings
unbind -
unbind _

# Vertical split (left/right panes)
bind v split-window -h

# Horizontal split (top/bottom panes)
bind s split-window -v
```

### Result

| Key      | Action                              |
|----------|-------------------------------------|
| prefix v | Vertical split (left/right panes)   |
| prefix s | Horizontal split (top/bottom panes) |

Visual representation

Vertical split (`prefix v`)

```text
+-----+-----+
|     |     |
|     |     |
+-----+-----+
```

Horizontal split (`prefix s`)

```text
+-----------+
|           |
+-----------+
|           |
+-----------+
```

### Reload tmux Configuration

From a shell

```bash
tmux source-file ~/.tmux.conf
```

Or inside tmux, use the Oh My tmux reload shortcut

```text
prefix r
```

These customizations apply to all tmux sessions regardless of whether you use Kitty, Ghostty, or a remote SSH terminal.

# Auto tmux Startup vs Manual tmux Startup

Both Kitty and Ghostty can automatically start or reconnect to a tmux session.

This is convenient when you want every terminal window to enter the same persistent workspace automatically.

## Auto-start tmux in Kitty

Enabled by this line in `kitty.conf`

```conf
shell tmux new-session -A -s main
```

Disable auto-start by commenting it out

```conf
# shell tmux new-session -A -s main
```

## Auto-start tmux in Ghostty

Enabled by this line in Ghostty config

```conf
command = tmux new-session -A -s main
```

Disable auto-start by commenting it out

```conf
# command = tmux new-session -A -s main
```

## Manual tmux Workflow

Create or attach to main session:

```bash
tmux new-session -A -s main
```

Create a named work session:

```bash
tmux new -s work
```

List sessions:

```bash
tmux ls
```

Attach to a session

```bash
tmux attach -t work
```

Detach from tmux

```text
prefix d
```

Recommended approach

* Use auto-start for daily development machines.
* Use manual tmux startup for servers, shared machines, production shells, or when you want normal shell startup
  behavior.

---

# Useful Kitty Commands

* Ctrl+Shift+C → Copy
* Ctrl+Shift+V → Paste
* Ctrl+Shift+T → New tab
* Ctrl+Shift+N → New OS window
* Ctrl+Shift+Enter → Vertical split
* Ctrl+Shift+- → Horizontal split
* Ctrl+Shift+Arrow → Navigate windows
* Ctrl+Shift+F5 → Reload config

---

# Useful Ghostty Commands

* Ctrl+Shift+C → Copy
* Ctrl+Shift+V → Paste
* `ghostty +show-config` → Show effective config
* `ghostty +list-fonts` → List available fonts
* `ghostty +list-keybinds` → List keybindings

---

# tmux Commands

Sessions:

```bash
tmux new -s work
tmux ls
tmux attach -t work
```

Detach:

```text
prefix d
```

Windows

```text
prefix c
prefix ,
prefix n
prefix p
```

Panes

```text
prefix v
prefix h
prefix Arrow
prefix z
```

---

# Clipboard

Terminal owns clipboard integration.

Vim

```vim
"+yy
"+y
"+p
```

bat on macOS

```bash
bat README.md | pbcopy
```

bat on Linux

```bash
bat README.md | xsel --clipboard
```

---

# Future Enhancements

* fd
* ripgrep
* fzf
* zoxide
* yazi
* eza
* btop
* lazydocker
* k9s

---

# References

Kitty

```text
https://sw.kovidgoyal.net/kitty/
```

Ghostty

```text
https://ghostty.org/
https://ghostty.org/docs/config
https://ghostty.org/docs/config/reference
```

tmux

```text
https://github.com/tmux/tmux/wiki
```

Oh My tmux

```text
https://github.com/gpakosz/.tmux
```

Nerd Fonts

```text
https://www.nerdfonts.com/
https://github.com/ryanoasis/nerd-fonts
```