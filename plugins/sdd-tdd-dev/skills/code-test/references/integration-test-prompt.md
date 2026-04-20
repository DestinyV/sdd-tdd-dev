# 集成测试提示词

用于指导实现子代理进行集成测试的设计和实现。

---

## 什么是集成测试

集成测试验证多个组件或模块在一起协作时是否正常工作。关键特点：

- **测试范围**：多个单元的交互
- **测试层级**：单元测试和E2E测试之间
- **覆盖场景**：组件间通信、数据流、模块依赖

**vs 单元测试**：
- 单元测试 → 单个函数/组件的隔离行为
- 集成测试 → 多个组件的协作行为

**vs E2E测试**：
- 集成测试 → 特定模块或功能的协作
- E2E测试 → 完整的业务流程（UI到后端）

---

## 何时需要集成测试

### ✅ 应该做集成测试

1. **多个Task的协作**
   - Task A生成的数据被Task B使用
   - 两个Task通过API或数据库交互

2. **跨模块通信**
   - 前端调用后端API
   - 服务间通信（微服务）
   - 数据库和业务逻辑协作

3. **复杂的数据流**
   - 数据经过多个处理步骤
   - 有状态转移或业务规则

4. **关键业务流程**
   - 支付流程（订单→支付→确认）
   - 认证流程（登录→授权→会话）
   - 工作流程（创建→审批→发布）

### ❌ 不需要集成测试

- 简单的单个组件（用单元测试就够）
- 完整的用户流程（应该用E2E测试）
- 纯粹的UI布局（无业务逻辑）

---

## 集成测试设计指南

### 步骤1: 识别测试边界

明确哪些Task或模块需要一起测试。

**识别问题**：
- 这个功能涉及哪几个Task？
- Task之间如何通信（API、数据库、消息队列）？
- 如果一个Task失败，会影响其他Task吗？

**示例 - 订单处理系统**：
```
Task 1: 创建订单（Order Service）
    ↓ API调用
Task 2: 扣款（Payment Service）
    ↓ 结果通知
Task 3: 发送确认邮件（Notification Service）

集成测试边界：Task 1 + Task 2 + Task 3的协作
```

### 步骤2: 定义Mock策略

决定哪些组件真实调用，哪些Mock。

**决策树**：

```
是否需要真实数据库？
├─ YES: 如果测试业务逻辑和数据持久化的交互
│   └─ 使用测试数据库（MySQL、MongoDB等）
│
└─ NO: 如果只测数据库API的正确调用
    └─ Mock数据库（jest.mock()）

是否需要真实外部API？
├─ YES: 如果API的行为影响测试
│   └─ 使用测试环境API或API Gateway
│
└─ NO: 如果只测本服务的调用逻辑
    └─ Mock外部API（nock, msw等）

是否需要真实消息队列？
├─ YES: 如果消息顺序和处理很关键
│   └─ 使用真实MQ或本地MQ
│
└─ NO: 如果只测消息发送逻辑
    └─ Mock消息队列
```

**最佳实践**：
- 依赖的外部服务：通常Mock
- 测试的核心逻辑：通常真实
- 数据库：如果涉及，应该真实（或用test DB）

### 步骤3: 设计测试场景

为不同的场景编写测试。

#### 3.1 Happy Path（正常流程）

多个Task按预期协作的场景。

```typescript
describe('订单处理集成测试 - Happy Path', () => {
  test('应该能完整处理订单流程', async () => {
    // Arrange
    const order = { id: '123', amount: 100 };
    jest.spyOn(paymentService, 'charge')
      .mockResolvedValue({ status: 'success' });

    // Act
    const result = await orderService.processOrder(order);

    // Assert
    expect(result.status).toBe('completed');
    expect(paymentService.charge).toHaveBeenCalledWith(100);
  });
});
```

#### 3.2 Failure Scenarios（失败场景）

一个Task失败时的影响。

```typescript
describe('订单处理集成测试 - 失败场景', () => {
  test('支付失败时应该回滚订单', async () => {
    // Arrange
    const order = { id: '123', amount: 100 };
    jest.spyOn(paymentService, 'charge')
      .mockRejectedValue(new Error('Payment failed'));

    // Act
    const result = await orderService.processOrder(order);

    // Assert
    expect(result.status).toBe('failed');
    expect(notificationService.send).not.toHaveBeenCalled();
  });
});
```

#### 3.3 Boundary Cases（边界情况）

特殊但可能发生的情况。

```typescript
describe('订单处理集成测试 - 边界情况', () => {
  test('应该处理并发订单请求', async () => {
    // Arrange
    const orders = [
      { id: '1', amount: 100 },
      { id: '2', amount: 200 },
      { id: '3', amount: 150 }
    ];

    // Act
    const results = await Promise.all(
      orders.map(o => orderService.processOrder(o))
    );

    // Assert
    expect(results).toHaveLength(3);
    expect(results.every(r => r.status === 'completed')).toBe(true);
  });

  test('应该处理超时的外部API调用', async () => {
    // Arrange
    jest.spyOn(paymentService, 'charge')
      .mockImplementation(() => new Promise(resolve =>
        setTimeout(() => resolve({ status: 'timeout' }), 5000)
      ));

    // Act & Assert
    await expect(
      Promise.race([
        orderService.processOrder(order),
        timeout(3000)
      ])
    ).rejects.toThrow('Timeout');
  });
});
```

#### 3.4 State Transitions（状态转移）

数据经过多个处理步骤的验证。

```typescript
describe('订单处理集成测试 - 状态转移', () => {
  test('订单应该经过正确的状态转移', async () => {
    // Arrange
    const order = new Order({ status: 'pending' });

    // Act
    expect(order.status).toBe('pending');

    await orderService.validate(order);
    expect(order.status).toBe('validating');

    await orderService.charge(order);
    expect(order.status).toBe('charged');

    await orderService.ship(order);
    expect(order.status).toBe('shipped');

    // Assert
    expect(order.status).toBe('shipped');
  });
});
```

---

## 集成测试实现

### 框架和工具选择

| 技术栈 | 推荐框架 | Mock工具 | 数据库 |
|--------|---------|---------|--------|
| Node.js | Jest / Mocha | jest.mock(), sinon | SQLite (test) |
| Python | Pytest | pytest.mock() | SQLite (test) |
| Java | JUnit + Mockito | Mockito | H2DB |
| Go | testify | testify/mock | SQLite |
| Frontend | Jest + RTL | jest.mock(), msw | - |

### 实现模板

#### Node.js/Jest示例

```typescript
// tests/integration/order.integration.test.ts

import { orderService } from '../../src/services/orderService';
import { paymentService } from '../../src/services/paymentService';
import { notificationService } from '../../src/services/notificationService';
import { db } from '../../src/database';

// Mock外部服务，但真实数据库
jest.mock('../../src/services/paymentService');
jest.mock('../../src/services/notificationService');

describe('Order Integration Tests', () => {
  beforeEach(async () => {
    // 初始化测试数据库
    await db.clear();
    jest.clearAllMocks();
  });

  afterEach(async () => {
    // 清理
    await db.close();
  });

  describe('Happy Path', () => {
    test('should complete order processing successfully', async () => {
      // Arrange
      const order = { id: '123', amount: 100 };
      (paymentService.charge as jest.Mock).mockResolvedValue({
        status: 'success',
        transactionId: 'TXN123'
      });

      // Act
      const result = await orderService.processOrder(order);

      // Assert
      expect(result.status).toBe('completed');
      expect(paymentService.charge).toHaveBeenCalledWith(100);
      expect(notificationService.send).toHaveBeenCalledWith(
        expect.objectContaining({ type: 'order_completed' })
      );

      // 验证数据库中的数据
      const dbOrder = await db.orders.findById('123');
      expect(dbOrder.status).toBe('completed');
    });
  });

  describe('Failure Scenarios', () => {
    test('should handle payment failure gracefully', async () => {
      // Arrange
      const order = { id: '123', amount: 100 };
      (paymentService.charge as jest.Mock).mockRejectedValue(
        new Error('Payment declined')
      );

      // Act & Assert
      await expect(orderService.processOrder(order))
        .rejects
        .toThrow('Payment declined');

      // 验证不会发送确认邮件
      expect(notificationService.send).not.toHaveBeenCalled();

      // 验证订单状态未更新
      const dbOrder = await db.orders.findById('123');
      expect(dbOrder.status).not.toBe('completed');
    });
  });

  describe('Boundary Cases', () => {
    test('should handle concurrent requests', async () => {
      // Arrange
      const orders = [
        { id: '1', amount: 100 },
        { id: '2', amount: 200 }
      ];
      (paymentService.charge as jest.Mock).mockResolvedValue({
        status: 'success'
      });

      // Act
      const results = await Promise.all(
        orders.map(o => orderService.processOrder(o))
      );

      // Assert
      expect(results).toHaveLength(2);
      expect(results.every(r => r.status === 'completed')).toBe(true);
    });
  });
});
```

#### Python/Pytest示例

```python
# tests/integration/test_order.py

import pytest
from unittest.mock import patch, MagicMock
from src.services.order_service import OrderService
from src.database import db

@pytest.fixture
def order_service():
    return OrderService()

@pytest.fixture(autouse=True)
def setup_teardown():
    # 初始化
    db.clear()
    yield
    # 清理
    db.close()

class TestOrderIntegration:
    @patch('src.services.payment_service.charge')
    def test_complete_order_processing(self, mock_charge, order_service):
        # Arrange
        order = {'id': '123', 'amount': 100}
        mock_charge.return_value = {'status': 'success'}

        # Act
        result = order_service.process_order(order)

        # Assert
        assert result['status'] == 'completed'
        mock_charge.assert_called_once_with(100)

        # 验证数据库
        db_order = db.orders.find_by_id('123')
        assert db_order['status'] == 'completed'

    @patch('src.services.payment_service.charge')
    def test_payment_failure_handling(self, mock_charge, order_service):
        # Arrange
        order = {'id': '123', 'amount': 100}
        mock_charge.side_effect = Exception('Payment declined')

        # Act & Assert
        with pytest.raises(Exception) as exc_info:
            order_service.process_order(order)

        assert 'Payment declined' in str(exc_info.value)
```

---

## 常见问题和解决方案

### Q1: 集成测试应该有多少？

**A**: 按照以下比例（金字塔测试）：
```
      E2E (少) - 5%
    集成 (中) - 15%
  单元 (多) - 80%
```

**实际建议**：
- 单元测试 (Jest): 100+个
- 集成测试: 20-30个（关键业务流程）
- E2E测试: 5-10个（核心场景）

### Q2: 集成测试应该有多快？

**A**:
- 单元测试 → 毫秒级 (< 1ms)
- 集成测试 → 秒级 (1-5s)
- E2E测试 → 秒级 (5-30s)

**优化建议**：
- 使用并发运行: `jest --maxWorkers=4`
- 避免真实网络调用: 使用Mock或VCR
- 使用轻量级数据库: SQLite而非PostgreSQL

### Q3: 如何处理集成测试的数据库state？

**A**: 三种方法：

1. **事务回滚** (推荐)
```typescript
beforeEach(async () => {
  await db.transaction(async () => {
    // 测试代码在事务中运行
    // 测试结束后自动回滚
  });
});
```

2. **删除后重建**
```typescript
beforeEach(async () => {
  await db.clear(); // 删除所有数据
  await db.seed();  // 导入初始数据
});
```

3. **快照隔离**
```typescript
beforeEach(async () => {
  await db.createSnapshot();
});

afterEach(async () => {
  await db.restoreSnapshot();
});
```

### Q4: 集成测试中何时Mock，何时真实调用？

**A**: 决策表：

| 对象 | 应该Mock? | 原因 |
|------|----------|------|
| 数据库 | ❌ 否 | 业务逻辑和持久化是核心 |
| 外部API | ✅ 是 | 网络不稳定，增加测试耗时 |
| 消息队列 | 情况而定 | 如果测试消息顺序，不Mock |
| 文件系统 | ✅ 是 | IO操作慢，可能有副作用 |
| 时间 | ✅ 是 | jest.useFakeTimers() |
| 随机数 | ✅ 是 | 保证测试可重复 |

### Q5: 集成测试失败时如何调试？

**A**: 调试步骤：

1. **查看错误信息**
```bash
npm test -- --verbose
```

2. **检查Mock调用**
```typescript
expect(mockService.method).toHaveBeenCalledWith(/* 参数 */);
// 如果失败，检查实际调用了什么
console.log(mockService.method.mock.calls);
```

3. **检查数据库状态**
```typescript
const state = await db.orders.find({});
console.log('Database state:', state);
```

4. **逐步运行**
```typescript
test.only('specific test', async () => {
  // 只运行这个测试，便于调试
});
```

5. **添加日志**
```typescript
test('order processing', async () => {
  console.log('Starting test');
  const result = await orderService.process(order);
  console.log('Result:', result);
  // ... assertions
});
```

### Q6: 如何避免集成测试的Flaky问题？

**A**: 常见原因和解决方案：

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 异步操作未等待 | 偶尔失败 | 使用 async/await, waitFor() |
| Mock状态未清理 | 测试顺序相关 | beforeEach清理 Mock |
| 时间依赖 | 运行慢时失败 | jest.useFakeTimers() |
| 并发问题 | 高负载时失败 | 添加并发测试 |
| 数据库锁 | 间歇性超时 | 使用事务或快照隔离 |

**最佳实践**：
```typescript
// ❌ Flaky - 依赖真实时间
setTimeout(() => {
  expect(result).toBe('done');
}, 1000);

// ✅ 稳定 - 使用fake timers
jest.useFakeTimers();
jest.advanceTimersByTime(1000);
expect(result).toBe('done');
```

---

## 集成测试检查清单

在提交集成测试前，确保：

- [ ] 识别了正确的测试边界（哪些Task需要协作）
- [ ] 定义了清晰的Mock策略（什么Mock，什么真实）
- [ ] 包含了Happy Path和Failure Scenarios
- [ ] 处理了并发和边界情况
- [ ] 测试数据库state管理（初始化、清理）
- [ ] 测试通过（npm test -- --coverage）
- [ ] 覆盖率 ≥ 80%
- [ ] 运行速度可接受（< 5s每个测试）
- [ ] Mock调用验证正确（expect().toHaveBeenCalledWith）
- [ ] 没有依赖测试执行顺序
- [ ] 没有真实网络调用或外部依赖

---

## 总结

集成测试是连接单元测试和E2E测试的桥梁：

✅ **重要**：多Task协作、跨模块通信、复杂数据流
✅ **适度**：不要过多（15%左右），焦点应在单元测试
✅ **清晰**：明确的Mock策略和测试边界
✅ **可靠**：避免Flaky，保证可重复运行

**从Phase 2继承的Context**：
- test-design已生成单元测试规范和fixtures
- code-execute已通过TDD实现代码
- 现在在Phase 3中，code-test负责集成、E2E、性能测试的设计

**开始集成测试设计！** 🚀

