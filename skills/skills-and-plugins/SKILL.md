---
name: skills-and-plugins
description: "Creating and managing skills, custom slash commands, and plugins. Use when extending Claude Code with reusable workflows, domain knowledge, or packaged integrations."
---

# Skills and Plugins

Skills extend Claude's capabilities with instructions, templates, and scripts. Plugins bundle skills with hooks, agents, and MCP servers.

## Skills Overview

A skill is a `SKILL.md` file that Claude loads when relevant or when you invoke it with `/skill-name`.

### Creating a Skill

```bash
mkdir -p ~/.claude/skills/explain-code
```

```markdown
# ~/.claude/skills/explain-code/SKILL.md
---
name: explain-code
description: Explains code with diagrams and analogies. Use when asking "how does this work?"
---

When explaining code:
1. Start with an analogy
2. Draw ASCII diagram
3. Walk through step-by-step
4. Highlight common gotchas
```

### Skill Locations

| Location | Scope |
|----------|-------|
| `~/.claude/skills/<name>/SKILL.md` | All your projects |
| `.claude/skills/<name>/SKILL.md` | Project only |
| `<plugin>/skills/<name>/SKILL.md` | Where plugin enabled |

### Invoking Skills

**Automatic**: Claude loads when description matches your request
**Manual**: `/skill-name` or `/skill-name arguments`

## Skill Configuration

### Frontmatter Fields

```yaml
---
name: my-skill
description: What this does and when to use it
argument-hint: "[issue-number]"
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Grep
model: haiku
context: fork
agent: Explore
---
```

| Field | Purpose |
|-------|---------|
| `name` | Slash command name |
| `description` | When Claude should use it |
| `argument-hint` | Autocomplete hint |
| `disable-model-invocation` | Only manual invocation |
| `user-invocable` | Show in `/` menu |
| `allowed-tools` | Restrict tool access |
| `model` | Override model |
| `context` | `fork` for subagent execution |
| `agent` | Which subagent to use |

### Invocation Control

| Configuration | You Can Invoke | Claude Can Invoke |
|---------------|----------------|-------------------|
| Default | Yes | Yes |
| `disable-model-invocation: true` | Yes | No |
| `user-invocable: false` | No | Yes |

## Skill Patterns

### Reference Knowledge

Knowledge Claude applies to current work:

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming
- Return consistent error formats
- Include validation
```

### Task Workflow

Step-by-step instructions for actions:

```yaml
---
name: deploy
description: Deploy application to production
disable-model-invocation: true
---

Deploy $ARGUMENTS to production:
1. Run test suite
2. Build application
3. Push to deployment target
4. Verify deployment
```

### With Subagent

Run in isolated context:

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:
1. Find relevant files
2. Read and analyze
3. Summarize with file references
```

## String Substitutions

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | All arguments passed |
| `$ARGUMENTS[N]` | Specific argument (0-indexed) |
| `$0`, `$1`, ... | Shorthand for `$ARGUMENTS[N]` |
| `${CLAUDE_SESSION_ID}` | Current session ID |

### Dynamic Context

Inject shell command output:

```yaml
---
name: pr-summary
description: Summarize current PR
---

## PR Context
- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
- Files: !`gh pr diff --name-only`

Summarize this pull request...
```

## Supporting Files

Skills can include additional files:

```
my-skill/
├── SKILL.md           # Main instructions
├── template.md        # Template to fill
├── examples/
│   └── sample.md      # Example output
└── scripts/
    └── validate.sh    # Executable script
```

Reference from SKILL.md:
```markdown
For complete API details, see [reference.md](reference.md)
```

## Plugins

Plugins bundle multiple extensions:

```
my-plugin/
├── plugin.json           # Manifest
├── skills/               # Skills
├── agents/               # Subagents
├── hooks/hooks.json      # Hooks
└── mcp/                  # MCP config
```

### Installing Plugins

```
/plugin
```

Or from URL:
```
/plugin https://github.com/user/plugin
```

### Plugin Manifest

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "homepage": "https://github.com/...",
  "requires": {
    "claudeCode": ">=1.0.0"
  }
}
```

### Discovering Plugins

```
/plugin
```

Browse marketplace or add custom sources.

### Code Intelligence Plugins

Install language-specific code intelligence:

```
/plugin
# Select "Code Intelligence" category
# Choose your language
```

Provides:
- Go to definition
- Find references
- Automatic error detection after edits

## Visual Output Skills

Skills can generate HTML visualizations:

```yaml
---
name: codebase-visualizer
description: Generate interactive tree visualization
allowed-tools: Bash(python *)
---

Generate visualization:
```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```

This creates codebase-map.html and opens in browser.
```

## Built-in Skills

View with `/help` or ask "what skills are available?"

Common built-ins:
- `/commit-push-pr` - Commit, push, and create PR
- `/explain-code` - Code explanation
- `/init` - Initialize CLAUDE.md

## Managing Skills

### View Available

```
What skills are available?
```

Or check `/help`.

### Control Access

In settings:
```json
{
  "permissions": {
    "allow": ["Skill(commit)"],
    "deny": ["Skill(deploy *)"]
  }
}
```

### Skill Budget

If too many skills, some may be excluded. Increase budget:
```bash
export SLASH_COMMAND_TOOL_CHAR_BUDGET=30000
```

## Decision Guide

| Need | Approach |
|------|----------|
| Reusable instructions | Skill with description |
| Manual-only workflow | `disable-model-invocation: true` |
| Background knowledge | `user-invocable: false` |
| Isolated execution | `context: fork` |
| Multiple extensions | Create plugin |
| Visual output | Script-based skill |

## Migration from Commands

`.claude/commands/` still works. Skills add:
- Supporting files
- Invocation control
- Subagent execution
- Hooks

If both exist with same name, skill takes precedence.

## References

- [Skill Examples](references/skill-examples.md)
- [Plugin Structure](references/plugin-structure.md)
