---
name: cli-fundamentals
description: "Core CLI commands, flags, and invocation patterns for Claude Code. Use when starting sessions, running commands, or configuring CLI behavior."
---

# CLI Fundamentals

Claude Code is an agentic coding tool that runs in your terminal. This skill covers installation, basic commands, flags, and invocation patterns.

## Installation

### Native Install (Recommended)

**macOS/Linux/WSL:**
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**
```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### Alternative Methods

| Method | Command | Auto-updates |
|--------|---------|--------------|
| Homebrew | `brew install --cask claude-code` | No |
| WinGet | `winget install Anthropic.ClaudeCode` | No |
| NPM (deprecated) | `npm install -g @anthropic-ai/claude-code` | No |

### Install Specific Version

```bash
# Install stable channel
curl -fsSL https://claude.ai/install.sh | bash -s stable

# Install specific version
curl -fsSL https://claude.ai/install.sh | bash -s 1.0.58
```

## Core Commands

| Command | Description | Example |
|---------|-------------|---------|
| `claude` | Start interactive REPL | `claude` |
| `claude "query"` | Start with initial prompt | `claude "explain this project"` |
| `claude -p "query"` | Non-interactive (print mode) | `claude -p "explain this function"` |
| `claude -c` | Continue most recent conversation | `claude -c` |
| `claude -r "<id>"` | Resume session by ID or name | `claude -r "auth-refactor"` |
| `claude update` | Update to latest version | `claude update` |
| `claude mcp` | Configure MCP servers | See [mcp-integration](../mcp-integration/SKILL.md) |
| `claude doctor` | Check installation health | `claude doctor` |

## Essential Flags

### Session Management

| Flag | Description |
|------|-------------|
| `--continue`, `-c` | Continue most recent conversation |
| `--resume`, `-r` | Resume specific session by ID/name |
| `--fork-session` | Create new session ID when resuming |
| `--from-pr <num>` | Resume sessions linked to a PR |

### Output Control

| Flag | Description |
|------|-------------|
| `--print`, `-p` | Non-interactive mode, print and exit |
| `--output-format <fmt>` | `text`, `json`, or `stream-json` |
| `--verbose` | Show detailed turn-by-turn output |
| `--json-schema` | Get validated JSON matching schema |

### Permission & Security

| Flag | Description |
|------|-------------|
| `--permission-mode <mode>` | `default`, `plan`, `acceptEdits`, `dontAsk`, `bypassPermissions` |
| `--allowedTools <tools>` | Auto-approve specific tools |
| `--disallowedTools <tools>` | Block specific tools |
| `--dangerously-skip-permissions` | Skip all permission prompts (use with caution) |

### Model & Configuration

| Flag | Description |
|------|-------------|
| `--model <model>` | Set model (`sonnet`, `opus`, or full name) |
| `--agent <name>` | Use specific agent for session |
| `--system-prompt <text>` | Replace entire system prompt |
| `--append-system-prompt <text>` | Add to default system prompt |

### Working Directories

| Flag | Description |
|------|-------------|
| `--add-dir <path>` | Add additional working directories |

## Piping & Integration

### Pipe Data Into Claude

```bash
# Process file contents
cat logs.txt | claude -p "explain these errors"

# Pipe command output
git diff | claude -p "review these changes"

# Chain with other tools
cat data.json | claude -p "analyze" --output-format json | jq '.result'
```

### Output Formats

```bash
# Plain text (default)
claude -p "summarize this" --output-format text

# JSON with metadata
claude -p "list functions" --output-format json

# Streaming JSON for real-time processing
claude -p "analyze logs" --output-format stream-json
```

### Structured Output with JSON Schema

```bash
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}}}'
```

## Common Patterns

### Quick One-Off Query
```bash
claude -p "what does this project do?"
```

### Continue Previous Work
```bash
claude --continue
# or
claude -c -p "now add tests"
```

### Start in Plan Mode
```bash
claude --permission-mode plan
```

### Use with Specific Model
```bash
claude --model opus
```

### Auto-Approve Safe Commands
```bash
claude --allowedTools "Bash(npm run *)" "Read" "Glob"
```

## System Requirements

- **OS**: macOS 13.0+, Ubuntu 20.04+/Debian 10+, Windows 10 1809+
- **RAM**: 4 GB+
- **Network**: Internet connection required
- **Shell**: Works best in Bash or Zsh

## Decision Guide

| Scenario | Command |
|----------|---------|
| Explore a new codebase | `claude` then ask questions |
| Quick question | `claude -p "your question"` |
| Continue previous task | `claude -c` |
| Run in CI/automation | `claude -p "task" --allowedTools "..." --output-format json` |
| Safe exploration only | `claude --permission-mode plan` |
| Review code changes | `git diff | claude -p "review"` |

## References

- [Full CLI Reference](references/cli-flags.md) - Complete flag documentation
- [Environment Variables](references/env-vars.md) - Configuration via environment
