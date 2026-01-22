# 示例项目 CLAUDE.md

这是一个示例项目级别的 CLAUDE.md 文件。将其放置在您的项目根目录中。

## 项目概述

[对您的项目的简要描述——它做什么，技术栈]

## 关键规则

### 1. 代码组织

- 使用大量小型文件，而非少量大型文件
- 高内聚、低耦合
- 单个文件通常为 200–400 行，最多不超过 800 行
- 按功能/领域组织，而非按类型

### 2. 代码风格

- 代码、注释或文档中不得使用表情符号
- 始终使用不可变性——从不修改对象或数组
- 生产环境中不得使用 console.log
- 使用 try/catch 进行适当的错误处理
- 使用 Zod 或类似工具进行输入验证

### 3. 测试

- TDD：先写测试
- 最低覆盖率 80%
- 工具函数使用单元测试
- API 使用集成测试
- 关键流程使用端到端测试

### 4. 安全

- 禁止硬编码密钥
- 使用环境变量存储敏感数据
- 验证所有用户输入
- 仅使用参数化查询
- 启用 CSRF 保护

## 文件结构

```
src/
|-- app/              # Next.js 应用路由
|-- components/       # 可复用的 UI 组件
|-- hooks/            # 自定义 React 钩子
|-- lib/              # 工具库
|-- types/            # TypeScript 定义
```

## 核心模式

### API 响应格式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### 错误处理

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('操作失败：', error)
  return { success: false, error: '用户友好的提示信息' }
}
```

## 环境变量

```bash
# 必填项
DATABASE_URL=
API_KEY=

# 可选项
DEBUG=false
```

## 可用命令

- `/tdd` - 测试驱动开发工作流
- `/plan` - 创建实施计划
- `/code-review` - 代码质量审查
- `/build-fix` - 修复构建错误

## Git 工作流

- 使用约定式提交：`feat:`、`fix:`、`refactor:`、`docs:`、`test:`
- 禁止直接向 main 提交
- Pull Request 必须经过审查
- 合并前所有测试必须通过
