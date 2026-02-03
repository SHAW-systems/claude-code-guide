# Claude Code Quick Reference

A cheat sheet for the most common commands and patterns.

## Installation

```bash
# macOS/Linux/WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Update
claude update
```

## Starting Sessions

| Command | Purpose |
|---------|---------|
| `claude` | Start interactive session |
| `claude "query"` | Start with initial prompt |
| `claude -p "query"` | Non-interactive (print mode) |
| `claude -c` | Continue most recent session |
| `claude -r "name"` | Resume named session |
| `claude --permission-mode plan` | Start in read-only Plan Mode |

## Essential Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit session |
| `Ctrl+G` | Open prompt in text editor |
| `Ctrl+O` | Toggle verbose output |
| `Ctrl+V` | Paste image |
| `Ctrl+B` | Background running task |
| `Esc + Esc` | Open rewind menu |
| `Shift+Tab` | Cycle permission modes |
| `Alt+T` | Toggle extended thinking |

## Slash Commands

### Session
| Command | Purpose |
|---------|---------|
| `/clear` | Reset conversation |
| `/compact` | Compress context |
| `/resume` | Resume session |
| `/rename` | Name current session |
| `/rewind` | Restore previous state |

### Information
| Command | Purpose |
|---------|---------|
| `/help` | Usage help |
| `/cost` | Token usage |
| `/context` | Context visualization |
| `/status` | Version, model, account |

### Configuration
| Command | Purpose |
|---------|---------|
| `/config` | Settings |
| `/permissions` | Permission rules |
| `/model` | Change model |
| `/memory` | Edit CLAUDE.md |
| `/init` | Initialize project |
| `/mcp` | MCP servers |
| `/hooks` | Configure hooks |
| `/agents` | Manage subagents |

## Quick Input Modes

| Prefix | Purpose | Example |
|--------|---------|---------|
| `/` | Commands/skills | `/help` |
| `!` | Direct bash | `! npm test` |
| `@` | File reference | `@src/auth.js` |

## Permission Modes

| Mode | Description |
|------|-------------|
| `default` | Normal prompts |
| `acceptEdits` | Auto-accept file edits |
| `plan` | Read-only exploration |
| `dontAsk` | Auto-deny unless allowed |
| `bypassPermissions` | Skip all (dangerous) |

Cycle with `Shift+Tab` or set with `--permission-mode`.

## Common CLI Flags

```bash
# Output control
--output-format text|json|stream-json
--verbose

# Permissions
--allowedTools "Bash(npm run *)" "Read"
--disallowedTools "Write"
--permission-mode plan

# Model
--model sonnet|opus|haiku

# Limits
--max-turns 10
--max-budget-usd 5.00

# Directories
--add-dir ../lib ../docs
```

## Piping

```bash
# Pipe in
cat error.log | claude -p "explain"
git diff | claude -p "review"

# Pipe out
claude -p "generate config" > config.json
claude -p "list issues" --output-format json | jq '.'
```

## CLAUDE.md Quick Setup

```bash
/init
```

Or create `CLAUDE.md`:
```markdown
# Commands
- Build: `npm run build`
- Test: `npm test`

# Code Style
- Use ES modules (import/export)
- Prefer TypeScript
```

## File Locations

| File | Purpose |
|------|---------|
| `~/.claude/CLAUDE.md` | Personal global |
| `./CLAUDE.md` | Project (shared) |
| `./CLAUDE.local.md` | Project (private) |
| `.claude/rules/*.md` | Modular rules |
| `~/.claude/settings.json` | User settings |
| `.claude/settings.json` | Project settings |
| `~/.claude/skills/` | Personal skills |
| `.claude/skills/` | Project skills |
| `.claude/agents/` | Custom subagents |

## MCP Quick Commands

```bash
# Add server
claude mcp add github npx -y @anthropic-ai/mcp-github

# List servers
claude mcp list

# Remove server
claude mcp remove github
```

## Hooks Quick Setup

Desktop notifications:
```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude' 'Needs attention'"
      }]
    }]
  }
}
```

Auto-format on edit:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs prettier --write"
      }]
    }]
  }
}
```

## Subagents

Built-in:
| Agent | Purpose |
|-------|---------|
| `Explore` | Fast codebase search (Haiku) |
| `Plan` | Read-only planning |
| `general-purpose` | Complex multi-step tasks |

Custom: Create `.claude/agents/<name>.md`

## Skill Quick Template

```markdown
---
name: my-skill
description: What it does and when to use it
---

Instructions for Claude...
```

Save to `~/.claude/skills/my-skill/SKILL.md`

## Headless/CI

```bash
# Simple query
claude -p "analyze this"

# With permissions
claude -p "build" --allowedTools "Bash(npm run *)"

# JSON output
claude -p "list functions" --output-format json

# Full automation (isolated only)
claude -p "task" --dangerously-skip-permissions
```

## Common Workflows

### Explore → Plan → Implement
```bash
claude --permission-mode plan  # Explore
# Plan the change
# Shift+Tab to exit Plan Mode
# Implement
```

### Continue Previous Work
```bash
claude -c -p "now add tests"
```

### Review Changes
```bash
git diff | claude -p "review for issues"
```

### Parallel Sessions
```bash
git worktree add ../feature-branch -b feature
cd ../feature-branch && claude
```

## Decision Quick Guide

| Need | Do |
|------|-----|
| Quick question | `claude -p "question"` |
| Explore codebase | Plan Mode |
| Continue work | `claude -c` |
| Keep context clean | Use subagents |
| Batch processing | `for f in *.js; do claude -p "process $f"; done` |
| CI integration | `--allowedTools` + `--output-format json` |

## Session State Recognition

| You See | State | Action |
|---------|-------|--------|
| Spinner moving | Working | Wait |
| `[Y/n]` or `[A/R/D]` | Needs permission | Respond |
| Blinking cursor at `>` | Idle/done | Review, give next task |
| Red error text | Failed | Diagnose |
| No output, frozen | Stuck | `Ctrl+C`, investigate |

## Permission Responses

| Key | Effect |
|-----|--------|
| `Y` / `Enter` | Allow once |
| `A` | Always allow (session) |
| `D` | Don't allow |
| `R` | Reject with feedback |
| `?` | Show more info |

## Notification Setup (Essential)

**macOS:**
```json
{"hooks":{"Notification":[{"matcher":"","hooks":[{"type":"command","command":"osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\" sound name \"Glass\"'"}]}]}}
```

**Linux:**
```json
{"hooks":{"Notification":[{"matcher":"","hooks":[{"type":"command","command":"notify-send -u critical 'Claude Code' \"$(cat | jq -r '.message')\""}]}]}}
```

**Terminal bell:**
```json
{"hooks":{"Notification":[{"matcher":"permission_prompt","hooks":[{"type":"command","command":"printf '\\a'"}]}]}}
```

## Multi-Session Management

```bash
# tmux (recommended for multiple sessions)
tmux new-session -d -s claude-1 'claude'
tmux new-session -d -s claude-2 'claude'
tmux attach -t claude-1
# Ctrl+B s to switch sessions

# List Claude sessions
claude --resume

# Check what's running
pgrep -af claude
```

## Before Leaving a Session

1. Check for pending permission prompts
2. Verify task state (done? blocked?)
3. Name session if important: `/rename feature-name`
4. Set up notifications if continuing unattended

## Recovery Quick Commands

| Problem | Command |
|---------|---------|
| Resume disconnected | `claude --resume` |
| Session stuck | `Ctrl+C`, then `claude --resume` |
| Undo bad changes | `/rewind` |
| Context full | `/compact` or `/clear` |
| Wrong direction | `Ctrl+C`, then redirect or `/rewind` |
| Find session by name | `claude --resume "name"` |
| Check health | `claude doctor` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Context filling up | `/clear` or use subagents |
| Claude ignores rules | CLAUDE.md too long, prune it |
| Repeated mistakes | `/clear`, write better prompt |
| Tool permission issues | Check `/permissions` |
| MCP not working | `claude mcp list`, restart server |
| Session won't resume | Try `--fork-session` flag |
| Notifications not working | Test hook script directly |

## Responsible Automation Checklist

Before unattended operation:
- [ ] Notifications configured (especially `permission_prompt`)
- [ ] Permission rules set for expected actions
- [ ] `--max-turns` or `--max-budget-usd` set
- [ ] Session named for tracking
- [ ] Recovery plan if things go wrong
