---
name: session-operations
description: "Operational management of Claude Code sessions - monitoring state, handling permissions, lifecycle management, and knowing when sessions need attention. Use when running Claude Code tasks that require oversight."
---

# Session Operations

How to responsibly manage Claude Code sessions: monitoring state, responding to prompts, and ensuring tasks complete successfully.

## Session States

Claude Code sessions cycle through these states:

| State | What's Happening | Your Action |
|-------|------------------|-------------|
| **Working** | Claude is thinking or executing | Wait, monitor progress |
| **Permission Prompt** | Waiting for approval | Review and respond |
| **User Input** | Waiting for your response | Provide input or direction |
| **Idle** | Finished or waiting | Check results, give next task |
| **Error** | Something failed | Diagnose and recover |
| **Backgrounded** | Running detached | Check back periodically |

## Recognizing Session State

### Visual Indicators (Interactive Mode)

| Indicator | Meaning |
|-----------|---------|
| Spinner animating | Claude is working |
| `[Y/n]` or `[A/R/D]` prompt | Permission needed |
| Blinking cursor at `>` | Waiting for your input |
| Response complete, cursor ready | Task finished |
| Error message in red | Something failed |
| `⏸ plan mode on` | Read-only mode active |
| `⏵⏵ accept edits on` | Auto-accepting edits |

### Programmatic State Detection

In headless mode, parse output for state:

```bash
# Stream JSON shows state changes
claude -p "task" --output-format stream-json | while read line; do
  type=$(echo "$line" | jq -r '.type // empty')
  case "$type" in
    "assistant_message") echo "Claude responding..." ;;
    "tool_use") echo "Using tool: $(echo "$line" | jq -r '.tool')" ;;
    "permission_request") echo "NEEDS PERMISSION" ;;
    "result") echo "Finished" ;;
    "error") echo "ERROR: $(echo "$line" | jq -r '.error')" ;;
  esac
done
```

## Permission Prompt Handling

### Permission Prompt Types

| Prompt | What It's Asking |
|--------|------------------|
| `Allow Bash command?` | Shell command execution |
| `Allow file edit?` | Modify existing file |
| `Allow file write?` | Create new file |
| `Allow MCP tool?` | External integration |

### Response Options

| Key | Action | Effect |
|-----|--------|--------|
| `Y` / `Enter` | Allow once | Permits this specific action |
| `A` | Always allow | Permits this pattern for session |
| `D` | Don't allow | Blocks this action |
| `R` | Reject | Blocks and tells Claude why |
| `?` | More info | Shows details about the action |

### Pre-Approving Common Actions

Reduce interruptions by pre-approving safe patterns:

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Read",
      "Glob",
      "Grep"
    ]
  }
}
```

### Permission Mode for Unattended Work

```bash
# Auto-accept file edits (review later via git)
claude --permission-mode acceptEdits

# Full automation (isolated environments only)
claude --permission-mode bypassPermissions
```

## Notification Setup

### Desktop Notifications (Critical)

**macOS:**
```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"$(/bin/cat | jq -r .message)\" with title \"Claude Code\" sound name \"Glass\"'"
      }]
    }]
  }
}
```

**Linux:**
```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "notify-send -u critical 'Claude Code' \"$(/bin/cat | jq -r .message)\""
      }]
    }]
  }
}
```

### Terminal Bell

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt|idle_prompt",
      "hooks": [{
        "type": "command",
        "command": "printf '\\a'"
      }]
    }]
  }
}
```

### Notification Types to Monitor

| Type | When | Priority |
|------|------|----------|
| `permission_prompt` | Needs approval | High - blocks progress |
| `idle_prompt` | Claude finished, awaiting input | Medium |
| `auth_success` | Login completed | Low |
| `elicitation_dialog` | Clarification needed | High |

### Webhook/External Alerts

```bash
#!/bin/bash
# .claude/hooks/slack-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message')
TYPE=$(echo "$INPUT" | jq -r '.notification_type')

if [ "$TYPE" = "permission_prompt" ] || [ "$TYPE" = "idle_prompt" ]; then
  curl -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"Claude Code: $MESSAGE\"}" \
    "$SLACK_WEBHOOK_URL"
fi
```

## Session Lifecycle Workflow

### 1. Starting a Task

```bash
# Name session for tracking
claude
> /rename feature-auth

# Or start with name intent
claude "implement OAuth login"
```

**Checklist before stepping away:**
- [ ] Session named meaningfully
- [ ] Notifications configured
- [ ] Permission rules set for expected operations
- [ ] Know how to check back

### 2. Monitoring Progress

**Active monitoring:**
```bash
# Watch verbose output
Ctrl+O  # Toggle verbose mode
```

**Passive monitoring (backgrounded):**
```bash
# Check background tasks
/tasks

# Read task output
# (Task output file path shown when backgrounded)
```

**Remote check-in:**
```bash
# From another terminal, check session transcript
tail -f ~/.claude/projects/*/sessions/*.jsonl | jq -r '.type'
```

### 3. Checking Back Appropriately

| Notification | Response Time | Action |
|--------------|---------------|--------|
| Permission prompt | ASAP | Review and approve/deny |
| Idle prompt | When convenient | Review results, next task |
| Error | Soon | Diagnose, potentially restart |

### 4. Handling Permission Prompts

**When you see a permission prompt:**

1. **Read the action** - What is Claude trying to do?
2. **Assess risk** - Is this expected? Safe?
3. **Decide scope** - Once, always, or never?

```
# Example prompt:
Allow Bash command: rm -rf node_modules && npm install
[Y]es, [A]lways, [D]on't allow, [R]eject with feedback
```

**Guidelines:**
- `Y` for one-off actions you've verified
- `A` for patterns you'll always want (like `npm run test`)
- `D` for unexpected or risky actions
- `R` when Claude is going the wrong direction

### 5. Knowing When It's Done

**Indicators of completion:**
- Claude says "I've completed..." or similar
- Cursor returns to prompt with no spinner
- `/cost` shows no recent token usage
- No pending permission prompts

**Verify completion:**
```
What's the status? Are there remaining tasks?
```

### 6. Not Abandoning Sessions

**Before leaving a session:**

1. Check for pending prompts
2. Verify task state (done, blocked, or in-progress)
3. If long-running, set up notifications
4. Name session for later resumption

**If you must leave mid-task:**
```bash
# Background if possible
Ctrl+B

# Or note session ID for resumption
/status  # Shows session ID
```

## Multi-Session Management

### Tracking Active Sessions

```bash
# List recent sessions
claude --resume
# Shows all sessions with names, timestamps, projects

# Check specific session
claude --resume "feature-auth" --fork-session
# Forks to inspect without affecting original
```

### Running Parallel Sessions

**Using terminal multiplexer (recommended):**
```bash
# tmux
tmux new-session -s claude-1 'claude'
tmux new-session -s claude-2 'claude'

# Switch between: Ctrl+B then 0/1/2
# List: tmux list-sessions
```

**Using git worktrees:**
```bash
git worktree add ../project-feature feature-branch
cd ../project-feature && claude
```

### Session Inventory Check

Periodic check of all Claude sessions:

```bash
#!/bin/bash
# check-sessions.sh
echo "=== Active Claude Sessions ==="
pgrep -a claude || echo "No running sessions"

echo -e "\n=== Recent Sessions ==="
ls -lt ~/.claude/projects/*/sessions/*.jsonl 2>/dev/null | head -10

echo -e "\n=== Background Tasks ==="
# Check for any backgrounded processes
```

### Preventing Lost Work

1. **Name sessions immediately** after starting important work
2. **Commit incrementally** - Ask Claude to commit working states
3. **Use worktrees** for parallel work (separate git state)
4. **Check session list** before starting new work

## Recovery Patterns

### Session Disconnected

```bash
# Find and resume
claude --resume
# Select the session from list

# Or by name if you named it
claude --resume "feature-auth"
```

### Claude Stuck in Loop

```bash
# Interrupt
Ctrl+C

# If unresponsive
Ctrl+C Ctrl+C  # Force stop

# Clear confused context
/clear
```

### Permission Prompt Missed

If Claude was waiting and you weren't there:

```bash
# Resume session
claude --resume

# Session state preserved, prompt will reappear
# Or check transcript for what was requested
```

### Error Recovery

```bash
# Check what happened
/rewind  # See checkpoints

# Restore to before error
# Select checkpoint from menu

# Or start fresh with context
/compact Keep the progress on X, discard the failed approach to Y
```

### Context Window Full

```bash
# Manual compaction with guidance
/compact Focus on [specific task], discard exploration of [irrelevant area]

# Start fresh with summary
/clear
"Continue from where we left off. Summary: [paste key context]"
```

### Session Won't Start

```bash
# Check health
claude doctor

# Check auth
claude --status

# Clear cache and retry
rm -rf ~/.claude/cache
claude
```

## Responsible Automation Checklist

Before running Claude Code unattended:

- [ ] **Notifications configured** for permission prompts
- [ ] **Permission rules set** for expected operations
- [ ] **Budget limits set** (`--max-budget-usd`)
- [ ] **Turn limits set** (`--max-turns`) for headless
- [ ] **Session named** for tracking
- [ ] **Recovery plan** if things go wrong
- [ ] **Monitoring method** decided (active/passive)
- [ ] **Check-in schedule** established

## Quick State Reference

| You See | State | Do |
|---------|-------|-----|
| Spinner moving | Working | Wait |
| `[Y/n]` prompt | Needs permission | Respond |
| Cursor at `>` | Idle/done | Review, next task |
| Red error text | Failed | Diagnose |
| No output, hung | Possibly stuck | `Ctrl+C`, investigate |
| "Compacting..." | Context management | Wait |

## References

- [Notification Recipes](references/notification-recipes.md)
- [Recovery Procedures](references/recovery-procedures.md)
