---
description: 使用 Playwright 生成并运行端到端测试。创建测试流程，运行测试，捕获屏幕截图/视频/轨迹，并上传工件。
---

# 端到端命令

此命令调用 **e2e-runner** 代理，使用 Playwright 生成、维护和执行端到端测试。

## 此命令的功能

1. **生成测试流程** - 为用户流程创建 Playwright 测试
2. **运行端到端测试** - 在浏览器中执行测试
3. **捕获工件** - 失败时的屏幕截图、视频和轨迹
4. **上传结果** - HTML 报告和 JUnit XML
5. **识别不稳定测试** - 将不稳定的测试隔离

## 使用时机

使用 `/e2e` 当：
- 测试关键用户流程（登录、交易、支付）
- 验证多步流程是否端到端正常工作
- 测试 UI 交互和导航
- 验证前端与后端之间的集成
- 准备上线部署

## 工作原理

e2e-runner 代理将：

1. **分析用户流程** 并识别测试场景
2. **使用页面对象模型模式生成 Playwright 测试**
3. **在多个浏览器（Chrome、Firefox、Safari）中运行测试**
4. **在失败时捕获截图、视频和轨迹**
5. **生成包含结果和工件的报告**
6. **识别不稳定测试并推荐修复方案**

## 示例用法

```
用户：/e2e 测试市场搜索和查看流程

代理（e2e-runner）：
# 端到端测试生成：市场搜索和查看流程

## 识别的测试场景

**用户流程：** 市场搜索 → 查看结果 → 点击市场 → 查看详情

**测试覆盖：**
1. 导航至市场页面
2. 执行语义搜索
3. 验证搜索结果
4. 点击第一个结果
5. 验证市场详情页面加载
6. 验证图表渲染

## 生成的测试代码

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('市场搜索和查看流程', () => {
  test('用户可以搜索市场并查看详情', async ({ page }) => {
    // 1. 导航至市场页面
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 验证页面加载
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. 执行语义搜索
    await marketsPage.searchMarkets('election')

    // 等待 API 响应
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 验证搜索结果
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 捕获搜索结果截图
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 点击第一个结果
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. 验证市场详情页面加载
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. 验证图表渲染
    await expect(detailsPage.priceChart).toBeVisible()

    // 验证市场名称匹配
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // 捕获市场详情截图
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('搜索无结果时显示空状态', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 搜索不存在的市场
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 验证空状态
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('可以清除搜索并再次查看所有市场', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 初始市场数量
    const initialCount = await marketsPage.marketCards.count()

    // 执行搜索
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // 验证过滤后的结果
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 清除搜索
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // 验证再次显示所有市场
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## 运行测试

```bash
# 运行生成的测试
npx playwright test tests/e2e/markets/search-and-view.spec.ts

运行 3 个测试，使用 3 个工作进程

  ✓  [chromium] › search-and-view.spec.ts:5:3 › 用户可以搜索市场并查看详情 (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › 搜索无结果时显示空状态 (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › 可以清除搜索并再次查看所有市场 (2.9s)

  全部通过 (9.1s)

生成的工件：
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## 测试报告

```
╔══════════════════════════════════════════════════════════════╗
║                    端到端测试结果                          ║
╠══════════════════════════════════════════════════════════════╣
║ 状态:     ✅ 所有测试通过                                   ║
║ 总数:      3 个测试                                         ║
║ 通过:     3 (100%)                                          ║
║ 失败:     0                                                 ║
║ 不稳定:   0                                                 ║
║ 持续时间:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

工件：
📸 屏幕截图: 2 个文件
📹 视频: 0 个文件 (仅在失败时)
🔍 轨迹: 0 个文件 (仅在失败时)
📊 HTML 报告: playwright-report/index.html

查看报告: npx playwright show-report
```

✅ 端到端测试套件已准备好集成至 CI/CD！

## 测试工件

当测试运行时，将捕获以下工件：

**所有测试中：**
- 包含时间线和结果的 HTML 报告
- 用于 CI 集成的 JUnit XML

**仅在失败时：**
- 失败状态截图
- 测试视频录制
- 用于调试的轨迹文件（逐步回放）
- 网络日志
- 控制台日志

## 查看工件

```bash
# 在浏览器中查看 HTML 报告
npx playwright show-report

# 查看特定轨迹文件
npx playwright show-trace artifacts/trace-abc123.zip

# 屏幕截图保存在 artifacts/ 目录中
open artifacts/search-results.png
```

## 不稳定测试检测

如果一个测试偶尔失败：

```
⚠️  检测到不稳定测试：tests/e2e/markets/trade.spec.ts

测试通过 7/10 次运行（70% 通过率）

常见失败：
"等待元素 '[data-testid="confirm-btn"]' 超时"

推荐修复方案：
1. 添加显式等待：await page.waitForSelector('[data-testid="confirm-btn"]')
2. 增加超时：{ timeout: 10000 }
3. 检查组件中的竞争条件
4. 验证元素未被动画隐藏

隔离建议：在修复前标记为 test.fixme()
```

## 浏览器配置

默认情况下，测试在多个浏览器中运行：
- ✅ Chromium（桌面 Chrome）
- ✅ Firefox（桌面）
- ✅ WebKit（桌面 Safari）
- ✅ 移动 Chrome（可选）

在 `playwright.config.ts` 中配置以调整浏览器。

## CI/CD 集成

将以下内容添加到你的 CI 流水线中：

```yaml
# .github/workflows/e2e.yml
- name: 安装 Playwright
  run: npx playwright install --with-deps

- name: 运行端到端测试
  run: npx playwright test

- name: 上传工件
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX 特定的关键流程

对于 PMX，优先运行这些端到端测试：

**🔴 关键（必须始终通过）：**
1. 用户可以连接钱包
2. 用户可以浏览市场
3. 用户可以搜索市场（语义搜索）
4. 用户可以查看市场详情
5. 用户可以下单交易（使用测试资金）
6. 市场正确结算
7. 用户可以提取资金

**🟡 重要：**
1. 市场创建流程
2. 用户档案更新
3. 实时价格更新
4. 图表渲染
5. 过滤和排序市场
6. 移动响应式布局

## 最佳实践

**建议：**
- ✅ 使用页面对象模型以提高可维护性
- ✅ 使用 data-testid 属性进行选择器定位
- ✅ 等待 API 响应，而非任意超时
- ✅ 测试关键用户流程端到端
- ✅ 在合并至主分支前运行测试
- ✅ 当测试失败时检查工件

**避免：**
- ❌ 使用脆弱的选择器（CSS 类可能改变）
- ❌ 测试实现细节
- ❌ 在生产环境运行测试
- ❌ 忽略不稳定测试
- ❌ 忽略失败时的工件审查
- ❌ 用端到端测试覆盖每个边缘情况（使用单元测试）
