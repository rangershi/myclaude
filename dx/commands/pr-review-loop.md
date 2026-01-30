---
allowed-tools: [Bash, Read, Glob, TodoWrite, Edit, Grep, Task]
description: 'Run multi-agent PR review + auto-fix loop with GitHub comments'
agent: sisyphus
---

# PR Review Loop

## 输入

- `{{PR_NUMBER}}`

## 固定 subagent_type（直接用 Task 调用，不要反复确认）

- `pr-precheck`
- `pr-context`
- `codex-reviewer`
- `claude-reviewer`
- `gemini-reviewer`
- `pr-review-aggregate`
- `pr-fix`

## 循环前

0. Task: `pr-precheck`

- prompt 必须包含：`PR #{{PR_NUMBER}}`
- 若返回 `{"error":"..."}`：立即终止
- 若返回 `{"ok":false,"fixFile":"..."}`：调用一次 `pr-fix` 修复，然后再调用一次 `pr-precheck`，必须拿到 `{"ok":true}` 才进入循环

## 循环（最多 3 轮）

每轮按顺序执行：

1. Task: `pr-context`

- prompt 必须包含：`PR #{{PR_NUMBER}}`、`round: <ROUND>`
- 若返回 `{"error":"..."}`：立即终止本轮并回传错误（不再调用 reviewers）
- 取出：`contextFile`、`runId`、`headOid`（如有）

2. Task（并行）: `codex-reviewer` + `claude-reviewer` + `gemini-reviewer`

- 每个 reviewer prompt 必须包含：
  - `PR #{{PR_NUMBER}}`
  - `round: <ROUND>`
  - `contextFile: <path>`
- reviewer 默认读 `contextFile`；必要时允许用 `git/gh` 只读命令拿 diff

- 每个 reviewer 输出：`reviewFile: <path>`（Markdown）

3. Task: `pr-review-aggregate`

- prompt 必须包含：`PR #{{PR_NUMBER}}`、`round: <ROUND>`、`runId: <RUN_ID>`、`contextFile: <path>`、三条 `reviewFile: <path>`
- 输出：`{"stop":true}` 或 `{"stop":false,"fixFile":"..."}`
- 若 `stop=true`：本轮结束并退出循环

4. Task: `pr-fix`

- prompt 必须包含：
  - `PR #{{PR_NUMBER}}`
  - `round: <ROUND>`
  - `fixFile: <path>`
- 约定：`pr-fix` 对每个 findingId 单独 commit + push（一个 findingId 一个 commit），结束后再 `git push` 兜底

- pr-fix 输出：`fixReportFile: <path>`（Markdown）

5. Task: `pr-review-aggregate`（发布修复评论）

- prompt 必须包含：`PR #{{PR_NUMBER}}`、`round: <ROUND>`、`runId: <RUN_ID>`、`fixReportFile: <path>`
- 输出：`{"ok":true}`

6. 下一轮

- 回到 1
