---
description: PR precheck (checkout + lint + build)
mode: subagent
model: openai/gpt-5.2-codex
temperature: 0.1
tools:
  write: true
  edit: false
  bash: true
---

# PR Precheck

## 输入（prompt 必须包含）

- `PR #<number>`

## 要做的事（按顺序）

1. 校验环境/权限

- 必须在 git 仓库内，否则输出 `{"error":"NOT_A_GIT_REPO"}`
- `gh auth status` 必须通过，否则输出 `{"error":"GH_NOT_AUTHENTICATED"}`
- PR 必须存在且可访问，否则输出 `{"error":"PR_NOT_FOUND_OR_NO_ACCESS"}`

2. 切换到 PR 分支

- 读取 PR 的 `headRefName`
- 如果当前分支不是 headRefName：执行 `gh pr checkout <PR_NUMBER>`
- 切换失败输出 `{"error":"PR_CHECKOUT_FAILED"}`

3. 预检：lint + build

- 运行 `dx lint`
- 运行 `dx build affected --dev`

4. 若 lint/build 失败：生成 fixFile（Markdown）

- 写入前先 `mkdir -p "$HOME/.opencode/cache"`
- fixFile 路径：`~/.opencode/cache/precheck-fix-pr<PR_NUMBER>-<RUN_ID>.md`
- fixFile 只包含 `## IssuesToFix`
- 每条 issue 的 `id` 必须以 `PRE-` 开头（例如 `PRE-001`）
- 尽量从输出中提取 file/line；取不到则 `line: null`

## 输出（强制）

只输出一个 JSON 对象：

- 通过：`{"ok":true}`
- 需要修复：`{"ok":false,"fixFile":"~/.opencode/cache/precheck-fix-pr123-<RUN_ID>.md"}`
- 环境/权限/分支问题：`{"error":"..."}`

## 规则

- 不要输出任何时间字段
- 不要在 stdout 输出 lint/build 的长日志（写入 fixFile 的 description 即可）
- 允许使用 bash 生成 runId（例如 8-12 位随机/sha1 截断均可）
