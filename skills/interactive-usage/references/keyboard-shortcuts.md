# Complete Keyboard Shortcuts Reference

## macOS Option Key Setup

For shortcuts using Option/Alt, configure your terminal:

**iTerm2**: Settings → Profiles → Keys → Set Left/Right Option key to "Esc+"
**Terminal.app**: Settings → Profiles → Keyboard → Check "Use Option as Meta Key"
**VS Code**: Settings → Profiles → Keys → Set Left/Right Option key to "Esc+"

## General Controls

| Shortcut | Description | Platform |
|----------|-------------|----------|
| `Ctrl+C` | Cancel input/generation | All |
| `Ctrl+D` | Exit session | All |
| `Ctrl+G` | Open in text editor | All |
| `Ctrl+L` | Clear terminal screen | All |
| `Ctrl+O` | Toggle verbose output | All |
| `Ctrl+R` | Reverse search history | All |
| `Ctrl+V` | Paste image | Mac/Linux |
| `Cmd+V` | Paste image | iTerm2 |
| `Alt+V` | Paste image | Windows |
| `Ctrl+B` | Background running task | All (press twice in tmux) |
| `Ctrl+T` | Toggle syntax highlighting | In `/theme` menu only |

## Navigation

| Shortcut | Description |
|----------|-------------|
| `Left/Right` | Cycle through dialog tabs |
| `Up/Down` | Navigate command history |
| `Esc + Esc` | Open rewind menu |

## Mode Switching

| Shortcut | Description |
|----------|-------------|
| `Shift+Tab` | Cycle permission modes |
| `Alt+M` | Toggle permission modes (some terminals) |
| `Alt+P` (Option+P) | Switch model |
| `Alt+T` (Option+T) | Toggle extended thinking |

## Text Editing

| Shortcut | Description |
|----------|-------------|
| `Ctrl+K` | Delete to end of line |
| `Ctrl+U` | Delete entire line |
| `Ctrl+Y` | Paste deleted text |
| `Alt+Y` | Cycle paste history |
| `Alt+B` | Move cursor back one word |
| `Alt+F` | Move cursor forward one word |

## Multiline Input Methods

| Method | Shortcut | Works In |
|--------|----------|----------|
| Backslash escape | `\` + `Enter` | All terminals |
| Option+Enter | `Option+Enter` | macOS default |
| Shift+Enter | `Shift+Enter` | iTerm2, WezTerm, Ghostty, Kitty |
| Control+J | `Ctrl+J` | All terminals |
| Paste mode | Paste directly | All terminals |

Run `/terminal-setup` to enable Shift+Enter in VS Code, Alacritty, Zed, Warp.

## Reverse Search (Ctrl+R)

| Action | How |
|--------|-----|
| Start search | `Ctrl+R` |
| Cycle older matches | `Ctrl+R` again |
| Accept and edit | `Tab` or `Esc` |
| Accept and execute | `Enter` |
| Cancel | `Ctrl+C` or `Backspace` on empty |

## Dialog Navigation

| Shortcut | Description |
|----------|-------------|
| `Left/Right` | Navigate tabs in permission dialogs |
| `Enter` | Confirm/select |
| `Esc` | Cancel/dismiss |

## Session Picker (/resume)

| Shortcut | Description |
|----------|-------------|
| `↑`/`↓` | Navigate between sessions |
| `→`/`←` | Expand/collapse grouped sessions |
| `Enter` | Select and resume |
| `P` | Preview session content |
| `R` | Rename session |
| `/` | Enter search mode |
| `A` | Toggle all projects view |
| `B` | Filter to current git branch |
| `Esc` | Exit picker or search mode |

## Quick Prefixes

| Prefix | Description |
|--------|-------------|
| `/` | Command or skill autocomplete |
| `!` | Bash mode (direct execution) |
| `@` | File path mention |
