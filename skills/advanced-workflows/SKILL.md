---
name: advanced-workflows
description: "Advanced patterns for parallel sessions, complex tasks, and power user workflows. Use when scaling Claude usage, running multiple sessions, or optimizing development."
---

# Advanced Workflows

Patterns for power users: parallel sessions, complex planning, multi-step workflows, and scaling Claude Code usage.

## The Context Window Constraint

**Most best practices stem from one truth: Claude's context window fills up fast, and performance degrades as it fills.**

### Managing Context

| Strategy | When |
|----------|------|
| `/clear` | Between unrelated tasks |
| `/compact` | Long session, need to continue |
| Subagents | Exploration that fills context |
| New session | Task requires fresh start |

### Check Context Usage

```
/context
```

## Explore, Plan, Implement

### Phase 1: Explore (Plan Mode)

```bash
claude --permission-mode plan
```

```
Read /src/auth and understand sessions and login.
Look at how we handle environment variables.
```

### Phase 2: Plan

```
I want to add Google OAuth. What files need to change?
Create a detailed implementation plan.
```

Press `Ctrl+G` to edit plan in text editor.

### Phase 3: Implement (Normal Mode)

Press `Shift+Tab` to exit Plan Mode, then:

```
Implement the OAuth flow from your plan.
Write tests for the callback handler.
Run tests and fix any failures.
```

### Phase 4: Commit

```
Commit with a descriptive message and open a PR.
```

## Parallel Sessions

### With Git Worktrees

```bash
# Create isolated worktrees
git worktree add ../project-feature-a -b feature-a
git worktree add ../project-bugfix bugfix-123

# Run Claude in each
cd ../project-feature-a && claude
cd ../project-bugfix && claude

# Clean up
git worktree remove ../project-feature-a
```

### Writer/Reviewer Pattern

| Session A (Writer) | Session B (Reviewer) |
|-------------------|---------------------|
| Implement rate limiter | (wait) |
| (wait) | Review implementation for edge cases |
| Address feedback | (wait) |

### Test-First Pattern

| Session A (Tests) | Session B (Implementation) |
|------------------|---------------------------|
| Write failing tests | (wait) |
| (wait) | Implement to pass tests |
| Verify tests pass | (wait) |

## Subagent Strategies

### Keep Main Context Clean

```
Use a subagent to investigate how authentication works.
Summarize findings without reading all the files into main context.
```

### Parallel Investigation

```
Use subagents to analyze:
1. Authentication patterns
2. Database schema
3. API conventions
Report back with summaries.
```

### Post-Implementation Review

```
Use a subagent to review this code for edge cases.
```

## Session Management

### Name Sessions Early

```
/rename auth-refactor
```

Resume later:
```bash
claude --resume auth-refactor
```

### Fork for Experiments

```bash
claude --resume abc123 --fork-session
```

Preserves context, creates new branch.

### PR-Linked Sessions

Sessions auto-link when you create PRs. Resume with:
```bash
claude --from-pr 123
```

## Fan-Out Processing

### Batch Migration

```bash
# Generate file list
claude -p "list Python files needing migration" > files.txt

# Process each file
for file in $(cat files.txt); do
  claude -p "Migrate $file from Python 2 to 3" \
    --allowedTools "Read" "Edit"
done
```

### Parallel Processing

```bash
cat files.txt | xargs -P 4 -I {} \
  claude -p "Process {}" --allowedTools "Read" "Edit"
```

## Interview-Driven Development

Let Claude interview you for complex features:

```
I want to build [brief description]. Interview me using AskUserQuestion.

Ask about technical implementation, UI/UX, edge cases, and tradeoffs.
Don't ask obvious questions—dig into hard parts.

Keep interviewing until we've covered everything,
then write a complete spec to SPEC.md.
```

After spec complete, start fresh session to implement.

## Effective Prompting

### Be Specific

| Before | After |
|--------|-------|
| "add tests" | "write tests for foo.py covering logged-out edge case" |
| "fix bug" | "login fails after session timeout, check token refresh in src/auth/" |
| "add widget" | "follow pattern in HotDogWidget.php to create calendar widget" |

### Point to Patterns

```
Look at how existing widgets are implemented.
HotDogWidget.php is a good example.
Follow that pattern for the new calendar widget.
```

### Scope Investigations

```
Look through ExecutionFactory's git history
and summarize how its API evolved.
```

## Verification Strategies

### Always Provide Tests

```
Write validateEmail function.
Test cases:
- user@example.com → true
- invalid → false
- user@.com → false
Run tests after implementing.
```

### Visual Verification

Paste screenshot, then:
```
Implement this design.
Take a screenshot of result.
Compare to original and list differences.
Fix any discrepancies.
```

### Root Cause Analysis

```
Build fails with this error: [paste error]
Fix it and verify build succeeds.
Address root cause, don't suppress error.
```

## CLAUDE.md Best Practices

### Keep It Concise

For each line ask: *Would removing this cause mistakes?*

If not, delete it.

### Good Examples

```markdown
# Commands
- Build: `npm run build`
- Test: `npm test`

# Code Style
- Use ES modules, not CommonJS
- Destructure imports

# Workflow
- Typecheck after code changes
- Run single tests, not full suite
```

### Bad Examples

```markdown
# Don't include:
- Standard language conventions
- Documentation Claude can read
- Obvious practices like "write clean code"
- Long tutorials
```

## Course Correction

### Stop and Redirect

Press `Esc` to stop Claude, then redirect:
```
Actually, use TypeScript instead of JavaScript.
```

### Rewind

Double-tap `Esc` or `/rewind` to restore previous state.

### Fresh Start

After 2+ corrections on same issue:
```
/clear
```

Then write better initial prompt incorporating lessons learned.

## Integration Patterns

### As Unix Utility

```bash
# In build script
cat error.log | claude -p "explain root cause" > diagnosis.txt

# As linter
claude -p "check for typos vs main" --allowedTools "Read" "Bash(git *)"
```

### In CI/CD

```bash
git diff origin/main | claude -p "review for issues" \
  --allowedTools "Read" \
  --output-format text
```

## Cost Optimization

### Use Haiku for Simple Tasks

```bash
claude --model haiku -p "quick analysis"
```

### Limit Turns

```bash
claude -p "investigate" --max-turns 5
```

### Budget Limits

```bash
claude -p "complex task" --max-budget-usd 10.00
```

## Troubleshooting Sessions

### Kitchen Sink Session

**Problem**: Too much unrelated content in context

**Fix**: `/clear` between unrelated tasks

### Repeated Corrections

**Problem**: Context polluted with failed attempts

**Fix**: After 2 failures, `/clear` and write better prompt

### Over-Specified CLAUDE.md

**Problem**: Claude ignores instructions

**Fix**: Prune ruthlessly, convert to skills

### Infinite Exploration

**Problem**: Claude reads too many files

**Fix**: Scope narrowly or use subagents

## Decision Guide

| Situation | Approach |
|-----------|----------|
| Complex feature | Interview → Spec → Fresh session |
| Parallel work | Git worktrees + multiple sessions |
| Code review | Separate reviewer session |
| Large migration | Fan-out batch processing |
| Context filling up | Subagents for exploration |
| Task unclear | Plan Mode first |

## References

- [Workflow Examples](references/workflow-examples.md)
- [Optimization Strategies](references/optimization.md)
