---
name: headless-automation
description: "Non-interactive and programmatic Claude Code usage for CI/CD, scripts, and automation. Use when running Claude in pipelines, batch processing, or integrating with other tools."
---

# Headless Automation

Run Claude Code non-interactively for automation, CI/CD, and scripting.

## Basic Headless Usage

### Print Mode (-p)

```bash
claude -p "explain this project"
```

- Runs query, prints result, exits
- No interactive prompts
- Returns text by default

### Output Formats

| Format | Flag | Use Case |
|--------|------|----------|
| Plain text | `--output-format text` | Simple integrations |
| JSON | `--output-format json` | Parsing with jq |
| Streaming JSON | `--output-format stream-json` | Real-time processing |

```bash
# JSON with metadata
claude -p "list functions" --output-format json

# Streaming for real-time
claude -p "analyze logs" --output-format stream-json
```

### Structured Output (JSON Schema)

```bash
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{
    "type": "object",
    "properties": {
      "functions": {"type": "array", "items": {"type": "string"}}
    }
  }'
```

## Piping Data

### Pipe In

```bash
cat error.log | claude -p "explain these errors"
git diff | claude -p "review these changes"
```

### Pipe Out

```bash
claude -p "generate config" > config.json
claude -p "list issues" | grep "TODO"
```

### Chain Commands

```bash
cat data.json | claude -p "transform" --output-format json | jq '.result'
```

## Permission Control

### Allow Specific Tools

```bash
claude -p "build and test" \
  --allowedTools "Bash(npm run *)" "Read" "Edit"
```

### Block Tools

```bash
claude -p "analyze" --disallowedTools "Write" "Bash"
```

### Skip All Permissions

```bash
claude -p "fix lint" --dangerously-skip-permissions
```

> **Warning**: Only use in isolated containers without network access.

### Permission Modes

```bash
# Read-only exploration
claude -p "analyze architecture" --permission-mode plan

# Auto-accept edits
claude -p "fix typos" --permission-mode acceptEdits
```

## Resource Limits

### Token/Turn Limits

```bash
# Limit agentic turns
claude -p "investigate" --max-turns 5

# Budget limit
claude -p "refactor" --max-budget-usd 2.00
```

### Timeout

```bash
timeout 300 claude -p "complex task"
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: curl -fsSL https://claude.ai/install.sh | bash

      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          git diff origin/main | claude -p "review these changes for issues" \
            --allowedTools "Read" \
            --output-format text > review.md

      - name: Post Review
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: review
            });
```

### GitLab CI

```yaml
code-review:
  image: ubuntu:latest
  script:
    - curl -fsSL https://claude.ai/install.sh | bash
    - git diff $CI_MERGE_REQUEST_DIFF_BASE_SHA | claude -p "review" --output-format text
  only:
    - merge_requests
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

staged=$(git diff --cached --name-only)
if [ -n "$staged" ]; then
  claude -p "Check these files for issues: $staged" \
    --allowedTools "Read" \
    --output-format text
fi
```

## Scripting Patterns

### Build Verification

```bash
#!/bin/bash
claude -p "Run the build and fix any errors" \
  --allowedTools "Bash(npm run *)" "Read" "Edit" \
  --max-turns 10
```

### Code Generation

```bash
#!/bin/bash
# Generate API client from OpenAPI spec
claude -p "Generate TypeScript API client from openapi.yaml" \
  --allowedTools "Read" "Write" \
  --output-format text > api-client.ts
```

### Batch Processing

```bash
#!/bin/bash
for file in src/*.js; do
  claude -p "Add JSDoc comments to $file" \
    --allowedTools "Read" "Edit" \
    --max-turns 3
done
```

### Migration Script

```bash
#!/bin/bash
# Migrate files in parallel
cat files-to-migrate.txt | xargs -P 4 -I {} \
  claude -p "Migrate {} from CommonJS to ES modules" \
    --allowedTools "Read" "Edit"
```

## Linting Integration

### As Build Script

```json
{
  "scripts": {
    "lint:claude": "claude -p 'review changes vs main for issues'"
  }
}
```

### As Linter

```bash
claude -p "You are a linter. Check for issues in src/. \
  Report format: filename:line: description" \
  --allowedTools "Read" "Glob"
```

## Resume in Automation

### Continue Session

```bash
# Continue most recent
claude -c -p "now add tests"

# Resume named session
claude --resume "feature-work" -p "continue implementation"
```

### Session Persistence

```bash
# Disable session saving
claude -p "one-off task" --no-session-persistence
```

## Error Handling

### Check Exit Codes

```bash
if claude -p "run tests" --allowedTools "Bash(npm test)"; then
  echo "Tests passed"
else
  echo "Tests failed"
  exit 1
fi
```

### Capture Output

```bash
result=$(claude -p "analyze" --output-format json 2>&1)
if echo "$result" | jq -e '.error' > /dev/null; then
  echo "Error: $(echo "$result" | jq -r '.error')"
  exit 1
fi
```

## Agent SDK

For programmatic control, use the Agent SDK:

```typescript
import { Claude } from "@anthropic-ai/claude-code";

const claude = new Claude();
const session = await claude.startSession();

const result = await session.query({
  prompt: "Analyze this codebase",
  allowedTools: ["Read", "Glob", "Grep"],
  outputFormat: "json"
});

console.log(result);
await session.close();
```

## Best Practices

### For CI/CD

1. Use explicit `--allowedTools` to whitelist needed tools
2. Set `--max-turns` to prevent infinite loops
3. Use `--max-budget-usd` for cost control
4. Capture output in JSON for parsing
5. Handle timeouts explicitly

### For Scripts

1. Test commands interactively first
2. Use `--output-format json` for reliable parsing
3. Set appropriate timeouts
4. Log stderr for debugging
5. Use retry logic for transient failures

## Decision Guide

| Use Case | Approach |
|----------|----------|
| Simple query | `claude -p "query"` |
| Parse output | `--output-format json | jq` |
| Strict output | `--json-schema` |
| CI code review | `git diff | claude -p "review"` |
| Full automation | `--dangerously-skip-permissions` in container |
| Continue work | `claude -c -p "next step"` |

## References

- [Output Format Details](references/output-formats.md)
- [Agent SDK Documentation](references/agent-sdk.md)
