# 编码风格

## 不可变性（至关重要的原则）

始终创建新对象，切勿修改原对象：

```javascript
// 错误：修改原对象
function updateUser(user, name) {
  user.name = name  // 修改！
  return user
}

// 正确：使用不可变性
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## 文件组织

推荐使用大量小型文件，而非少量大型文件：
- 高内聚、低耦合
- 典型每文件 200–400 行，最多 800 行
- 从大型组件中提取工具函数
- 按功能/领域组织，而非按类型

## 错误处理

始终全面处理错误：

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('操作失败：', error)
  throw new Error('详细的用户友好提示信息')
}
```

## 输入验证

始终验证用户输入：

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## 代码质量检查清单

在标记工作完成前，请确保：
- [ ] 代码可读且命名清晰
- [ ] 函数简短（<50 行）
- [ ] 文件专注（<800 行）
- [ ] 无深层嵌套（>4 层）
- [ ] 正确处理错误
- [ ] 无 console.log 语句
- [ ] 无硬编码值
- [ ] 无修改原对象（使用不可变模式）
