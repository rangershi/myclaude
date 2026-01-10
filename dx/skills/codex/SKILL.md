---
name: codex
description: Execute Codex backend via codeagent-wrapper for code analysis, refactoring, and automated code changes. Use when you need to delegate complex code tasks to Codex AI with file references (@syntax) and structured output.
---

# Codex Backend Integration

> **Note**: This skill is accessed through `codeagent-wrapper --backend codex`. For complete documentation including parallel execution and all features, see the [codeagent skill](../codeagent/SKILL.md).

## Overview

Execute Codex backend commands via codeagent-wrapper and parse structured JSON responses. Supports file references via `@` syntax, multiple models, and sandbox controls.

## When to Use

- Complex code analysis requiring deep understanding
- Large-scale refactoring across multiple files
- Automated code generation with safety controls
- Deep reasoning tasks and algorithm optimization

## Usage

**Mandatory**: Run every automated invocation through the Bash tool in the foreground with **HEREDOC syntax** to avoid shell quoting issues, keeping the `timeout` parameter fixed at `7200000` milliseconds.

```bash
codeagent-wrapper --backend codex - [working_dir] <<'EOF'
<task content here>
EOF
```

**Why HEREDOC?** Tasks often contain code blocks, nested quotes, shell metacharacters (`$`, `` ` ``, `\`), and multiline text. HEREDOC (Here Document) syntax passes these safely without shell interpretation, eliminating quote-escaping nightmares.

**Simple tasks** (backward compatibility):
For simple single-line tasks without special characters, you can still use direct quoting:
```bash
codeagent-wrapper --backend codex "simple task here" [working_dir]
```

**Resume a session with HEREDOC:**
```bash
codeagent-wrapper --backend codex resume <session_id> - [working_dir] <<'EOF'
<task content>
EOF
```

## Environment Variables

- **CODEX_TIMEOUT**: Override timeout in milliseconds (default: 7200000 = 2 hours)
  - Example: `export CODEX_TIMEOUT=3600000` for 1 hour

## Timeout Control

- **Built-in**: Binary enforces 2-hour timeout by default
- **Override**: Set `CODEX_TIMEOUT` environment variable (in milliseconds)
- **Behavior**: On timeout, sends SIGTERM, then SIGKILL after 5s if process doesn't exit
- **Exit code**: Returns 124 on timeout (consistent with GNU timeout)
- **Bash tool**: Always set `timeout: 7200000` parameter for double protection

### Parameters

- `task` (required): Task description, supports `@file` references
- `working_dir` (optional): Working directory (default: current)

### Return Format

Extracts `agent_message` from Codex JSON stream and appends session ID:
```
Agent response text here...

---
SESSION_ID: 019a7247-ac9d-71f3-89e2-a823dbd8fd14
```

Error format (stderr):
```
ERROR: Error message
```

### Invocation Pattern

All automated executions must use HEREDOC syntax through the Bash tool in the foreground, with `timeout` fixed at `7200000` (non-negotiable):

```
Bash tool parameters:
- command: codeagent-wrapper --backend codex - [working_dir] <<'EOF'
  <task content>
  EOF
- timeout: 7200000
- description: <brief description of the task>
```

### Examples

**Basic code analysis:**
```bash
# Recommended: with HEREDOC (handles any special characters)
codeagent-wrapper --backend codex - <<'EOF'
explain @src/main.ts
EOF
# timeout: 7200000

# Alternative: simple direct quoting (if task is simple)
codeagent-wrapper --backend codex "explain @src/main.ts"
```

**Refactoring with multiline instructions:**
```bash
codeagent-wrapper --backend codex - <<'EOF'
refactor @src/utils for performance:
- Extract duplicate code into helpers
- Use memoization for expensive calculations
- Add inline comments for non-obvious logic
EOF
# timeout: 7200000
```

**Multi-file analysis:**
```bash
codeagent-wrapper --backend codex - "/path/to/project" <<'EOF'
analyze @. and find security issues:
1. Check for SQL injection vulnerabilities
2. Identify XSS risks in templates
3. Review authentication/authorization logic
4. Flag hardcoded credentials or secrets
EOF
# timeout: 7200000
```

**Resume previous session:**
```bash
# First session
codeagent-wrapper --backend codex - <<'EOF'
add comments to @utils.js explaining the caching logic
EOF
# Output includes: SESSION_ID: 019a7247-ac9d-71f3-89e2-a823dbd8fd14

# Continue the conversation with more context
codeagent-wrapper --backend codex resume 019a7247-ac9d-71f3-89e2-a823dbd8fd14 - <<'EOF'
now add TypeScript type hints and handle edge cases where cache is null
EOF
# timeout: 7200000
```

## Parallel Execution

For parallel execution with multiple tasks, use `codeagent-wrapper --parallel` with per-task backend specification:

```bash
codeagent-wrapper --parallel <<'EOF'
---TASK---
id: analyze_1732876800
backend: codex
workdir: /home/user/project
---CONTENT---
analyze @spec.md and summarize API requirements
---TASK---
id: implement_1732876801
backend: codex
workdir: /home/user/project
dependencies: analyze_1732876800
---CONTENT---
implement features from analyze_1732876800 summary
EOF
```

See [codeagent skill](../codeagent/SKILL.md) for complete parallel execution documentation.

## Critical Rules

**NEVER kill codeagent processes.** Long-running tasks (2-10 minutes) are normal. Instead:

1. Check task status via log file: `tail -f /tmp/claude/<workdir>/tasks/<task_id>.output`
2. Wait with timeout: `TaskOutput(task_id="<id>", block=true, timeout=300000)`

## Notes

- **Unified entry point**: Use `codeagent-wrapper --backend codex` for all Codex tasks
- **Binary distribution**: Single Go binary, zero dependencies
- **Cross-platform compatible**: Linux (amd64/arm64), macOS (amd64/arm64)
- Uses `--skip-git-repo-check` to work in any directory
- Streams progress, returns only final agent message
- Every execution returns a session ID for resuming conversations
- Requires Codex CLI installed and authenticated
