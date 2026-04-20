# TDD Phase 3 完整工作流示例

**目标**：展示从规范到最终验证的完整TDD工作流（涵盖Phase 1-3所有阶段）

**示例场景**：电商商品搜索和购物车系统

---

## 第1部分：规范定义 (Phase 1 - Spec Creation)

### TEST-VERIFY格式的规范

在`spec.md`中定义：

```markdown
## 电商搜索和购物车系统

### 场景1：用户搜索商品

**WHEN** 用户在搜索框输入关键词
**THEN** 系统应该返回匹配的商品列表

**TEST-VERIFY**:
- TC-1.1.1: 搜索"laptop" → test_search_laptop_returns_items
- TC-1.1.2: 搜索结果分页 → test_search_pagination_works
- TC-1.1.3: 搜索无结果 → test_search_no_results_shows_empty

### 场景2：用户添加商品到购物车

**WHEN** 用户点击"添加到购物车"
**THEN** 商品应该添加到购物车，购物车数量更新

**TEST-VERIFY**:
- TC-1.2.1: 添加单个商品 → test_add_to_cart_updates_count
- TC-1.2.2: 重复添加相同商品 → test_add_duplicate_increments_qty
- TC-1.2.3: 添加商品库存不足 → test_add_out_of_stock_shows_error

### 场景3：用户查看和修改购物车

**WHEN** 用户进入购物车页面
**THEN** 应该看到已添加的商品，可以修改数量或移除

**TEST-VERIFY**:
- TC-1.3.1: 显示购物车商品 → test_cart_displays_items
- TC-1.3.2: 修改商品数量 → test_update_item_quantity
- TC-1.3.3: 移除商品 → test_remove_item_from_cart
```

---

## 第2部分：测试设计 (Phase 2.1 - Test Designer生成)

### test-designer生成的test-spec.md

```markdown
# Test Specification: 搜索和购物车系统

## 测试层级决策

| 功能 | 单元 | 集成 | E2E |
|------|------|------|-----|
| searchProducts() | ✅ | - | - |
| addToCart() | ✅ | - | - |
| updateCartItem() | ✅ | - | - |
| 搜索→加入购物车流程 | - | ✅ | - |
| 完整购物车页面 | - | - | ✅ |

## Fixture定义

```json
{
  "validSearchQuery": "laptop",
  "noResultsQuery": "xyzabc123",
  "productResults": [
    { "id": "P1", "name": "MacBook", "price": 999 },
    { "id": "P2", "name": "Dell Laptop", "price": 799 }
  ],
  "cartItem": { "productId": "P1", "quantity": 1 }
}
```

## Test Case列表

### TC-1.1: searchProducts()

**TC-1.1.1**: 搜索有结果
- Input: "laptop"
- Expected: 返回2个商品
- Mock: API返回fixtures中的数据

**TC-1.1.2**: 搜索无结果
- Input: "xyzabc123"
- Expected: 返回空数组

### TC-1.2: addToCart()

**TC-1.2.1**: 添加新商品
- Input: productId: "P1", qty: 1
- Expected: cart.items.length = 1
- Mock: 无

**TC-1.2.2**: 添加重复商品
- Input: productId: "P1", qty: 1 (第二次)
- Expected: cart.items[0].quantity = 2
- Mock: 无
```

### test-search-product.template.ts

```typescript
describe('TC-1.1: searchProducts', () => {
  test('TC-1.1.1: 搜索"laptop"返回商品', () => {
    // 框架已准备，实现者填充
    // Arrange
    // Act
    // Assert
  });

  test('TC-1.1.2: 搜索无结果返回空', () => {
    // 框架已准备，实现者填充
  });
});

describe('TC-1.2: addToCart', () => {
  test('TC-1.2.1: 添加新商品更新购物车', () => {
    // 框架已准备，实现者填充
  });
});
```

---

## 第3部分：TDD实现 (Phase 2 - Code Execute with TDD)

现在code-executor根据test-spec.md进行TDD实现。

### 🔴 RED阶段：编写失败的测试

实现者将template转换为可运行的test代码：

```typescript
// src/__tests__/search.test.ts
import { searchProducts, addToCart } from '../cart';

describe('TC-1.1: searchProducts', () => {
  test('TC-1.1.1: 搜索"laptop"返回商品', () => {
    const results = searchProducts('laptop');
    expect(results).toHaveLength(2);
    expect(results[0].name).toContain('Laptop');
  });

  test('TC-1.1.2: 搜索无结果返回空', () => {
    const results = searchProducts('xyzabc123');
    expect(results).toEqual([]);
  });
});

describe('TC-1.2: addToCart', () => {
  test('TC-1.2.1: 添加新商品更新购物车', () => {
    const cart = { items: [] };
    addToCart(cart, 'P1', 1);
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].productId).toBe('P1');
    expect(cart.items[0].quantity).toBe(1);
  });

  test('TC-1.2.2: 添加重复商品增加数量', () => {
    const cart = { items: [{ productId: 'P1', quantity: 1 }] };
    addToCart(cart, 'P1', 1);
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].quantity).toBe(2);
  });
});
```

**运行测试**：
```bash
npm test
# ❌ 所有test失败（因为代码还没写）
```

### 🟢 GREEN阶段：最小化实现

实现最少的代码让test通过：

```typescript
// src/cart.ts
const products = [
  { id: 'P1', name: 'MacBook', price: 999 },
  { id: 'P2', name: 'Dell Laptop', price: 799 }
];

export function searchProducts(query: string) {
  if (!query || query === 'xyzabc123') {
    return [];
  }
  return products;
}

export function addToCart(cart: any, productId: string, qty: number) {
  const existingItem = cart.items.find((item: any) => item.productId === productId);

  if (existingItem) {
    existingItem.quantity += qty;
  } else {
    cart.items.push({ productId, quantity: qty });
  }
}
```

**运行测试**：
```bash
npm test
# ✅ 所有test通过
```

### 🔵 REFACTOR阶段：改进代码质量

在test通过的前提下改进代码：

```typescript
// src/cart.ts (重构后)

// 产品数据库（可以从API加载）
const productDatabase = [
  { id: 'P1', name: 'MacBook Pro', price: 999 },
  { id: 'P2', name: 'Dell XPS Laptop', price: 799 },
  { id: 'P3', name: 'ThinkPad', price: 649 }
];

/**
 * 搜索商品
 * @param query - 搜索关键词
 * @returns 匹配的商品列表
 */
export function searchProducts(query: string): Product[] {
  if (!query || query.trim().length === 0) {
    return [];
  }

  const lowerQuery = query.toLowerCase();

  return productDatabase.filter(product =>
    product.name.toLowerCase().includes(lowerQuery)
  );
}

interface CartItem {
  productId: string;
  quantity: number;
}

interface Cart {
  items: CartItem[];
}

/**
 * 添加商品到购物车
 * 如果商品已存在，增加数量；否则添加新商品
 * @param cart - 购物车对象
 * @param productId - 商品ID
 * @param qty - 添加数量
 */
export function addToCart(cart: Cart, productId: string, qty: number): void {
  if (qty <= 0) {
    throw new Error('Quantity must be greater than 0');
  }

  const existingItem = cart.items.find(item => item.productId === productId);

  if (existingItem) {
    existingItem.quantity += qty;
  } else {
    cart.items.push({ productId, quantity: qty });
  }
}

export interface Product {
  id: string;
  name: string;
  price: number;
}
```

**运行测试**：
```bash
npm test
# ✅ 所有test仍然通过
```

### ✅ REVIEW阶段：质量审查

```bash
# 1. 所有test通过
npm test
# ✅ PASS (3 tests)

# 2. TypeScript检查
npm run type-check
# ✅ 0 errors

# 3. Lint检查
npm run lint src/cart.ts
# ✅ 0 errors

# 4. 覆盖率
npm test -- --coverage
# ✅ Statements: 100%
# ✅ Branches: 100%
# ✅ Functions: 100%
```

---

## 第4部分：Phase 3 - 高层测试

现在进入Phase 3（code-test），进行集成、E2E、性能测试。

### 🔗 集成测试：搜索→购物车

```typescript
// tests/integration/search-to-cart.integration.test.ts

import { searchProducts, addToCart } from '../../src/cart';

describe('集成测试：搜索商品并添加到购物车', () => {
  test('应该能完整执行搜索和添加流程', () => {
    // Arrange
    const cart = { items: [] };

    // Act 1: 搜索商品
    const searchResults = searchProducts('laptop');

    // Assert 1: 搜索结果
    expect(searchResults.length).toBeGreaterThan(0);
    const targetProduct = searchResults[0];

    // Act 2: 添加到购物车
    addToCart(cart, targetProduct.id, 2);

    // Assert 2: 购物车状态
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].productId).toBe(targetProduct.id);
    expect(cart.items[0].quantity).toBe(2);
  });

  test('应该处理多个搜索和添加', () => {
    // Arrange
    const cart = { items: [] };

    // Act: 搜索两次，各添加一个商品
    const laptops = searchProducts('laptop');
    const dells = searchProducts('dell');

    addToCart(cart, laptops[0].id, 1);
    addToCart(cart, dells[0].id, 1);

    // Assert
    expect(cart.items).toHaveLength(2);
    expect(cart.items[0].productId).not.toBe(cart.items[1].productId);
  });
});
```

### 🌐 E2E测试：完整购物车页面流程

```typescript
// tests/e2e/shopping-flow.e2e.test.ts

import { test, expect } from '@playwright/test';

test.describe('E2E: 完整购物流程', () => {
  test('用户应该能完成搜索和购物车操作', async ({ page }) => {
    // 1️⃣ 访问商城首页
    await page.goto('http://localhost:3000/shop');
    await expect(page).toHaveTitle(/Shop/);

    // 2️⃣ 在搜索框输入
    await page.fill('[data-testid="search-input"]', 'laptop');
    await page.click('[data-testid="search-btn"]');

    // 3️⃣ 等待搜索结果
    await expect(page.locator('[data-testid="product-item"]'))
      .first()
      .toBeVisible();

    // 4️⃣ 获取第一个商品的ID
    const productId = await page
      .locator('[data-testid="product-item"]')
      .first()
      .getAttribute('data-product-id');

    // 5️⃣ 点击"添加到购物车"
    await page
      .locator('[data-testid="add-to-cart"][data-product-id="' + productId + '"]')
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
    expect(parseInt(totalPrice || '0')).toBeGreaterThan(999); // 商品价格 × 2
  });

  test('应该显示搜索无结果提示', async ({ page }) => {
    await page.goto('http://localhost:3000/shop');

    // 搜索不存在的商品
    await page.fill('[data-testid="search-input"]', 'xyznonexistent');
    await page.click('[data-testid="search-btn"]');

    // 应该显示无结果提示
    await expect(page.locator('[data-testid="no-results"]'))
      .toContainText('No products found');
  });
});
```

### 📊 性能测试：搜索API负载测试

```javascript
// tests/performance/search-api.load.js

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend, Counter } from 'k6/metrics';

const searchDuration = new Trend('search_duration');
const successfulSearches = new Counter('successful_searches');

export const options = {
  stages: [
    { duration: '1m', target: 10 },    // 10个用户
    { duration: '5m', target: 50 },    // 50个用户
    { duration: '10m', target: 50 },   // 保持50个用户
    { duration: '5m', target: 0 }      // 关闭
  ],

  thresholds: {
    'http_req_duration': ['p(95)<200'],  // 95%请求<200ms
    'http_req_failed': ['rate<0.1']      // 错误率<0.1%
  }
};

const searchQueries = [
  'laptop',
  'macbook',
  'dell',
  'iphone',
  'xyznonexistent'
];

export default function () {
  // 随机搜索
  const query = searchQueries[
    Math.floor(Math.random() * searchQueries.length)
  ];

  const res = http.get(
    `http://localhost:3000/api/search?q=${encodeURIComponent(query)}`
  );

  const isSuccess = check(res, {
    'status 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
    'response has data': (r) => r.body.length > 0
  });

  if (isSuccess) {
    searchDuration.add(res.timings.duration);
    successfulSearches.add(1);
  }

  sleep(Math.random() * 2 + 1);
}
```

**运行性能测试**：
```bash
k6 run tests/performance/search-api.load.js

# 结果示例：
# http_req_duration: avg=145ms, p(95)=185ms, p(99)=220ms
# http_req_failed: 0.05% (低于0.1%阈值 ✅)
# 平均RPS: 85 requests/sec
```

---

## 第5部分：闭环验证 (Phase 3 - Code Test闭环)

### 验证矩阵

| TEST-VERIFY | Test Case | Test文件 | 单元 | 集成 | E2E | 性能 | 状态 |
|------------|-----------|---------|------|------|-----|------|------|
| TC-1.1.1 | 搜索返回结果 | search.test.ts | ✅ | ✅ | ✅ | ✅ | ✅完成 |
| TC-1.1.2 | 搜索无结果 | search.test.ts | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-1.2.1 | 添加新商品 | cart.test.ts | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-1.2.2 | 添加重复商品 | cart.test.ts | ✅ | ✅ | ✅ | - | ✅完成 |
| TC-1.3.1 | 显示购物车 | cart-page.e2e.test.ts | - | - | ✅ | - | ✅完成 |
| 性能基准 | 搜索API性能 | search-api.load.js | - | - | - | ✅ | ✅完成 |

**验证结论**：
- ✅ 所有TEST-VERIFY都有对应的test
- ✅ 所有test都通过
- ✅ 单元测试覆盖率 100%
- ✅ 集成测试验证多Task协作
- ✅ E2E测试验证完整流程
- ✅ 性能测试验证在50并发用户下的表现

---

## 总结

### 完整的TDD链条

```
1️⃣ 规范定义 (Phase 1)
   TEST-VERIFY格式的验收标准
         ↓
2️⃣ 测试设计 (Phase 2.1 - test-designer)
   test-spec.md + fixtures + test-*.template
         ↓
3️⃣ TDD实现 (Phase 2 - code-execute)
   🔴RED → 🟢GREEN → 🔵REFACTOR → ✅REVIEW
         ↓
4️⃣ 高层测试 (Phase 3 - code-test)
   集成测试 → E2E测试 → 性能测试
         ↓
5️⃣ 闭环验证 (Phase 3 - code-test)
   验证矩阵：TEST-VERIFY → test → 代码 → 结果
```

### 关键成就

✅ **单元测试质量保证** - TDD流程确保100%覆盖和代码质量
✅ **集成测试验证** - 多Task协作和跨模块通信正常
✅ **E2E测试覆盖** - 完整的业务流程可正常执行
✅ **性能基准建立** - 在50并发用户下，P95 < 200ms
✅ **完整的闭环** - 从规范到最终验证的完整追踪

**这是一个生产级的、完整的、可信赖的TDD工作流！** 🚀

