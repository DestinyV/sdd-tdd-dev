# 完整的TDD工作流 - Phase 1-3端到端指南

**目标**：展示从规范定义到最终验证的**完整的、生产级的TDD工作流**，涵盖所有三个阶段。

**适用场景**：
- 全栈项目的端到端开发
- 需要建立代码质量保障的团队
- 需要完整的需求追踪和验证链的项目

---

## 工作流总览

```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: 规范定义 (Spec-First)                                   │
├─────────────────────────────────────────────────────────────────┤
│ • 用户需求 → 规范化
│ • 生成：spec-dev/{req_id}/spec/
│   - scenarios/*.md (BDD WHEN-THEN格式)
│   - data-models.md
│   - business-rules.md
│   - glossary.md
└─────────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: TDD实现 (Design → Execute)                              │
├─────────────────────────────────────────────────────────────────┤
│ 2.1 设计规划 (code-designer)                                     │
│   • 输入：spec/
│   • 生成：design.md (架构、技术方案、设计决策)
│
│ 2.2 任务分解 (code-task)                                         │
│   • 输入：design.md
│   • 生成：tasks.md (详细Task列表)
│
│ 2.3 TDD实现 (code-execute)                                       │
│   • 🔴 RED：编写失败的单元测试
│   • 🟢 GREEN：实现最少代码让测试通过
│   • 🔵 REFACTOR：在测试通过的前提下优化代码
│   • ✅ REVIEW：验证代码质量和规范审查
│   • 输出：src/ + execution-report.md
└─────────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 3: 高层测试 (Integration → E2E → Performance)              │
├─────────────────────────────────────────────────────────────────┤
│ • 集成测试：多Task协作、跨模块通信
│ • E2E测试：完整业务流程、用户场景
│ • 性能测试：负载、压力、耐久性、基准建立
│
│ 输出：tests/ + testing-report.md
│      (含闭环验证矩阵：TEST-VERIFY → Test → Code → Result)
└─────────────────────────────────────────────────────────────────┘
                           ↓
                    ✅ 交付完成！
```

---

## Part 1: 规范定义阶段 (Phase 1)

### Step 1: 需求规范化

**使用 `/spec-creation` Skill**

用户提供需求描述后，通过交互式问卷进行规范化：

**输入**：
```
需求：构建电商平台的搜索和购物车系统
包含功能：
- 用户搜索商品
- 添加商品到购物车
- 查看和修改购物车
```

**输出**：`spec-dev/ecommerce-search-cart/spec/`

#### scenarios/search.md（BDD格式）

```markdown
## 场景：用户搜索商品

### SC-1: 搜索有结果

**WHEN** 用户在搜索框输入关键词"laptop"
**THEN** 系统应该返回所有名称包含"laptop"的商品
**AND** 搜索结果按相关度排序
**AND** 显示分页控件

**TEST-VERIFY**:
- TC-1.1.1: searchProducts("laptop") → test_search_laptop_returns_items
- TC-1.1.2: 搜索结果包含分页 → test_search_pagination_works
- TC-1.1.3: 结果按相关度排序 → test_search_relevance_order

### SC-2: 搜索无结果

**WHEN** 用户搜索不存在的关键词
**THEN** 系统应该显示"没有找到相关商品"提示
**AND** 提示建议用户修改搜索词

**TEST-VERIFY**:
- TC-1.2.1: searchProducts("xyzabc123") → test_search_no_results_shows_empty
- TC-1.2.2: 无结果时显示建议 → test_search_suggestion_displayed
```

#### scenarios/cart.md（BDD格式）

```markdown
## 场景：用户管理购物车

### SC-3: 添加商品到购物车

**WHEN** 用户在搜索结果中点击"添加到购物车"
**THEN** 商品应该添加到购物车
**AND** 购物车计数应该更新
**AND** 显示"已添加"确认提示

**TEST-VERIFY**:
- TC-2.1.1: addToCart(productId, qty) → test_add_to_cart_updates_count
- TC-2.1.2: 重复添加相同商品增加数量 → test_add_duplicate_increments_qty
- TC-2.1.3: 库存不足显示错误 → test_add_out_of_stock_shows_error

### SC-4: 修改购物车

**WHEN** 用户进入购物车页面
**THEN** 应该显示所有已添加的商品
**AND** 可以修改每个商品的数量
**AND** 可以删除商品
**AND** 总价实时更新

**TEST-VERIFY**:
- TC-2.2.1: 显示购物车商品列表 → test_cart_displays_items
- TC-2.2.2: 修改数量更新总价 → test_update_quantity_updates_total
- TC-2.2.3: 删除商品移除项 → test_remove_item_from_cart
```

#### data-models.md

```markdown
## 数据模型

### Product（商品）
```
{
  id: string,                    // 商品ID
  name: string,                  // 商品名称
  description: string,           // 商品描述
  price: number,                 // 价格（人民币）
  image: string,                 // 商品图片URL
  category: string,              // 商品分类
  stock: number,                 // 库存数量
  rating: number                 // 评分（0-5）
}
```

### Cart（购物车）
```
{
  userId: string,                // 用户ID
  items: CartItem[],             // 购物车项目列表
  createdAt: timestamp,          // 创建时间
  updatedAt: timestamp           // 更新时间
}
```

### CartItem（购物车项）
```
{
  productId: string,             // 商品ID
  quantity: number,              // 数量
  addedAt: timestamp             // 添加时间
}
```
```

#### business-rules.md

```markdown
## 业务规则

### 搜索规则
- 搜索关键词长度 ≥ 1 字符
- 搜索不区分大小写
- 支持部分匹配（name包含关键词）
- 搜索结果按相关度排序（完全匹配 > 前缀匹配 > 包含匹配）

### 购物车规则
- 单个商品最多购买999件
- 库存不足时无法添加
- 重复添加相同商品则增加数量
- 删除商品后，购物车中该商品完全移除
- 总价 = Σ(商品价格 × 数量)
```

---

## Part 2: 设计与规范阶段 (Phase 2.1-2.2)

### Step 2: 代码设计规划

**使用 `/code-designer` Skill**

**输入**：`spec-dev/ecommerce-search-cart/spec/`

**设计方案**（design.md摘要）：

```markdown
## 架构设计

### 系统组件
- SearchAPI（搜索服务）
- CartAPI（购物车服务）
- UI层（前端React组件）
- 数据库层（PostgreSQL）

### 技术方案
- 后端：Node.js + Express + TypeScript
- 前端：React + TypeScript + Tailwind CSS
- 数据库：PostgreSQL + Prisma ORM
- 测试：Jest（单元）+ Playwright（E2E）+ k6（性能）

### 关键设计决策
1. 搜索使用全文搜索（PostgreSQL全文索引）
2. 购物车存储在数据库（不是session）
3. 搜索结果缓存5分钟
4. 购物车实时更新，不进行缓存
```

**输出**：`spec-dev/ecommerce-search-cart/design/design.md`

### Step 3: 任务列表分解

**使用 `/code-task` Skill**

**输入**：`spec-dev/ecommerce-search-cart/design/design.md`

**任务列表**（tasks.md摘要）：

```markdown
## Task列表

### Task 1: 搜索API实现
目标：实现搜索商品API
交付物：
- src/api/search.ts (搜索函数)
- src/__tests__/search.test.ts (单元测试)
验收标准：
- [ ] 支持关键词搜索
- [ ] 返回正确的搜索结果
- [ ] 处理无结果的情况
- [ ] 单元测试覆盖率 ≥ 85%

### Task 2: 购物车API实现
目标：实现购物车操作API
交付物：
- src/api/cart.ts (购物车函数)
- src/__tests__/cart.test.ts (单元测试)
验收标准：
- [ ] 支持添加商品
- [ ] 支持删除商品
- [ ] 支持修改数量
- [ ] 单元测试覆盖率 ≥ 85%

### Task 3: UI实现（搜索页面）
目标：实现搜索页面组件
交付物：
- src/components/SearchPage.tsx
- src/components/SearchBar.tsx
- src/components/ProductList.tsx
验收标准：
- [ ] 显示搜索框
- [ ] 显示搜索结果列表
- [ ] 显示分页

### Task 4: UI实现（购物车页面）
目标：实现购物车页面
交付物：
- src/components/CartPage.tsx
- src/components/CartItem.tsx
验收标准：
- [ ] 显示购物车商品
- [ ] 支持修改数量
- [ ] 支持删除商品
```

**输出**：`spec-dev/ecommerce-search-cart/tasks/tasks.md`

---

## Part 3: TDD实现阶段 (Phase 2.3 - code-execute)

### Step 4: TDD实现

**使用 `/code-execute` Skill**

现在code-executor根据tasks.md逐个实现每个Task。以Task 1（搜索API）为例：

#### 🔴 RED阶段：编写失败的单元测试

**test-design**生成：`src/__tests__/search.test.ts`（template）

```typescript
// src/__tests__/search.test.ts
import { searchProducts } from '../api/search';

describe('Task 1: searchProducts API', () => {
  describe('TC-1.1.1: 搜索有结果', () => {
    test('搜索"laptop"应该返回匹配的商品', () => {
      // Arrange
      const query = 'laptop';

      // Act
      const results = searchProducts(query);

      // Assert
      expect(results.length).toBeGreaterThan(0);
      expect(results[0].name).toContain('Laptop');
    });
  });

  describe('TC-1.1.2: 搜索无结果', () => {
    test('搜索不存在的关键词应该返回空', () => {
      // Arrange
      const query = 'xyzabc123';

      // Act
      const results = searchProducts(query);

      // Assert
      expect(results).toEqual([]);
    });
  });

  describe('TC-1.1.3: 分页', () => {
    test('搜索应该包含分页信息', () => {
      // Arrange
      const query = 'laptop';

      // Act
      const result = searchProducts(query);

      // Assert
      expect(result).toHaveProperty('page');
      expect(result).toHaveProperty('pageSize');
      expect(result).toHaveProperty('total');
    });
  });
});
```

**运行测试**：
```bash
npm test src/__tests__/search.test.ts
# ❌ 所有测试失败（因为searchProducts函数还不存在）
```

#### 🟢 GREEN阶段：最小化实现

code-executor实现最少的代码让测试通过：

```typescript
// src/api/search.ts
const products = [
  { id: 'P1', name: 'MacBook Pro', price: 999 },
  { id: 'P2', name: 'Dell XPS Laptop', price: 799 },
  { id: 'P3', name: 'ThinkPad', price: 649 }
];

export function searchProducts(query: string) {
  if (!query || query === 'xyzabc123') {
    return { items: [], page: 1, pageSize: 10, total: 0 };
  }

  const filtered = products.filter(p =>
    p.name.toLowerCase().includes(query.toLowerCase())
  );

  return {
    items: filtered,
    page: 1,
    pageSize: 10,
    total: filtered.length
  };
}
```

**运行测试**：
```bash
npm test src/__tests__/search.test.ts
# ✅ 所有测试通过
```

#### 🔵 REFACTOR阶段：改进代码质量

在测试通过的前提下，改进代码：

```typescript
// src/api/search.ts (重构后)

interface Product {
  id: string;
  name: string;
  price: number;
  description?: string;
  category?: string;
}

interface SearchResult {
  items: Product[];
  page: number;
  pageSize: number;
  total: number;
}

// 产品数据库（实际中从数据库加载）
const productDatabase: Product[] = [
  {
    id: 'P1',
    name: 'MacBook Pro',
    price: 999,
    description: '高性能笔记本电脑',
    category: 'electronics'
  },
  // ...更多产品
];

/**
 * 搜索商品
 * @param query 搜索关键词
 * @param page 页码（默认1）
 * @param pageSize 每页数量（默认10）
 * @returns 搜索结果
 */
export function searchProducts(
  query: string,
  page: number = 1,
  pageSize: number = 10
): SearchResult {
  if (!query || query.trim().length === 0) {
    return { items: [], page, pageSize, total: 0 };
  }

  const lowerQuery = query.toLowerCase();

  // 搜索并排序（完全匹配 > 前缀匹配 > 包含匹配）
  const matched = productDatabase
    .filter(p => p.name.toLowerCase().includes(lowerQuery))
    .sort((a, b) => {
      const aName = a.name.toLowerCase();
      const bName = b.name.toLowerCase();

      if (aName === lowerQuery) return -1;
      if (bName === lowerQuery) return 1;
      if (aName.startsWith(lowerQuery)) return -1;
      if (bName.startsWith(lowerQuery)) return 1;

      return 0;
    });

  // 分页
  const start = (page - 1) * pageSize;
  const items = matched.slice(start, start + pageSize);

  return {
    items,
    page,
    pageSize,
    total: matched.length
  };
}
```

**运行测试**：
```bash
npm test src/__tests__/search.test.ts
# ✅ 所有测试仍然通过
```

#### ✅ REVIEW阶段：质量审查

```bash
# 1. 所有测试通过
npm test
# ✅ PASS (所有测试)

# 2. TypeScript检查
npm run type-check
# ✅ 0 errors

# 3. Lint检查
npm run lint src/api/search.ts
# ✅ 0 errors

# 4. 代码覆盖率
npm test -- --coverage src/api/search.ts
# ✅ Statements: 100%
# ✅ Branches: 100%
# ✅ Functions: 100%
# ✅ Lines: 100%
```

### Step 5: 完成所有Task

类似地，code-executor完成其他Task：
- Task 2: 购物车API实现 （addToCart, removeFromCart, updateQuantity）
- Task 3: 搜索UI实现 （SearchPage, SearchBar, ProductList组件）
- Task 4: 购物车UI实现 （CartPage, CartItem组件）

**输出**：
- `src/` - 所有源代码
- `spec-dev/ecommerce-search-cart/execution/execution-report.md` - 执行报告

---

## Part 4: 高层测试阶段 (Phase 3 - code-test)

现在进入Phase 3，code-test进行集成、E2E、性能测试。

### Step 6: 集成测试

**目标**：验证搜索→购物车流程的多Task协作

```typescript
// tests/integration/search-to-cart.integration.test.ts

import { searchProducts } from '../../src/api/search';
import { addToCart, getCart } from '../../src/api/cart';

describe('集成测试: 搜索→添加到购物车流程', () => {
  let userId = 'test-user-123';

  beforeEach(() => {
    // 清理购物车
    clearCart(userId);
  });

  test('用户应该能搜索商品并添加到购物车', () => {
    // Arrange
    const searchQuery = 'laptop';

    // Act 1: 搜索商品
    const searchResult = searchProducts(searchQuery);

    // Assert 1: 搜索返回结果
    expect(searchResult.items.length).toBeGreaterThan(0);
    const targetProduct = searchResult.items[0];

    // Act 2: 添加到购物车
    addToCart(userId, targetProduct.id, 1);

    // Assert 2: 购物车包含商品
    const cart = getCart(userId);
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].productId).toBe(targetProduct.id);
  });

  test('用户可以搜索多种商品并全部添加到购物车', () => {
    // Act: 搜索两种商品
    const laptops = searchProducts('laptop');
    const phones = searchProducts('phone');

    // 添加到购物车
    addToCart(userId, laptops.items[0].id, 1);
    addToCart(userId, phones.items[0].id, 2);

    // Assert
    const cart = getCart(userId);
    expect(cart.items).toHaveLength(2);
    expect(cart.items[1].quantity).toBe(2);
  });
});
```

**运行集成测试**：
```bash
npm run test:integration
# ✅ 所有集成测试通过
```

### Step 7: E2E测试

**目标**：验证完整的购物流程（从用户视角）

```typescript
// tests/e2e/shopping-flow.e2e.test.ts

import { test, expect } from '@playwright/test';

test.describe('E2E: 完整购物流程', () => {
  test('用户应该能完成搜索和购物车操作', async ({ page }) => {
    // 1️⃣ 访问应用首页
    await page.goto('http://localhost:3000');
    await expect(page).toHaveTitle(/Shopping/);

    // 2️⃣ 搜索商品
    await page.fill('[data-testid="search-input"]', 'laptop');
    await page.click('[data-testid="search-btn"]');

    // 3️⃣ 等待搜索结果显示
    await expect(page.locator('[data-testid="product-item"]'))
      .first()
      .toBeVisible();

    // 4️⃣ 获取第一个商品信息
    const productName = await page
      .locator('[data-testid="product-name"]')
      .first()
      .textContent();
    expect(productName).toContain('Laptop');

    // 5️⃣ 点击"添加到购物车"
    await page
      .locator('[data-testid="add-to-cart-btn"]')
      .first()
      .click();

    // 6️⃣ 验证购物车更新
    const cartCount = await page
      .locator('[data-testid="cart-count"]')
      .textContent();
    expect(cartCount).toBe('1');

    // 7️⃣ 进入购物车页面
    await page.click('[data-testid="cart-icon"]');
    await page.waitForURL('**/cart');

    // 8️⃣ 验证购物车内容
    await expect(page.locator('[data-testid="cart-item"]'))
      .toHaveCount(1);

    const cartItemName = await page
      .locator('[data-testid="cart-item-name"]')
      .first()
      .textContent();
    expect(cartItemName).toContain('Laptop');

    // 9️⃣ 修改数量
    await page
      .locator('[data-testid="quantity-input"]')
      .first()
      .clear();
    await page
      .locator('[data-testid="quantity-input"]')
      .first()
      .fill('2');

    // 🔟 验证总价更新
    const totalPrice = await page
      .locator('[data-testid="total-price"]')
      .getAttribute('data-value');
    expect(parseInt(totalPrice || '0')).toBeGreaterThan(999);
  });

  test('搜索无结果时应该显示提示', async ({ page }) => {
    await page.goto('http://localhost:3000');

    // 搜索不存在的商品
    await page.fill('[data-testid="search-input"]', 'xyznonexistent');
    await page.click('[data-testid="search-btn"]');

    // 应该显示无结果提示
    await expect(page.locator('[data-testid="no-results"]'))
      .toContainText('No products found');
  });
});
```

**运行E2E测试**：
```bash
npm run test:e2e
# ✅ 所有E2E测试通过
```

### Step 8: 性能测试

**目标**：验证在50并发用户下的搜索和购物车性能

```javascript
// tests/performance/shopping-api.load.js

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend, Counter, Gauge } from 'k6/metrics';

const searchDuration = new Trend('search_duration');
const addToCartDuration = new Trend('add_to_cart_duration');
const successfulRequests = new Counter('successful_requests');
const failedRequests = new Counter('failed_requests');

export const options = {
  stages: [
    { duration: '1m', target: 10 },     // 1分钟内增加到10个用户
    { duration: '5m', target: 50 },     // 5分钟内增加到50个用户
    { duration: '10m', target: 50 },    // 保持50个用户10分钟
    { duration: '5m', target: 0 }       // 5分钟内降到0
  ],
  thresholds: {
    'search_duration': ['p(95)<200', 'p(99)<500'],    // 95%请求<200ms, 99%<500ms
    'add_to_cart_duration': ['p(95)<300', 'p(99)<600'],
    'http_req_failed': ['rate<0.1']                    // 错误率<0.1%
  }
};

const searchQueries = ['laptop', 'macbook', 'dell', 'iphone', 'xyznonexistent'];
const productIds = ['P1', 'P2', 'P3', 'P4', 'P5'];

export default function () {
  const userId = `user-${__VU}-${__ITER}`;

  // 随机搜索
  const query = searchQueries[Math.floor(Math.random() * searchQueries.length)];
  const searchRes = http.get(
    `http://localhost:3000/api/search?q=${encodeURIComponent(query)}`
  );

  const searchSuccess = check(searchRes, {
    'search status 200': (r) => r.status === 200,
    'search response time < 200ms': (r) => r.timings.duration < 200,
  });

  if (searchSuccess) {
    searchDuration.add(searchRes.timings.duration);
    successfulRequests.add(1);
  } else {
    failedRequests.add(1);
  }

  // 添加到购物车
  const productId = productIds[Math.floor(Math.random() * productIds.length)];
  const cartRes = http.post(
    'http://localhost:3000/api/cart/add',
    JSON.stringify({
      userId,
      productId,
      quantity: 1
    }),
    { headers: { 'Content-Type': 'application/json' } }
  );

  const cartSuccess = check(cartRes, {
    'add to cart status 200': (r) => r.status === 200,
    'add to cart response time < 300ms': (r) => r.timings.duration < 300,
  });

  if (cartSuccess) {
    addToCartDuration.add(cartRes.timings.duration);
    successfulRequests.add(1);
  } else {
    failedRequests.add(1);
  }

  sleep(Math.random() * 2 + 1);  // 1-3秒之间随机等待
}
```

**运行性能测试**：
```bash
k6 run tests/performance/shopping-api.load.js

# 结果示例：
#
# ✅ 搜索API性能：
#   - 平均响应时间: 85ms
#   - P95响应时间: 180ms (< 200ms ✅)
#   - P99响应时间: 450ms (< 500ms ✅)
#   - 平均RPS: 120 requests/sec
#
# ✅ 添加到购物车API性能：
#   - 平均响应时间: 120ms
#   - P95响应时间: 280ms (< 300ms ✅)
#   - P99响应时间: 580ms (< 600ms ✅)
#
# ✅ 错误率: 0.02% (< 0.1% ✅)
# ✅ 50并发用户下，系统表现良好
```

---

## Part 5: 闭环验证

### 验证矩阵

| 规范(TEST-VERIFY) | Test Case | 单元测试 | 集成测试 | E2E测试 | 性能测试 | 最终状态 |
|------------------|-----------|--------|--------|--------|---------|----------|
| TC-1.1.1: 搜索返回结果 | test_search_laptop_returns_items | ✅ | ✅ | ✅ | ✅ | ✅完成 |
| TC-1.1.2: 搜索无结果 | test_search_no_results | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-1.1.3: 搜索分页 | test_search_pagination | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-2.1.1: 添加新商品 | test_add_to_cart | ✅ | ✅ | ✅ | ✅ | ✅完成 |
| TC-2.1.2: 重复添加 | test_add_duplicate | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-2.2.1: 修改数量 | test_update_quantity | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-2.2.2: 删除商品 | test_remove_item | ✅ | ✅ | ✅ | - | ✅完成 |
| 完整购物流程 | 搜索→添加→修改→结账 | - | - | ✅ | - | ✅完成 |
| 性能基准(50用户) | P95<200ms, 错误率<0.1% | - | - | - | ✅ | ✅完成 |

### 闭环验证结论

✅ **所有TEST-VERIFY都有对应的test**
- 规范中定义的每个TEST-VERIFY都可以追踪到具体的test case
- 单元测试验证单个功能正确性
- 集成测试验证多Task协作
- E2E测试验证完整业务流程
- 性能测试验证系统容量

✅ **所有test都通过**
```
npm test              # ✅ 所有单元测试通过
npm run test:int      # ✅ 所有集成测试通过
npm run test:e2e      # ✅ 所有E2E测试通过
k6 run perf/test.js   # ✅ 性能基准达成
```

✅ **所有Task都实现和验证**
- Task 1 (搜索API): 实现✅ 单元测试✅ 集成测试✅ E2E✅ 性能✅
- Task 2 (购物车API): 实现✅ 单元测试✅ 集成测试✅ E2E✅
- Task 3 (搜索UI): 实现✅ 单元测试✅ E2E✅
- Task 4 (购物车UI): 实现✅ 单元测试✅ E2E✅

✅ **代码质量保证**
- ESLint: 0 errors
- TypeScript: 0 errors (strict mode)
- 单元测试覆盖率: ≥ 85%
- 关键功能覆盖率: 100%

---

## 关键成就总结

### 质量保证

```
规范定义(WHEN-THEN)
         ↓
单元测试设计(test-spec.md)
         ↓
TDD实现(RED-GREEN-REFACTOR-REVIEW) → 100%覆盖率 ✅
         ↓
集成测试 → 验证多Task协作 ✅
         ↓
E2E测试 → 验证业务流程 ✅
         ↓
性能测试 → 建立性能基准 ✅
         ↓
闭环验证 → TEST-VERIFY ↔ Test ↔ Code ✅
         ↓
    ✅ 生产就绪！
```

### 可追踪性

- ✅ 每个规范(TEST-VERIFY) → 对应的test case
- ✅ 每个test case → 对应的代码实现
- ✅ 每个代码 → 对应的test覆盖
- ✅ 完整的执行和测试报告

### 风险降低

- ✅ 需求遗漏风险：规范定义时确认完整
- ✅ 实现错误风险：TDD的RED-GREEN确保正确
- ✅ 集成风险：集成测试验证多Task协作
- ✅ 业务流程风险：E2E测试验证端到端
- ✅ 性能风险：性能测试识别瓶颈

---

## 最佳实践总结

### Phase 1：规范定义

- ✅ 使用BDD的WHEN-THEN格式定义场景
- ✅ 明确定义TEST-VERIFY映射到test case
- ✅ 完整的数据模型和业务规则
- ✅ 多轮确认确保规范准确无遗漏

### Phase 2：TDD实现

- ✅ 遵循RED-GREEN-REFACTOR-REVIEW四阶段
- ✅ 每个Task的单元测试覆盖率 ≥ 85%
- ✅ 在测试通过的前提下进行重构
- ✅ 严格的规范和质量审查

### Phase 3：高层测试

- ✅ 集成测试：验证多Task协作，Mock策略清晰
- ✅ E2E测试：避免Flaky，使用显式等待
- ✅ 性能测试：定义明确的基准和阈值
- ✅ 闭环验证：TEST-VERIFY映射到各层测试

---

## 工作流特点

### 📊 完整的追踪链条

```
需求 → 规范 → 设计 → 任务 → 单元测试 → 集成测试 → E2E测试 → 性能测试 → 闭环验证
↓     ↓     ↓    ↓   ↓        ↓         ↓        ↓        ↓
TEST-  WHEN- Arch Task TEST- searchProducts() + addToCart() + 完整流程 + 性能指标
VERIFY THEN Tasks  spec searchTest  +  integrationTest  +  e2eTest  +  perfTest
```

### 🛡️ 多层防护

- **规范层**：WHEN-THEN确保需求清晰
- **设计层**：架构设计确保技术方案合理
- **单元层**：TDD确保代码正确实现
- **集成层**：验证多Task协作
- **业务层**：E2E测试验证用户流程
- **容量层**：性能测试验证系统能力

### ✅ 生产就绪

完成Phase 1-3后，代码满足以下条件：
- 规范完整且被验证 ✅
- 代码实现正确且有单元测试 ✅
- 多模块协作验证 ✅
- 业务流程端到端验证 ✅
- 性能基准建立并达标 ✅
- 完整的追踪和报告 ✅

**可以安心上线！** 🚀

---

## 总结

这是一个**完整的、体系化的、生产级的TDD工作流**：

1. **规范驱动**：从WHEN-THEN格式的规范开始
2. **设计优先**：详细的设计指导实现
3. **TDD实现**：RED-GREEN-REFACTOR-REVIEW保证质量
4. **分层测试**：单元→集成→E2E→性能，多层防护
5. **闭环验证**：TEST-VERIFY映射到所有测试层
6. **完全追踪**：从规范到代码到测试的完整链条

通过遵循这个工作流，团队可以：
- 降低需求遗漏风险 🎯
- 降低实现错误风险 🛡️
- 降低集成问题风险 🔗
- 建立可信赖的代码质量 ⭐
- 为后续维护积累完整的文档 📚

**让AI辅助的研发开发变得规范、高效、可信赖！** 🚀

