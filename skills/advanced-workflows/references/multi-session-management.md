# Multi-Session Management

Comprehensive guide to running, tracking, and managing multiple Claude Code sessions.

## Session Inventory

### List All Sessions

```bash
# Interactive session picker
claude --resume

# List by project
ls -lt ~/.claude/projects/*/sessions/*.jsonl | head -20

# Find sessions by name
grep -l "session_name" ~/.claude/projects/*/sessions/*.jsonl
```

### Session Metadata

Each session has:
- **Session ID**: UUID for unique identification
- **Name**: Optional human-readable name (`/rename`)
- **Project**: Directory where started
- **Branch**: Git branch (if applicable)
- **Transcript**: Full conversation history

### Naming Conventions

Adopt consistent naming for easy tracking:

```
/rename feature-auth-oauth
/rename bugfix-login-timeout
/rename refactor-api-v2
/rename explore-caching-options
```

Pattern: `{type}-{area}-{description}`

## Running Multiple Sessions

### Terminal Multiplexer (tmux)

**Setup:**
```bash
# Create named sessions
tmux new-session -d -s claude-feature 'cd ~/project && claude'
tmux new-session -d -s claude-bugfix 'cd ~/project && claude'
tmux new-session -d -s claude-review 'cd ~/project && claude'

# Attach to one
tmux attach -t claude-feature
```

**Navigation:**
- `Ctrl+B d` - Detach (leave running)
- `Ctrl+B s` - Session list
- `Ctrl+B (` / `)` - Previous/next session
- `tmux list-sessions` - Show all

**Status monitoring:**
```bash
# See all sessions
tmux list-sessions

# Check specific session output
tmux capture-pane -t claude-feature -p | tail -20
```

### Screen Alternative

```bash
# Create sessions
screen -dmS claude-1 bash -c 'cd ~/project && claude'
screen -dmS claude-2 bash -c 'cd ~/project && claude'

# Attach
screen -r claude-1

# List
screen -ls
```

### Git Worktrees (Isolated File State)

When sessions need separate file states:

```bash
# Create worktrees
git worktree add ../project-feature-auth feature/auth
git worktree add ../project-bugfix-123 bugfix/issue-123
git worktree add ../project-experiment experiment

# Start Claude in each
cd ../project-feature-auth && claude &
cd ../project-bugfix-123 && claude &
cd ../project-experiment && claude &

# List worktrees
git worktree list

# Clean up when done
git worktree remove ../project-feature-auth
```

**When to use worktrees:**
- Parallel work on different branches
- Need to run/test code independently
- Conflicting changes between sessions

### Simple Background

For quick parallel tasks:

```bash
# Start in background
claude -p "analyze auth system" --output-format json > auth-analysis.json &
claude -p "analyze database schema" --output-format json > db-analysis.json &

# Wait for all
wait

# Review results
cat auth-analysis.json db-analysis.json | jq '.'
```

## Tracking Active Sessions

### Session Dashboard Script

```bash
#!/bin/bash
# ~/bin/claude-dashboard.sh

echo "=== Running Claude Processes ==="
pgrep -a node | grep claude || echo "None"

echo -e "\n=== tmux Sessions ==="
tmux list-sessions 2>/dev/null | grep claude || echo "None"

echo -e "\n=== Recent Sessions (last 24h) ==="
find ~/.claude/projects -name "*.jsonl" -mtime -1 -exec ls -lt {} + | head -10

echo -e "\n=== Background Jobs ==="
jobs -l 2>/dev/null || echo "None"
```

### Session Status Check

```bash
#!/bin/bash
# Check if any session needs attention
# Run periodically via cron

NEEDS_ATTENTION=false

# Check for permission prompts in recent transcripts
for transcript in ~/.claude/projects/*/sessions/*.jsonl; do
  if [ -f "$transcript" ]; then
    LAST_LINE=$(tail -1 "$transcript" 2>/dev/null)
    if echo "$LAST_LINE" | grep -q "permission_request"; then
      echo "Permission needed: $transcript"
      NEEDS_ATTENTION=true
    fi
  fi
done

if [ "$NEEDS_ATTENTION" = true ]; then
  notify-send -u critical "Claude Sessions" "Some sessions need attention"
fi
```

## Workload Distribution

### Task Assignment Strategy

| Task Type | Session Strategy |
|-----------|------------------|
| Feature development | Dedicated named session |
| Bug investigation | Separate exploration session |
| Code review | Independent session per PR |
| Batch operations | Fan-out headless sessions |
| Experiments | Forked sessions |

### Load Balancing Example

Distribute migration across sessions:

```bash
#!/bin/bash
# Distribute 100 files across 4 parallel sessions

FILES=($(find src -name "*.js" | head -100))
BATCH_SIZE=25

for i in 0 1 2 3; do
  START=$((i * BATCH_SIZE))
  BATCH=("${FILES[@]:$START:$BATCH_SIZE}")

  {
    for file in "${BATCH[@]}"; do
      claude -p "Migrate $file to TypeScript" --allowedTools "Read" "Edit" "Write"
    done
  } &

  PIDS+=($!)
done

# Wait for all batches
for pid in "${PIDS[@]}"; do
  wait $pid
done
```

## Session Handoffs

### Continue in Different Terminal

```bash
# Terminal 1: Start work
claude
> /rename auth-work
# ... do some work
# Close terminal

# Terminal 2: Continue
claude --resume auth-work
```

### Hand Off to Colleague

```bash
# You: Commit current state
git add -A && git commit -m "WIP: auth implementation"
git push origin feature/auth

# Record session context
claude -p "Summarize current progress and next steps" > handoff-notes.txt
git add handoff-notes.txt && git commit -m "Add handoff notes"
git push

# Colleague: Pick up
git pull origin feature/auth
cat handoff-notes.txt
claude  # Start fresh with context
```

### PR-Linked Sessions

Sessions automatically link to PRs:

```bash
# Resume sessions for a specific PR
claude --from-pr 123
```

## Session Cleanup

### Identify Old Sessions

```bash
# Sessions older than 7 days
find ~/.claude/projects -name "*.jsonl" -mtime +7

# Large sessions (>10MB)
find ~/.claude/projects -name "*.jsonl" -size +10M
```

### Manual Cleanup

```bash
# Remove specific session
rm ~/.claude/projects/PROJECT_HASH/sessions/SESSION_ID.jsonl

# Remove old sessions (careful!)
find ~/.claude/projects -name "*.jsonl" -mtime +30 -delete
```

### Automatic Cleanup

In settings:
```json
{
  "cleanupPeriodDays": 14
}
```

## Session Recovery

### Find Lost Session

```bash
# By approximate time
find ~/.claude/projects -name "*.jsonl" -newermt "2024-01-15" -not -newermt "2024-01-16"

# By content
grep -l "keyword from session" ~/.claude/projects/*/sessions/*.jsonl

# By project directory
ls ~/.claude/projects/
# Find hash for your project, then:
ls -lt ~/.claude/projects/PROJECT_HASH/sessions/
```

### Inspect Session Content

```bash
# View recent messages
tail -50 ~/.claude/projects/PROJECT_HASH/sessions/SESSION_ID.jsonl | jq -r '.content // .message // empty'

# Find specific action
grep "tool_use" SESSION.jsonl | jq '.tool'
```

### Fork for Recovery

When original session is corrupted but has useful context:

```bash
# Fork creates new session with same history
claude --resume SESSION_ID --fork-session

# Now you have a working copy
```

## Coordination Patterns

### Sequential Pipeline

```bash
#!/bin/bash
# Session 1: Implement
IMPL_SESSION=$(claude -p "Implement feature X" --output-format json | jq -r '.session_id')
echo "Implementation session: $IMPL_SESSION"

# Session 2: Test (after implementation)
claude -p "Write tests for the implementation from session $IMPL_SESSION" \
  --allowedTools "Read" "Write" "Bash(npm test *)"

# Session 3: Review (after tests)
claude -p "Review implementation and tests for edge cases" \
  --allowedTools "Read"
```

### Parallel with Merge

```bash
# Start parallel investigations
claude -p "Analyze authentication patterns" > auth.md &
PID1=$!
claude -p "Analyze database patterns" > db.md &
PID2=$!
claude -p "Analyze API patterns" > api.md &
PID3=$!

wait $PID1 $PID2 $PID3

# Merge findings
cat auth.md db.md api.md | claude -p "Synthesize these analyses into recommendations"
```

### Supervisor Pattern

One session coordinates others:

```bash
# Supervisor session
claude -p "You are coordinating a refactoring project.
  1. Start investigation sessions for each module
  2. Collect their findings
  3. Create unified plan
  4. Delegate implementation
  Use subagents for each step."
```

## Monitoring Dashboard

### Real-time Session Monitor

```bash
#!/bin/bash
# ~/bin/claude-monitor.sh
# Run in a dedicated terminal: watch -n 5 ~/bin/claude-monitor.sh

clear
echo "=== Claude Session Monitor ==="
echo "Updated: $(date)"
echo

echo "Active Processes:"
pgrep -af "claude" | grep -v "claude-monitor" || echo "  None"
echo

echo "Recent Activity (last 10 min):"
find ~/.claude/projects -name "*.jsonl" -mmin -10 -exec basename {} \; | sort -u | head -5 || echo "  None"
echo

echo "tmux Sessions:"
tmux list-sessions 2>/dev/null | grep -i claude || echo "  None"
echo

echo "Background Jobs:"
jobs -l 2>/dev/null || echo "  None"
```

### Cron-based Alerts

```cron
# Check every 5 minutes for sessions needing attention
*/5 * * * * ~/bin/check-claude-sessions.sh
```

## Best Practices

1. **Name sessions immediately** when starting important work
2. **Use tmux/screen** for long-running sessions you'll revisit
3. **Worktrees for isolation** when sessions modify files
4. **Monitor actively** during critical operations
5. **Clean up regularly** to avoid confusion
6. **Document handoffs** when passing work to others
7. **Set notifications** so you know when sessions need input
8. **Review before closing** to ensure work is saved
