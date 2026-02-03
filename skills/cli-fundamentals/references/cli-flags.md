# Complete CLI Flags Reference

## Session Management

| Flag | Description | Example |
|------|-------------|---------|
| `--continue`, `-c` | Load most recent conversation | `claude -c` |
| `--resume`, `-r` | Resume session by ID or name | `claude -r "auth-refactor"` |
| `--fork-session` | Create new session ID when resuming | `claude --resume abc --fork-session` |
| `--from-pr <num>` | Resume sessions linked to a PR | `claude --from-pr 123` |
| `--session-id <uuid>` | Use specific session ID | `claude --session-id "550e8400..."` |
| `--rename <name>` | Rename current session | Via `/rename` command |

## Output & Formatting

| Flag | Description | Example |
|------|-------------|---------|
| `--print`, `-p` | Non-interactive mode | `claude -p "query"` |
| `--output-format <fmt>` | Output format: `text`, `json`, `stream-json` | `claude -p "q" --output-format json` |
| `--verbose` | Show detailed turn-by-turn output | `claude --verbose` |
| `--json-schema <schema>` | Validate output against JSON schema | `claude -p "q" --json-schema '{...}'` |
| `--include-partial-messages` | Include streaming events (with stream-json) | `claude -p --output-format stream-json --include-partial-messages` |
| `--input-format <fmt>` | Input format: `text`, `stream-json` | `claude -p --input-format stream-json` |

## Permissions & Security

| Flag | Description | Example |
|------|-------------|---------|
| `--permission-mode <mode>` | Set permission mode | `claude --permission-mode plan` |
| `--allowedTools <tools>` | Auto-approve tools | `claude --allowedTools "Bash(git *)" "Read"` |
| `--disallowedTools <tools>` | Block tools | `claude --disallowedTools "Write" "Edit"` |
| `--dangerously-skip-permissions` | Skip all prompts (dangerous) | `claude --dangerously-skip-permissions` |
| `--allow-dangerously-skip-permissions` | Enable bypass as option | Combined with `--permission-mode` |
| `--permission-prompt-tool <tool>` | MCP tool for permission prompts | `claude -p --permission-prompt-tool mcp_auth` |

### Permission Modes

| Mode | Description |
|------|-------------|
| `default` | Standard prompts for first use of each tool |
| `acceptEdits` | Auto-accept file edit permissions |
| `plan` | Read-only exploration, no modifications |
| `dontAsk` | Auto-deny unless pre-approved |
| `bypassPermissions` | Skip all checks (isolated environments only) |

## Model & System Prompt

| Flag | Description | Example |
|------|-------------|---------|
| `--model <model>` | Set model for session | `claude --model opus` |
| `--fallback-model <model>` | Fallback when default overloaded | `claude -p --fallback-model sonnet` |
| `--system-prompt <text>` | Replace entire system prompt | `claude --system-prompt "You are..."` |
| `--system-prompt-file <file>` | Load system prompt from file | `claude -p --system-prompt-file prompt.txt` |
| `--append-system-prompt <text>` | Append to default prompt | `claude --append-system-prompt "Use TypeScript"` |
| `--append-system-prompt-file <file>` | Append file contents to prompt | `claude -p --append-system-prompt-file rules.txt` |

## Agents & Subagents

| Flag | Description | Example |
|------|-------------|---------|
| `--agent <name>` | Use specific agent | `claude --agent my-custom-agent` |
| `--agents <json>` | Define subagents via JSON | `claude --agents '{"reviewer":{...}}'` |

### Agents JSON Format

```json
{
  "agent-name": {
    "description": "When to use this agent",
    "prompt": "System prompt for the agent",
    "tools": ["Read", "Edit", "Bash"],
    "model": "sonnet"
  }
}
```

## Working Directories

| Flag | Description | Example |
|------|-------------|---------|
| `--add-dir <path>` | Add additional directories | `claude --add-dir ../apps ../lib` |

## MCP Configuration

| Flag | Description | Example |
|------|-------------|---------|
| `--mcp-config <file>` | Load MCP servers from JSON | `claude --mcp-config ./mcp.json` |
| `--strict-mcp-config` | Only use servers from --mcp-config | `claude --strict-mcp-config --mcp-config mcp.json` |

## Automation & Limits

| Flag | Description | Example |
|------|-------------|---------|
| `--max-turns <n>` | Limit agentic turns (print mode) | `claude -p --max-turns 3` |
| `--max-budget-usd <n>` | Maximum spend before stopping | `claude -p --max-budget-usd 5.00` |
| `--no-session-persistence` | Don't save session to disk | `claude -p --no-session-persistence` |

## Features & Integrations

| Flag | Description | Example |
|------|-------------|---------|
| `--chrome` | Enable Chrome integration | `claude --chrome` |
| `--no-chrome` | Disable Chrome integration | `claude --no-chrome` |
| `--ide` | Auto-connect to IDE | `claude --ide` |
| `--remote` | Create web session on claude.ai | `claude --remote "Fix login bug"` |
| `--teleport` | Resume web session locally | `claude --teleport` |

## Settings & Configuration

| Flag | Description | Example |
|------|-------------|---------|
| `--settings <file>` | Load settings from file | `claude --settings ./settings.json` |
| `--setting-sources <list>` | Setting sources to load | `claude --setting-sources user,project` |
| `--plugin-dir <path>` | Load plugins from directory | `claude --plugin-dir ./my-plugins` |
| `--disable-slash-commands` | Disable skills and slash commands | `claude --disable-slash-commands` |

## Debugging & Development

| Flag | Description | Example |
|------|-------------|---------|
| `--debug [categories]` | Enable debug mode | `claude --debug "api,mcp"` |
| `--verbose` | Enable verbose logging | `claude --verbose` |
| `--init` | Run initialization hooks | `claude --init` |
| `--init-only` | Run init hooks and exit | `claude --init-only` |
| `--maintenance` | Run maintenance hooks | `claude --maintenance` |
| `--version`, `-v` | Show version | `claude -v` |
| `--betas <headers>` | Beta headers for API | `claude --betas interleaved-thinking` |
