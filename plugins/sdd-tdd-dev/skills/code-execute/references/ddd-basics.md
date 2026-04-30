# DDD 基础概念参考

> code-designer 阶段后端领域模型设计和 code-execute 阶段后端实现参考

---

## 什么是 DDD（领域驱动设计）

DDD 是一种通过**业务领域**来组织软件架构的设计方法。核心思想是：软件系统的复杂性应该由领域逻辑决定，而不是由技术框架决定。

---

## 核心概念

### 1. 限界上下文 (Bounded Context)

**定义**：一个明确的边界，在这个边界内，领域模型有统一的、明确的含义。

**实际意义**：同一个概念在不同上下文中有不同含义。

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   订单上下文     │    │   库存上下文     │    │   支付上下文     │
│                 │    │                 │    │                 │
│ Order:          │    │ Product:        │    │ Payment:        │
│  - orderId      │    │  - productId    │    │  - paymentId    │
│  - items[]      │    │  - stockCount   │    │  - amount       │
│  - totalAmount  │    │  - reservedQty  │    │  - status       │
│  - status       │    │                 │    │  - method       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

**代码组织**：
```
src/
├── order/                    # 订单限界上下文
│   ├── entities/
│   ├── services/
│   ├── repositories/
│   └── dtos/
├── inventory/                # 库存限界上下文
│   ├── entities/
│   ├── services/
│   └── repositories/
├── payment/                  # 支付限界上下文
│   ├── entities/
│   ├── services/
│   └── repositories/
```

### 2. 实体 (Entity)

**定义**：有唯一标识的对象，其相等性由标识决定而非属性值。实体有生命周期，属性会随时间变化。

**关键特征**：
- 有唯一标识（ID）
- 属性会变化
- 有业务行为（不仅仅是数据容器）

```typescript
class User {
  private readonly id: string;  // 唯一标识
  private name: string;
  private email: string;
  private status: UserStatus;

  constructor(id: string, name: string, email: string) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.status = 'ACTIVE';
  }

  // 业务行为：改名
  changeName(newName: string): void {
    if (!newName || newName.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    this.name = newName;
  }

  // 业务行为：封禁
  ban(): void {
    if (this.status === 'BANNED') {
      throw new Error('User is already banned');
    }
    this.status = 'BANNED';
  }

  getId(): string { return this.id; }
  getStatus(): UserStatus { return this.status; }
}
```

### 3. 值对象 (Value Object)

**定义**：通过属性值定义的对象，没有唯一标识。值对象是不可变的。

**关键特征**：
- 无唯一标识
- 不可变（一旦创建，属性不变化）
- 相等性由属性值决定
- 可以替换为具有相同值的另一个对象

```typescript
// ❌ 错误：用实体存储地址
class User {
  addressStreet: string;
  addressCity: string;
  addressProvince: string;
  addressZipCode: string;
}

// ✅ 正确：用值对象
class Address {
  constructor(
    readonly street: string,
    readonly city: string,
    readonly province: string,
    readonly zipCode: string,
  ) {}

  // 值对象的方法：返回新的值对象（不可变）
  withZipCode(newZip: string): Address {
    return new Address(this.street, this.city, this.province, newZip);
  }

  equals(other: Address): boolean {
    return this.street === other.street
      && this.city === other.city
      && this.province === other.province
      && this.zipCode === other.zipCode;
  }
}

class User {
  private address: Address;  // 使用值对象

  updateAddress(newAddress: Address): void {
    this.address = newAddress;  // 整体替换，不是逐个属性修改
  }
}
```

### 4. 聚合根 (Aggregate Root)

**定义**：一组相关对象的集合和一致性边界。外部只能通过聚合根访问内部对象。

**规则**：
- 聚合内的对象必须有强一致性
- 外部只能通过聚合根引用内部对象
- 对聚合的修改必须通过聚合根
- 一个事务只修改一个聚合

```typescript
// Order 是聚合根，OrderItem 是聚合内的实体
class Order {
  private readonly id: string;
  private items: OrderItem[] = [];
  private status: OrderStatus;

  // 通过聚合根操作内部对象
  addItem(product: Product, quantity: number): void {
    if (this.status !== 'DRAFT') {
      throw new Error('Can only add items to draft orders');
    }
    const existing = this.items.find(i => i.productId === product.id);
    if (existing) {
      existing.increaseQuantity(quantity);
    } else {
      this.items.push(new OrderItem(product.id, quantity, product.price));
    }
    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    this.items = this.items.filter(i => i.productId !== productId);
    this.recalculateTotal();
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce((sum, item) => sum + item.subtotal, 0);
  }
}

// ❌ 错误：外部直接修改聚合内对象
// order.items[0].quantity = 5;

// ✅ 正确：通过聚合根的方法
order.addItem(product, 5);
```

### 5. 领域服务 (Domain Service)

**定义**：不属于任何单个实体或值对象、涉及多个领域对象协作的业务逻辑。

**识别标准**：操作涉及多个领域对象，且不天然属于其中任何一个。

```typescript
// ❌ 错误：把跨领域逻辑放在某个实体里
class Order {
  async processPayment(paymentGateway: PaymentGateway): Promise<void> {
    // Order 不应该直接调用支付网关
  }
}

// ✅ 正确：提取为领域服务
class OrderPaymentService {
  constructor(
    private paymentGateway: PaymentGateway,
    private orderRepository: OrderRepository,
  ) {}

  async processPayment(order: Order): Promise<Payment> {
    const payment = await this.paymentGateway.charge(order.getTotal());
    order.confirmPayment(payment.id);
    await this.orderRepository.save(order);
    return payment;
  }
}
```

### 6. 应用服务 (Application Service)

**定义**：协调用例执行的薄层，负责事务管理、权限校验、调用领域服务和仓储。

**与领域服务的区别**：
- 应用服务：编排和协调（What to do）
- 领域服务：业务规则（How to do）

```typescript
class CreateOrderApplicationService {
  constructor(
    private orderRepository: OrderRepository,
    private inventoryService: InventoryService,
    private eventBus: EventBus,
  ) {}

  async execute(request: CreateOrderRequest): Promise<OrderResult> {
    // 1. 权限校验
    this.ensureUserCanOrder(request.userId);

    // 2. 开启事务
    return await this.withTransaction(async () => {
      // 3. 调用领域服务
      const order = Order.create(request);
      await this.inventoryService.reserve(order.items);

      // 4. 持久化
      const saved = await this.orderRepository.save(order);

      // 5. 发布事件
      this.eventBus.publish(new OrderCreatedEvent(saved));

      return OrderResult.from(saved);
    });
  }
}
```

### 7. 仓储 (Repository)

**定义**：提供访问聚合根的接口，隐藏数据访问的细节。

```typescript
// 接口定义在领域层
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<Order>;
  delete(id: string): Promise<void>;
  findAllByUserId(userId: string, page: number, size: number): Promise<Order[]>;
}

// 实现在基础设施层
class MySQLOrderRepository implements OrderRepository {
  constructor(private db: Knex) {}

  async findById(id: string): Promise<Order | null> {
    const row = await this.db('orders').where({ id }).first();
    if (!row) return null;
    const items = await this.db('order_items').where({ orderId: id });
    return Order.reconstruct({ ...row, items });
  }

  async save(order: Order): Promise<Order> {
    // ...
  }
}
```

### 8. 统一语言 (Ubiquitous Language)

**定义**：开发人员和领域专家使用的共同语言，体现在代码命名中。

**实践**：
- 类名、方法名、变量名使用业务术语
- 避免技术术语泄露到业务代码中

```typescript
// ❌ 错误：技术术语
class DataProcessor {
  processRow(row: DBRecord): void { ... }
}

// ✅ 正确：业务语言
class OrderProcessor {
  processOrder(order: Order): void { ... }
}
```

---

## 实体 vs 值对象 vs 聚合根 决策树

```
需要建模一个业务对象？
  ↓
是否有唯一标识？─── 否 ──→ 值对象 ✅
  ↓是
是否需要通过它访问内部对象？─── 是 ──→ 聚合根 ✅
  ↓否
实体 ✅
```

---

## 分层架构中的 DDD

```
┌─────────────────────────────────────────┐
│  接口层 (Controllers, API Routes)        │  ← 接收请求、返回响应
├─────────────────────────────────────────┤
│  应用层 (Application Services)            │  ← 用例编排、事务管理
├─────────────────────────────────────────┤
│  领域层 (Entities, Value Objects,         │  ← 业务规则、领域服务
│         Domain Services, Aggregates)      │
├─────────────────────────────────────────┤
│  基础设施层 (Repositories, External APIs) │  ← 数据访问、外部服务
└─────────────────────────────────────────┘
```
