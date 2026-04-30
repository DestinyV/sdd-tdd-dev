# 架构模式指南

> code-designer 阶段架构决策参考

## 架构风格对比

| 维度 | 分层架构 | Clean Architecture | Hexagonal | MVC/MVVM |
|------|---------|-------------------|-----------|---------|
| 复杂度 | 低 | 高 | 中 | 中 |
| 学习曲线 | 平缓 | 陡峭 | 中等 | 中等 |
| 测试友好度 | 中 | 高 | 高 | 中 |
| 适合项目 | CRUD | 复杂业务 | 多外部依赖 | 前端应用 |
| 依赖方向 | 单向 | 向内 | 向核心 | MVC/MVVM |

---

## Clean Architecture（整洁架构）

**适用场景**：中大型后端应用，业务逻辑复杂。

**四层结构**（从内到外，内层不依赖外层）：

```
┌─────────────────────────────────────────────────┐
│          Frameworkes & Drivers (框架层)           │
│  数据库、文件系统、UI、外部API                      │
├─────────────────────────────────────────────────┤
│          Interface Adapters (接口适配层)           │
│  Controllers、Presenters、Gateways               │
├─────────────────────────────────────────────────┤
│          Use Cases (用例层)                       │
│  应用服务、业务流程编排                            │
├─────────────────────────────────────────────────┤
│          Entities (实体层)                        │
│  业务实体、核心业务规则                            │
└─────────────────────────────────────────────────┘
```

**TypeScript**：
```typescript
// 实体层（最内层，无外部依赖）
class Order {
  constructor(private id: string, private items: OrderItem[]) {}
  getTotal(): number { return this.items.reduce(...); }
  cancel(): void { this.status = 'CANCELLED'; }
}

// 用例层
class CreateOrderUseCase {
  constructor(private repo: OrderRepository, private gateway: PaymentGateway) {}
  async execute(input: CreateOrderInput): Promise<Order> {
    const order = Order.create(input.items);
    const payment = await this.gateway.charge(order.getTotal());
    order.confirmPayment(payment.id);
    return this.repo.save(order);
  }
}

// 接口适配层
class OrderController {
  constructor(private usecase: CreateOrderUseCase) {}
  async handle(req, res) {
    const order = await this.usecase.execute(req.body);
    res.json({ code: 0, data: order });
  }
}
```

**Go**：
```go
// entities/order.go
type Order struct { ID string; Items []OrderItem; Status string }
func (o *Order) Cancel() error { ... }

// usecases/create_order.go
type CreateOrderUseCase struct {
    Repo OrderRepository
    Gateway PaymentGateway
}
func (uc *CreateOrderUseCase) Execute(input CreateOrderInput) (*Order, error) { ... }

// interfaces/order_controller.go
type OrderController struct { Usecase *CreateOrderUseCase }
func (c *OrderController) Handle(w http.ResponseWriter, r *http.Request) { ... }
```

---

## Hexagonal Architecture（六边形架构 / Ports & Adapters）

**适用场景**：需要隔离核心业务与外部依赖，支持多种数据库/服务切换。

```
                    ┌──────────────┐
                    │   驱动适配器   │ ← 外部触发核心
                    │  Controller  │
┌───────────────────┴──────────────┴───────────────────┐
│                    核心领域                            │
│  ┌────────────┐     ┌──────────────┐     ┌─────────┐ │
│  │  端口(Port) │ ←→ │  领域服务     │ ←→ │  实体    │ │
│  │  (接口)     │     │  (业务逻辑)   │     │         │ │
│  └────────────┘     └──────────────┘     └─────────┘ │
├───────────────────┬──────────────┬───────────────────┤
│                   │  被驱动适配器 │                   │
│                   │ Repository   │                   │
│                   │ External API │                   │
└───────────────────┴──────────────┴───────────────────┘
```

**Python**：
```python
# ports/order_repository.py
class OrderRepository(Protocol):
    def find_by_id(self, id: str) -> Optional[Order]: ...
    def save(self, order: Order) -> Order: ...

# domain/order.py
class Order:
    def __init__(self, id: str, items: list[OrderItem]): ...
    def add_item(self, item: OrderItem): ...

# adapters/sql_order_repository.py
class SQLOrderRepository:
    def __init__(self, db: Session): ...
    def find_by_id(self, id: str) -> Optional[Order]: ...
    def save(self, order: Order) -> Order: ...

# 组合根
repo = SQLOrderRepository(db_session)
service = OrderService(repo)
```

---

## Layered Architecture（分层架构）

**适用场景**：最常见的企业级应用，适合 CRUD 和中等复杂度项目。

```
表现层（Controller/Handler） → 接收请求、返回响应
    ↓
业务层（Service） → 业务规则、事务管理
    ↓
数据层（Repository/DAO） → 数据库 CRUD
    ↓
实体层（Entity/Model） → 数据模型、DTO
```

**依赖规则**：
- 表现层 → 业务层
- 业务层 → 数据层
- 数据层 → 实体层
- ❌ 禁止反向依赖

**Go**：
```go
// handlers/order_handler.go (表现层)
func (h *OrderHandler) CreateOrder(w http.ResponseWriter, r *http.Request) {
    order, _ := h.service.CreateOrder(req.Body)
    json.NewEncoder(w).Encode(order)
}

// services/order_service.go (业务层)
func (s *OrderService) CreateOrder(data OrderData) (*Order, error) {
    order := NewOrder(data)
    if err := s.repo.Save(order); err != nil { return nil, err }
    return order, nil
}

// repositories/order_repository.go (数据层)
func (r *OrderRepository) Save(order *Order) error {
    return r.db.Exec("INSERT INTO orders...", order.ID, ...).Error
}

// models/order.go (实体层)
type Order struct { ID string; Items []OrderItem; Status string }
```

---

## 架构选择建议

| 项目特征 | 推荐架构 | 理由 |
|---------|---------|------|
| 简单 CRUD | 分层架构 | 简单直接，团队熟悉 |
| 中等复杂度 | 分层 + 部分 Clean | 核心业务逻辑独立 |
| 高复杂度、多外部依赖 | Clean / Hexagonal | 隔离核心业务，易测试 |
| 读写差异大 | 分层 + CQRS | 读写独立优化 |
| 前端应用 | MVC/MVVM | 框架天然支持 |
| 需要切换数据库/服务 | Hexagonal | 端口适配模式 |
| 需要快速交付 MVP | 分层架构 | 开发效率高 |
