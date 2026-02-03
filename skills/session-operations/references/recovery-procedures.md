# Recovery Procedures

Step-by-step procedures for recovering from common problems.

## Session Disconnected

**Symptoms:** Terminal closed, SSH dropped, network interruption

**Recovery:**

1. **Find the session:**
   ```bash
   claude --resume
   # Lists all recent sessions
   ```

2. **Identify your session** by:
   - Session name (if you named it)
   - Project directory
   - Timestamp
   - Last message preview

3. **Resume:**
   ```bash
   claude --resume "session-name"
   # or select from list
   ```

4. **Check state:**
   ```
   What was I working on? What's the current status?
   ```

**Prevention:**
- Name sessions immediately: `/rename meaningful-name`
- Use tmux/screen for SSH sessions
- Enable notifications to know if prompts are waiting

## Claude Unresponsive / Hung

**Symptoms:** No output, spinner frozen, no response to input

**Recovery:**

1. **Try interrupt:**
   ```
   Ctrl+C
   ```

2. **If still frozen:**
   ```
   Ctrl+C Ctrl+C  # Force interrupt
   ```

3. **If terminal frozen:**
   - Open new terminal
   - Kill process: `pkill -f claude`

4. **Resume:**
   ```bash
   claude --resume
   ```

**Common causes:**
- Network timeout to API
- Very long-running bash command
- MCP server hanging

## Context Window Full

**Symptoms:** "Context limit reached", auto-compaction triggered, degraded responses

**Recovery:**

1. **Check context:**
   ```
   /context
   ```

2. **Compact with guidance:**
   ```
   /compact Keep: [specific task context]. Discard: [exploration, failed attempts]
   ```

3. **If compaction not enough, fresh start:**
   ```
   /clear
   ```
   Then provide summary:
   ```
   Continue task X. Status: Y completed, Z remaining. Key files: A, B, C.
   ```

**Prevention:**
- Use subagents for exploration
- `/clear` between unrelated tasks
- Set lower auto-compact threshold:
  ```bash
  export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50
  ```

## Claude in Wrong Direction

**Symptoms:** Claude implementing wrong approach, misunderstanding requirements

**Recovery:**

1. **Stop immediately:**
   ```
   Ctrl+C
   ```

2. **Undo changes:**
   ```
   Undo those changes
   ```
   or
   ```
   /rewind
   # Select checkpoint before wrong direction
   ```

3. **Redirect clearly:**
   ```
   Stop. That's not what I need. The requirement is [specific].
   Don't do X, instead do Y.
   ```

4. **If context polluted with failed attempts:**
   ```
   /clear
   ```
   Start fresh with better initial prompt.

## Permission Prompt Timeout

**Symptoms:** Permission prompt expired while you were away

**Recovery:**

1. **Resume session:**
   ```bash
   claude --resume
   ```

2. **The prompt should reappear.** If not:
   ```
   Continue where you left off
   ```

3. **Check what was requested:**
   ```
   What permission were you asking for?
   ```

**Prevention:**
- Set up notifications for `permission_prompt` events
- Pre-approve expected operations in settings

## Corrupted Session

**Symptoms:** Session won't load, crashes on resume, garbled output

**Recovery:**

1. **Try forking:**
   ```bash
   claude --resume "session-id" --fork-session
   ```

2. **If that fails, check transcript:**
   ```bash
   cat ~/.claude/projects/*/sessions/SESSION_ID.jsonl | tail -50
   ```

3. **Start fresh, salvage context:**
   ```bash
   claude
   ```
   Then:
   ```
   I was working on [task]. Here's the last known state: [summary].
   Continue from there.
   ```

4. **Report if reproducible:**
   ```
   /bug
   ```

## API Errors

**Symptoms:** "API error", "Rate limited", "Server error"

### Rate Limited

1. **Wait and retry:**
   - Usually resolves in 1-5 minutes
   - Session state preserved

2. **Check limits:**
   ```
   /usage
   ```

3. **Reduce request size:**
   ```
   /compact
   ```

### Authentication Error

1. **Re-authenticate:**
   ```bash
   claude --logout
   claude  # Will prompt for login
   ```

2. **Check API key (if using):**
   ```bash
   echo $ANTHROPIC_API_KEY
   ```

### Server Error (500s)

1. **Retry in a few minutes**
2. **Check status:** https://status.anthropic.com
3. **Resume session when service restored:**
   ```bash
   claude --resume
   ```

## Build/Test Failures After Claude Changes

**Symptoms:** Claude made changes that broke the build

**Recovery:**

1. **See what changed:**
   ```bash
   git diff
   git status
   ```

2. **Options:**
   - Ask Claude to fix:
     ```
     The build is failing with this error: [error]. Fix it.
     ```
   - Revert and try again:
     ```bash
     git checkout .
     ```
     Then start fresh approach
   - Partial revert:
     ```bash
     git checkout -- path/to/broken/file.js
     ```

3. **Use checkpoints:**
   ```
   /rewind
   # Select checkpoint before breaking changes
   ```

**Prevention:**
- Ask Claude to run tests after changes
- Use hooks to auto-run linting/tests

## Lost Work

**Symptoms:** Changes disappeared, can't find session

### Finding Lost Sessions

```bash
# List all sessions by date
ls -lt ~/.claude/projects/*/sessions/*.jsonl

# Search by content
grep -l "keyword" ~/.claude/projects/*/sessions/*.jsonl

# Check recently modified
find ~/.claude -name "*.jsonl" -mmin -60
```

### Recovering Code Changes

```bash
# Check git reflog
git reflog

# Check git stash
git stash list

# Check Claude checkpoints
claude --resume  # Browse sessions
/rewind          # Browse checkpoints within session
```

### If Git Has Changes

```bash
# See uncommitted changes
git diff

# See staged changes
git diff --staged

# Recover deleted file
git checkout HEAD -- path/to/file
```

## MCP Server Issues

**Symptoms:** MCP tools not working, timeouts, connection errors

### Server Not Starting

1. **Check configuration:**
   ```bash
   claude mcp get server-name
   ```

2. **Test manually:**
   ```bash
   # Run the command directly
   npx -y @anthropic-ai/mcp-server-name
   ```

3. **Check for port conflicts:**
   ```bash
   lsof -i :PORT
   ```

4. **Remove and re-add:**
   ```bash
   claude mcp remove server-name
   claude mcp add server-name [command]
   ```

### Server Timeout

1. **Increase timeout:**
   ```bash
   export MCP_TIMEOUT=60000  # 60 seconds
   ```

2. **Check server health:**
   ```
   /mcp
   ```

## Diagnosis Commands

Quick diagnostic commands to understand state:

```bash
# Claude Code health
claude doctor

# Session status
/status

# Context usage
/context

# Cost/usage
/cost

# Recent sessions
claude --resume  # View without selecting

# MCP status
/mcp

# Background tasks
/tasks
```

## Emergency Reset

When all else fails:

```bash
# 1. Save any important unsaved work
git stash  # if you have uncommitted changes

# 2. Kill any Claude processes
pkill -f claude

# 3. Clear cache (preserves settings)
rm -rf ~/.claude/cache

# 4. Start fresh
claude

# 5. Resume session if needed
claude --resume
```

**Nuclear option (loses all local state):**
```bash
# ⚠️ DESTRUCTIVE - removes all sessions and settings
rm -rf ~/.claude
claude  # Fresh start
```

## Getting Help

If you can't resolve the issue:

1. **In-app bug report:**
   ```
   /bug
   ```

2. **GitHub issues:**
   https://github.com/anthropics/claude-code/issues

3. **Discord:**
   https://anthropic.com/discord

Include:
- What you were doing
- Error message (exact text)
- `claude doctor` output
- OS and terminal info
