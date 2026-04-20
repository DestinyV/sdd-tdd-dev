# 前端浏览器测试完整指南

## 目录

1. [概述](#概述)
2. [环境安装和配置](#环境安装和配置)
3. [E2E 端到端测试](#e2e-端到端测试)
4. [视觉回归测试](#视觉回归测试)
5. [组件 UI 测试](#组件-ui-测试)
6. [MCP Browser 探索性测试](#mcp-browser-探索性测试)
7. [测试数据管理](#测试数据管理)
8. [API Mock 策略](#api-mock-策略)
9. [CI/CD 集成](#cicd-集成)
10. [常见问题和解决方案](#常见问题和解决方案)

---

## 概述

前端浏览器测试在 SDD 工作流中的定位：

```
spec 创建 → 定义 BROWSER-TESTABLE 验收标准
    ↓
code-designer → 定义 data-testid 策略和可测试交互路径
    ↓
code-task → 生成浏览器测试任务 + Browser Test ID 映射
    ↓
code-execute → TDD 编写浏览器测试 + 实现
    ↓
code-test → 执行 Playwright E2E + 视觉回归 + 组件 UI + MCP Browser
```

### 测试分层

| 层级 | 工具 | 目标 | 覆盖率 |
|------|------|------|--------|
| E2E 端到端 | Playwright | 完整用户流程 | 核心流程 100% |
| 视觉回归 | Playwright Screenshots | UI 布局一致性 | 关键页面 100% |
| 组件 UI | Playwright/Vitest | 组件渲染和交互 | 所有组件 |

---

## 环境安装和配置

### 1. 安装 Playwright

```bash
# 在目标前端项目中执行
npm install -D @playwright/test
npx playwright install chromium  # 或安装全部浏览器
npx playwright install            # 安装所有浏览器（Chromium, Firefox, WebKit）
```

### 2. 配置 playwright.config.ts

将模板 `skills/code-test/templates/playwright.config.ts` 复制到项目根目录：

```bash
cp plugins/sdd-tdd-dev/skills/code-test/templates/playwright.config.ts ./playwright.config.ts
```

根据需要修改：
- `testDir`: 测试文件目录
- `baseURL`: 开发服务器地址
- `webServer.command`: 启动开发服务器的命令
- `projects`: 选择需要的浏览器项目

### 3. 配置 package.json scripts

```json
{
  "scripts": {
    "test:e2e": "npx playwright test --grep @e2e",
    "test:visual": "npx playwright test --grep @visual",
    "test:component": "npx playwright test --grep @component",
    "test:browser": "bash plugins/sdd-tdd-dev/skills/code-test/scripts/run-browser-tests.sh all"
  }
}
```

### 4. 复制工具函数

```bash
mkdir -p tests/helpers
cp plugins/sdd-tdd-dev/skills/code-test/templates/browser-test-helpers.ts tests/helpers/
```

### 5. 验证安装

```bash
npx playwright --version
npx playwright test --list
```

---

## E2E 端到端测试

### 测试编写原则

1. **从用户视角**：测试应该模拟真实用户的操作路径
2. **基于 spec 的 BROWSER-TESTABLE 标准**：每个测试对应 spec 中定义的验收标准
3. **使用 data-testid**：通过 design.md 中定义的选择器定位元素
4. **包含 Happy Path 和异常路径**

### 测试结构

```
tests/e2e/
├── auth.e2e.test.ts          # 登录/注册 E2E
├── [module].e2e.test.ts      # 功能模块 E2E
├── [module].e2e.test.ts      # 功能模块 E2E
└── helpers/
    └── browser-test-helpers.ts
```

### 编写 E2E 测试

参考模板：`skills/code-test/templates/frontend-e2e-test.template.ts`

```typescript
import { test, expect } from '@playwright/test';
import { loginAs, fillForm, waitForStable } from '../helpers/browser-test-helpers';

test.describe('@e2e 用户登录', () => {
  test('应该能用正确的凭据登录', async ({ page }) => {
    // 导航到登录页
    await page.goto('/login');

    // 填写表单（使用 data-testid）
    await page.getByTestId('login-username').fill('test@example.com');
    await page.getByTestId('login-password').fill('password123');
    await page.getByTestId('login-submit').click();

    // 验证跳转
    await expect(page).toHaveURL(/\/dashboard/);

    // 验证欢迎信息（BROWSER-TESTABLE 标准）
    await expect(page.getByTestId('welcome-message')).toContainText('欢迎');
  });
});
```

### 运行 E2E 测试

```bash
# 运行所有 E2E 测试
npm run test:e2e

# 运行单个文件
npx playwright test tests/e2e/auth.e2e.test.ts

# 在特定浏览器运行
npx playwright test --project=chromium-e2e

#  headed 模式（可视化调试）
npx playwright test --headed
```

---

## 视觉回归测试

### 基准截图生成

第一次运行视觉回归测试时，Playwright 会自动生成基准截图：

```bash
npx playwright test --grep @visual --update-snapshots
```

基准文件存储在 `test-results/` 或 `playwright/.expected/` 目录。

### 编写视觉回归测试

参考模板：`skills/code-test/templates/visual-regression.template.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('@visual 首页视觉回归', () => {
  test('首页桌面端应该与基准一致', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');

    await expect(page).toHaveScreenshot('homepage-desktop.png', {
      maxDiffPixels: 100,    // 允许的最大差异像素数
      threshold: 0.2,        // 像素差异阈值 (0-1)
      fullPage: true,        // 全页面截图
    });
  });
});
```

### 处理预期更新

当 UI 有意变更时，更新基准截图：

```bash
# 更新所有基准截图
npx playwright test --grep @visual --update-snapshots

# 查看差异报告
npx playwright show-report
```

### 运行视觉回归测试

```bash
npm run test:visual
```

---

## 组件 UI 测试

### 编写组件 UI 测试

参考模板：`skills/code-test/templates/frontend-component-ui.template.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('@component Button 组件', () => {
  test('应该渲染按钮并显示文本', async ({ page }) => {
    await page.goto('/__components/Button?text=点击我&variant=primary');

    await expect(page.getByRole('button')).toBeVisible();
    await expect(page.getByRole('button')).toContainText('点击我');
  });

  test('点击按钮时应该触发 onClick', async ({ page }) => {
    await page.goto('/__components/Button?text=点击我');

    await page.getByRole('button').click();
    await expect(page.getByTestId('click-count')).toContainText('1');
  });

  test('禁用状态下应该不可点击', async ({ page }) => {
    await page.goto('/__components/Button?text=点击我&disabled=true');

    await expect(page.getByRole('button')).toBeDisabled();
  });
});
```

### 运行组件 UI 测试

```bash
npm run test:component
```

---

## MCP Browser 探索性测试

### 配置

详见：`templates/mcp-browser-server.md`

在项目的 `.mcp.json` 中配置 Playwright MCP Server：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp-server"]
    }
  }
}
```

### 在 code-test 阶段的使用

code-test 的 Step 3.5 通过 MCP Browser 进行探索性测试：

1. **导航验证**：Claude 操控浏览器访问各页面，截图验证渲染
2. **交互探索**：点击各交互元素，验证响应
3. **状态检查**：执行 JS 检查 localStorage、Cookie、页面状态
4. **跨浏览器验证**：在不同浏览器上下文中重复关键测试

### 典型流程

```
Claude: 通过 MCP 工具执行以下步骤：

1. browser_navigate → http://localhost:5173
2. browser_screenshot → 截图验证首页渲染
3. browser_click → 点击导航项
4. browser_screenshot → 截图验证页面跳转
5. browser_get_text → 验证关键文本
6. browser_evaluate → 检查应用状态
7. 重复以上步骤覆盖关键流程
```

---

## 测试数据管理

### 策略

| 策略 | 适用场景 | 实现方式 |
|------|---------|---------|
| API 直接插入 | 需要预设业务数据 | `/api/test/data/setup` |
| UI 操作创建 | 需要测试创建流程 | 通过 UI 表单创建 |
| 数据库种子 | 需要大量初始数据 | `beforeAll` 批量插入 |
| 每次独立创建 | 需要数据隔离 | `beforeEach` 创建唯一数据 |

### 实现示例

使用 `browser-test-helpers.ts` 中的工具函数：

```typescript
test.beforeEach(async ({ page }) => {
  // 通过测试 API 设置初始数据
  await setupTestData(page, {
    users: [{ id: 'test-user', name: '测试用户' }],
    products: [{ id: 'prod-1', name: '测试商品', price: 99 }],
  });
});

test.afterEach(async ({ page }) => {
  // 清理测试数据
  await cleanupTestData(page);
});
```

### 数据隔离

```typescript
// 使用唯一标识避免测试间冲突
const testId = `test-${Date.now()}-${Math.random().toString(36).slice(2)}`;

test(`测试场景 - ${testId}`, async ({ page }) => {
  // ...
});
```

---

## API Mock 策略

### 何时 Mock

| 应该 Mock | 不应该 Mock |
|-----------|-------------|
| 第三方 API（支付、短信） | 自己的后端 API |
| 不稳定的外部服务 | 需要验证的数据流 |
| 费用相关的调用 | 关键业务逻辑 |
| 速率限制的服务 | 需要真实响应的场景 |

### Mock 实现

使用 `browser-test-helpers.ts` 中的 `mockAPI`：

```typescript
test('应该在 API 失败时显示错误', async ({ page }) => {
  // Mock API 返回 500
  await mockAPI(page, '**/api/submit', {
    status: 500,
    body: { error: '服务器错误' },
  });

  // 执行操作
  await page.getByTestId('form-submit').click();

  // 验证错误处理
  await expect(page.getByTestId('error-banner')).toBeVisible();
});
```

### 延迟模拟

```typescript
// 模拟慢请求
await mockAPI(page, '**/api/slow-endpoint', {
  status: 200,
  body: { data: 'ok' },
  delay: 3000, // 3 秒延迟
});
```

---

## CI/CD 集成

### GitHub Actions

```yaml
name: Frontend Browser Tests

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  browser-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Run visual regression tests
        run: npm run test:visual
        continue-on-error: true  # 视觉回归不阻断 CI

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

### 关键环境变量

| 变量 | 用途 | CI 值 |
|------|------|-------|
| `CI` | 启用 CI 模式 | `true` |
| `TEST_BASE_URL` | 应用 URL | 部署后的 URL |
| `PLAYWRIGHT_BROWSERS_PATH` | 浏览器安装路径 | `0` |

---

## 常见问题和解决方案

### Q1: 测试 Flaky（不稳定）

**原因和解决方案**：

| 原因 | 解决 |
|------|------|
| 等待时间不足 | 使用 `waitForSelector` / `waitForLoadState` |
| 动画未完成 | 使用 `waitForTimeout(300)` 等待 CSS 动画 |
| 网络请求竞态 | Mock 所有外部 API |
| 测试数据污染 | `beforeEach` 清理，使用唯一 ID |

```typescript
// ❌ Flaky
await page.click('[data-testid="submit"]');
await expect(page.getByTestId('success')).toBeVisible();

// ✅ 稳定
await page.getByTestId('submit').click();
await page.waitForLoadState('networkidle');
await expect(page.getByTestId('success')).toBeVisible({ timeout: 5000 });
```

### Q2: 截图对比差异过大

**原因**：
- 字体渲染差异（CI vs 本地）
- 系统主题不同
- 动态内容（时间戳、随机数）

**解决**：
```typescript
await expect(page).toHaveScreenshot('page.png', {
  maxDiffPixels: 100,     // 增加容差
  threshold: 0.3,         // 提高差异阈值
  animations: 'disabled', // 禁用动画
  mask: [                 // 遮挡动态内容
    page.getByTestId('timestamp'),
  ],
});
```

### Q3: 如何测试文件上传？

```typescript
test('应该支持文件上传', async ({ page }) => {
  await page.goto('/upload');

  // 设置文件选择器的值
  const fileInput = page.getByTestId('file-input');
  await fileInput.setInputFiles('test-file.pdf');

  // 触发上传
  await page.getByTestId('upload-submit').click();

  // 验证上传成功
  await expect(page.getByTestId('upload-success')).toBeVisible();
});
```

### Q4: 如何测试弹窗和对话框？

```typescript
test('应该能处理系统对话框', async ({ page }) => {
  page.on('dialog', async (dialog) => {
    expect(dialog.message()).toContain('确定要删除吗');
    await dialog.accept();
  });

  await page.getByTestId('delete-button').click();
  await expect(page.getByTestId('delete-success')).toBeVisible();
});
```

### Q5: 浏览器测试执行太慢

**优化策略**：

1. **只安装需要的浏览器**：
   ```bash
   npx playwright install chromium  # 只装 Chromium
   ```

2. **并行执行**：
   ```bash
   npx playwright test --workers=4
   ```

3. **选择性测试**：
   ```bash
   npx playwright test --grep "@e2e"  # 只跑 E2E
   ```

4. **复用登录状态**：
   ```typescript
   // 在 globalSetup 中登录，保存 auth state
   // 在各测试中复用
   export default defineConfig({
     use: { storageState: 'auth.json' },
   });
   ```

---

## 在 SDD 工作流中的使用

### code-test 阶段的完整浏览器测试流程

```bash
# 方式1: 使用自动化脚本（推荐）
bash plugins/sdd-tdd-dev/skills/code-test/scripts/run-browser-tests.sh all

# 方式2: 分步执行
npm run test:e2e       # E2E 端到端测试
npm run test:visual     # 视觉回归测试
npm run test:component  # 组件 UI 测试

# 方式3: 单类型测试
bash scripts/run-browser-tests.sh e2e
bash scripts/run-browser-tests.sh visual
bash scripts/run-browser-tests.sh component
```

### 测试通过标准

| 测试类型 | 通过标准 |
|---------|---------|
| E2E | 所有 `@e2e` 标签测试 100% 通过 |
| 视觉回归 | 无意外差异（或已更新基准） |
| 组件 UI | 所有 `@component` 标签测试 100% 通过 |

### 测试失败处理

1. **查看 HTML 报告**：`npx playwright show-report`
2. **查看失败截图**：检查 `test-results/` 目录
3. **Headed 模式调试**：`npx playwright test --headed`
4. **修复后重新执行**：确保修复有效
