# Git 工作流程

## 提交信息格式

```
<type>: <description>

<optional body>
```

类型：feat, fix, refactor, docs, test, chore, perf, ci

注意：通过 ~/.claude/settings.json 全局禁用归属声明。

## 拉取请求工作流程

创建 PR 时：
1. 分析完整的提交历史（不仅限于最新一次提交）
2. 使用 `git diff [base-branch]...HEAD` 查看所有变更
3. 草拟全面的 PR 摘要
4. 包含带 TODO 的测试计划
5. 如为新分支，使用 `-u` 标志推送

## 功能实现工作流程

1. **首先规划**
   - 使用 **planner** agent 制定实现计划
   - 识别依赖项与风险
   - 分解为各个阶段

2. **测试驱动开发方法**
   - 使用 **tdd-guide** agent
   - 先编写测试（RED）
   - 实现以通过测试（GREEN）
   - 重构（IMPROVE）
   - 验证覆盖率超过 80%

3. **代码审查**
   - 编写代码后立即使用 **code-reviewer** agent
   - 解决 CRITICAL 和 HIGH 级别问题
   - 在可能的情况下修复 MEDIUM 级别问题

4. **提交与推送**
   - 详细的提交信息
   - 遵循约定式提交格式
