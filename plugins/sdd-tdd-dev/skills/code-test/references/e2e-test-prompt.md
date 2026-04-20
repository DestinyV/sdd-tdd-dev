# E2E测试提示词

用于指导实现子代理进行端到端(E2E)测试的设计和实现。

> **📌 更新提示**：本项目已配备完整的 Playwright 浏览器测试基础设施。
> - 配置模板：`../templates/playwright.config.ts`
> - 工具函数：`../templates/browser-test-helpers.ts`
> - E2E 测试模板：`../templates/frontend-e2e-test.template.ts`
> - 完整指南：`../references/frontend-browser-testing.md`
> - 自动化脚本：`../scripts/run-browser-tests.sh`
>
> 优先使用上述模板和工具函数，而非从零编写。

---

## 什么是E2E测试

E2E (End-to-End) 测试验证整个应用从用户界面到后端系统的完整业务流程。

**关键特点**：
- **测试范围**：完整的业务场景（从用户操作到最终结果）
- **涉及层级**：UI层 → 业务层 → 数据层 → 外部系统
- **用户视角**：从用户角度验证应用是否正常工作

**三层测试对比**：
```
单元测试    → 单个函数/组件的行为
集成测试    → 多个组件的协作
E2E测试     → 完整的业务流程（用户视角）
```

---

## 何时需要E2E测试

### ✅ 应该做E2E测试

1. **核心业务流程**
   - 用户注册和登录
   - 商品购买和支付
   - 文章发布和分享
   - 订单创建和追踪

2. **跨越前后端**
   - UI交互触发后端操作
   - 后端数据更新反映到UI
   - 用户操作会产生持久化数据

3. **关键用户场景**
   - 首次使用的新用户
   - 常见操作路径
   - 特殊但重要的边界情况

4. **多个功能协作**
   - 用户A和B的交互（如评论）
   - 实时数据更新（如库存）
   - 多步骤工作流（如申请和审批）

### ❌ 不需要E2E测试

- 单个UI组件的显示（用单元测试）
- 纯逻辑函数（用单元测试）
- 两个服务的调用（用集成测试）
- 每一个细节都测（会很慢）

---

## E2E测试设计指南

### 步骤1: 定义用户场景

明确要测试哪个用户故事。

#### 1.1 角色定义

```
系统中有哪些用户角色？
├─ 管理员 → 权限最高，能管理内容和用户
├─ 普通用户 → 可以创建和管理自己的内容
├─ 访客 → 只读权限，无登录
└─ 特殊角色 → 如审核员、开发者等
```

#### 1.2 业务流程定义

**示例：电商购买流程**

```
场景：用户完成购买

1️⃣ 用户行为阶段
   - 访问商城首页
   - 搜索商品
   - 点击商品进入详情页
   - 查看评价和库存
   - 点击"加入购物车"

2️⃣ 购物车阶段
   - 查看购物车
   - 修改数量
   - 移除不需要的商品
   - 点击"结账"

3️⃣ 订单阶段
   - 填写收货地址
   - 选择配送方式
   - 选择支付方式

4️⃣ 支付阶段
   - 进入支付界面（第三方）
   - 完成支付
   - 返回应用

5️⃣ 确认阶段
   - 显示订单确认
   - 发送确认邮件
   - 用户可查看订单列表

验证点：
✅ 订单已创建
✅ 库存已扣减
✅ 支付记录存在
✅ 邮件已发送
✅ 用户界面显示正确
```

### 步骤2: 定义验证点

明确要验证哪些东西。

#### 2.1 UI验证

```typescript
// 验证UI是否反映了业务状态
expect(page).toContainText('Order #12345 confirmed');
expect(page.locator('[data-testid="status"]'))
  .toHaveText('Payment received');
expect(page.locator('[data-testid="total"]'))
  .toHaveValue('$299.99');
```

#### 2.2 数据库验证

```typescript
// 验证后端数据是否正确持久化
const order = await db.orders.findById('123');
expect(order).toMatchObject({
  userId: '456',
  status: 'completed',
  amount: 29999,
  items: [{ sku: 'ITEM1', qty: 2 }]
});
```

#### 2.3 外部系统验证

```typescript
// 验证是否调用了外部服务
expect(emailService.sendMock).toHaveBeenCalledWith({
  to: 'user@example.com',
  subject: 'Order Confirmation'
});

expect(paymentGateway.chargeMock).toHaveBeenCalledWith({
  amount: 29999,
  currency: 'USD'
});
```

### 步骤3: 测试环境准备

#### 3.1 环境配置

```typescript
// 使用测试环境而非生产环境
const config = {
  baseURL: process.env.TEST_BASE_URL || 'http://localhost:3000',
  timeout: 30000,
  headless: true, // 无头浏览器
  slowMo: 0, // 不延迟操作
  retries: 2, // 失败重试
};
```

#### 3.2 初始数据准备

```typescript
// 在测试前设置初始状态
beforeEach(async () => {
  // 清理数据库
  await db.orders.deleteMany({});
  await db.users.deleteMany({});

  // 创建测试用户
  const user = await db.users.create({
    email: 'test@example.com',
    password: bcrypt.hash('password123'),
    balance: 10000 // 可用余额
  });

  // 创建测试商品
  await db.products.create({
    id: 'ITEM1',
    name: 'Test Product',
    price: 29999,
    stock: 100
  });

  testContext.userId = user.id;
});
```

#### 3.3 浏览器和设备配置

```typescript
// 测试多个浏览器
const browsers = ['chromium', 'firefox', 'webkit'];

// 测试多个设备
const devices = {
  desktop: { width: 1280, height: 720 },
  mobile: { width: 375, height: 667 },
  tablet: { width: 768, height: 1024 }
};
```

---

## E2E测试实现

### 工具选择

| 工具 | 推荐场景 | 特点 |
|------|---------|------|
| Cypress | SPA应用、快速反馈 | JavaScript优先，快速 |
| Playwright | 跨浏览器、性能 | 多浏览器，并发强 |
| Puppeteer | Chrome自动化 | 轻量级，Chrome专用 |
| WebdriverIO | 完整测试套件 | Selenium基础，多语言 |
| Nightwatch | 简洁API | 学习曲线低 |

### 实现模板

#### Playwright示例（推荐）

```typescript
// tests/e2e/checkout.e2e.test.ts

import { test, expect, Page } from '@playwright/test';

test.describe('E2E: 购物车结账流程', () => {
  let page: Page;
  let testContext: any;

  test.beforeEach(async ({ browser, context }) => {
    // 初始化页面
    page = await context.newPage();

    // 初始化测试数据
    await initTestData();

    // 登录用户
    await page.goto('http://localhost:3000/login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-btn"]');
    await page.waitForURL('**/dashboard');

    testContext = {
      page,
      userId: 'test-user-123'
    };
  });

  test.afterEach(async () => {
    await page.close();
    await cleanupTestData();
  });

  test('应该能完成购物流程', async () => {
    const { page } = testContext;

    // 1️⃣ 访问商城
    await page.goto('http://localhost:3000/shop');
    await expect(page).toHaveTitle(/Shop/);

    // 2️⃣ 搜索商品
    await page.fill('[data-testid="search"]', 'Test Product');
    await page.click('[data-testid="search-btn"]');

    // 等待搜索结果
    await page.waitForSelector('[data-testid="product-item"]');
    const productCount = await page.locator('[data-testid="product-item"]')
      .count();
    expect(productCount).toBeGreaterThan(0);

    // 3️⃣ 点击商品
    await page.click('[data-testid="product-item"]:first-child');
    await page.waitForURL('**/products/*');

    // 4️⃣ 查看商品详情
    const title = await page.locator('[data-testid="product-title"]')
      .textContent();
    expect(title).toContain('Test Product');

    const price = await page.locator('[data-testid="product-price"]')
      .getAttribute('data-value');
    expect(price).toBe('29999');

    // 5️⃣ 加入购物车
    await page.fill('[data-testid="quantity"]', '2');
    await page.click('[data-testid="add-to-cart"]');

    // 验证购物车更新
    const cartCount = await page.locator('[data-testid="cart-count"]')
      .textContent();
    expect(cartCount).toBe('2');

    // 6️⃣ 进入购物车
    await page.click('[data-testid="cart-icon"]');
    await page.waitForURL('**/cart');

    // 7️⃣ 验证购物车内容
    const cartItems = await page.locator('[data-testid="cart-item"]')
      .count();
    expect(cartItems).toBe(1);

    const itemTotal = await page.locator('[data-testid="item-total"]')
      .getAttribute('data-value');
    expect(itemTotal).toBe('59998'); // 29999 * 2

    // 8️⃣ 结账
    await page.click('[data-testid="checkout-btn"]');
    await page.waitForURL('**/checkout');

    // 9️⃣ 填写收货地址
    await page.fill('[data-testid="address"]', '123 Main St');
    await page.fill('[data-testid="city"]', 'Springfield');
    await page.fill('[data-testid="zip"]', '12345');

    // 🔟 选择配送方式
    await page.click('[data-testid="shipping-standard"]');

    // 1️⃣1️⃣ 选择支付方式
    await page.click('[data-testid="payment-card"]');

    // 1️⃣2️⃣ 完成支付
    await page.click('[data-testid="pay-btn"]');

    // 等待支付处理（可能跳转到第三方）
    await page.waitForNavigation();

    // 1️⃣3️⃣ 验证确认页面
    await expect(page).toHaveURL('**/order-confirmation');
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('Order confirmed');

    const orderNumber = await page.locator('[data-testid="order-number"]')
      .textContent();
    expect(orderNumber).toMatch(/^ORD-\d+$/);

    // 1️⃣4️⃣ 验证数据库
    const order = await db.orders.findOne({ orderNumber });
    expect(order).toMatchObject({
      userId: testContext.userId,
      status: 'completed',
      totalAmount: 59998,
      itemCount: 1
    });

    // 1️⃣5️⃣ 验证邮件已发送
    const emails = await emailService.getReceivedEmails(
      'test@example.com'
    );
    const confirmationEmail = emails.find(e =>
      e.subject.includes('Order Confirmation')
    );
    expect(confirmationEmail).toBeDefined();
    expect(confirmationEmail?.body).toContain(orderNumber);
  });

  test('应该处理库存不足', async () => {
    const { page } = testContext;

    // 前置条件：设置商品库存为1
    await db.products.update(
      { id: 'ITEM1' },
      { stock: 1 }
    );

    // 访问商品页面
    await page.goto('http://localhost:3000/products/ITEM1');

    // 尝试加入2个
    await page.fill('[data-testid="quantity"]', '2');
    await page.click('[data-testid="add-to-cart"]');

    // 验证错误提示
    await expect(page.locator('[data-testid="error"]'))
      .toContainText('Only 1 item available');
  });

  test('应该能处理并发购买', async () => {
    // 前置条件：库存为2
    await db.products.update(
      { id: 'ITEM1' },
      { stock: 2 }
    );

    // 模拟两个用户同时购买
    const user1Page = testContext.page;
    const user2Browser = await chromium.launch();
    const user2Page = await user2Browser.newPage();

    // User 1 加入购物车
    await user1Page.goto('http://localhost:3000/products/ITEM1');
    await user1Page.fill('[data-testid="quantity"]', '1');
    await user1Page.click('[data-testid="add-to-cart"]');

    // User 2 加入购物车
    await user2Page.goto('http://localhost:3000/products/ITEM1');
    await user2Page.fill('[data-testid="quantity"]', '1');
    await user2Page.click('[data-testid="add-to-cart"]');

    // 两个都应该成功
    expect(
      await user1Page.locator('[data-testid="cart-count"]').textContent()
    ).toBe('1');
    expect(
      await user2Page.locator('[data-testid="cart-count"]').textContent()
    ).toBe('1');

    // 库存应该变为0
    const product = await db.products.findById('ITEM1');
    expect(product.stock).toBe(0);

    await user2Browser.close();
  });

  test('应该能在支付失败后重试', async () => {
    const { page } = testContext;

    // Mock支付失败的第一次，然后成功
    let paymentAttempts = 0;
    await mockPaymentGateway({
      charge: async (amount) => {
        paymentAttempts++;
        if (paymentAttempts === 1) {
          throw new Error('Payment declined');
        }
        return { status: 'success' };
      }
    });

    // 进行购买流程
    await page.goto('http://localhost:3000/shop');
    await page.click('[data-testid="product-item"]:first-child');
    await page.click('[data-testid="add-to-cart"]');
    await page.click('[data-testid="cart-icon"]');
    await page.click('[data-testid="checkout-btn"]');

    // 填写信息并支付
    await page.fill('[data-testid="address"]', '123 Main St');
    await page.click('[data-testid="pay-btn"]');

    // 应该看到支付失败提示
    await expect(page.locator('[data-testid="error"]'))
      .toContainText('Payment failed');

    // 用户点击重试
    await page.click('[data-testid="retry-btn"]');

    // 这次应该成功
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('Order confirmed');
  });
});
```

#### Cypress示例

```typescript
// cypress/e2e/checkout.cy.ts

describe('Checkout Flow', () => {
  beforeEach(() => {
    // 初始化数据
    cy.initTestData();

    // 登录
    cy.visit('/login');
    cy.get('[data-testid="email"]').type('test@example.com');
    cy.get('[data-testid="password"]').type('password123');
    cy.get('[data-testid="login-btn"]').click();
    cy.url().should('include', '/dashboard');
  });

  it('should complete purchase', () => {
    // 访问商城
    cy.visit('/shop');

    // 搜索商品
    cy.get('[data-testid="search"]').type('Test Product');
    cy.get('[data-testid="search-btn"]').click();

    // 点击商品
    cy.get('[data-testid="product-item"]').first().click();
    cy.url().should('include', '/products/');

    // 加入购物车
    cy.get('[data-testid="quantity"]').clear().type('2');
    cy.get('[data-testid="add-to-cart"]').click();

    // 进入购物车
    cy.get('[data-testid="cart-icon"]').click();
    cy.url().should('include', '/cart');

    // 验证购物车
    cy.get('[data-testid="cart-item"]').should('have.length', 1);

    // 结账
    cy.get('[data-testid="checkout-btn"]').click();
    cy.url().should('include', '/checkout');

    // 填写地址
    cy.get('[data-testid="address"]').type('123 Main St');
    cy.get('[data-testid="city"]').type('Springfield');
    cy.get('[data-testid="zip"]').type('12345');

    // 选择配送和支付
    cy.get('[data-testid="shipping-standard"]').click();
    cy.get('[data-testid="payment-card"]').click();

    // 支付
    cy.get('[data-testid="pay-btn"]').click();

    // 验证确认
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-testid="success-message"]')
      .should('contain', 'Order confirmed');
  });
});
```

---

## 常见问题和解决方案

### Q1: E2E测试太慢，如何加速？

**A**: 优化策略：

1. **并发运行**
```bash
# Playwright
npx playwright test --workers=4

# Cypress
npx cypress run --parallel
```

2. **选择性测试**
```typescript
// 只测关键流程
test.only('should complete checkout', () => {
  // ...
});
```

3. **减少等待时间**
```typescript
// ❌ 慢 - 默认30秒超时
await page.waitForSelector('[data-testid="result"]');

// ✅ 快 - 显式指定
await page.waitForSelector('[data-testid="result"]', {
  timeout: 5000
});
```

4. **使用轻量级初始化**
```typescript
// ❌ 慢 - 真实API调用
await createRealUser();
await createRealProducts();

// ✅ 快 - 数据库直接插入
await db.users.insert(testUser);
await db.products.insert(testProducts);
```

5. **并发创建数据**
```typescript
// 并行初始化而非顺序
await Promise.all([
  db.users.insert(testUser),
  db.products.insert(testProducts),
  db.orders.insert(testOrders)
]);
```

### Q2: E2E测试Flaky（不稳定），如何修复？

**A**: 常见原因和解决方案：

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 等待时间不够 | 有时过快，有时超时 | 使用waitFor()和显式条件 |
| 网络问题 | 随机失败 | Mock API，避免真实网络 |
| 异步操作 | 偶发性竞态 | await所有异步操作 |
| DOM变化 | 找不到元素 | 等待元素visible |
| 浏览器状态 | 多次运行时失败 | beforeEach清理状态 |

**最佳实践**：
```typescript
// ❌ Flaky - 依赖等待时间
setTimeout(() => {
  expect(result).toBe('done');
}, 1000);

// ✅ 稳定 - 等待具体条件
await expect(page.locator('[data-testid="result"]'))
  .toContainText('done', { timeout: 5000 });
```

### Q3: 如何处理第三方支付、登录等外部系统？

**A**: 三种方法：

1. **Mock外部系统**（推荐）
```typescript
await mockPaymentGateway({
  charge: (amount) => ({ status: 'success', id: 'TXN123' })
});

await mockOAuth({
  login: () => ({ token: 'jwt-token' })
});
```

2. **使用测试账户**
```typescript
// 第三方提供的测试账户
const testCreditCard = {
  number: '4242424242424242',
  exp: '12/25',
  cvc: '123'
};
```

3. **录制和回放 (VCR)**
```typescript
// 第一次真实调用，后续回放录制
import * as nock from 'nock';

// 录制
nock('https://payment.example.com')
  .post('/charge')
  .reply(200, { status: 'success' });

// 后续测试使用录制内容
```

### Q4: 多个E2E测试同时运行时数据冲突？

**A**: 隔离策略：

```typescript
// 使用独立的测试账户
const userId = `test-${Date.now()}-${Math.random()}`;

// 或使用独立的数据库模式
const schemaName = `test_schema_${uuid()}`;
await db.createSchema(schemaName);

// 后续清理
afterEach(async () => {
  await db.dropSchema(schemaName);
});
```

### Q5: 如何在CI/CD中运行E2E测试？

**A**: CI/CD集成：

```yaml
# GitHub Actions
- name: Run E2E tests
  run: npx playwright test

- name: Upload results
  if: always()
  uses: actions/upload-artifact@v2
  with:
    name: playwright-report
    path: playwright-report/
```

---

## E2E测试检查清单

在编写E2E测试前，确保：

- [ ] 明确定义了用户场景和角色
- [ ] 识别了关键的验证点（UI、数据库、邮件等）
- [ ] 准备了测试环境（初始数据、Mock等）
- [ ] 使用了等待条件而非固定延迟
- [ ] 测试是独立的，不依赖执行顺序
- [ ] 包含Happy Path和失败场景
- [ ] 测试速度可接受（< 10s）
- [ ] 没有flaky（稳定性高）
- [ ] 能在CI/CD中运行
- [ ] 有清晰的错误提示
- [ ] 能捕获屏幕截图和日志用于调试

---

## 总结

E2E测试是验证整个应用的最后一道防线：

✅ **重要**：核心业务流程必须E2E测试
✅ **不过度**：选择关键场景（5-10个）而非全部
✅ **可靠**：避免Flaky，使用显式等待
✅ **维护**：定期更新以适应UI变化

**从Phase 2继承的Context**：
- code-execute已通过TDD确保单元测试完整
- 现在code-test在Phase 3负责E2E验证完整业务流程

**开始E2E测试设计！** 🚀

