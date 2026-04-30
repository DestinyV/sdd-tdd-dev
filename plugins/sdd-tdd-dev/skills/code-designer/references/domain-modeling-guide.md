# 领域建模指南

> code-designer 阶段后端领域模型设计参考

## 何时使用 DDD

### 适用场景

| 项目特征 | 是否适合 DDD |
|---------|-------------|
| 业务逻辑复杂，有很多规则和约束 | ✅ 适合 |
| 多个业务领域需要清晰边界 | ✅ 适合 |
| 团队需要和领域专家使用统一语言 | ✅ 适合 |
| 长期维护、持续迭代的系统 | ✅ 适合 |
| 简单 CRUD 操作 | ❌ 不适合 |
| 数据录入/查询为主 | ❌ 不适合 |
| 快速原型/一次性脚本 | ❌ 不适合 |

---

## 核心概念

### 1. 实体 (Entity)

有唯一标识，属性会随时间变化的对象。

**TypeScript**：
```typescript
class User {
  private readonly id: string;
  private name: string;
  private email: string;

  constructor(id: string, name: string, email: string) {
    this.id = id; this.name = name; this.email = email;
  }

  changeName(newName: string): void {
    if (!newName || newName.length < 2) throw new Error('Name too short');
    this.name = newName;
  }

  getId(): string { return this.id; }
}
```

**Go**：
```go
type User struct {
    id    string
    name  string
    email string
}

func (u *User) ChangeName(newName string) error {
    if len(newName) < 2 { return errors.New("name too short") }
    u.name = newName
    return nil
}

func (u *User) ID() string { return u.id }
```

### 2. 值对象 (Value Object)

通过属性值定义，无唯一标识，不可变。

**TypeScript**：
```typescript
class Address {
  constructor(
    readonly street: string,
    readonly city: string,
    readonly province: string,
    readonly zipCode: string,
  ) {}

  withZipCode(newZip: string): Address {
    return new Address(this.street, this.city, this.province, newZip);
  }

  equals(other: Address): boolean {
    return this.street === other.street && this.city === other.city
      && this.province === other.province && this.zipCode === other.zipCode;
  }
}
```

**Python**：
```python
from dataclasses import dataclass

@dataclass(frozen=True)  # 不可变
class Money:
    amount: float
    currency: str

    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)
```

### 3. 聚合 (Aggregate)

一组相关对象的集合和一致性边界。外部只能通过聚合根访问。

**TypeScript**：
```typescript
class Order {
  private readonly id: string;
  private items: OrderItem[] = [];
  private status: OrderStatus;

  // 通过聚合根操作内部对象
  addItem(product: Product, quantity: number): void {
    if (this.status !== 'DRAFT') throw new Error('Only draft orders can be modified');
    const existing = this.items.find(i => i.productId === product.id);
    if (existing) existing.increaseQuantity(quantity);
    else this.items.push(new OrderItem(product.id, quantity, product.price));
    this.recalculateTotal();
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce((sum, i) => sum + i.subtotal, 0);
  }
}
```

**聚合规则**：
- 外部只能通过聚合根引用内部对象
- 聚合内对象必须有强一致性
- 一个事务只修改一个聚合

### 4. 仓储 (Repository)

提供访问聚合根的接口，隐藏数据访问细节。

**TypeScript**：
```typescript
// 领域层：接口
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<Order>;
}

// 基础设施层：实现
class MySQLOrderRepository implements OrderRepository {
  constructor(private db: Knex) {}
  async findById(id: string): Promise<Order | null> { ... }
  async save(order: Order): Promise<Order> { ... }
}
```

**Python**：
```python
class OrderRepository(ABC):
    @abstractmethod
    def find_by_id(self, id: str) -> Optional[Order]: ...

    @abstractmethod
    def save(self, order: Order) -> Order: ...

class SQLOrderRepository(OrderRepository):
    def __init__(self, db: Session): ...
    def find_by_id(self, id: str) -> Optional[Order]: ...
    def save(self, order: Order) -> Order: ...
```

### 5. 限界上下文 (Bounded Context)

一个明确的业务边界，在边界内领域模型有统一含义。

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

---

## 从 Spec 提取领域模型四步法

1. **识别名词** → 从 spec 场景中提取所有业务名词
2. **分类** → 判断每个名词是实体还是值对象
3. **划定聚合** → 确定哪些对象属于同一聚合，聚合根是谁
4. **提取行为** → 从场景的 WHEN-THEN 中提取业务行为

**示例**：
```
Spec: "当用户提交订单时，系统扣减库存，创建支付记录，更新订单状态"

步骤1 名词: 订单, 库存, 支付记录, 订单状态
步骤2 分类: 订单(实体), 订单项(值对象), 支付(实体), 库存(实体)
步骤3 聚合: [订单 + 订单项] (聚合根: Order), [支付] (聚合根: Payment)
步骤4 行为: Order.submit(), Inventory.deduct(), Payment.create(), Order.updateStatus()
```

---

## 贫血模型 vs 富血模型

### 贫血模型（只有数据）

```typescript
class Order {
  items: OrderItem[];
  total: number;
  status: string;
}

class OrderService {
  addItem(order: Order, item: OrderItem) { order.items.push(item); order.total = ...; }
  cancel(order: Order) { order.status = 'CANCELLED'; }
}
```

### 富血模型（有行为）

```typescript
class Order {
  private items: OrderItem[];
  private status: OrderStatus;

  addItem(item: OrderItem) { this.items.push(item); this.recalculateTotal(); }
  cancel() {
    if (this.status === 'SHIPPED') throw new Error('Cannot cancel shipped order');
    this.status = 'CANCELLED';
  }
  private recalculateTotal() { ... }
}
```

**取舍**：
- 贫血模型 + 服务类：简单，但容易导致 God Service
- 富血模型：业务逻辑内聚，但对象可能膨胀
- 建议：核心业务逻辑放领域模型，协调逻辑放领域服务
