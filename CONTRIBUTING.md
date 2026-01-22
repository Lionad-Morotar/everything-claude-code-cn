# 为 Everything Claude Code 贡献代码

感谢您希望做出贡献。此仓库旨在成为 Claude Code 用户的社区资源。

## 我们希望获得的内容

### 代理（Agents）

能够有效处理特定任务的新代理：
- 语言特定的审阅者（Python、Go、Rust）
- 框架专家（Django、Rails、Laravel、Spring）
- DevOps 专家（Kubernetes、Terraform、CI/CD）
- 领域专家（ML 流水线、数据工程、移动开发）

### 技能（Skills）

工作流定义与领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南
- 领域特定知识

### 命令（Commands）

调用有用工作流的斜杠命令：
- 部署命令
- 测试命令
- 文档命令
- 代码生成命令

### 钩子（Hooks）

有用的自动化功能：
- 语法检查/格式化钩子
- 安全性检查
- 验证钩子
- 通知钩子

### 规则（Rules）

始终遵循的指南：
- 安全规则
- 代码风格规则
- 测试要求
- 命名约定

### MCP 配置（MCP Configurations）

新的或改进的 MCP 服务器配置：
- 数据库集成
- 云提供商 MCP
- 监控工具
- 通信工具

---

## 如何贡献

### 1. Fork 此仓库

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. 创建分支

```bash
git checkout -b add-python-reviewer
```

### 3. 添加您的贡献

将文件放入适当的目录中：
- `agents/` 用于新代理
- `skills/` 用于技能（可以是单个 .md 文件或目录）
- `commands/` 用于斜杠命令
- `rules/` 用于规则文件
- `hooks/` 用于钩子配置
- `mcp-configs/` 用于 MCP 服务器配置

### 4. 遵循格式

**代理（Agents）** 应具有前置元数据：

```markdown
---
name: agent-name
description: What it does
tools: Read, Grep, Glob, Bash
model: sonnet
---

Instructions here...
```

**技能（Skills）** 应清晰且可操作：

```markdown
# Skill Name

## When to Use

...

## How It Works

...

## Examples

...
```

**命令（Commands）** 应解释其作用：

```markdown
---
description: Brief description of command
---

# Command Name

Detailed instructions...
```

**钩子（Hooks）** 应包含描述：

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "What this hook does"
}
```

### 5. 测试您的贡献

在提交前确保配置能与 Claude Code 正常配合使用。

### 6. 提交拉取请求（PR）

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

然后打开一个 PR，包含以下内容：
- 您添加的内容
- 其为何有用
- 您是如何测试的

---

## 行为准则

### 做到

- 保持配置集中且模块化
- 包含清晰的描述
- 提交前进行测试
- 遵循现有模式
- 记录任何依赖项

### 避免

- 包含敏感数据（API 密钥、令牌、路径）
- 添加过度复杂或冷门的配置
- 提交未经测试的配置
- 创建重复功能
- 添加需要特定付费服务才能运行的配置，而无替代方案

---

## 文件命名规则

- 使用小写字母与连字符：`python-reviewer.md`
- 描述性强：`tdd-workflow.md` 而非 `workflow.md`
- 文件名应与代理/技能名称一致

---

## 有疑问？

开启一个 issue 或通过 X 联系我：[@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献。让我们一起构建一个出色的资源库。
