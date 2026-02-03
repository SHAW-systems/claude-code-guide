# Claude Code Guide Builder

Your job: Read the Claude Code documentation and create a complete set of skills that will teach an AI agent to use Claude Code like an expert user.

## Sources to Read

Read these thoroughly before creating anything:

1. **Official Docs**: https://code.claude.com/docs/en/overview (navigate all sections)
2. **Tutorials**: https://claude.com/resources/tutorials
3. **Blog**: https://claude.com/blog (Claude Code related posts)
4. **GitHub**: https://github.com/anthropics/claude-code
5. **Use Cases**: https://claude.com/resources/use-cases

## Output: Skills

Create skills in the `skills/` directory. Each skill is a folder following this structure:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name + description)
│   └── Markdown instructions
└── Optional resources:
    ├── scripts/      - Executable code for deterministic tasks
    ├── references/   - Detailed docs to load as needed
    └── assets/       - Templates, examples, etc.
```

### SKILL.md Format

```markdown
---
name: skill-name
description: "What this skill does and WHEN to use it. This is the trigger — be specific about contexts where this skill applies."
---

# Skill Title

Instructions go here...
```

### Key Rules

1. **Description is the trigger** — Must clearly state what the skill does AND when to use it
2. **Keep SKILL.md under 500 lines** — Split details into `references/` files
3. **Progressive disclosure** — Core workflow in SKILL.md, detailed reference material in separate files
4. **Concise > verbose** — Include only what's needed, no fluff
5. **No extra docs** — No README.md, CHANGELOG.md, INSTALLATION.md, etc.
6. **Actionable content** — Real commands, real examples, decision trees for when to use what
7. **Link references from SKILL.md** — Always reference supporting files so they're discoverable

### Skill Organization

Determine the logical skill breakdown based on what you find in the documentation. Consider:

- **CLI fundamentals** — Flags, options, invocation patterns
- **Interactive usage** — Slash commands, keyboard shortcuts, TUI navigation
- **Agents** — Creating, configuring, invoking custom agents
- **Permissions** — Permission modes, headless operation, automation
- **Hooks** — Lifecycle events, automation triggers
- **Context management** — CLAUDE.md, settings, project structure
- **MCP integration** — If applicable
- **Advanced workflows** — Multi-session, tmux usage, orchestration patterns

This is a suggestion — organize based on what the documentation actually contains.

## Goal

The AI agent using these skills should be able to:

1. Run Claude Code interactively like a power user (slash commands, navigation, shortcuts)
2. Run Claude Code headlessly for automated tasks
3. Create and manage custom agents
4. Configure permissions appropriately for different use cases
5. Set up hooks for automation
6. Structure projects for optimal Claude Code usage
7. Make informed decisions about when to use interactive vs headless, agents vs raw prompts, etc.

## Quality Standards

- Every command/flag mentioned should be accurate (verify against docs)
- Include decision guidance: "Use X when... Use Y when..."
- Cover edge cases and gotchas
- Practical examples for common scenarios
- Quick reference patterns for frequently used operations

## After Completion

Create a `quick-reference.md` in the root with the most common commands and patterns — a cheat sheet for fast lookup.
