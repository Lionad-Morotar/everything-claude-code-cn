# Everything Claude Code

**由 Anthropic 黑客松获胜者提供的 Claude Code 配置完整集合。**

此仓库包含我每日与 Claude Code 一起使用的生产就绪代理、技能、钩子、命令、规则和 MCP 配置。这些配置经过超过十个月的密集使用，用于构建真实产品。

---

## 先阅读完整指南

**在深入研究这些配置之前，请先阅读我在 X 上发布的完整指南：**

<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />

**[Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)**

该指南解释了：
- 每种配置类型的作用及其适用场景
- 如何构建 Claude Code 设置结构
- 上下文窗口管理（对性能至关重要）
- 并行工作流和高级技巧
- 这些配置背后的哲学理念

**本仓库仅包含配置！更多技巧、示例和文章链接将随着 README 的更新而添加。**

---

## 内容概览

```
everything-claude-code/
|-- agents/           # 用于委托的专用子代理
|   |-- planner.md           # 功能实现规划
|   |-- architect.md         # 系统设计决策
|   |-- tdd-guide.md         # 测试驱动开发指南
|   |-- code-reviewer.md     # 质量与安全审查
|   |-- security-reviewer.md # 漏洞分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright 端到端测试
|   |-- refactor-cleaner.md  # 冗余代码清理
|   |-- doc-updater.md       # 文档同步更新
|
|-- skills/           # 工作流定义与领域知识
|   |-- coding-standards.md         # 编程语言最佳实践
|   |-- backend-patterns.md         # API、数据库、缓存模式
|   |-- frontend-patterns.md        # React、Next.js 模式
|   |-- project-guidelines-example.md # 示例项目特定技能
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- clickhouse-io.md            # ClickHouse 分析
|
|-- commands/         # 快速执行的斜杠命令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现规划
|   |-- e2e.md              # /e2e - 端到端测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 修复构建错误
|   |-- refactor-clean.md   # /refactor-clean - 删除冗余代码
|   |-- test-coverage.md    # /test-coverage - 覆盖率分析
|   |-- update-codemaps.md  # /update-codemaps - 刷新文档
|   |-- update-docs.md      # /update-docs - 同步文档
|
|-- rules/            # 始终遵循的指导原则
|   |-- security.md         # 必须的安全检查
|   |-- coding-style.md     # 不可变性、文件组织
|   |-- testing.md          # TDD，覆盖率要求 80%
|   |-- git-workflow.md     # 提交格式、PR 流程
|   |-- agents.md           # 何时委托给子代理
|   |-- performance.md      # 模型选择、上下文管理
|   |-- patterns.md         # API 响应格式、钩子
|   |-- hooks.md            # 钩子文档
|
|-- hooks/            # 基于触发的自动化
|   |-- hooks.json          # PreToolUse、PostToolUse、Stop 钩子
|
|-- mcp-configs/      # MCP 服务器配置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- plugins/          # 插件生态系统文档
|   |-- README.md           # 插件、市场、技能指南
|
|-- examples/         # 示例配置
    |-- CLAUDE.md           # 示例项目级配置
    |-- user-CLAUDE.md      # 示例用户级配置 (~/.claude/CLAUDE.md)
    |-- statusline.json     # 自定义状态行配置
```

---

## 快速开始

### 1. 拷贝所需内容

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 拷贝代理到你的 Claude 配置中
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 拷贝规则
cp everything-claude-code/rules/*.md ~/.claude/rules/

# 拷贝命令
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 拷贝技能
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. 将钩子添加到 settings.json

将 `hooks/hooks.json` 中的钩子拷贝至你的 `~/.claude/settings.json`。

### 3. 配置 MCP

从 `mcp-configs/mcp-servers.json` 中拷贝所需的 MCP 服务器到你的 `~/.claude.json`。

**重要提示：** 将 `YOUR_*_HERE` 占位符替换为实际 API 密钥。

### 4. 阅读指南

认真阅读 [该指南](https://x.com/affaanmustafa/status/2012378465664745795)。这些配置需要上下文才能真正理解。

---

## 核心概念

### 代理

子代理负责处理委托的任务，作用域有限。例如：

```markdown
---
name: code-reviewer
description: 审查代码以确保质量、安全性和可维护性
tools: Read, Grep, Glob, Bash
model: opus
---

你是一个资深代码审查者...
```

### 技能

技能是通过命令或代理调用的工作流定义：

```markdown
# TDD 工作流程

1. 首先定义接口
2. 编写失败的测试（RED）
3. 实现最小代码（GREEN）
4. 重构（IMPROVE）
5. 验证覆盖率超过 80%
```

### 钩子

钩子在工具事件触发时执行。例如——警告 console.log：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### 规则

规则是始终遵循的指导原则。请保持模块化：

```
~/.claude/rules/
  security.md      # 不得硬编码密钥
  coding-style.md  # 不可变性、文件限制
  testing.md       # TDD，覆盖率要求
```

---

## 贡献

**欢迎并鼓励贡献。**

此仓库旨在成为社区资源。如果你有：
- 实用的代理或技能
- 精巧的钩子
- 更好的 MCP 配置
- 改进的规则

请贡献！详情参见 [CONTRIBUTING.md](CONTRIBUTING.md)。

### 贡献思路

- 语言特定技能（Python、Go、Rust 模式）
- 框架特定配置（Django、Rails、Laravel）
- DevOps 代理（Kubernetes、Terraform、AWS）
- 测试策略（不同框架）
- 领域特定知识（机器学习、数据工程、移动端）

---

## 背景

我从实验性发布阶段起就在使用 Claude Code。2025 年 9 月，我和 [@DRodriguezFX](https://x.com/DRodriguezFX) 一起开发了 [zenith.chat](https://zenith.chat)，完全使用 Claude Code 构建，赢得了 Anthropic x Forum Ventures 黑客松。

这些配置已在多个生产应用中经受考验。

---

## 重要说明

### 上下文窗口管理

**关键：** 不要一次性启用所有 MCP。若启用过多工具，你的 200k 上下文窗口可能缩小至 70k。

经验法则：
- 配置 20–30 个 MCP
- 每个项目保持启用少于 10 个
- 单项目保持活动工具数低于 80 个

在项目配置中使用 `disabledMcpServers` 来禁用未使用的 MCP。

### 自定义

这些配置适用于我的工作流。你应该：
1. 从你共鸣的部分开始
2. 根据你的技术栈进行修改
3. 删除你不需要的内容
4. 添加你自己的模式

---

## 链接

- **完整指南：** [Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)
- **关注：** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat：** [zenith.chat](https://zenith.chat)

---

## 许可证

MIT —— 自由使用、按需修改、如可请回馈。

---

**如果对你有帮助，请给本仓库点星。阅读指南，构建伟大之物。**
