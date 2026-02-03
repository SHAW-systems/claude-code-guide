# Environment Variables Reference

## Authentication

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for Claude SDK |
| `ANTHROPIC_AUTH_TOKEN` | Custom Authorization header value |
| `ANTHROPIC_CUSTOM_HEADERS` | Custom request headers |
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry authentication |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry resource URL |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock API key |

## Model Configuration

| Variable | Description | Example |
|----------|-------------|---------|
| `ANTHROPIC_MODEL` | Model to use | `claude-sonnet-4-5` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Default Haiku model | `claude-3-5-haiku-latest` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Default Sonnet model | `claude-3-5-sonnet-latest` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Default Opus model | `claude-opus-4-1-latest` |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model for subagents | |
| `MAX_THINKING_TOKENS` | Extended thinking budget | `10000` or `0` to disable |

## Behavior Control

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_SHELL` | Override shell detection |
| `CLAUDE_CODE_SHELL_PREFIX` | Command wrapper prefix |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Return to original dir after bash |
| `BASH_DEFAULT_TIMEOUT_MS` | Default bash timeout |
| `BASH_MAX_TIMEOUT_MS` | Maximum bash timeout |
| `BASH_MAX_OUTPUT_LENGTH` | Max bash output characters |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Exit delay in ms |
| `CLAUDE_CODE_TASK_LIST_ID` | Share task list across sessions |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable background tasks (`1`) |

## Output & Display

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | Hide email/org from UI |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Disable terminal title updates |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Enable prompt suggestions |
| `CLAUDE_CODE_ENABLE_TASKS` | Use task tracking system |
| `IS_DEMO` | Enable demo mode |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Context capacity threshold (1-100) |

## File & Directory

| Variable | Description |
|----------|-------------|
| `CLAUDE_CONFIG_DIR` | Custom config directory |
| `CLAUDE_CODE_TMPDIR` | Custom temp directory |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | File read token limit |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens (default: 32000, max: 64000) |

## Provider Configuration

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Skip Bedrock auth |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Skip Foundry auth |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Skip Vertex auth |
| `VERTEX_REGION_*` | Vertex region overrides |

## Monitoring & Telemetry

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Enable OpenTelemetry |
| `DISABLE_TELEMETRY` | Disable Statsig telemetry |
| `DISABLE_ERROR_REPORTING` | Disable Sentry error reporting |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Disable session quality surveys |

## Features

| Variable | Description |
|----------|-------------|
| `ENABLE_TOOL_SEARCH` | MCP tool search: `auto`, `auto:5`, `true`, `false` |
| `DISABLE_PROMPT_CACHING` | Disable all prompt caching |
| `DISABLE_PROMPT_CACHING_HAIKU` | Disable caching for Haiku |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | Disable flavor text |
| `DISABLE_COST_WARNINGS` | Disable cost warnings |
| `MAX_MCP_OUTPUT_TOKENS` | Max MCP response tokens (default: 25000) |
| `MCP_TIMEOUT` | MCP server startup timeout |

## Updates & Installation

| Variable | Description |
|----------|-------------|
| `DISABLE_AUTOUPDATER` | Disable auto-updates |
| `FORCE_AUTOUPDATE_PLUGINS` | Force plugin auto-updates |
| `DISABLE_INSTALLATION_CHECKS` | Disable installation warnings |
| `DISABLE_BUG_COMMAND` | Disable `/bug` command |

## Network & Security

| Variable | Description |
|----------|-------------|
| `HTTP_PROXY` | HTTP proxy server |
| `HTTPS_PROXY` | HTTPS proxy server |
| `NO_PROXY` | Domains/IPs to bypass proxy |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Proxy handles DNS |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS certificate path |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS private key path |

## Miscellaneous

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Credential refresh interval |
| `USE_BUILTIN_RIPGREP` | Use built-in ripgrep |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Skill metadata char limit (default: 15000) |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | Skip IDE extension auto-install |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | Load CLAUDE.md from additional dirs |
