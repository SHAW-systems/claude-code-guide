# Notification Recipes

Complete notification setups for different environments and needs.

## macOS Desktop Notifications

### Basic Notification

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\" sound name \"Glass\"'"
      }]
    }]
  }
}
```

### With Message Content

```bash
#!/bin/bash
# ~/.claude/hooks/macos-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message // "Notification"')
TITLE=$(echo "$INPUT" | jq -r '.title // "Claude Code"')

osascript -e "display notification \"$MESSAGE\" with title \"$TITLE\" sound name \"Glass\""
```

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/macos-notify.sh"
      }]
    }]
  }
}
```

### Permission-Only (High Priority)

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display alert \"Claude Code\" message \"Permission required\" as critical'"
      }]
    }]
  }
}
```

## Linux Desktop Notifications

### Basic with notify-send

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' \"$(cat | jq -r '.message')\""
      }]
    }]
  }
}
```

### Urgent Notifications

```bash
#!/bin/bash
# ~/.claude/hooks/linux-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message // "Notification"')
TYPE=$(echo "$INPUT" | jq -r '.notification_type // "info"')

URGENCY="normal"
if [ "$TYPE" = "permission_prompt" ]; then
  URGENCY="critical"
fi

notify-send -u "$URGENCY" "Claude Code" "$MESSAGE"
```

### With Icon

```bash
notify-send -i terminal "Claude Code" "$MESSAGE"
```

## Terminal Bell

### Simple Bell

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

### Repeated Bell (Attention-Getting)

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt",
      "hooks": [{
        "type": "command",
        "command": "for i in 1 2 3; do printf '\\a'; sleep 0.3; done"
      }]
    }]
  }
}
```

## tmux Integration

### Status Bar Update

```bash
#!/bin/bash
# ~/.claude/hooks/tmux-status.sh
INPUT=$(cat)
TYPE=$(echo "$INPUT" | jq -r '.notification_type')

if [ "$TYPE" = "permission_prompt" ]; then
  tmux set-option -g status-style "bg=red,fg=white"
  tmux display-message "Claude needs permission!"
elif [ "$TYPE" = "idle_prompt" ]; then
  tmux set-option -g status-style "bg=yellow,fg=black"
  tmux display-message "Claude waiting for input"
fi

# Reset after delay
(sleep 30 && tmux set-option -g status-style "bg=green,fg=black") &
```

### Window Alert

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt",
      "hooks": [{
        "type": "command",
        "command": "tmux select-window -t :claude 2>/dev/null; printf '\\a'"
      }]
    }]
  }
}
```

## Slack Integration

### Webhook Notification

```bash
#!/bin/bash
# ~/.claude/hooks/slack-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message')
TYPE=$(echo "$INPUT" | jq -r '.notification_type')

# Only notify for important events
case "$TYPE" in
  permission_prompt|idle_prompt|error)
    curl -s -X POST -H 'Content-type: application/json' \
      --data "{\"text\":\"🤖 Claude Code: $MESSAGE\"}" \
      "${SLACK_WEBHOOK_URL}"
    ;;
esac
```

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/slack-notify.sh"
      }]
    }]
  }
}
```

## Email Notification

### Via mail Command

```bash
#!/bin/bash
# ~/.claude/hooks/email-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message')
TYPE=$(echo "$INPUT" | jq -r '.notification_type')

if [ "$TYPE" = "permission_prompt" ]; then
  echo "$MESSAGE" | mail -s "Claude Code Needs Attention" your@email.com
fi
```

## Pushover (Mobile Push)

```bash
#!/bin/bash
# ~/.claude/hooks/pushover-notify.sh
INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message')
TYPE=$(echo "$INPUT" | jq -r '.notification_type')

PRIORITY=0
[ "$TYPE" = "permission_prompt" ] && PRIORITY=1

curl -s \
  --form-string "token=${PUSHOVER_APP_TOKEN}" \
  --form-string "user=${PUSHOVER_USER_KEY}" \
  --form-string "message=$MESSAGE" \
  --form-string "priority=$PRIORITY" \
  --form-string "title=Claude Code" \
  https://api.pushover.net/1/messages.json
```

## Sound Alerts

### macOS System Sounds

```bash
# Different sounds for different events
#!/bin/bash
TYPE=$(cat | jq -r '.notification_type')

case "$TYPE" in
  permission_prompt)
    afplay /System/Library/Sounds/Sosumi.aiff
    ;;
  idle_prompt)
    afplay /System/Library/Sounds/Glass.aiff
    ;;
  *)
    afplay /System/Library/Sounds/Pop.aiff
    ;;
esac
```

### Linux with paplay

```bash
#!/bin/bash
TYPE=$(cat | jq -r '.notification_type')

case "$TYPE" in
  permission_prompt)
    paplay /usr/share/sounds/freedesktop/stereo/dialog-warning.oga
    ;;
  *)
    paplay /usr/share/sounds/freedesktop/stereo/message.oga
    ;;
esac
```

## Combined Setup (Recommended)

Full-featured notification system:

```bash
#!/bin/bash
# ~/.claude/hooks/notify-all.sh
set -e

INPUT=$(cat)
MESSAGE=$(echo "$INPUT" | jq -r '.message // "Notification"')
TYPE=$(echo "$INPUT" | jq -r '.notification_type // "info"')
TITLE=$(echo "$INPUT" | jq -r '.title // "Claude Code"')

# Terminal bell (always)
printf '\a'

# Desktop notification
if command -v osascript &>/dev/null; then
  # macOS
  SOUND="Glass"
  [ "$TYPE" = "permission_prompt" ] && SOUND="Sosumi"
  osascript -e "display notification \"$MESSAGE\" with title \"$TITLE\" sound name \"$SOUND\""
elif command -v notify-send &>/dev/null; then
  # Linux
  URGENCY="normal"
  [ "$TYPE" = "permission_prompt" ] && URGENCY="critical"
  notify-send -u "$URGENCY" "$TITLE" "$MESSAGE"
fi

# Optional: Slack for permission prompts
if [ "$TYPE" = "permission_prompt" ] && [ -n "$SLACK_WEBHOOK_URL" ]; then
  curl -s -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"⚠️ $TITLE: $MESSAGE\"}" \
    "$SLACK_WEBHOOK_URL" &
fi

exit 0
```

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/notify-all.sh"
      }]
    }]
  }
}
```

## Session Start/End Notifications

Track when sessions begin and end:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' 'Session started'"
      }]
    }],
    "SessionEnd": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' 'Session ended'"
      }]
    }]
  }
}
```

## Testing Notifications

Verify your setup works:

```bash
# Test the hook script directly
echo '{"message":"Test notification","notification_type":"permission_prompt","title":"Test"}' | ~/.claude/hooks/notify-all.sh

# In Claude Code, trigger a permission prompt
# by asking Claude to run a bash command
```
