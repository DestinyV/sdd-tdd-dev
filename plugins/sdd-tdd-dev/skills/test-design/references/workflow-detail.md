# test-design 详细工作流

本文档包含 test-design SKILL.md 中省略的详细工作流步骤、示例和最佳实践。

---

## 完整工作流步骤

### 阶段1: 读取和分析需求

#### 1.1 从规范中提取TEST-VERIFY

**读取spec中的TEST-VERIFY**：

```
spec-dev/order-form/spec/
├── requirement.md           # 需求总览
├── scenarios/
│   ├── user-create-order.md
│   ├── user-edit-order.md
│   └── user-cancel-order.md
├── test-verify.md          # ← TEST-VERIFY在这里
├── data-models.md
├── business-rules.md
└── glossary.md
```

**test-verify.md 中的内容示例**：

```markdown
# 测试验证标准（TEST-VERIFY）

## 订单创建功能

### TV-Order-1: 用户能创建订单
```
WHEN 用户在订单表单中填入以下有效数据：
  - 商品ID: "product-123"（有效的商品ID）
  - 数量: 2
  - 地址: "北京市朝阳区..."
THEN 系统应该：
  1. 创建订单记录
  2. 订单状态为 "pending"
  3. 返回订单ID给用户
  4. 发送确认邮件

Mock数据文件：
- fixtures/product-valid.json
- fixtures/address-valid.json
```

### TV-Order-2: 系统应该验证商品ID
```
WHEN 用户提交的商品ID不存在（如 "invalid-product"）
THEN 系统应该：
  1. 拒绝订单创建
  2. 返回错误信息 "商品不存在"
  3. 不创建订单记录

Mock数据文件：
- fixtures/product-invalid.json
```

### TV-Order-3: 系统应该验证数量范围
```
WHEN 用户提交的数量不在有效范围内：
  - 边界值1: 数量 = 0（应该被拒绝）
  - 边界值2: 数量 = -5（应该被拒绝）
  - 边界值3: 数量 = 1000000（超过库存）
  - 有效值: 数量 = 1 到 100
THEN 系统应该拒绝无效的数量

Mock数据文件：
- fixtures/quantity-boundary.json
```

...（更多TV项）
```

**提取要点**：

1. ✅ 识别所有TV项（TV-Order-1, TV-Order-2, ...）
2. ✅ 理解每个TV的WHEN-THEN条件
3. ✅ 收集Mock数据文件位置
4. ✅ 识别边界值和特殊值

#### 1.2 从任务中提取test-case-mapping

**读取tasks.md中的映射表**：

```markdown
## Task 1: 订单创建

### Test Case Mapping

| TEST-VERIFY | TC-ID | 测试类型 | Mock数据 | 实现点 |
|------------|-------|---------|---------|-------|
| TV-Order-1 | TC-1.1.1 | Unit | fixtures/product-valid.json | createOrder() 函数 |
| TV-Order-2 | TC-1.1.2 | Unit | fixtures/product-invalid.json | validateProduct() 函数 |
| TV-Order-3 | TC-1.2.1 | Unit | fixtures/quantity-boundary.json | validateQuantity() 函数 |
| TV-Order-1 | TC-1.3.1 | Integration | fixtures/*.json | 订单API完整流程 |
| TV-Order-1 | TC-1.4.1 | E2E | fixtures/*.json | 用户完整订单流程 |
```

**分析关键点**：

1. **TC-ID命名规则**：
   - TC-1.1.1: Task1 的第1个describe suite 的第1个test
   - TC-1.2.1: Task1 的第2个describe suite 的第1个test
   - TC-1.3.1: Task1 的第3个describe suite（集成测试）的第1个test
   - TC-1.4.1: Task1 的第4个describe suite（E2E）的第1个test

2. **测试分布**：
   - Unit tests: 最多（单个函数）
   - Integration tests: 中等（多个组件）
   - E2E tests: 最少（完整流程）

3. **Mock数据来源**：
   - 来自spec中的test-verify部分
   - 位置：fixtures/目录

#### 1.3 分析设计方案

**读取design.md中的关键信息**：

```markdown
## 技术栈选型

### 前端
- 框架：React
- 测试框架：Jest + React Testing Library
- Mock库：Jest Mock
- 覆盖率工具：Istanbul

### 后端
- 框架：Node.js + Express
- 测试框架：Jest
- Mock库：Jest Mock
- 数据库：PostgreSQL（测试用SQLite）

## 架构设计

### 订单模块
- OrderService: 业务逻辑
- OrderController: API端点
- OrderValidator: 验证逻辑
- OrderRepository: 数据持久化
```

**提取信息**：

1. ✅ 测试框架：Jest
2. ✅ 被测对象：OrderService, OrderController, OrderValidator, OrderRepository
3. ✅ Mock策略：Mock OrderRepository 用 Jest Mock

### 阶段2: 生成测试规范

#### 2.1 生成test-spec.md结构

```markdown
# 测试规范

## 项目信息
- 项目名：Order Management System
- 需求：[order-form 需求](../spec/requirement.md)
- 任务列表：[tasks.md](../tasks/tasks.md)
- 测试设计阶段：2026-03-26

## 测试框架和工具

### 前端
- 测试框架：Jest 28.x
- 测试库：React Testing Library 13.x
- Mock库：Jest Mock
- 覆盖率工具：Istanbul (Jest内置)

### 后端
- 测试框架：Jest 28.x
- Mock库：Jest Mock
- 数据库：SQLite (内存)

## 测试分层

### 单元测试（Unit Tests）

#### Task 1: 订单创建
- 目标：验证 OrderService.createOrder() 的正确性
- 依赖：Mock OrderRepository
- Test Cases：6个
  - TC-1.1.1: 创建有效订单
  - TC-1.1.2: 验证商品ID
  - TC-1.1.3: 验证数量（最小值）
  - TC-1.1.4: 验证数量（最大值）
  - TC-1.1.5: 验证地址
  - TC-1.1.6: 验证邮箱

#### Task 2: 订单编辑
- 目标：验证 OrderService.updateOrder() 的正确性
- 依赖：Mock OrderRepository, AuthService
- Test Cases：4个
  - TC-1.2.1: 编辑已存在的订单
  - TC-1.2.2: 拒绝编辑他人订单
  - TC-1.2.3: 拒绝编辑已确认订单
  - TC-1.2.4: 验证编辑字段

...（更多Task）

### 集成测试（Integration Tests）

#### 订单API完整流程
- 目标：验证 OrderController 和 OrderService 的协作
- Mock：OrderRepository（内存SQLite）
- Test Cases：4个
  - TC-1.3.1: 创建 → 查询 → 完整流程
  - TC-1.3.2: 创建 → 编辑 → 查询 → 完整流程
  - TC-1.3.3: 错误处理链路
  - TC-1.3.4: 并发订单创建

### E2E测试（E2E Tests）

#### 用户订单流程
- 工具：Cypress
- Mock：后端API（仅Mock外部服务如支付、邮件）
- Test Cases：3个
  - TC-1.4.1: 用户完整订单流程
  - TC-1.4.2: 用户编辑订单
  - TC-1.4.3: 用户取消订单

## 完整的Test Case表格

| TC-ID | Task | TEST-VERIFY | 测试类型 | Mock方式 | 状态 |
|-------|------|-----------|---------|---------|------|
| TC-1.1.1 | 订单创建 | TV-Order-1 | Unit | Mock OrderRepository | ✅ |
| TC-1.1.2 | 订单创建 | TV-Order-2 | Unit | Mock OrderRepository | ✅ |
| TC-1.1.3 | 订单创建 | TV-Order-3 | Unit | Mock OrderRepository | ✅ |
| TC-1.2.1 | 订单编辑 | TV-Order-4 | Unit | Mock OrderRepository | ✅ |
| TC-1.3.1 | API整合 | TV-Order-1 | Integration | Mock DB | ✅ |
| TC-1.4.1 | 用户流程 | TV-Order-1 | E2E | Mock支付服务 | ✅ |
| ... | ... | ... | ... | ... | ... |

## Mock和Fixture定义

### OrderRepository Mock
```typescript
const mockRepository = {
  create: jest.fn(),
  findById: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
};
```

### Fixture数据
- fixtures/order-valid.json: 有效订单数据
- fixtures/order-boundary.json: 边界值数据
- fixtures/order-invalid.json: 无效数据

## 覆盖率分析

- 单元测试覆盖率：85%+
- 集成测试覆盖率：80%+
- E2E测试覆盖率：70%+
- **TEST-VERIFY覆盖率：100%** ✅

## 执行计划

### Phase 1: 单元测试（code-execute TDD）
- 时间：Day 1-2
- 文件：test-Task1.test.ts, test-Task2.test.ts...
- 验证：Jest test:unit 覆盖率 ≥85%

### Phase 2: 集成测试（code-test）
- 时间：Day 3
- 文件：test-integration.test.ts
- 验证：npm run test:integration 全部通过

### Phase 3: E2E测试（code-test）
- 时间：Day 4
- 文件：cypress/e2e/*.cy.ts
- 验证：npm run test:e2e 全部通过
```

#### 2.2 每个Task的测试规范详情

**Task 1: 订单创建的详细规范**：

```markdown
## Task 1: 订单创建

### 概述
- 目标：实现 OrderService.createOrder() 函数
- 被测对象：OrderService 类的 createOrder 方法
- 依赖：OrderRepository, OrderValidator
- Mock：OrderRepository（隔离数据库）

### 来源验收标准
- TV-Order-1: 用户能创建订单
- TV-Order-2: 系统应该验证商品ID
- TV-Order-3: 系统应该验证数量范围

### 测试用例详情

#### TC-1.1.1: 创建有效订单
```
来源：TV-Order-1
类型：Unit Test
前置条件：
  - 商品"product-123"存在
  - 库存充足（≥2）
输入：
  - productId: "product-123"
  - quantity: 2
  - address: "北京市朝阳区..."
  - email: "user@example.com"
预期结果：
  - 返回订单对象
  - 订单.status = "pending"
  - 订单.createdAt 为当前时间
  - OrderRepository.create() 被调用一次
Mock策略：
  - Mock OrderRepository.create() 返回成功
  - Mock OrderRepository.findProductById() 返回有效商品
```

#### TC-1.1.2: 验证无效商品ID
```
来源：TV-Order-2
类型：Unit Test
前置条件：
  - 商品"invalid-product"不存在
输入：
  - productId: "invalid-product"
  - quantity: 2
  - address: "北京市朝阳区..."
预期结果：
  - 抛出 ValidationError("商品不存在")
  - OrderRepository.create() 不被调用
Mock策略：
  - Mock OrderRepository.findProductById() 返回 null
```

#### TC-1.1.3: 验证数量边界值 (0)
```
来源：TV-Order-3
类型：Unit Test
前置条件：
  - 无
输入：
  - quantity: 0
预期结果：
  - 抛出 ValidationError("数量必须大于0")
Mock策略：
  - 不需要Mock数据库
```

#### TC-1.1.4: 验证数量边界值 (1000000)
```
来源：TV-Order-3
类型：Unit Test
前置条件：
  - 商品库存 < 1000000
输入：
  - quantity: 1000000
预期结果：
  - 抛出 ValidationError("库存不足")
Mock策略：
  - Mock OrderRepository.findProductById() 返回库存 100 的商品
```

#### TC-1.1.5: 验证有效的数量范围 (1-100)
```
来源：TV-Order-3
类型：Unit Test
前置条件：
  - 商品"product-123"库存 = 50
输入：
  - productId: "product-123"
  - quantity: 25
预期结果：
  - 创建成功
  - 订单.quantity = 25
Mock策略：
  - Mock OrderRepository.findProductById() 返回库存 50 的商品
```

### 测试数据（fixtures）

```json
{
  "Task1": {
    "validInputs": [
      {
        "name": "valid_order_full_data",
        "data": {
          "productId": "product-123",
          "quantity": 2,
          "address": "北京市朝阳区中关村",
          "email": "user@example.com"
        },
        "expectedResult": {
          "status": "pending",
          "productId": "product-123",
          "quantity": 2
        }
      }
    ],
    "boundaryValues": [
      {
        "name": "min_quantity",
        "data": {
          "quantity": 1
        },
        "expectedResult": "success"
      },
      {
        "name": "max_quantity",
        "data": {
          "quantity": 100
        },
        "expectedResult": "success"
      },
      {
        "name": "quantity_zero",
        "data": {
          "quantity": 0
        },
        "expectedError": "数量必须大于0"
      },
      {
        "name": "quantity_negative",
        "data": {
          "quantity": -5
        },
        "expectedError": "数量必须大于0"
      }
    ],
    "specialValues": [
      {
        "name": "invalid_product_id",
        "data": {
          "productId": "nonexistent"
        },
        "expectedError": "商品不存在"
      },
      {
        "name": "empty_email",
        "data": {
          "email": ""
        },
        "expectedError": "邮箱不能为空"
      }
    ]
  }
}
```
```

### 阶段3: 定义Mock和Fixture

#### 3.1 fixtures.json 结构

```json
{
  "metadata": {
    "project": "Order Management System",
    "createdAt": "2026-03-26",
    "version": "1.0",
    "description": "测试数据和Mock定义"
  },
  "fixtures": {
    "Task1_OrderCreation": {
      "validInputs": [
        {
          "name": "standard_order",
          "data": {
            "productId": "PROD-001",
            "quantity": 2,
            "address": "北京市朝阳区",
            "email": "user@example.com",
            "paymentMethod": "credit_card"
          },
          "expectedResult": {
            "orderId": "ORD-xxx",
            "status": "pending",
            "totalPrice": 1998
          }
        }
      ],
      "boundaryValues": [
        {
          "name": "min_quantity",
          "data": { "quantity": 1 },
          "expectedResult": "should accept"
        },
        {
          "name": "max_quantity",
          "data": { "quantity": 100 },
          "expectedResult": "should accept"
        }
      ],
      "specialValues": [
        {
          "name": "zero_quantity",
          "data": { "quantity": 0 },
          "expectedError": "Quantity must be > 0"
        }
      ]
    }
  },
  "mocks": {
    "OrderRepository": {
      "methods": {
        "create": {
          "returnValue": { "orderId": "ORD-123", "status": "pending" }
        },
        "findProductById": {
          "validProduct": { "id": "PROD-001", "price": 999, "stock": 50 },
          "invalidProduct": null
        }
      }
    },
    "ExternalServices": {
      "PaymentGateway": {
        "chargeCard": {
          "success": { "transactionId": "TXN-123", "status": "succeeded" },
          "failure": { "error": "Card declined" }
        }
      },
      "EmailService": {
        "sendConfirmationEmail": {
          "success": { "messageId": "MSG-123" },
          "failure": { "error": "SMTP error" }
        }
      }
    }
  }
}
```

#### 3.2 Mock策略说明

```markdown
## Mock策略定义

### OrderRepository Mock

**为什么Mock**：
- 隔离数据库依赖，单元测试不应该依赖真实数据库
- 加快测试速度（内存操作 vs I/O）
- 便于控制返回值（正常、错误、边界）

**Mock方式**：
```typescript
jest.mock('../database/OrderRepository', () => ({
  OrderRepository: jest.fn().mockImplementation(() => ({
    create: jest.fn(),
    findById: jest.fn(),
    findProductById: jest.fn(),
    update: jest.fn(),
  })),
}));
```

**Mock返回值**：
```typescript
const mockRepo = OrderRepository.mock.results[0].value;

// 成功情况
mockRepo.findProductById.mockResolvedValue({
  id: 'PROD-001',
  name: '商品1',
  price: 999,
  stock: 50,
});

// 失败情况（商品不存在）
mockRepo.findProductById.mockResolvedValue(null);
```

### PaymentGateway Mock

**为什么Mock**：
- 支付服务是外部API，单元测试不应该真实调用
- 避免测试中产生真实扣款
- 便于测试错误场景

**Mock方式**：
```typescript
jest.mock('../services/PaymentGateway', () => ({
  charge: jest.fn(),
}));
```

**Mock返回值**：
```typescript
// 成功扣款
PaymentGateway.charge.mockResolvedValue({
  transactionId: 'TXN-123',
  status: 'succeeded',
});

// 失败扣款
PaymentGateway.charge.mockRejectedValue(
  new Error('Card declined')
);
```

### 不应该Mock的内容

```typescript
// ❌ 不要Mock业务逻辑
jest.mock('../services/OrderValidator');

// ✅ 应该直接调用OrderValidator的真实实现
import { OrderValidator } from '../services/OrderValidator';
const validator = new OrderValidator();
const isValid = validator.validate(order);
```
```

### 阶段4: 生成测试框架代码

#### 4.1 Jest单元测试模板

**test-Task1.template.ts**：

```typescript
// test-Task1.template.ts
// Task 1: 订单创建
// 来源：TV-Order-1, TV-Order-2, TV-Order-3

import { OrderService } from '../src/services/OrderService';
import { OrderRepository } from '../src/database/OrderRepository';
import { PaymentGateway } from '../src/services/PaymentGateway';
import { fixtures } from './fixtures.json';

// Mock所有外部依赖
jest.mock('../src/database/OrderRepository');
jest.mock('../src/services/PaymentGateway');

describe('Task 1: Order Creation (创建订单)', () => {
  let orderService: OrderService;
  let mockRepository: jest.Mocked<OrderRepository>;
  let mockPaymentGateway: jest.Mocked<typeof PaymentGateway>;

  beforeEach(() => {
    // 清除所有Mock
    jest.clearAllMocks();

    // 初始化Mock
    mockRepository = OrderRepository as jest.Mocked<typeof OrderRepository>;
    mockPaymentGateway = PaymentGateway as jest.Mocked<typeof PaymentGateway>;

    // 创建被测对象
    orderService = new OrderService(mockRepository, mockPaymentGateway);
  });

  describe('正常路径 (Happy Path)', () => {
    // TC-1.1.1: 创建有效订单
    test('TC-1.1.1: should create order with valid data', async () => {
      // Arrange
      const input = fixtures.Task1_OrderCreation.validInputs[0];
      mockRepository.findProductById.mockResolvedValue({
        id: input.data.productId,
        price: 999,
        stock: 100,
      });
      mockRepository.create.mockResolvedValue({
        orderId: 'ORD-123',
        ...input.data,
        status: 'pending',
        totalPrice: 999 * 2,
      });

      // Act
      const result = await orderService.createOrder(input.data);

      // Assert
      expect(result.status).toBe('pending');
      expect(result.orderId).toBeDefined();
      expect(mockRepository.create).toHaveBeenCalledWith(
        expect.objectContaining({
          productId: input.data.productId,
          quantity: input.data.quantity,
        })
      );
    });
  });

  describe('验证 - 商品ID (TV-Order-2)', () => {
    // TC-1.1.2: 拒绝无效商品ID
    test('TC-1.1.2: should reject invalid product ID', async () => {
      // Arrange
      const input = {
        productId: 'nonexistent',
        quantity: 2,
        address: '北京市朝阳区',
        email: 'user@example.com',
      };
      mockRepository.findProductById.mockResolvedValue(null);

      // Act & Assert
      await expect(orderService.createOrder(input)).rejects.toThrow(
        '商品不存在'
      );
      expect(mockRepository.create).not.toHaveBeenCalled();
    });
  });

  describe('验证 - 数量范围 (TV-Order-3)', () => {
    // TC-1.1.3: 拒绝零数量
    test('TC-1.1.3: should reject zero quantity', async () => {
      // Arrange
      const input = fixtures.Task1_OrderCreation.boundaryValues.find(
        (v) => v.name === 'quantity_zero'
      );

      // Act & Assert
      await expect(
        orderService.createOrder({
          ...input.data,
          productId: 'PROD-001',
          address: '北京',
          email: 'user@example.com',
        })
      ).rejects.toThrow('数量必须大于0');
    });

    // TC-1.1.4: 拒绝过大数量
    test('TC-1.1.4: should reject quantity > stock', async () => {
      // Arrange
      const input = {
        productId: 'PROD-001',
        quantity: 1000, // 超过库存
        address: '北京市朝阳区',
        email: 'user@example.com',
      };
      mockRepository.findProductById.mockResolvedValue({
        id: 'PROD-001',
        price: 999,
        stock: 50, // 库存只有50
      });

      // Act & Assert
      await expect(orderService.createOrder(input)).rejects.toThrow(
        '库存不足'
      );
    });

    // TC-1.1.5: 接受有效数量
    test('TC-1.1.5: should accept valid quantity (1-100)', async () => {
      // Arrange
      const input = {
        productId: 'PROD-001',
        quantity: 50,
        address: '北京市朝阳区',
        email: 'user@example.com',
      };
      mockRepository.findProductById.mockResolvedValue({
        id: 'PROD-001',
        price: 999,
        stock: 100,
      });
      mockRepository.create.mockResolvedValue({
        orderId: 'ORD-456',
        ...input,
        status: 'pending',
      });

      // Act
      const result = await orderService.createOrder(input);

      // Assert
      expect(result.status).toBe('pending');
      expect(mockRepository.create).toHaveBeenCalled();
    });
  });

  afterEach(() => {
    // 清理
    jest.restoreAllMocks();
  });
});
```

#### 4.2 集成测试模板

**test-integration.template.ts**：

```typescript
// test-integration.template.ts
// 集成测试：订单API完整流程
// 来源：TV-Order-1

import request from 'supertest';
import { app } from '../src/app';
import { Database } from '../src/database/Database';

describe('Integration: Order API (订单API整合)', () => {
  let db: Database;

  beforeAll(async () => {
    // 初始化测试数据库（内存SQLite）
    db = new Database(':memory:');
    await db.initialize();
    await db.seedTestData();
  });

  afterAll(async () => {
    await db.close();
  });

  describe('TC-1.3.1: 创建 → 查询 → 完整流程', () => {
    test('should create order and retrieve it', async () => {
      // Arrange
      const orderData = {
        productId: 'PROD-001',
        quantity: 2,
        address: '北京市朝阳区',
        email: 'user@example.com',
      };

      // Act: 创建订单
      const createResponse = await request(app)
        .post('/api/orders')
        .send(orderData);

      expect(createResponse.status).toBe(201);
      const orderId = createResponse.body.orderId;

      // Act: 查询订单
      const getResponse = await request(app).get(`/api/orders/${orderId}`);

      // Assert
      expect(getResponse.status).toBe(200);
      expect(getResponse.body).toMatchObject({
        orderId,
        productId: 'PROD-001',
        quantity: 2,
        status: 'pending',
      });
    });
  });

  describe('TC-1.3.2: 创建 → 编辑 → 查询 → 完整流程', () => {
    test('should create, edit, and retrieve order', async () => {
      // Arrange & Act & Assert
      // 详细步骤...
    });
  });
});
```

#### 4.3 E2E测试模板

**cypress/e2e/order-creation.cy.ts**：

```typescript
// cypress/e2e/order-creation.cy.ts
// E2E测试：用户订单完整流程
// 来源：TV-Order-1

describe('E2E: User Order Creation Flow (用户订单流程)', () => {
  beforeEach(() => {
    // 访问应用
    cy.visit('http://localhost:3000');
    // Mock外部服务（支付、邮件）
    cy.intercept('POST', '**/payment/charge', {
      statusCode: 200,
      body: { transactionId: 'TXN-123', status: 'succeeded' },
    });
  });

  it('TC-1.4.1: should complete full order flow', () => {
    // 1. 点击创建订单
    cy.contains('Create Order').click();

    // 2. 选择商品
    cy.get('[data-testid="product-selector"]').select('PROD-001');

    // 3. 输入数量
    cy.get('[data-testid="quantity-input"]').type('2');

    // 4. 输入收货地址
    cy.get('[data-testid="address-input"]').type('北京市朝阳区');

    // 5. 输入邮箱
    cy.get('[data-testid="email-input"]').type('user@example.com');

    // 6. 提交订单
    cy.get('[data-testid="submit-button"]').click();

    // 7. 验证成功
    cy.contains('Order created successfully').should('be.visible');
    cy.get('[data-testid="order-id"]').should('exist');

    // 8. 验证订单详情
    cy.contains('Status: pending').should('be.visible');
    cy.contains('Product: 商品1').should('be.visible');
    cy.contains('Quantity: 2').should('be.visible');
  });
});
```

### 阶段5: 验证覆盖率

#### 5.1 覆盖率检查清单

```markdown
## 覆盖率验证清单

### TEST-VERIFY覆盖率 (必须100%)

- [x] TV-Order-1: 用户能创建订单
  - Unit Test: TC-1.1.1 ✅
  - Integration Test: TC-1.3.1 ✅
  - E2E Test: TC-1.4.1 ✅

- [x] TV-Order-2: 系统应该验证商品ID
  - Unit Test: TC-1.1.2 ✅

- [x] TV-Order-3: 系统应该验证数量范围
  - Unit Test: TC-1.1.3, TC-1.1.4, TC-1.1.5 ✅

...

**总计**：所有 TV 项都有对应的 test case ✅

### 测试分层完整性

- [x] Unit Tests: 至少 5 个 per Task
- [x] Integration Tests: 至少 2 个 per Task 组（多Task协作）
- [x] E2E Tests: 至少 1 个 per 主流程

### Mock完整性

- [x] OrderRepository: 所有方法都有Mock定义 ✅
- [x] PaymentGateway: 成功/失败场景都有 ✅
- [x] EmailService: 发送成功/失败都有 ✅

### 框架代码完整性

- [x] 每个 test case 都有对应的框架代码
- [x] 框架代码有 Arrange-Act-Assert 结构
- [x] Mock 初始化和清理都完整
```

#### 5.2 生成覆盖率矩阵

```markdown
## 覆盖率矩阵

| TEST-VERIFY | TC数量 | Unit | Integration | E2E | 状态 |
|-------------|--------|------|-------------|-----|------|
| TV-Order-1  | 3      | ✅   | ✅          | ✅  | ✅   |
| TV-Order-2  | 1      | ✅   | -           | -   | ✅   |
| TV-Order-3  | 3      | ✅   | -           | -   | ✅   |
| TV-Order-4  | 2      | ✅   | ✅          | -   | ✅   |
| TV-Order-5  | 1      | ✅   | -           | ✅  | ✅   |
| ...         | ...    | ... | ...         | ... | ... |

**总计**：
- 总TEST-VERIFY: 12个
- 总Test Cases: 25个
- 覆盖率：12/12 = **100%** ✅
```

---

## 最佳实践

### 1. TC-ID命名规范

```
TC-{Task}.{Suite}.{Index}

TC-1.1.1
  ↑ ↑ ↑
  | | └─ 该Suite中的第1个test
  | └──── Task1的第1个describe suite（通常是正常路径）
  └────── Task1
```

### 2. 完整的追踪链

```
TEST-VERIFY (spec)
   ↓ 来自
test-case-mapping (tasks.md)
   ↓ 对应
TC-ID (test-spec.md)
   ↓ 实现
test-*.template (框架代码)
```

### 3. Mock数据的来源

```markdown
所有Mock数据都应该来自spec中的test-verify部分：

spec/test-verify.md:
  WHEN 商品ID = "product-123" (有效值)
  OR 商品ID = "invalid" (特殊值)
  OR 数量 = 0 (边界值)

→ 使用这些具体的值作为test fixtures
```

### 4. 分层的粒度

```
单元测试：单个函数/方法
  - OrderValidator.validate()
  - OrderService.createOrder()

集成测试：多个相关的类/模块
  - OrderService + OrderRepository + PaymentGateway

E2E测试：完整的业务流程
  - 用户从UI到数据库的完整订单创建流程
```

---

## 常见问题

### Q: 如何处理异步操作的Mock？

A: 使用 mockResolvedValue 和 mockRejectedValue：

```typescript
// 成功情况
mockRepository.create.mockResolvedValue({ orderId: 'ORD-123' });

// 失败情况
mockPaymentGateway.charge.mockRejectedValue(new Error('Card declined'));
```

### Q: 如何处理数据库相关的集成测试？

A: 使用内存数据库（SQLite）和事务回滚：

```typescript
beforeEach(async () => {
  await db.beginTransaction();
});

afterEach(async () => {
  await db.rollback();
});
```

### Q: TC-ID太长了怎么办？

A: 可以简化为 TC-1.1 （省略最后的索引），主要是保证唯一性。

---

**关键理念**：测试规范应该清晰、完整、可追踪，使得后续的代码实现和测试执行可以有序进行。
