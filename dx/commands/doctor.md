---
allowed-tools: [Bash, AskUserQuestion]
description: '环境诊断：检测并安装 codeagent-wrapper 及后端 CLI 依赖'
---

## Usage

```bash
/dx:doctor
```

检测当前系统的开发环境配置，确保必要的工具已正确安装。

## 检测项

### 1. codeagent-wrapper (必需)

统一的多后端代码代理封装，是 DX 工具集的核心依赖。

**检测命令**：

```bash
which codeagent-wrapper || command -v codeagent-wrapper
codeagent-wrapper --version 2>/dev/null
```

### 2. codex CLI (可选)

OpenAI Codex CLI，用于支持 `--codex` 参数。

**检测命令**：

```bash
which codex || command -v codex
codex --version 2>/dev/null
```

### 3. gemini CLI (可选)

Google Gemini CLI，用于支持 `--gemini` 参数。

**检测命令**：

```bash
which gemini || command -v gemini
gemini --version 2>/dev/null
```

## 工作流程

### 阶段 1：环境检测

执行以下检测：

```bash
# 1. 检查 codeagent-wrapper
if command -v codeagent-wrapper &> /dev/null; then
    CODEAGENT_STATUS="已安装"
    CODEAGENT_VERSION=$(codeagent-wrapper --version 2>/dev/null || echo "未知")
else
    CODEAGENT_STATUS="未安装"
    CODEAGENT_VERSION="-"
fi

# 2. 检查 codex CLI
if command -v codex &> /dev/null; then
    CODEX_STATUS="已安装"
    CODEX_VERSION=$(codex --version 2>/dev/null || echo "未知")
else
    CODEX_STATUS="未安装"
    CODEX_VERSION="-"
fi

# 3. 检查 gemini CLI
if command -v gemini &> /dev/null; then
    GEMINI_STATUS="已安装"
    GEMINI_VERSION=$(gemini --version 2>/dev/null || echo "未知")
else
    GEMINI_STATUS="未安装"
    GEMINI_VERSION="-"
fi
```

### 阶段 2：输出诊断报告

根据检测结果输出报告：

```
环境诊断报告

工具               | 状态     | 版本      | 说明
-------------------|----------|-----------|------------------
codeagent-wrapper  | 已安装   | v1.2.3    | 核心依赖
codex              | 已安装   | v1.0.0    | 支持 --codex 参数
gemini             | 未安装   | -         | 支持 --gemini 参数
```

### 阶段 3：警告信息

**如果 codex CLI 未安装**：

```
⚠️  警告: codex CLI 未安装
   - 命令参数 --codex 将无法使用
   - 安装方式: npm install -g @openai/codex 或参考官方文档
```

**如果 gemini CLI 未安装**：

```
⚠️  警告: gemini CLI 未安装
   - 命令参数 --gemini 将无法使用
   - 安装方式: 参考 Google Gemini CLI 官方文档
```

### 阶段 4：处理 codeagent-wrapper 未安装情况

如果 `codeagent-wrapper` 未安装：

1. 使用 `AskUserQuestion` 询问用户是否要自动安装：
   - 选项 1: 自动安装（推荐）- 下载并运行官方安装脚本
   - 选项 2: 手动安装 - 显示安装命令供用户复制

2. **如果用户选择自动安装**：

   执行安装命令：
   ```bash
   curl -fsSL https://raw.githubusercontent.com/cexll/myclaude/master/install.sh | bash
   ```

   - 如果安装成功，输出成功消息并重新检测版本
   - 如果安装失败，输出错误信息并提示用户手动安装

3. **如果用户选择手动安装**：

   输出以下信息：
   ```
   请手动执行以下命令安装 codeagent-wrapper：

   curl -fsSL https://raw.githubusercontent.com/cexll/myclaude/master/install.sh | bash

   或者访问以下链接获取更多安装选项：
   https://github.com/cexll/myclaude/blob/master/install.sh
   ```

### 阶段 5：安装失败处理

如果自动安装失败，输出：

```
安装失败

自动安装过程中遇到错误。请尝试手动安装：

1. 下载安装脚本：
   curl -fsSL https://raw.githubusercontent.com/cexll/myclaude/master/install.sh -o install.sh

2. 查看脚本内容（可选）：
   cat install.sh

3. 执行安装：
   bash install.sh

如果问题持续，请访问：
https://github.com/cexll/myclaude/issues
```

## 输出格式

### 全部检测通过

```
环境诊断完成

工具               | 状态     | 版本
-------------------|----------|--------
codeagent-wrapper  | 已安装   | v1.2.3
codex              | 已安装   | v1.0.0
gemini             | 已安装   | v2.0.0

所有依赖已就绪，全部功能可用。
```

### 部分功能可用

```
环境诊断完成

工具               | 状态     | 版本
-------------------|----------|--------
codeagent-wrapper  | 已安装   | v1.2.3
codex              | 已安装   | v1.0.0
gemini             | 未安装   | -

⚠️  警告: gemini CLI 未安装
   - 命令参数 --gemini 将无法使用
   - 其他功能正常可用

核心功能已就绪。如需使用 --gemini 参数，请安装 gemini CLI。
```

### 核心依赖缺失

```
环境诊断完成

工具               | 状态     | 版本
-------------------|----------|--------
codeagent-wrapper  | 未安装   | -
codex              | 未安装   | -
gemini             | 未安装   | -

❌ 错误: codeagent-wrapper 未安装（核心依赖）

是否要自动安装 codeagent-wrapper？
```

### 安装成功

```
安装成功

codeagent-wrapper 已成功安装 (v1.2.3)

现在可以正常使用 dx 工具集了。
```

## 注意事项

- 安装脚本需要网络连接
- 某些系统可能需要 sudo 权限
- 如果使用代理，确保代理配置正确
- codex 和 gemini CLI 为可选依赖，不影响核心功能
- 未安装可选 CLI 时，对应的 `--codex` 或 `--gemini` 参数将不可用
