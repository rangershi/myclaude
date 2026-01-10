---
name: gemini
description: Execute Gemini backend via codeagent-wrapper for AI-powered code analysis and generation. Use when you need to leverage Google's Gemini models for UI/UX tasks and visual design.
---

# Gemini Backend Integration

> **Note**: This skill is accessed through `codeagent-wrapper --backend gemini`. For complete documentation including parallel execution and all features, see the [codeagent skill](../codeagent/SKILL.md).

## Overview

Execute Gemini backend commands via codeagent-wrapper. Integrates Google's Gemini AI models into Claude Code workflows for UI/UX focused development.

## When to Use

- UI component scaffolding and layout prototyping
- Design system implementation with style consistency
- Interactive element generation with accessibility support
- Visual design tasks and frontend development

## Usage

**Mandatory**: Run via codeagent-wrapper with fixed timeout 7200000ms (foreground):

```bash
codeagent-wrapper --backend gemini - [working_dir] <<'EOF'
<task content here>
EOF
```

**Simple tasks**:
```bash
codeagent-wrapper --backend gemini "simple task here" [working_dir]
```

**Resume a session**:
```bash
codeagent-wrapper --backend gemini resume <session_id> - <<'EOF'
<follow-up task>
EOF
```

## Environment Variables

- **CODEX_TIMEOUT**: Override timeout in milliseconds (default: 7200000 = 2 hours)

## Timeout Control

- **Fixed**: 7200000 milliseconds (2 hours)
- **Bash tool**: Always set `timeout: 7200000` for double protection

### Parameters

- `task` (required): Task prompt or question
- `working_dir` (optional): Working directory (default: current directory)

### Return Format

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

When calling via Bash tool, always include the timeout parameter:

```
Bash tool parameters:
- command: codeagent-wrapper --backend gemini - [working_dir] <<'EOF'
  <task content>
  EOF
- timeout: 7200000
- description: <brief description of the task>
```

### Examples

**Basic UI task:**
```bash
codeagent-wrapper --backend gemini - <<'EOF'
Create a responsive navigation component with mobile hamburger menu
EOF
# timeout: 7200000
```

**Design system implementation:**
```bash
codeagent-wrapper --backend gemini - <<'EOF'
Implement a button component following Material Design 3 guidelines:
- Primary, secondary, and tertiary variants
- Loading and disabled states
- Proper accessibility attributes
EOF
# timeout: 7200000
```

**With specific working directory:**
```bash
codeagent-wrapper --backend gemini - "/path/to/project" <<'EOF'
analyze @src/components and suggest UI improvements
EOF
# timeout: 7200000
```

**Dashboard layout:**
```bash
codeagent-wrapper --backend gemini - <<'EOF'
Create a responsive dashboard layout with:
- Sidebar navigation with collapsible sections
- Header with search and user profile
- Main content area with data visualization cards
- Mobile-first responsive design
EOF
# timeout: 7200000
```

## Parallel Execution

For parallel execution with multiple tasks, use `codeagent-wrapper --parallel` with per-task backend specification:

```bash
codeagent-wrapper --parallel <<'EOF'
---TASK---
id: ui_components_1732876800
backend: gemini
workdir: /home/user/project
---CONTENT---
Create reusable button and input components
---TASK---
id: ui_layout_1732876801
backend: gemini
workdir: /home/user/project
dependencies: ui_components_1732876800
---CONTENT---
Build dashboard layout using the new components
EOF
```

See [codeagent skill](../codeagent/SKILL.md) for complete parallel execution documentation.

## Critical Rules

**NEVER kill codeagent processes.** Long-running tasks (2-10 minutes) are normal. Instead:

1. Check task status via log file: `tail -f /tmp/claude/<workdir>/tasks/<task_id>.output`
2. Wait with timeout: `TaskOutput(task_id="<id>", block=true, timeout=300000)`

## Notes

- **Unified entry point**: Use `codeagent-wrapper --backend gemini` for all Gemini tasks
- Cross-platform compatible (Windows/macOS/Linux)
- Requires Gemini CLI installed and authenticated
- Best suited for UI/UX prototyping and visual design tasks
- Supports all Gemini model variants
