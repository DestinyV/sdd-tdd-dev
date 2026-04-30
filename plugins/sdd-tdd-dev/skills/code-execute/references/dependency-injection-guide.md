# 依赖注入 (DI) 指南

> code-execute 阶段代码实现参考

## 注入方式对比

| 方式 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **构造函数注入** | 必须依赖（无此依赖对象无法工作） | 明确依赖、不可变、易测试 | 构造函数参数可能过多 |
| **属性注入** | 可选依赖（有默认行为） | 灵活、可按需设置 | 依赖不明确、测试时需要手动设置 |
| **方法注入** | 临时依赖（只在某个方法中需要） | 精确控制作用域 | 每次调用都要传入 |

---

## TypeScript

### 构造函数注入

```typescript
class OrderService {
  private readonly orderRepo: OrderRepository;
  private readonly paymentGateway: PaymentGateway;
  private readonly eventBus: EventBus;

  constructor(
    orderRepo: OrderRepository,
    paymentGateway: PaymentGateway,
    eventBus: EventBus,
  ) {
    this.orderRepo = orderRepo;
    this.paymentGateway = paymentGateway;
    this.eventBus = eventBus;
  }

  async createOrder(data: CreateOrderInput): Promise<Order> {
    const order = Order.create(data);
    await this.orderRepo.save(order);
    const payment = await this.paymentGateway.charge(order.getTotal());
    order.confirmPayment(payment.id);
    this.eventBus.publish(new OrderCreatedEvent(order));
    return order;
  }
}
```

### 简单 IoC Container

```typescript
type Factory<T> = () => T;

class Container {
  private factories = new Map<string, Factory<any>>();
  private instances = new Map<string, any>();

  register<T>(token: string, factory: Factory<T>): void {
    this.factories.set(token, factory);
  }

  resolve<T>(token: string): T {
    if (!this.instances.has(token)) {
      const factory = this.factories.get(token)!;
      this.instances.set(token, factory());
    }
    return this.instances.get(token);
  }
}

// 注册
const container = new Container();
container.register('OrderRepository', () => new MySQLOrderRepository(db));
container.register('PaymentGateway', () => new StripePaymentGateway(apiKey));
container.register('OrderService', () =>
  new OrderService(
    container.resolve('OrderRepository'),
    container.resolve('PaymentGateway'),
  )
);

// 使用
const orderService = container.resolve<OrderService>('OrderService');
```

---

## Python

### 构造函数注入

```python
class OrderService:
    def __init__(
        self,
        order_repo: OrderRepository,
        payment_gateway: PaymentGateway,
    ):
        self._order_repo = order_repo
        self._payment_gateway = payment_gateway

    def create_order(self, data: CreateOrderInput) -> Order:
        order = Order.create(data)
        self._order_repo.save(order)
        payment = self._payment_gateway.charge(order.get_total())
        order.confirm_payment(payment.id)
        return order
```

### 依赖注入装饰器

```python
from typing import TypeVar, Generic, Type

T = TypeVar('T')

class Injectable(Generic[T]):
    _registry: dict[type, T] = {}

    @classmethod
    def register(cls, instance: T):
        cls._registry[instance.__class__] = instance

    @classmethod
    def resolve(cls, cls_type: Type[T]) -> T:
        return cls._registry[cls_type]

# 注册实例
Injectable.register(MySQLOrderRepository())
Injectable.register(StripePaymentGateway())

# 解析实例
repo = Injectable.resolve(MySQLOrderRepository)
gateway = Injectable.resolve(StripePaymentGateway)
```

---

## Go

### 构造函数注入

```go
type OrderService struct {
    repo    OrderRepository
    gateway PaymentGateway
    event   EventBus
}

func NewOrderService(repo OrderRepository, gateway PaymentGateway, event EventBus) *OrderService {
    return &OrderService{repo: repo, gateway: gateway, event: event}
}

func (s *OrderService) CreateOrder(data CreateOrderInput) (*Order, error) {
    order := NewOrder(data)
    if err := s.repo.Save(order); err != nil {
        return nil, err
    }
    payment, err := s.gateway.Charge(order.GetTotal())
    if err != nil {
        return nil, err
    }
    order.ConfirmPayment(payment.ID)
    s.event.Publish(OrderCreatedEvent{Order: order})
    return order, nil
}
```

---

## Java (Spring)

```java
@Service
public class OrderService {
    private final OrderRepository orderRepo;
    private final PaymentGateway paymentGateway;

    // Spring 自动注入（构造函数注入）
    public OrderService(OrderRepository orderRepo, PaymentGateway paymentGateway) {
        this.orderRepo = orderRepo;
        this.paymentGateway = paymentGateway;
    }
}
```

---

## 何时不需要 DI

- **简单工具函数**：纯函数，无状态依赖
- **一次性脚本**：不需要测试和复用
- **配置对象**：全局只读配置，不需要替换
- **DTO/ValueObject**：纯数据载体

---

## 常见反模式

### Service Locator（服务定位器）

```typescript
// ❌ 隐藏依赖，难以测试
class OrderService {
  createOrder(data: CreateOrderInput) {
    const repo = ServiceLocator.get<OrderRepository>('OrderRepository');
    const gateway = ServiceLocator.get<PaymentGateway>('PaymentGateway');
    // ...
  }
}
```

### 过度注入

```typescript
// ❌ 构造函数参数过多（>7个）
class GodService {
  constructor(
    private a: ServiceA,
    private b: ServiceB,
    private c: ServiceC,
    private d: ServiceD,
    private e: ServiceE,
    private f: ServiceF,
    private g: ServiceG,
  ) {}
}
// ✅ 应该使用 Facade 或进一步拆分
```

### 注入具体实现而非接口

```typescript
// ❌ 耦合到具体实现
class OrderService {
  constructor(private repo: MySQLOrderRepository) {}
}

// ✅ 依赖接口
class OrderService {
  constructor(private repo: OrderRepository) {}
}
```
