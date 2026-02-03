---
name: hooks
description: "Lifecycle hooks for automation and custom behavior. Use when you need to run commands automatically on file edits, notifications, session events, or tool usage."
---

# Hooks

Hooks are shell commands that execute automatically at specific points in Claude Code's lifecycle. They provide deterministic control—certain actions always happen rather than relying on Claude to choose.

## Quick Start

### Using /hooks Menu

1. Run `/hooks`
2. Select an event (e.g., `Notification`)
3. Set matcher (e.g., `*` for all)
4. Add shell command
5. Choose storage location

### Example: Desktop Notifications

**macOS:**
```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\"'"
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
        "command": "notify-send 'Claude Code' 'Claude needs attention'"
      }]
    }]
  }
}
```

## Hook Events

| Event | When It Fires | Can Block |
|-------|---------------|-----------|
| `SessionStart` | Session begins/resumes | No |
| `UserPromptSubmit` | Prompt submitted | Yes |
| `PreToolUse` | Before tool executes | Yes |
| `PermissionRequest` | Permission dialog appears | Yes |
| `PostToolUse` | After tool succeeds | No |
| `PostToolUseFailure` | After tool fails | No |
| `Notification` | Notification sent | No |
| `SubagentStart` | Subagent spawned | No |
| `SubagentStop` | Subagent finishes | Yes |
| `Stop` | Claude finishes responding | Yes |
| `PreCompact` | Before context compaction | No |
| `SessionEnd` | Session terminates | No |

## Matchers

Filter when hooks fire:

| Event | Matcher Filters | Examples |
|-------|-----------------|----------|
| Tool events | Tool name | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | How started | `startup`, `resume`, `compact` |
| `SessionEnd` | Why ended | `clear`, `logout`, `other` |
| `Notification` | Type | `permission_prompt`, `idle_prompt` |
| `SubagentStart/Stop` | Agent type | `Explore`, `Plan`, custom names |
| `PreCompact` | Trigger | `manual`, `auto` |

## Configuration

### Hook Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "pattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-script.sh",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

### Storage Locations

| Location | Scope |
|----------|-------|
| `~/.claude/settings.json` | All your projects |
| `.claude/settings.json` | Current project (shared) |
| `.claude/settings.local.json` | Current project (private) |
| Managed settings | Organization-wide |

### Hook Fields

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `command`, `prompt`, or `agent` |
| `command` | Yes* | Shell command to run |
| `timeout` | No | Seconds before timeout (default: 600) |
| `statusMessage` | No | Custom spinner message |
| `async` | No | Run in background |

## Input and Output

### Hook Input (stdin)

Hooks receive JSON via stdin:

```json
{
  "session_id": "abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

### Exit Codes

| Code | Effect |
|------|--------|
| `0` | Success, action proceeds |
| `2` | Block the action, stderr becomes feedback |
| Other | Non-blocking error, logged |

### JSON Output

For structured control, exit 0 and print JSON:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Blocked for safety"
  }
}
```

## Common Patterns

### Auto-Format After Edits

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
      }]
    }]
  }
}
```

### Block Protected Files

```bash
#!/bin/bash
# .claude/hooks/protect-files.sh
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern" >&2
    exit 2
  fi
done
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
      }]
    }]
  }
}
```

### Re-inject Context After Compaction

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "compact",
      "hooks": [{
        "type": "command",
        "command": "echo 'Reminder: use Bun, not npm. Run tests before commit.'"
      }]
    }]
  }
}
```

### Validate Bash Commands

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q "rm -rf"; then
  echo "Blocked: rm -rf not allowed" >&2
  exit 2
fi
exit 0
```

## Prompt-Based Hooks

Use LLM to make decisions:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Check if all tasks complete. Return {\"ok\": false, \"reason\": \"...\"} if not."
      }]
    }]
  }
}
```

Response format:
```json
{"ok": true}
// or
{"ok": false, "reason": "Tests not run yet"}
```

## Agent-Based Hooks

Multi-turn verification with tool access:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "agent",
        "prompt": "Verify all unit tests pass by running the test suite.",
        "timeout": 120
      }]
    }]
  }
}
```

## Async Hooks

Run in background without blocking:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/run-tests.sh",
        "async": true,
        "timeout": 300
      }]
    }]
  }
}
```

## Debugging

- Run `claude --debug` for detailed hook logs
- Toggle verbose mode with `Ctrl+O`
- Check `/hooks` menu for configuration status

## Decision Guide

| Need | Hook Type |
|------|-----------|
| Format code after edits | `PostToolUse` with Edit|Write matcher |
| Block dangerous commands | `PreToolUse` with exit code 2 |
| Get notified when idle | `Notification` |
| Inject context on startup | `SessionStart` |
| Verify before stopping | `Stop` with prompt/agent type |
| Custom permission logic | `PreToolUse` with JSON output |

## References

- [Hook Events Reference](references/hook-events.md)
- [Input/Output Schemas](references/hook-schemas.md)
