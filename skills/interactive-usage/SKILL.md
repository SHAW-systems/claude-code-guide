---
name: interactive-usage
description: "Interactive mode features including slash commands, keyboard shortcuts, vim mode, and TUI navigation. Use when working in interactive Claude Code sessions."
---

# Interactive Usage

This skill covers Claude Code's interactive mode: slash commands, keyboard shortcuts, input modes, and navigation.

## Built-in Commands

Type `/` followed by command name. Type `/` to see all available commands.

### Session Management

| Command | Purpose |
|---------|---------|
| `/clear` | Clear conversation history |
| `/compact [instructions]` | Compact conversation with optional focus |
| `/resume [session]` | Resume conversation by ID/name |
| `/rename <name>` | Rename current session |
| `/rewind` | Rewind conversation and/or code |
| `/exit` | Exit the REPL |

### Information & Status

| Command | Purpose |
|---------|---------|
| `/help` | Get usage help |
| `/cost` | Show token usage statistics |
| `/context` | Visualize current context usage |
| `/status` | Show version, model, account info |
| `/stats` | Visualize daily usage and history |
| `/usage` | Show plan limits (subscription only) |
| `/doctor` | Check installation health |

### Configuration

| Command | Purpose |
|---------|---------|
| `/config` | Open Settings interface |
| `/permissions` | View/update permissions |
| `/model` | Select or change AI model |
| `/memory` | Edit CLAUDE.md memory files |
| `/init` | Initialize project with CLAUDE.md |
| `/theme` | Change color theme |
| `/statusline` | Configure status line UI |
| `/vim` | Enable vim editing mode |
| `/terminal-setup` | Install terminal shortcuts |

### Tools & Features

| Command | Purpose |
|---------|---------|
| `/mcp` | Manage MCP server connections |
| `/agents` | View and manage subagents |
| `/hooks` | View and manage hooks |
| `/tasks` | List background tasks |
| `/todos` | List current TODO items |
| `/copy` | Copy last response to clipboard |
| `/export [filename]` | Export conversation |
| `/teleport` | Resume remote session locally |

## Keyboard Shortcuts

> Press `?` to see available shortcuts for your environment.

### General Controls

| Shortcut | Description |
|----------|-------------|
| `Ctrl+C` | Cancel current input or generation |
| `Ctrl+D` | Exit Claude Code session |
| `Ctrl+G` | Open prompt in text editor |
| `Ctrl+L` | Clear terminal screen |
| `Ctrl+O` | Toggle verbose output |
| `Ctrl+R` | Reverse search command history |
| `Ctrl+V` | Paste image from clipboard |
| `Ctrl+B` | Background running task (press twice in tmux) |
| `Esc + Esc` | Open rewind menu |
| `Shift+Tab` | Cycle permission modes |
| `Alt+P` | Switch model |
| `Alt+T` | Toggle extended thinking |

### Text Editing

| Shortcut | Description |
|----------|-------------|
| `Ctrl+K` | Delete to end of line |
| `Ctrl+U` | Delete entire line |
| `Ctrl+Y` | Paste deleted text |
| `Alt+Y` | Cycle paste history (after Ctrl+Y) |
| `Alt+B` | Move cursor back one word |
| `Alt+F` | Move cursor forward one word |

### Multiline Input

| Method | How |
|--------|-----|
| Quick escape | `\` + `Enter` |
| macOS default | `Option+Enter` |
| Shift+Enter | Works in iTerm2, WezTerm, Ghostty, Kitty |
| Control sequence | `Ctrl+J` |
| Paste mode | Paste directly |

> Run `/terminal-setup` to enable `Shift+Enter` in other terminals.

### Quick Commands

| Shortcut | Description |
|----------|-------------|
| `/` at start | Trigger command/skill autocomplete |
| `!` at start | Run bash command directly |
| `@` | File path autocomplete |

## Quick Input Modes

### Bash Mode with `!`

Run commands directly without Claude approval:

```bash
! npm test
! git status
! ls -la
```

- Output is added to conversation context
- Supports `Ctrl+B` backgrounding
- History-based autocomplete with Tab

### File References with `@`

Reference files directly in prompts:

```
Explain the logic in @src/utils/auth.js
```

- Single file: `@src/file.js` - includes full content
- Directory: `@src/components` - shows file listing
- MCP resources: `@github:issue://123`
- Multiple files: `@file1.js and @file2.js`

## Vim Mode

Enable with `/vim` command or via `/config`.

### Mode Switching

| Command | Action | From |
|---------|--------|------|
| `Esc` | Enter NORMAL mode | INSERT |
| `i` | Insert before cursor | NORMAL |
| `I` | Insert at line start | NORMAL |
| `a` | Insert after cursor | NORMAL |
| `A` | Insert at line end | NORMAL |
| `o` | Open line below | NORMAL |
| `O` | Open line above | NORMAL |

### Navigation (NORMAL mode)

| Command | Action |
|---------|--------|
| `h`/`j`/`k`/`l` | Move left/down/up/right |
| `w` | Next word |
| `b` | Previous word |
| `0` | Beginning of line |
| `$` | End of line |
| `gg` | Beginning of input |
| `G` | End of input |

### Editing (NORMAL mode)

| Command | Action |
|---------|--------|
| `x` | Delete character |
| `dd` | Delete line |
| `D` | Delete to end of line |
| `dw`/`db` | Delete word forward/back |
| `yy` | Yank (copy) line |
| `p` | Paste after cursor |
| `P` | Paste before cursor |
| `.` | Repeat last change |

## Session Management

### Resume Conversations

```bash
# Continue most recent
claude --continue

# Resume specific session
claude --resume auth-refactor

# Open session picker
claude --resume
```

### Session Picker Shortcuts

| Shortcut | Action |
|----------|--------|
| `↑`/`↓` | Navigate sessions |
| `→`/`←` | Expand/collapse groups |
| `Enter` | Select and resume |
| `P` | Preview session |
| `R` | Rename session |
| `/` | Search/filter |
| `A` | Toggle all projects view |
| `B` | Filter to current branch |
| `Esc` | Exit picker |

### Name Sessions

```
/rename auth-refactor
```

Then resume later:
```bash
claude --resume auth-refactor
```

## Background Tasks

Claude can run commands in the background while you continue working.

### Backgrounding Commands

- Ask Claude to run something in the background
- Press `Ctrl+B` to move a running command to background
- Use `/tasks` to list background tasks

### Common Background Tasks

- Build tools (webpack, vite, make)
- Package managers (npm, yarn)
- Test runners (jest, pytest)
- Development servers

## Prompt Suggestions

Claude Code shows grayed-out suggestions based on your conversation:

- Press `Tab` to accept suggestion
- Press `Enter` to accept and submit
- Start typing to dismiss

Disable with:
```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

## Permission Mode Cycling

Press `Shift+Tab` to cycle through modes:

1. **Normal Mode** - Standard permission checking
2. **Auto-Accept Mode** (`⏵⏵ accept edits on`) - Auto-approve file edits
3. **Plan Mode** (`⏸ plan mode on`) - Read-only exploration

## Image Input

Add images to conversations:

1. **Drag and drop** into Claude Code window
2. **Copy/paste** with `Ctrl+V` (not `Cmd+V`)
3. **Provide path**: "Analyze this image: /path/to/image.png"

## References

- [Keyboard Shortcuts by Platform](references/keyboard-shortcuts.md)
- [Vim Mode Complete Reference](references/vim-mode.md)
