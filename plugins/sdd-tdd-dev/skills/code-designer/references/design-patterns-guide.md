# 设计模式指南

> code-designer 阶段设计决策参考，code-execute 阶段实现参考

## 模式决策树

```
需要创建新对象？
  ├─ 对象类型运行时决定？ ──→ Factory Method
  ├─ 对象由多组相互依赖的具体组件构成？ ──→ Abstract Factory
  └─ 构建步骤复杂、有多步配置？ ──→ Builder

需要组合或扩展对象行为？
  ├─ 需要兼容不匹配的接口？ ──→ Adapter
  ├─ 需要在运行时动态添加职责？ ──→ Decorator
  ├─ 需要为复杂子系统提供简化接口？ ──→ Facade
  └─ 需要延迟实例化或控制访问？ ──→ Proxy

需要在对象间建立关系或行为协作？
  ├─ 同一操作有多种算法？ ──→ Strategy
  ├─ 一个对象变化需要通知多个对象？ ──→ Observer
  ├─ 需要将请求封装为对象？ ──→ Command
  ├─ 对象行为根据内部状态变化？ ──→ State
  ├─ 需要定义对象间的公共通信？ ──→ Mediator
  ├─ 请求需要由多个处理器依次处理？ ──→ Chain of Responsibility
  ├─ 算法骨架固定但某些步骤可变？ ──→ Template Method
  └─ 需要遍历聚合对象内部？ ──→ Iterator
```

---

## 创建型模式

### Factory Method（工厂方法）

**适用场景**：创建对象的逻辑复杂或需要根据运行时条件决定创建哪个类型。

**TypeScript**：
```typescript
interface Notifier {
  notify(message: string): void;
}

class EmailNotifier implements Notifier {
  notify(message: string) { /* 发送邮件 */ }
}

class SMSNotifier implements Notifier {
  notify(message: string) { /* 发送短信 */ }
}

class NotifierFactory {
  static create(type: 'email' | 'sms'): Notifier {
    switch (type) {
      case 'email': return new EmailNotifier();
      case 'sms': return new SMSNotifier();
    }
  }
}
```

**Python**：
```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def notify(self, message: str) -> None: ...

class EmailNotifier(Notifier):
    def notify(self, message: str) -> None: ...

class SMSNotifier(Notifier):
    def notify(self, message: str) -> None: ...

class NotifierFactory:
    @staticmethod
    def create(type: str) -> Notifier:
        return {'email': EmailNotifier(), 'sms': SMSNotifier()}[type]
```

### Builder（建造者）

**适用场景**：对象构建步骤复杂，有多个可选参数。

**TypeScript**：
```typescript
class QueryBuilder {
  private table: string;
  private conditions: string[] = [];
  private orderBy?: string;
  private limit?: number;

  from(table: string): this { this.table = table; return this; }
  where(condition: string): this { this.conditions.push(condition); return this; }
  orderBy(field: string): this { this.orderBy = field; return this; }
  limit(count: number): this { this.limit = count; return this; }

  build(): string {
    let sql = `SELECT * FROM ${this.table}`;
    if (this.conditions.length) sql += ` WHERE ${this.conditions.join(' AND ')}`;
    if (this.orderBy) sql += ` ORDER BY ${this.orderBy}`;
    if (this.limit) sql += ` LIMIT ${this.limit}`;
    return sql;
  }
}

const query = new QueryBuilder()
  .from('users')
  .where('status = "active"')
  .orderBy('created_at DESC')
  .limit(10)
  .build();
```

---

## 结构型模式

### Strategy（策略模式）

**适用场景**：同一操作有多种实现方式，需要运行时切换。

**TypeScript**：
```typescript
interface PricingStrategy {
  calculate(basePrice: number): number;
}

class RegularPricing implements PricingStrategy {
  calculate(base: number) { return base; }
}

class DiscountPricing implements PricingStrategy {
  constructor(private discount: number) {}
  calculate(base: number) { return base * (1 - this.discount); }
}

class Order {
  private strategy: PricingStrategy;
  constructor(strategy: PricingStrategy) { this.strategy = strategy; }
  setStrategy(s: PricingStrategy) { this.strategy = s; }
  getPrice(base: number) { return this.strategy.calculate(base); }
}
```

### Observer（观察者模式）

**适用场景**：一个对象状态变化需要通知多个依赖方。

**TypeScript**：
```typescript
type Listener<T> = (data: T) => void;

class EventEmitter<T> {
  private listeners: Listener<T>[] = [];
  on(listener: Listener<T>) { this.listeners.push(listener); }
  emit(data: T) { this.listeners.forEach(l => l(data)); }
}

// 使用
const orderEvents = new EventEmitter<OrderEvent>();
orderEvents.on((event) => emailService.sendConfirmation(event));
orderEvents.on((event) => inventoryService.releaseReserved(event));
orderEvents.on((event) => analytics.trackOrder(event));
```

---

## 行为型模式

### Command（命令模式）

**适用场景**：需要将请求封装为对象以支持撤销、排队、日志。

**TypeScript**：
```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
}

class CreateOrderCommand implements Command {
  constructor(private data: OrderData, private service: OrderService) {}
  private createdOrder?: Order;

  async execute() { this.createdOrder = await this.service.create(this.data); }
  async undo() { if (this.createdOrder) await this.service.cancel(this.createdOrder.id); }
}

// 命令队列 + 撤销支持
const commands: Command[] = [
  new CreateOrderCommand(data1, orderService),
  new CreateOrderCommand(data2, orderService),
];

for (const cmd of commands) await cmd.execute();
// 需要撤销时
for (const cmd of commands.reverse()) await cmd.undo();
```

### State（状态模式）

**适用场景**：对象行为根据内部状态变化。

**TypeScript**：
```typescript
interface OrderState {
  pay(order: Order): void;
  ship(order: Order): void;
  cancel(order: Order): void;
}

class PendingState implements OrderState {
  pay(order: Order) { order.state = new PaidState(); }
  ship() { throw new Error('Cannot ship unpaid order'); }
  cancel(order: Order) { order.state = new CancelledState(); }
}

class PaidState implements OrderState {
  pay() { throw new Error('Already paid'); }
  ship(order: Order) { order.state = new ShippedState(); }
  cancel(order: Order) { order.state = new RefundState(); }
}

class Order {
  state: OrderState = new PendingState();
  pay() { this.state.pay(this); }
  ship() { this.state.ship(this); }
  cancel() { this.state.cancel(this); }
}
```

### Template Method（模板方法）

**适用场景**：算法骨架固定但某些步骤由子类实现。

**Python**：
```python
from abc import ABC, abstractmethod

class ReportGenerator(ABC):
    def generate(self) -> str:
        data = self.fetch_data()
        formatted = self.format_data(data)
        return self.render(formatted)

    @abstractmethod
    def fetch_data(self) -> list[dict]: ...

    @abstractmethod
    def format_data(self, data: list[dict]) -> str: ...

    def render(self, content: str) -> str:
        return f"<html><body>{content}</body></html>"

class PDFReportGenerator(ReportGenerator):
    def fetch_data(self): ...
    def format_data(self, data): ...
```

---

## 反模式识别

### Singleton 滥用

**何时避免**：
- 单例使测试困难（全局状态共享）
- 大多数情况可以用依赖注入替代
- 仅在确实是全局唯一资源时使用（连接池、配置管理器）

**替代方案**：
```typescript
// ❌ 滥用 Singleton
class Logger {
  private static instance: Logger;
  static getInstance() { return this.instance ??= new Logger(); }
}

// ✅ 依赖注入
class Logger { constructor(private writer: LogWriter) {} }
class OrderService { constructor(private logger: Logger) {} }
```

### 模式贫血

**问题**：实体类只有 getter/setter，没有业务行为。

```typescript
// ❌ 贫血模型
class Order {
  items: OrderItem[];
  total: number;
  // 所有业务逻辑都在外部 Service 中
}

// ✅ 富血模型
class Order {
  private items: OrderItem[];
  addItem(product: Product, qty: number): void { ... }
  getTotal(): number { return this.items.reduce(...); }
  cancel(): void { ... }
}
```

### 过度设计（金锤子）

**问题**：简单 CRUD 却使用 5 种设计模式。

**何时不需要设计模式**：
- 简单的 CRUD 操作
- 没有多种实现方式需要切换
- 没有复杂的对象创建逻辑
- 没有多观察者通知需求

**原则**：模式是解决问题的工具，不是目的。先有问题，再选模式。

---

## 模式选择矩阵

| 问题特征 | 推荐模式 | 理由 |
|---------|---------|------|
| 根据类型创建不同对象 | Factory Method | 封装创建逻辑 |
| 多种算法可切换 | Strategy | 运行时切换 |
| 对象变化需通知多方 | Observer | 一对多通知 |
| 需要简化复杂子系统 | Facade | 统一入口 |
| 兼容不匹配接口 | Adapter | 接口转换 |
| 需要支持撤销操作 | Command | 封装请求为对象 |
| 行为随状态变化 | State | 状态机 |
| 多处理器依次处理 | Chain of Responsibility | 责任链传递 |
| 构建步骤复杂 | Builder | 链式构建 |
| 动态添加职责 | Decorator | 运行时装饰 |
