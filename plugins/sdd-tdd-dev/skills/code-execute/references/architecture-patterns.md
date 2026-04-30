# 常用架构模式参考

> code-designer 阶段架构决策和 code-execute 阶段实现参考

---

## Clean Architecture（整洁架构）

**适用场景**：中大型后端应用，业务逻辑复杂，需要独立测试。

**核心依赖规则（从外到内，内层不依赖外层）**：

```
┌─────────────────────────────────────────────────┐
│                外层：基础设施层                     │
│  数据库、文件系统、外部API、框架、UI                │
├─────────────────────────────────────────────────┤
│             中间层：应用层                         │
│  用例（Use Cases）、应用服务、业务流程编排           │
├─────────────────────────────────────────────────┤
│             内层：领域层                           │
│  实体（Entities）、值对象、领域服务、业务规则         │
└─────────────────────────────────────────────────┘

依赖方向：外层 → 中间层 → 内层
内层绝对不能依赖外层
```

**各层职责**：

| 层 | 职责 | 示例文件 |
|---|------|---------|
| **领域层** | 业务实体、核心业务规则、不依赖任何外部技术 | `domain/entities/User.ts`、`domain/services/OrderService.ts` |
| **应用层** | 用例编排、事务管理、调用领域服务 | `application/usecases/CreateOrderUseCase.ts` |
| **基础设施层** | 数据库操作、外部 API 调用、缓存、日志 | `infrastructure/repositories/UserRepository.ts` |

**代码示例**：
```typescript
// 领域层：实体（纯业务逻辑，无框架依赖）
class Order {
  constructor(
    public readonly id: string,
    private items: OrderItem[],
    private status: OrderStatus,
  ) {}

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  cancel(): void {
    if (this.status === 'SHIPPED') throw new Error('Cannot cancel shipped order');
    this.status = 'CANCELLED';
  }
}

// 应用层：用例（编排领域层和基础设施层）
class CreateOrderUseCase {
  constructor(
    private orderRepo: OrderRepository,
    private paymentGateway: PaymentGateway,
  ) {}

  async execute(input: CreateOrderInput): Promise<Order> {
    const order = Order.create(input.items);
    const payment = await this.paymentGateway.charge(order.getTotal());
    order.confirmPayment(payment.id);
    return this.orderRepo.save(order);
  }
}

// 基础设施层：实现（依赖具体技术）
class MySQLUserRepository implements UserRepository {
  constructor(private db: Knex) {}
  async findById(id: string): Promise<User> { ... }
}
```

**与项目对标**：如果项目已有 `services/`、`controllers/`、`repositories/` 分层，可映射为 Clean Architecture 的应用层、基础设施层（控制器）、基础设施层（数据访问）。

---

## Hexagonal Architecture（六边形架构 / Ports & Adapters）

**适用场景**：需要隔离核心业务逻辑与外部依赖的项目，支持多种数据库/外部服务切换。

**核心概念**：

```
                    ┌──────────────┐
                    │   驱动适配器   │ ← 外部触发核心（控制器、CLI、MQ消费者）
                    │  (Driving)    │
┌───────────────────┴──────────────┴───────────────────┐
│                    核心领域                            │
│  ┌────────────┐     ┌──────────────┐     ┌─────────┐ │
│  │  端口(Port) │ ←→ │  领域服务     │ ←→ │  实体    │ │
│  │  (接口)     │     │  (业务逻辑)   │     │         │ │
│  └────────────┘     └──────────────┘     └─────────┘ │
├───────────────────┬──────────────┬───────────────────┤
│                   │  被驱动适配器 │                   │
│                   │  (Driven)    │                   │
│                   │ DB适配器、    │                   │
│                   │ 外部API适配器 │                   │
└───────────────────┴──────────────┴───────────────────┘

核心规则：核心只依赖端口（接口），不依赖适配器（实现）
```

**端口（Port）vs 适配器（Adapter）**：

| 概念 | 说明 | 示例 |
|------|------|------|
| **驱动端口** | 核心对外提供的接口 | `OrderService` 接口 |
| **驱动适配器** | 将外部请求转换为核心调用 | `OrderController` |
| **被驱动端口** | 核心需要的能力接口 | `OrderRepository` 接口 |
| **被驱动适配器** | 能力的具体实现 | `MySQLOrderRepository`、`RedisOrderRepository` |

**代码示例**：
```typescript
// 端口（接口，核心定义）
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order>;
}

// 被驱动适配器（具体实现，基础设施层）
class MySQLOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> { /* SQL 实现 */ }
  async findById(id: string): Promise<Order> { /* SQL 实现 */ }
}

class RedisOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> { /* Redis 实现 */ }
  async findById(id: string): Promise<Order> { /* Redis 实现 */ }
}
```

---

## Layered Architecture（分层架构）

**适用场景**：最常见的企业级应用架构，适合大多数 CRUD 和中等复杂度项目。

**层级和依赖规则**：

```
┌─────────────────────────────────────────┐
│  表现层（Presentation/Controller）       │  ← 接收请求、返回响应
│  - 路由定义                               │
│  - 请求验证                               │
│  - 响应格式化                             │
├─────────────────────────────────────────┤
│  业务层（Service/Business Logic）        │  ← 业务规则、事务管理
│  - 业务逻辑                               │
│  - 事务边界                               │
│  - 权限校验                               │
├─────────────────────────────────────────┤
│  数据层（Repository/DAO）                │  ← 数据访问
│  - 数据库 CRUD                           │
│  - 查询构建                               │
│  - 缓存策略                               │
├─────────────────────────────────────────┤
│  实体层（Entity/Model）                  │  ← 数据模型
│  - 领域对象                               │
│  - DTO                                   │
│  - 枚举                                   │
└─────────────────────────────────────────┘

依赖方向：表现层 → 业务层 → 数据层 → 实体层
禁止反向依赖：数据层不能调用业务层，业务层不能调用表现层
```

**禁止的依赖**：
- ❌ 数据层不能依赖业务层
- ❌ 实体层不能依赖任何层（最底层）
- ❌ 表现层不能直接调用数据层（必须经过业务层）

**代码示例**：
```typescript
// 实体层
class User { id: number; name: string; email: string; }

// 数据层
class UserRepository {
  async findById(id: number): Promise<User> { ... }
}

// 业务层
class UserService {
  constructor(private repo: UserRepository) {}
  async getUser(id: number): Promise<User> {
    const user = await this.repo.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return user;
  }
}

// 表现层
class UserController {
  constructor(private service: UserService) {}
  async handle(req, res) {
    const user = await this.service.getUser(req.params.id);
    res.json({ code: 0, data: user });
  }
}
```

---

## MVC / MVVM（前端架构）

**适用场景**：前端组件化开发，Vue/React 等现代前端框架。

### MVC（Model-View-Controller）

```
View (UI组件)  ←→  Controller (事件处理/状态管理)  ←→  Model (数据/API)
```

| 层 | 职责 | Vue/React 对应 |
|---|------|---------------|
| **View** | UI 展示、用户交互 | 模板/SFC 的 template 部分 |
| **Controller** | 事件处理、状态管理、数据转换 | script setup、Hooks |
| **Model** | 数据获取、状态存储 | API 调用、Store (Pinia/Redux) |

### 前端组件分层实践

```vue
<!-- View: 模板 -->
<template>
  <div class="order-list">
    <OrderCard v-for="order in orders" :key="order.id" :order="order" />
  </div>
</template>

<!-- Controller + Model: 逻辑和数据 -->
<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useOrderStore } from '@/stores/order';

const store = useOrderStore();
const orders = computed(() => store.orders);

onMounted(async () => {
  await store.fetchOrders();  // Model 层：数据获取
});

function handleOrderClick(id: number) {  // Controller 层：事件处理
  router.push(`/orders/${id}`);
}
</script>
```

---

## CQRS（命令查询职责分离）

**适用场景**：读写负载差异大、读写模型差异大的后端系统。

**核心思想**：将写操作（Command）和读操作（Query）使用不同的模型和处理路径。

```
┌─────────────────────┐    ┌─────────────────────┐
│     命令侧 (Write)   │    │     查询侧 (Read)    │
│                     │    │                     │
│  Command → Handler  │    │  Query → Handler    │
│      ↓              │    │      ↓              │
│  Domain Model       │    │  Read Model (DTO)   │
│      ↓              │    │      ↓              │
│  Write DB           │    │  Read DB / Cache    │
└─────────────────────┘    └─────────────────────┘
```

**代码示例**：
```typescript
// 命令侧
class CreateOrderCommand {
  constructor(public readonly userId: number, public readonly items: OrderItem[]) {}
}

class CreateOrderHandler {
  async handle(cmd: CreateOrderCommand): Promise<Order> {
    const order = Order.create(cmd.items);
    return this.orderRepo.save(order);
  }
}

// 查询侧
class OrderDetailQuery {
  constructor(public readonly orderId: number) {}
}

class OrderDetailQueryHandler {
  async handle(query: OrderDetailQuery): Promise<OrderDetailView> {
    return this.readDb.query(`
      SELECT o.*, oi.* FROM orders o
      JOIN order_items oi ON oi.order_id = o.id
      WHERE o.id = ?
    `, [query.orderId]);
  }
}
```

---

## 架构模式选择指南

| 项目特征 | 推荐模式 | 理由 |
|---------|---------|------|
| 简单 CRUD | Layered | 简单直接，团队熟悉 |
| 中等复杂度 | Layered + 部分 Clean | 核心业务逻辑独立 |
| 高复杂度、多外部依赖 | Clean / Hexagonal | 隔离核心业务，易测试 |
| 读写差异大 | Layered + CQRS | 读写独立优化 |
| 前端项目 | MVC/MVVM（组件化） | 框架天然支持 |
| 需要切换数据库/服务 | Hexagonal | 端口适配模式 |
