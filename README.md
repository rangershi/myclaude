# DX - Developer Experience Toolkit

团队内部使用的开发者体验工具集，专注于 Git 工作流自动化和多智能体 PR 评审。

## 功能特性

- **Git 工作流自动化**: Issue 创建、代码提交、PR 创建一站式流程
- **智能版本发布**: 从 release 分支自动生成 changelog 并发布版本
- **多智能体 PR 评审**: 并行使用 Claude、Codex、Gemini 进行代码评审
- **自动修复循环**: PR 评审问题自动修复并推送

## 快速开始

### 1. 环境检测

首次使用前，运行环境诊断命令检测依赖是否已安装：

```bash
/dx:doctor
```

该命令会检测必要的工具是否已安装，如未安装会提示安装方法。

### 2. 安装插件

#### 方式一：通过 Claude Code 安装（推荐）

```bash
# 在 Claude Code 中运行
claude /plugin add https://github.com/rangershi/mydx
```

或在 Claude Code 交互模式中：
```bash
/plugin add https://github.com/rangershi/mydx
```

安装完成后，插件会自动加载，无需额外配置。

#### 方式二：手动安装

```bash
# 克隆仓库
git clone https://github.com/rangershi/mydx.git

# 在 Claude Code 中加载插件
claude --plugin-dir /path/to/mydx
```

#### 方式三：项目级安装

将插件克隆到项目的 `.claude-plugin/` 目录下，Claude Code 会自动识别并加载：

```bash
cd your-project
git clone https://github.com/rangershi/mydx.git .claude-plugin/mydx
```

## 命令详解

### `/dx:doctor` - 环境诊断

检测并安装所需的依赖工具。

```bash
/dx:doctor
```

**检测内容**:
- `opencode` CLI 工具
- `dx` CLI 工具
- `agent-browser` 工具
- `AGENTS.md` 配置文件
- `opencode.json` 配置文件
- OpenCode 插件安装状态

### `/dx:git-commit-and-pr` - Git 工作流自动化

自动化的 Issue/Commit/PR 创建流程。

```bash
# 自动检测并执行所需阶段
/dx:git-commit-and-pr

# 指定关联 Issue
/dx:git-commit-and-pr --issue <ID>

# 仅创建 Issue
/dx:git-commit-and-pr --issue-only

# 仅创建 PR
/dx:git-commit-and-pr --pr --base <BRANCH>
```

**执行流程**:

1. **Issue 创建**（可选）
   - 从对话历史提取问题描述
   - 分析代码变更
   - 生成结构化 Issue 内容（背景、问题、期望、计划、影响范围）
   - 自动添加合适的标签

2. **Commit 流程**
   - 暂存所有变更
   - 分析变更内容
   - 生成符合规范的 commit message
   - 自动关联 Issue

3. **PR 创建**
   - 推送分支到远程
   - 分析与主分支的差异
   - 生成 PR 描述
   - 自动关联 Issue

**输出示例**:
```
✅ 完成

Issue: #123 [Backend] 优化用户认证流程
Commit: a1b2c3d feat: add JWT authentication
PR: #45 → https://github.com/owner/repo/pull/45

💡 下一步：运行以下命令启动自动评审
/dx:pr-review-loop --pr 45
```

### `/dx:git-release` - 智能版本发布

从 release 分支自动生成 changelog 并创建 GitHub Release。

```bash
# 必须在 release/vX.Y.Z 或 release/vX.Y.Z-<prerelease>.N 分支上执行
/dx:git-release
```

**前置条件**:
- 必须在 `release/vX.Y.Z` 或 `release/vX.Y.Z-<prerelease>.N` 格式的分支上
- 工作目录必须干净（无未提交的修改）
- 版本号从分支名自动提取

**支持的版本格式**:
- 正式版本: `release/v0.1.10`、`release/v1.0.0`
- Beta 版本: `release/v0.2.6-beta.5`
- Alpha 版本: `release/v1.0.0-alpha.3`
- RC 版本: `release/v1.0.0-rc.2`

**执行流程**:

1. **状态检查**
   - 验证分支名格式
   - 检查工作目录状态
   - 提取并确认版本号
   - 检查版本号冲突

2. **更新版本号**
   - 更新所有 `package.json` 文件的 version 字段
   - 提交版本号变更

3. **收集变更信息**
   - 从 GitHub Releases 获取上一个发布版本
   - 提取 PR 信息（标题、标签、描述）
   - 分析提交记录
   - 识别运维提醒事项

4. **生成发行说明**
   - 发布摘要（3-5 条核心变更）
   - 分类变更（新增、优化、修复、技术改进）
   - 运维提醒（环境变量、部署步骤、依赖更新）
   - 引用（PRs、Issues、提交数）
   - 升级指南

5. **创建 Release**
   - 创建 Git annotated tag
   - 推送 tag 到远程
   - 创建 GitHub Release

**示例**:

```bash
# 1. 创建 release 分支
git checkout -b release/v0.1.10

# 2. 执行发布
/dx:git-release

# 输出:
📊 发布前状态检查...
✅ 工作目录干净
✅ 当前分支: release/v0.1.10
✅ 版本号: v0.1.10

🔄 更新版本号...
✅ 已更新 4 个 package.json 文件

📝 变更分析...
发现 15 commits, 8 PRs

📄 生成发行说明...
[预览发行说明]

🎉 v0.1.10 发布成功!
URL: https://github.com/owner/repo/releases/tag/v0.1.10
```

### `/dx:pr-review-loop` - 多智能体 PR 评审循环

使用多个 AI 模型（Claude、Codex、Gemini）并行评审 PR，自动修复问题并推送。

```bash
# 默认评审最近的 PR
/dx:pr-review-loop

# 指定 PR 编号
/dx:pr-review-loop --pr 123
```

**架构设计**:

采用 **Multi-Agent Orchestration** 模式，包含以下 Agent：

| Agent | 职责 |
|-------|------|
| `pr-precheck` | PR 前置检查（CI 状态、分支验证） |
| `pr-context` | 提取 PR 上下文信息 |
| `codex-reviewer` | 使用 Codex 进行代码评审 |
| `claude-reviewer` | 使用 Claude 进行代码评审 |
| `gemini-reviewer` | 使用 Gemini 进行代码评审 |
| `pr-review-aggregate` | 聚合评审结果并决策 |
| `pr-fix` | 自动修复评审发现的问题 |

**执行流程**:

```
┌─────────────────┐
│  pr-precheck    │  检查 CI、分支状态
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   循环 (最多3轮) │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│  pr-context     │  提取 PR 上下文
└────────┬────────┘
         │
         ▼
┌──────────────────────────────────┐
│  并行评审                         │
│  ┌────────────┐                  │
│  │ codex      │                  │
│  └────────────┘                  │
│  ┌────────────┐                  │
│  │ claude     │                  │
│  └────────────┘                  │
│  ┌────────────┐                  │
│  │ gemini     │                  │
│  └────────────┘                  │
└────────┬─────────────────────────┘
         │
         ▼
┌─────────────────┐
│ pr-review-      │  决策：通过 or 需修复
│ aggregate       │
└────────┬────────┘
         │
         ├─── 通过 ──→ 结束
         │
         └─── 需修复 ──→ pr-fix ──→ 下一轮
```

**特性**:

1. **并行评审**: 三个 AI 模型同时评审，获取多视角反馈
2. **智能聚合**: 自动去重、分类、优先级排序评审意见
3. **自动修复**: 每个问题单独 commit 并推送
4. **循环迭代**: 最多 3 轮评审-修复循环
5. **质量门控**: 所有 reviewer 都通过才结束

**输出示例**:

```
🔍 PR #123 评审开始

Round 1:
  ✅ 前置检查通过
  📝 提取上下文完成
  🤖 Codex: 发现 3 个问题
  🤖 Claude: 发现 2 个问题
  🤖 Gemini: 发现 1 个问题
  📊 聚合评审: 需修复 4 个问题
  🔧 自动修复完成 (4 commits pushed)

Round 2:
  📝 提取上下文完成
  🤖 Codex: ✅ 通过
  🤖 Claude: 发现 1 个问题
  🤖 Gemini: ✅ 通过
  📊 聚合评审: 需修复 1 个问题
  🔧 自动修复完成 (1 commit pushed)

Round 3:
  📝 提取上下文完成
  🤖 Codex: ✅ 通过
  🤖 Claude: ✅ 通过
  🤖 Gemini: ✅ 通过
  📊 聚合评审: ✅ 全部通过

✅ PR #123 评审完成
```

## 工作流推荐

### 日常开发流程

```bash
# 1. 创建 Issue 并提交代码
/dx:git-commit-and-pr

# 2. 启动 PR 评审
/dx:pr-review-loop --pr <PR_NUMBER>

# 3. 评审通过后合并 PR
# （手动操作或通过 CI/CD）
```

### 版本发布流程

```bash
# 1. 创建 release 分支
git checkout -b release/v1.2.3

# 2. 执行发布
/dx:git-release

# 3. 合并 release 分支到 main
git checkout main
git merge release/v1.2.3

# 4. 删除 release 分支
git branch -d release/v1.2.3
git push origin --delete release/v1.2.3
```

## 目录结构

```
mydx/
├── .claude-plugin/
│   └── marketplace.json    # 插件配置
├── dx/
│   ├── commands/           # 命令定义（统一 /dx:* 前缀）
│   │   ├── doctor.md
│   │   ├── git-commit-and-pr.md
│   │   ├── git-release.md
│   │   └── pr-review-loop.md
│   └── agents/             # Agent 定义
│       ├── claude-reviewer.md
│       ├── codex-reviewer.md
│       ├── gemini-reviewer.md
│       ├── pr-context.md
│       ├── pr-fix.md
│       ├── pr-precheck.md
│       └── pr-review-aggregate.md
├── opencode.json           # OpenCode 配置
├── AGENTS.md               # 项目指令入口
└── README.md
```

## 配置文件

### opencode.json

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "ruler/**/*.md"
  ]
}
```

### AGENTS.md

```markdown
# AGENTS.md

OpenCode 项目指令入口文件
```

## 版本历史

### v2.0.0 - 2026-01-30

**重大重构**:
- 移除复杂的 BMAD、Requirements-Driven、Feature-Dev 工作流
- 移除所有 Skills (codex-cli, gemini-cli, omo, product-requirements, agent-browser)
- 专注于核心功能：Git 工作流 + PR 评审

**保留功能**:
- ✅ Git 工作流自动化 (`/dx:git-commit-and-pr`)
- ✅ 智能版本发布 (`/dx:git-release`)
- ✅ 多智能体 PR 评审 (`/dx:pr-review-loop`)
- ✅ 环境诊断 (`/dx:doctor`)

**新增 Agents**:
- `pr-precheck` - PR 前置检查
- `pr-context` - PR 上下文提取
- `claude-reviewer` - Claude 评审
- `codex-reviewer` - Codex 评审
- `gemini-reviewer` - Gemini 评审
- `pr-review-aggregate` - 评审聚合
- `pr-fix` - 自动修复

## 致谢

本项目大量代码来源于以下项目，在此表示感谢：

- [cexll/myclaude](https://github.com/cexll/myclaude) - 本项目的主要代码基础
- [Anthropic Claude Code](https://claude.ai/claude-code) - Claude Code 官方插件系统和最佳实践

## License

- **本项目新增/修改的代码**：采用 [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) 协议，放弃所有版权，可自由使用
- **继承的代码**：按照原项目的协议授权
