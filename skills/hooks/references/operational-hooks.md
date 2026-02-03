# Operational Hooks

Hooks for monitoring sessions, ensuring quality, and maintaining responsible automation.

## Session State Monitoring

### Track Session Activity

Log all session activity for monitoring:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -Iseconds) SESSION_START $(jq -r '.source')\" >> ~/.claude/session-log.txt"
      }]
    }],
    "SessionEnd": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -Iseconds) SESSION_END $(jq -r '.reason')\" >> ~/.claude/session-log.txt"
      }]
    }]
  }
}
```

### Alert on Idle

Notify when Claude finishes and is waiting:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "notify-send -u normal 'Claude Code' 'Task completed, awaiting input'"
      }]
    }]
  }
}
```

### Permission Request Alerts

High-priority alert when permission needed:

```bash
#!/bin/bash
# ~/.claude/hooks/permission-alert.sh
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input | tostring' | head -c 100)

# Desktop notification
if command -v osascript &>/dev/null; then
  osascript -e "display alert \"Permission Required\" message \"Tool: $TOOL\" as critical"
elif command -v notify-send &>/dev/null; then
  notify-send -u critical "Claude Code Permission" "Tool: $TOOL"
fi

# Terminal bell
printf '\a\a\a'

# Log it
echo "$(date -Iseconds) PERMISSION_REQUEST $TOOL $TOOL_INPUT" >> ~/.claude/session-log.txt
```

```json
{
  "hooks": {
    "PermissionRequest": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/permission-alert.sh"
      }]
    }]
  }
}
```

## Quality Assurance Hooks

### Verify Tests Pass Before Stopping

Don't let Claude finish without passing tests:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "agent",
        "prompt": "Verify that all tests pass. Run 'npm test' and check results. Return {\"ok\": false, \"reason\": \"Tests failing: [errors]\"} if any fail.",
        "timeout": 120
      }]
    }]
  }
}
```

### Lint Check After Edits

Auto-run linter after any file change:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npm run lint --silent 2>&1 || echo 'Lint issues found' >&2",
        "timeout": 30
      }]
    }]
  }
}
```

### Type Check After TypeScript Changes

```bash
#!/bin/bash
# ~/.claude/hooks/typecheck.sh
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# Only check TypeScript files
if [[ "$FILE" == *.ts ]] || [[ "$FILE" == *.tsx ]]; then
  npx tsc --noEmit 2>&1 | head -20
fi
```

### Prevent Commits Without Tests

Block git commit if tests fail:

```bash
#!/bin/bash
# ~/.claude/hooks/pre-commit-check.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [[ "$COMMAND" == git\ commit* ]]; then
  if ! npm test --silent 2>/dev/null; then
    echo '{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Tests must pass before committing"}}'
    exit 0
  fi
fi
```

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/pre-commit-check.sh"
      }]
    }]
  }
}
```

## Safety Hooks

### Block Dangerous Commands

```bash
#!/bin/bash
# ~/.claude/hooks/safety-check.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Dangerous patterns
BLOCKED_PATTERNS=(
  "rm -rf /"
  "rm -rf ~"
  "rm -rf \$HOME"
  "> /dev/sda"
  "mkfs"
  "dd if="
  ":(){:|:&};:"
)

for pattern in "${BLOCKED_PATTERNS[@]}"; do
  if [[ "$COMMAND" == *"$pattern"* ]]; then
    cat << EOF
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Blocked dangerous command pattern: $pattern"}}
EOF
    exit 0
  fi
done

exit 0
```

### Protect Sensitive Files

```bash
#!/bin/bash
# ~/.claude/hooks/protect-sensitive.sh
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED=(
  ".env"
  ".env.local"
  ".env.production"
  "credentials"
  "secrets"
  ".ssh"
  ".aws"
  ".gnupg"
)

for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    cat << EOF
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Cannot modify sensitive file: $FILE"}}
EOF
    exit 0
  fi
done
```

### Limit Modifications Per Session

Track and limit how many files Claude modifies:

```bash
#!/bin/bash
# ~/.claude/hooks/limit-modifications.sh
INPUT=$(cat)
SESSION=$(echo "$INPUT" | jq -r '.session_id')
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

COUNTER_FILE="/tmp/claude-mod-count-$SESSION"
MAX_MODIFICATIONS=50

# Get current count
COUNT=$(cat "$COUNTER_FILE" 2>/dev/null || echo "0")
COUNT=$((COUNT + 1))
echo "$COUNT" > "$COUNTER_FILE"

if [ "$COUNT" -gt "$MAX_MODIFICATIONS" ]; then
  cat << EOF
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Session modification limit ($MAX_MODIFICATIONS) reached. Review changes before continuing."}}
EOF
  exit 0
fi
```

## Progress Tracking Hooks

### Log All Tool Usage

```bash
#!/bin/bash
# ~/.claude/hooks/log-tool-use.sh
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')
SESSION=$(echo "$INPUT" | jq -r '.session_id')

echo "$(date -Iseconds) $SESSION $TOOL" >> ~/.claude/tool-usage.log
```

### Track File Changes

```bash
#!/bin/bash
# ~/.claude/hooks/track-changes.sh
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
SESSION=$(echo "$INPUT" | jq -r '.session_id')
EVENT=$(echo "$INPUT" | jq -r '.hook_event_name')

if [ -n "$FILE" ]; then
  echo "$(date -Iseconds) $SESSION $EVENT $FILE" >> ~/.claude/file-changes.log
fi
```

### Session Summary on End

```bash
#!/bin/bash
# ~/.claude/hooks/session-summary.sh
INPUT=$(cat)
SESSION=$(echo "$INPUT" | jq -r '.session_id')
TRANSCRIPT=$(echo "$INPUT" | jq -r '.transcript_path')

# Count actions from log
TOOL_COUNT=$(grep "$SESSION" ~/.claude/tool-usage.log 2>/dev/null | wc -l)
FILE_COUNT=$(grep "$SESSION" ~/.claude/file-changes.log 2>/dev/null | wc -l)

notify-send "Claude Session Ended" "Tools used: $TOOL_COUNT, Files changed: $FILE_COUNT"
```

## Automation Safety Net

### Prevent Runaway Loops

Detect if Claude is repeating the same action:

```bash
#!/bin/bash
# ~/.claude/hooks/loop-detector.sh
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name')
TOOL_INPUT_HASH=$(echo "$INPUT" | jq -r '.tool_input' | md5sum | cut -d' ' -f1)
SESSION=$(echo "$INPUT" | jq -r '.session_id')

HISTORY_FILE="/tmp/claude-action-history-$SESSION"
CURRENT="$TOOL:$TOOL_INPUT_HASH"

# Check last 5 actions
if grep -c "^$CURRENT$" "$HISTORY_FILE" 2>/dev/null | grep -q "^[3-9]"; then
  cat << EOF
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Possible loop detected - same action attempted multiple times"}}
EOF
  exit 0
fi

echo "$CURRENT" >> "$HISTORY_FILE"
# Keep only last 10 entries
tail -10 "$HISTORY_FILE" > "$HISTORY_FILE.tmp" && mv "$HISTORY_FILE.tmp" "$HISTORY_FILE"
```

### Cost Monitoring

Alert when session gets expensive:

```bash
#!/bin/bash
# ~/.claude/hooks/cost-alert.sh
# Run on Stop events to check cumulative cost

# This would need to track costs - simplified example
SESSION=$(cat | jq -r '.session_id')
# In practice, you'd track token counts and calculate cost

# Alert if estimate exceeds threshold
THRESHOLD=5.00
# CURRENT_COST=$(calculate_cost)  # Your implementation
# if (( $(echo "$CURRENT_COST > $THRESHOLD" | bc -l) )); then
#   notify-send -u critical "Claude Cost Alert" "Session cost: \$$CURRENT_COST"
# fi
```

## Complete Monitoring Setup

Comprehensive operational monitoring:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -Iseconds) START\" >> ~/.claude/session.log; notify-send 'Claude' 'Session started'"
      }]
    }],
    "Notification": [{
      "matcher": "permission_prompt",
      "hooks": [{
        "type": "command",
        "command": "notify-send -u critical 'Claude Code' 'Permission needed'; printf '\\a\\a'"
      }]
    }, {
      "matcher": "idle_prompt",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' 'Waiting for input'; printf '\\a'"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' >> ~/.claude/modified-files.log"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' 'Task completed'"
      }]
    }],
    "SessionEnd": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -Iseconds) END\" >> ~/.claude/session.log"
      }]
    }]
  }
}
```
