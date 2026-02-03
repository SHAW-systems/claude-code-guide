---
name: custom-agents
description: "Creating and using specialized AI subagents in Claude Code. Use when you need isolated task execution, specialized workflows, or want to delegate complex tasks."
---

# Custom Agents (Subagents)

Subagents are specialized AI assistants that run in their own context with custom prompts, specific tool access, and independent permissions.

## Why Use Subagents

- **Preserve context**: Keep exploration out of main conversation
- **Enforce constraints**: Limit tools for specific tasks
- **Specialize behavior**: Focused prompts for specific domains
- **Control costs**: Route tasks to faster/cheaper models like Haiku

## Built-in Subagents

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| **Explore** | Haiku | Read-only | Fast codebase search and analysis |
| **Plan** | Inherit | Read-only | Gather context for planning |
| **general-purpose** | Inherit | All | Complex multi-step tasks |
| **Bash** | Inherit | Bash | Terminal commands in separate context |

### Using Built-in Agents

Claude auto-delegates based on task. You can also request explicitly:

```
Use a subagent to investigate how authentication works
```

```
Use the Explore agent to find all API endpoints
```

## Creating Custom Subagents

### Using /agents Command

1. Run `/agents`
2. Select **Create new agent**
3. Choose scope: **User-level** or **Project-level**
4. Either **Generate with Claude** or write manually
5. Select allowed tools
6. Choose model
7. Pick a display color

### Manual Creation

Create markdown files with YAML frontmatter:

**User-level** (all projects): `~/.claude/agents/<name>.md`
**Project-level**: `.claude/agents/<name>.md`

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a senior code reviewer. When invoked, analyze code and provide
specific, actionable feedback on quality, security, and best practices.
```

### CLI-Defined Agents

Define agents inline for the current session:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after changes.",
    "prompt": "You are a senior code reviewer...",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

## Configuration Options

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should use this agent |
| `tools` | No | Allowed tools (inherits all if omitted) |
| `disallowedTools` | No | Tools to deny |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | No | Permission mode override |
| `skills` | No | Skills to preload |
| `hooks` | No | Lifecycle hooks |

### Tool Restrictions

```yaml
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit
---
```

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Standard permission checking |
| `acceptEdits` | Auto-accept file edits |
| `dontAsk` | Auto-deny unless pre-approved |
| `bypassPermissions` | Skip all checks (use with caution) |
| `plan` | Read-only exploration |

### Preloading Skills

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---
```

## Example Subagents

### Code Reviewer (Read-only)

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high code quality.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Clear, readable code
- Well-named functions/variables
- No duplicated code
- Proper error handling
- No exposed secrets
- Good test coverage

Provide feedback by priority: Critical, Warnings, Suggestions.
```

### Debugger

```markdown
---
name: debugger
description: Debugging specialist for errors and test failures.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works
```

### Data Scientist

```markdown
---
name: data-scientist
description: Data analysis expert for SQL and data insights.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

Key practices:
- Write optimized SQL queries
- Use appropriate aggregations
- Format results clearly
- Provide data-driven recommendations
```

### Database Query Validator

Using hooks for conditional validation:

```markdown
---
name: db-reader
description: Execute read-only database queries.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access.
Execute SELECT queries to answer data questions.
```

## Working with Subagents

### Foreground vs Background

- **Foreground**: Blocks main conversation, permission prompts pass through
- **Background**: Run concurrently, auto-deny unpermitted actions

Press `Ctrl+B` to background a running task.

### Resuming Subagents

Subagents can be resumed with their full context:

```
Continue that code review and now analyze the authorization logic
```

### Disabling Subagents

Add to settings `deny` rules:

```json
{
  "permissions": {
    "deny": ["Task(Explore)", "Task(my-custom-agent)"]
  }
}
```

Or via CLI:
```bash
claude --disallowedTools "Task(Explore)"
```

## Decision Guide

| Use Case | Approach |
|----------|----------|
| Quick codebase search | Built-in Explore agent |
| Complex multi-step task | general-purpose agent |
| Specialized domain work | Custom subagent |
| Isolated experiments | Subagent (preserves main context) |
| Cost-sensitive tasks | Haiku-based agent |

## References

- [Subagent Configuration](references/subagent-config.md)
- [Hook Configuration for Subagents](references/subagent-hooks.md)
