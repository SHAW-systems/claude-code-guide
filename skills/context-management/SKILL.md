---
name: context-management
description: "Managing Claude's memory with CLAUDE.md files, settings hierarchy, and project configuration. Use when setting up projects, configuring persistent instructions, or optimizing context usage."
---

# Context Management

Claude Code uses CLAUDE.md files and settings to maintain context across sessions.

## CLAUDE.md Files

### Memory Hierarchy

| Type | Location | Purpose | Shared |
|------|----------|---------|--------|
| **Managed policy** | System dirs | Organization-wide | All users |
| **User memory** | `~/.claude/CLAUDE.md` | Personal preferences | You (all projects) |
| **Project memory** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team instructions | Via git |
| **Project rules** | `./.claude/rules/*.md` | Modular instructions | Via git |
| **Local memory** | `./CLAUDE.local.md` | Personal project prefs | Just you |

All files load automatically at session start. Higher hierarchy = higher precedence.

### Creating CLAUDE.md

Bootstrap with `/init`:
```
/init
```

Or create manually:

```markdown
# Project Instructions

## Code Style
- Use ES modules (import/export), not CommonJS
- Destructure imports when possible

## Workflow
- Always typecheck after code changes
- Run single tests, not full suite, for speed

## Commands
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
```

### What to Include

| ✅ Include | ❌ Exclude |
|-----------|-----------|
| Bash commands Claude can't guess | Obvious patterns Claude can infer |
| Non-standard code style rules | Standard language conventions |
| Testing instructions | Long API documentation |
| Repository etiquette | Frequently changing info |
| Architectural decisions | Self-evident practices |
| Developer environment quirks | File-by-file descriptions |

### Imports

Reference other files with `@path/to/file`:

```markdown
See @README.md for project overview
Git workflow: @docs/git-instructions.md
Personal: @~/.claude/my-project-prefs.md
```

- Relative paths resolve from the importing file
- Max depth: 5 hops
- Not evaluated inside code blocks

## Modular Rules

Organize instructions in `.claude/rules/`:

```
.claude/rules/
├── code-style.md
├── testing.md
├── security.md
└── frontend/
    ├── react.md
    └── styles.md
```

All `.md` files discovered recursively.

### Path-Specific Rules

Use frontmatter to scope rules:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules
- Include input validation
- Use standard error format
- Add OpenAPI comments
```

### Glob Patterns

| Pattern | Matches |
|---------|---------|
| `**/*.ts` | All TypeScript files |
| `src/**/*` | Everything under src/ |
| `*.md` | Markdown in root only |
| `src/**/*.{ts,tsx}` | TS and TSX files |

## Settings Files

### Hierarchy (Precedence Order)

1. **Managed** (highest) - Cannot be overridden
2. **CLI arguments** - Session overrides
3. **Local** (`.claude/settings.local.json`) - Personal project
4. **Project** (`.claude/settings.json`) - Team shared
5. **User** (`~/.claude/settings.json`) - Personal global

### Key Settings

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)"],
    "deny": ["Read(./.env)"],
    "defaultMode": "acceptEdits"
  },
  "model": "claude-sonnet-4-5-20250929",
  "env": {
    "NODE_ENV": "development"
  }
}
```

## Context Window Management

### Why It Matters

- Claude's performance degrades as context fills
- Context is the most important resource to manage
- Long sessions with irrelevant context reduce quality

### Strategies

**Clear between tasks:**
```
/clear
```

**Compact with focus:**
```
/compact Focus on the API changes
```

**Check context usage:**
```
/context
```

**Use subagents for exploration** - keeps main context clean:
```
Use a subagent to investigate authentication patterns
```

### Auto-Compaction

Claude automatically compacts when context fills (~95% capacity).

Customize threshold:
```bash
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50
```

### Preserving Context During Compaction

Add to CLAUDE.md:
```markdown
# Compaction Instructions
When compacting, always preserve:
- List of modified files
- Test commands used
- Key architectural decisions
```

## Session Management

### Resume Conversations

```bash
# Continue most recent
claude --continue

# Resume by name
claude --resume auth-refactor

# Pick from list
claude --resume
```

### Name Sessions

```
/rename feature-auth
```

### Fork Sessions

Create branch from checkpoint:
```bash
claude --resume abc123 --fork-session
```

## Memory Commands

| Command | Purpose |
|---------|---------|
| `/init` | Bootstrap CLAUDE.md |
| `/memory` | Edit memory files |
| `/compact` | Manually compact context |
| `/context` | Visualize context usage |
| `/clear` | Reset context entirely |
| `/rewind` | Restore previous state |

## Best Practices

### Keep CLAUDE.md Concise

For each line, ask: *"Would removing this cause mistakes?"*

If not, delete it. Bloated files cause Claude to ignore instructions.

### Use Skills for Domain Knowledge

CLAUDE.md = always loaded, essential instructions
Skills = loaded on demand, specialized knowledge

### Review After Problems

When Claude repeatedly fails:
1. Check if CLAUDE.md is too long
2. Look for ambiguous phrasing
3. Test specific instruction changes

### Version Control

- Check `.claude/CLAUDE.md` into git for team
- Use `CLAUDE.local.md` for personal preferences
- `.local.md` files auto-added to `.gitignore`

## Decision Guide

| Need | Location |
|------|----------|
| All my projects | `~/.claude/CLAUDE.md` |
| Team conventions | `.claude/CLAUDE.md` (committed) |
| Personal project prefs | `CLAUDE.local.md` |
| File-specific rules | `.claude/rules/` with paths |
| Temporary instructions | Tell Claude directly |

## References

- [Settings Reference](references/settings.md)
- [Memory File Syntax](references/memory-syntax.md)
