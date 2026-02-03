---
name: permissions-and-security
description: "Permission modes, tool restrictions, sandboxing, and security settings. Use when configuring what Claude can access, controlling permissions, or setting up secure environments."
---

# Permissions and Security

Claude Code uses a tiered permission system to balance power and safety.

## Permission Tiers

| Tool Type | Example | Approval Required | "Don't ask again" Behavior |
|-----------|---------|-------------------|---------------------------|
| Read-only | File reads, Grep | No | N/A |
| Bash commands | Shell execution | Yes | Per project directory and command |
| File modification | Edit/write files | Yes | Until session end |

## Permission Modes

Set via `/permissions`, `--permission-mode`, or settings:

| Mode | Description |
|------|-------------|
| `default` | Standard prompts for first use |
| `acceptEdits` | Auto-accept file edit permissions |
| `plan` | Read-only, no modifications |
| `dontAsk` | Auto-deny unless pre-approved |
| `bypassPermissions` | Skip all checks (isolated environments only) |

### Switching Modes

**During session**: Press `Shift+Tab` to cycle modes

**At startup**:
```bash
claude --permission-mode plan
```

**In settings**:
```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

## Permission Rules

Rules follow: **deny → ask → allow** (first match wins)

### Rule Syntax

| Pattern | Effect |
|---------|--------|
| `Bash` | Matches all Bash commands |
| `Bash(npm run build)` | Matches exact command |
| `Bash(npm run *)` | Prefix match with wildcard |
| `Read(./.env)` | Specific file |
| `Read(./secrets/**)` | Directory pattern |
| `WebFetch(domain:example.com)` | Domain restriction |
| `mcp__server__tool` | MCP tool |

### Configure in Settings

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Read"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ]
  }
}
```

### CLI Flags

```bash
# Auto-approve specific tools
claude --allowedTools "Bash(git log *)" "Read"

# Block specific tools
claude --disallowedTools "Write" "Edit"
```

## File Path Patterns

Read/Edit rules use gitignore-style patterns:

| Pattern | Meaning | Example |
|---------|---------|---------|
| `//path` | Absolute from root | `Read(//Users/alice/secrets/**)` |
| `~/path` | From home directory | `Read(~/Documents/*.pdf)` |
| `/path` | Relative to settings file | `Edit(/src/**/*.ts)` |
| `path` | Relative to cwd | `Read(*.env)` |

## Sandboxing

Enable OS-level isolation for Bash commands:

```bash
/sandbox
```

### Sandbox Configuration

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "allowLocalBinding": true
    }
  }
}
```

### Sandbox + Permissions

- **Permissions**: Control which tools Claude can use
- **Sandbox**: OS-level enforcement for Bash only

Use both for defense-in-depth.

## Managing Permissions

### View Current Permissions

```
/permissions
```

### Working Directories

Add directories Claude can access:

**At startup**:
```bash
claude --add-dir ../docs ../lib
```

**During session**:
```
/add-dir ../shared
```

**In settings**:
```json
{
  "permissions": {
    "additionalDirectories": ["../docs/"]
  }
}
```

## Managed Settings (Enterprise)

IT admins can deploy organization-wide policies via `managed-settings.json`:

| Platform | Location |
|----------|----------|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux/WSL | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-settings.json` |

### Managed-Only Settings

| Setting | Description |
|---------|-------------|
| `disableBypassPermissionsMode` | Prevent bypass mode |
| `allowManagedPermissionRulesOnly` | Only managed rules apply |
| `allowManagedHooksOnly` | Only managed hooks allowed |

## Security Best Practices

### For Interactive Use

1. Start with default mode
2. Allowlist specific safe commands in project settings
3. Use Plan Mode for exploration
4. Review changes before approving

### For Automation

1. Use `--allowedTools` to explicitly permit needed tools
2. Enable sandboxing for untrusted codebases
3. Set `--max-budget-usd` to limit spending
4. Use `--max-turns` to limit agentic loops

### For CI/CD

```bash
claude -p "task" \
  --allowedTools "Bash(npm run *)" "Read" "Edit" \
  --output-format json
```

## Decision Guide

| Scenario | Recommendation |
|----------|----------------|
| Exploring new codebase | Plan Mode (`--permission-mode plan`) |
| Trusted project, productivity | `acceptEdits` mode |
| Untrusted/shared codebase | Sandboxing + deny rules |
| CI/CD automation | Explicit `--allowedTools` |
| Full autonomy (isolated) | `--dangerously-skip-permissions` in container |

## References

- [Permission Rule Syntax](references/permission-rules.md)
- [Sandbox Configuration](references/sandbox-config.md)
