# 编码标准与最佳实践

适用于所有项目的通用编码标准。

## 代码质量原则

### 1. 可读性优先
- 代码的阅读次数远超编写次数
- 清晰的变量与函数命名
- 优先使用自文档化代码而非注释
- 统一的格式化风格

### 2. KISS（保持简单）
- 最简单的有效解决方案
- 避免过度工程化
- 避免过早优化
- 易于理解 > 精巧代码

### 3. DRY（不要重复自己）
- 将通用逻辑提取至函数中
- 创建可复用组件
- 在模块间共享工具函数
- 避免复制粘贴编程

### 4. YAGNI（你不会需要它）
- 不要在功能尚未需要前构建
- 避免推测性通用化
- 仅在确实需要时增加复杂度
- 开始时保持简单，必要时再重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// ✅ 正确：具有描述性的名称
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 错误：命名不明确
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// ✅ 正确：动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 错误：命名不明确或仅名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变性模式（关键）

```typescript
// ✅ 始终使用扩展运算符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 永远不要直接修改
user.name = 'New Name'  // 错误
items.push(newItem)     // 错误
```

### 错误处理

```typescript
// ✅ 正确：全面的错误处理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ 错误：无错误处理
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### 异步/等待最佳实践

```typescript
// ✅ 正确：在可能时并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 错误：在不必要的场合顺序执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 类型安全

```typescript
// ✅ 正确：正确类型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 实现
}

// ❌ 错误：使用 'any'
function getMarket(id: any): Promise<any> {
  // 实现
}
```

## React 最佳实践

### 组件结构

```typescript
// ✅ 正确：带类型的函数组件
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ 错误：无类型，结构不清晰
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 自定义钩子

```typescript
// ✅ 正确：可复用的自定义钩子
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用方式
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 状态管理

```typescript
// ✅ 正确：适当的更新状态方式
const [count, setCount] = useState(0)

// 基于先前状态的函数式更新
setCount(prev => prev + 1)

// ❌ 错误：直接引用状态
setCount(count + 1)  // 在异步场景中可能过时
```

### 条件渲染

```typescript
// ✅ 正确：清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 错误：三元地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 约定

```
GET    /api/markets              # 列出所有市场
GET    /api/markets/:id          # 获取特定市场
POST   /api/markets              # 创建新市场
PUT    /api/markets/:id          # 更新市场（完整）
PATCH  /api/markets/:id          # 更新市场（部分）
DELETE /api/markets/:id          # 删除市场

# 查询参数用于过滤
GET /api/markets?status=active&limit=10&offset=0
```

### 响应格式

```typescript
// ✅ 正确：一致的响应结构
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功响应
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 错误响应
return NextResponse.json({
  success: false,
  error: 'Invalid request'
}, { status: 400 })
```

### 输入验证

```typescript
import { z } from 'zod'

// ✅ 正确：模式验证
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 使用已验证的数据
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## 文件组织

### 项目结构

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 路由
│   ├── markets/           # 市场页面
│   └── (auth)/           # 认证页面（路由组）
├── components/            # React 组件
│   ├── ui/               # 通用 UI 组件
│   ├── forms/            # 表单组件
│   └── layouts/          # 布局组件
├── hooks/                # 自定义 React 钩子
├── lib/                  # 工具函数与配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型
└── styles/              # 全局样式
```

### 文件命名

```
components/Button.tsx          # 组件使用 PascalCase
hooks/useAuth.ts              # 工具函数使用 camelCase 并以 'use' 开头
lib/formatDate.ts             # 工具函数使用 camelCase
types/market.types.ts         # 类型文件使用 camelCase 并以 .types 结尾
```

## 注释与文档

### 何时注释

```typescript
// ✅ 正确：解释为何，而非如何
// 使用指数退避策略以避免在中断期间压垮 API
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 明确在此处使用变异以提高处理大量数组时的性能
items.push(newItem)

// ❌ 错误：显而易见的内容
// 将计数器加一
count++

// 将名称设为用户名称
name = user.name
```

### 公共 API 的 JSDoc

```typescript
/**
 * 使用语义相似性搜索市场。
 *
 * @param query - 自然语言搜索查询
 * @param limit - 最大返回结果数（默认值：10）
 * @returns 按相似度得分排序的市场数组
 * @throws {Error} 如果 OpenAI API 失败或 Redis 不可用
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 实现逻辑
}
```

## 性能最佳实践

### 记忆化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 正确：记忆昂贵的计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 正确：记忆回调函数
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 懒加载

```typescript
import { lazy, Suspense } from 'react'

// ✅ 正确：懒加载重型组件
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### 数据库查询

```typescript
// ✅ 正确：仅选择需要的列
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ 错误：选择所有列
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 测试标准

### 测试结构（AAA 模式）

```typescript
test('正确计算相似度', () => {
  // Arrange（准备）
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act（执行）
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert（断言）
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// ✅ 正确：描述性强的测试名称
test('当无市场匹配查询时返回空数组', () => { })
test('当 OpenAI API 密钥缺失时抛出错误', () => { })
test('当 Redis 不可用时回退至子字符串搜索', () => { })

// ❌ 错误：模糊的测试名称
test('工作正常', () => { })
test('测试搜索', () => { })
```

## 代码异味检测

注意这些反模式：

### 1. 长函数
```typescript
// ❌ 错误：函数超过 50 行
function processMarketData() {
  // 100 行代码
}

// ✅ 正确：拆分为较小的函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深层嵌套
```typescript
// ❌ 错误：5+ 层嵌套
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 做一些事情
        }
      }
    }
  }
}

// ✅ 正确：提前返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 做一些事情
```

### 3. 魔数
```typescript
// ❌ 错误：未解释的数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 正确：命名常量
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**记住**：代码质量不可妥协。清晰、可维护的代码能够实现快速开发和自信重构。
