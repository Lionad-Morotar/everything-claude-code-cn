# 系统钩子

## 钩子类型

- **PreToolUse**：工具执行前（验证、参数修改）
- **PostToolUse**：工具执行后（自动格式化、检查）
- **Stop**：会话结束时（最终验证）

## 当前钩子（位于 ~/.claude/settings.json 中）

### PreToolUse
- **tmux 提醒**：建议对长时间运行的命令（npm、pnpm、yarn、cargo 等）使用 tmux
- **git push 审查**：推送前在 Zed 中打开以供审查
- **文档阻止器**：阻止创建不必要的 .md/.txt 文件

### PostToolUse
- **拉取请求创建**：记录 PR 链接和 GitHub Actions 状态
- **Prettier**：编辑后自动格式化 JS/TS 文件
- **TypeScript 检查**：编辑 .ts/.tsx 文件后运行 tsc
- **console.log 警告**：提示编辑文件中的 console.log

### Stop
- **console.log 审计**：会话结束前检查所有修改文件中是否存在 console.log

## 自动接受权限

谨慎使用：
- 仅用于可信且定义明确的计划
- 探索性工作时应禁用
- 永远不要使用 dangerously-skip-permissions 标志
- 应在 ~/.claude.json 中配置 `allowedTools` 代替

## TodoWrite 最佳实践

使用 TodoWrite 工具：
- 跟踪多步骤任务的进展
- 验证对指令的理解
- 实现实时控制
- 展示具体的实施步骤

待办事项列表揭示：
- 顺序错误的步骤
- 缺失项
- 多余不必要的项
- 粒度错误
- 要求误读
