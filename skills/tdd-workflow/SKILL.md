# 测试驱动开发工作流

此技能确保所有代码开发遵循 TDD 原则并具备全面的测试覆盖率。

## 使用时机

- 编写新功能或特性
- 修复漏洞或问题
- 重构现有代码
- 添加 API 端点
- 创建新组件

## 核心原则

### 1. 测试先于代码
始终优先编写测试，然后实现代码以使测试通过。

### 2. 覆盖率要求
- 最低 80% 覆盖率（单元测试 + 集成测试 + 端到端测试）
- 所有边缘情况均被覆盖
- 错误场景已测试
- 边界条件已验证

### 3. 测试类型

#### 单元测试
- 单个函数和工具
- 组件逻辑
- 纯函数
- 辅助函数和工具

#### 集成测试
- API 端点
- 数据库操作
- 服务交互
- 外部 API 调用

#### 端到端测试（Playwright）
- 关键用户流程
- 完整工作流
- 浏览器自动化
- UI 交互

## TDD 工作流程步骤

### 第一步：编写用户旅程
```
作为一个 [角色]，我希望 [操作]，以便 [好处]

示例：
作为一个用户，我希望能够语义化搜索市场，
以便我可以在没有确切关键词的情况下找到相关市场。
```

### 第二步：生成测试用例
对于每个用户旅程，创建全面的测试用例：

```typescript
describe('语义搜索', () => {
  it('为查询返回相关市场', async () => {
    // 测试实现
  })

  it('优雅处理空查询', async () => {
    // 测试边缘情况
  })

  it('当 Redis 不可用时回退到子字符串搜索', async () => {
    // 测试回退行为
  })

  it('根据相似度分数排序结果', async () => {
    // 测试排序逻辑
  })
})
```

### 第三步：运行测试（它们应失败）
```bash
npm test
# 测试应失败——我们尚未实现
```

### 第四步：实现代码
编写最小的代码使测试通过：

```typescript
// 由测试指导的实现
export async function searchMarkets(query: string) {
  // 实现在此处
}
```

### 第五步：再次运行测试
```bash
npm test
# 测试应通过
```

### 第六步：重构
在保持测试绿色的情况下改进代码质量：
- 删除重复项
- 改进命名
- 优化性能
- 提高可读性

### 第七步：验证覆盖率
```bash
npm run test:coverage
# 验证达到 80%+ 的覆盖率
```

## 测试模式

### 单元测试模式（Jest/Vitest）
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('按钮组件', () => {
  it('用正确的文本渲染', () => {
    render(<Button>点击我</Button>)
    expect(screen.getByText('点击我')).toBeInTheDocument()
  })

  it('点击时调用 onClick', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>点击</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('当 disabled 属性为 true 时是禁用状态', () => {
    render(<Button disabled>点击</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API 集成测试模式
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('成功返回市场', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('验证查询参数', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('优雅处理数据库错误', async () => {
    // 模拟数据库失败
    const request = new NextRequest('http://localhost/api/markets')
    // 测试错误处理
  })
})
```

### 端到端测试模式（Playwright）
```typescript
import { test, expect } from '@playwright/test'

test('用户可以搜索并筛选市场', async ({ page }) => {
  // 导航到市场页面
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 验证页面加载
  await expect(page.locator('h1')).toContainText('市场')

  // 搜索市场
  await page.fill('input[placeholder="搜索市场"]', '选举')

  // 等待去抖动和结果
  await page.waitForTimeout(600)

  // 验证显示搜索结果
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 验证结果包含搜索词
  const firstResult = results.first()
  await expect(firstResult).toContainText('选举', { ignoreCase: true })

  // 按状态筛选
  await page.click('button:has-text("活跃")')

  // 验证筛选后的结果
  await expect(results).toHaveCount(3)
})

test('用户可以创建新市场', async ({ page }) => {
  // 先登录
  await page.goto('/creator-dashboard')

  // 填写市场创建表单
  await page.fill('input[name="name"]', '测试市场')
  await page.fill('textarea[name="description"]', '测试描述')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // 提交表单
  await page.click('button[type="submit"]')

  // 验证成功消息
  await expect(page.locator('text=市场创建成功')).toBeVisible()

  // 验证重定向到市场页面
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 测试文件组织

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # 单元测试
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 集成测试
└── e2e/
    ├── markets.spec.ts               # 端到端测试
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 模拟外部服务

### Supabase 模拟
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: '测试市场' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis 模拟
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI 模拟
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // 模拟 1536 维向量
  ))
}))
```

## 测试覆盖率验证

### 运行覆盖率报告
```bash
npm run test:coverage
```

### 覆盖率阈值
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 常见测试错误避免

### ❌ 错误：测试实现细节
```typescript
// 不要测试内部状态
expect(component.state.count).toBe(5)
```

### ✅ 正确：测试用户可见行为
```typescript
// 测试用户看到的内容
expect(screen.getByText('计数: 5')).toBeInTheDocument()
```

### ❌ 错误：脆弱的选择器
```typescript
// 容易断裂
await page.click('.css-class-xyz')
```

### ✅ 正确：语义化选择器
```typescript
// 对更改有弹性
await page.click('button:has-text("提交")')
await page.click('[data-testid="submit-button"]')
```

### ❌ 错误：无测试隔离
```typescript
// 测试之间相互依赖
test('创建用户', () => { /* ... */ })
test('更新同一用户', () => { /* 依赖于前一个测试 */ })
```

### ✅ 正确：独立测试
```typescript
// 每个测试都设置自己的数据
test('创建用户', () => {
  const user = createTestUser()
  // 测试逻辑
})

test('更新用户', () => {
  const user = createTestUser()
  // 更新逻辑
})
```

## 持续测试

### 开发期间的监视模式
```bash
npm test -- --watch
# 文件更改时自动运行测试
```

### 提交前钩子
```bash
# 每次提交前运行
npm test && npm run lint
```

### CI/CD 集成
```yaml
# GitHub Actions
- name: 运行测试
  run: npm test -- --coverage
- name: 上传覆盖率
  uses: codecov/codecov-action@v3
```

## 最佳实践

1. **先写测试** - 始终 TDD
2. **每个测试一个断言** - 聚焦单一行为
3. **描述性测试名称** - 解释测试内容
4. **安排-执行-断言** - 清晰的测试结构
5. **模拟外部依赖** - 隔离单元测试
6. **测试边缘情况** - 空、未定义、空、大值
7. **测试错误路径** - 不仅是正常路径
8. **保持测试快速** - 单元测试每项 < 50ms
9. **测试后清理** - 没有副作用
10. **审查覆盖率报告** - 识别空白

## 成功指标

- 达到 80%+ 的代码覆盖率
- 所有测试通过（绿色）
- 没有跳过或禁用的测试
- 测试执行快速（单元测试 < 30s）
- 端到端测试覆盖关键用户流程
- 测试在生产之前捕获漏洞
