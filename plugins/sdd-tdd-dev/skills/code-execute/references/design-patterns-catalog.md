# 常用设计模式目录

> code-designer 阶段设计决策和 code-execute 阶段实现参考

---

## 创建型模式

### Factory Method（工厂方法）

**适用场景**：WHEN 需要根据条件创建不同类型的对象，THEN 使用工厂方法封装创建逻辑。

**示例**：
```typescript
interface PaymentProcessor {
  process(amount: number): Promise<PaymentResult>;
}

class AlipayProcessor implements PaymentProcessor { /* ... */ }
class WechatPayProcessor implements PaymentProcessor { /* ... */ }

class PaymentProcessorFactory {
  static create(type: PaymentType): PaymentProcessor {
    switch (type) {
      case 'alipay': return new AlipayProcessor();
      case 'wechat': return new WechatPayProcessor();
      default: throw new Error(`Unknown payment type: ${type}`);
    }
  }
}

// 使用
const processor = PaymentProcessorFactory.create('alipay');
await processor.process(100);
```

### Builder（建造者）

**适用场景**：WHEN 对象构建步骤复杂、有多个可选参数，THEN 使用 Builder 模式逐步构建。

**示例**：
```typescript
class OrderBuilder {
  private items: OrderItem[] = [];
  private address?: string;
  private coupon?: string;

  addItem(item: OrderItem): this { this.items.push(item); return this; }
  setAddress(addr: string): this { this.address = addr; return this; }
  useCoupon(code: string): this { this.coupon = code; return this; }

  build(): Order {
    if (!this.address) throw new Error('Address is required');
    return new Order(this.items, this.address, this.coupon);
  }
}

// 使用
const order = new OrderBuilder()
  .addItem({ productId: 1, quantity: 2, price: 99.99 })
  .setAddress('上海市浦东新区...')
  .useCoupon('SAVE10')
  .build();
```

### Singleton（单例，慎用）

**适用场景**：WHEN 整个系统只需要一个实例（如配置管理器、连接池），THEN 使用单例。

**⚠️ 注意事项**：
- 单例使测试困难（状态全局共享）
- 大多数情况可以用依赖注入替代
- 仅在确实是全局唯一资源时使用

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connection: any;

  private constructor() { /* 私有构造函数 */ }

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
}
```

---

## 结构型模式

### Strategy（策略模式）

**适用场景**：WHEN 同一操作有多种实现方式，需要在运行时切换，THEN 使用策略模式。

**示例**：
```typescript
interface DiscountStrategy {
  calculate(originalPrice: number): number;
}

class PercentageDiscount implements DiscountStrategy {
  constructor(private percentage: number) {}
  calculate(originalPrice: number): number {
    return originalPrice * (1 - this.percentage);
  }
}

class FixedAmountDiscount implements DiscountStrategy {
  constructor(private amount: number) {}
  calculate(originalPrice: number): number {
    return Math.max(0, originalPrice - this.amount);
  }
}

class PriceCalculator {
  calculate(price: number, strategy: DiscountStrategy): number {
    return strategy.calculate(price);
  }
}

// 运行时切换策略
const calc = new PriceCalculator();
calc.calculate(100, new PercentageDiscount(0.2));  // 80
calc.calculate(100, new FixedAmountDiscount(30));   // 70
```

### Adapter（适配器模式）

**适用场景**：WHEN 两个接口不兼容但需要协作，THEN 使用适配器转换接口。

**示例**：
```typescript
// 旧版 API
interface LegacyPaymentAPI {
  makePayment(amount: number, currency: string): boolean;
}

// 新版接口
interface NewPaymentAPI {
  processPayment(request: PaymentRequest): PaymentResponse;
}

// 适配器
class LegacyPaymentAdapter implements NewPaymentAPI {
  constructor(private legacy: LegacyPaymentAPI) {}

  processPayment(request: PaymentRequest): PaymentResponse {
    const success = this.legacy.makePayment(request.amount, request.currency);
    return {
      success,
      transactionId: success ? generateId() : null,
    };
  }
}
```

### Facade（外观模式）

**适用场景**：WHEN 子系统有多个复杂步骤需要简化对外接口，THEN 使用 Facade 提供统一入口。

**示例**：
```typescript
// 复杂的子系统
class InventoryService { /* ... */ }
class PaymentService { /* ... */ }
class NotificationService { /* ... */ }
class AuditService { /* ... */ }

// Facade 提供简化接口
class OrderFacade {
  constructor(
    private inventory: InventoryService,
    private payment: PaymentService,
    private notification: NotificationService,
    private audit: AuditService,
  ) {}

  async placeOrder(request: PlaceOrderRequest): Promise<OrderResult> {
    this.inventory.reserve(request.items);
    const payment = await this.payment.charge(request.payment);
    const order = this.createOrder(request, payment);
    this.notification.sendOrderConfirmation(order);
    this.audit.logOrder(order);
    return order;
  }
}
```

### Observer（观察者模式）

**适用场景**：WHEN 一个对象状态变化需要通知多个观察者，THEN 使用观察者模式。

**示例**：
```typescript
type EventHandler<T> = (data: T) => void;

class EventEmitter<T> {
  private handlers: EventHandler<T>[] = [];

  on(handler: EventHandler<T>): void {
    this.handlers.push(handler);
  }

  emit(data: T): void {
    this.handlers.forEach(h => h(data));
  }
}

// 使用：订单状态变化时通知多个观察者
const orderEvents = new EventEmitter<OrderEvent>();

orderEvents.on((event) => emailService.sendOrderUpdate(event));
orderEvents.on((event) => smsService.sendOrderUpdate(event));
orderEvents.on((event) => analytics.trackOrderEvent(event));

// 触发
orderEvents.emit({ orderId: '123', status: 'SHIPPED' });
```

### Decorator（装饰器模式）

**适用场景**：WHEN 需要动态给对象添加职责而不改变其接口，THEN 使用装饰器。

**示例**：
```typescript
interface DataSource {
  read(): string;
  write(data: string): void;
}

class FileDataSource implements DataSource { /* ... */ }

class EncryptionDecorator implements DataSource {
  constructor(private source: DataSource) {}

  read(): string {
    return this.decrypt(this.source.read());
  }

  write(data: string): void {
    this.source.write(this.encrypt(data));
  }
}

// 使用：在原始数据源上叠加加密
const source = new FileDataSource('/data.txt');
const encryptedSource = new EncryptionDecorator(source);
encryptedSource.write('secret data');  // 自动加密后写入
```

---

## 行为型模式

### Template Method（模板方法）

**适用场景**：WHEN 算法的骨架固定但某些步骤可变，THEN 使用模板方法定义骨架，子类实现可变步骤。

**示例**：
```typescript
abstract class ReportGenerator {
  // 模板方法：定义算法骨架
  generate(): string {
    const data = this.fetchData();
    const formatted = this.formatData(data);
    const rendered = this.render(formatted);
    return rendered;
  }

  // 抽象方法：子类实现
  protected abstract fetchData(): Record[];
  protected abstract formatData(data: Record[]): string;

  // 具体方法：共享实现
  private render(content: string): string {
    return `<html><body>${content}</body></html>`;
  }
}

class PdfReportGenerator extends ReportGenerator {
  protected fetchData(): Record[] { /* PDF 特定的数据获取 */ }
  protected formatData(data: Record[]): string { /* PDF 特定的格式化 */ }
}
```

### Command（命令模式）

**适用场景**：WHEN 需要将请求封装为对象以支持撤销/排队/日志，THEN 使用命令模式。

**示例**：
```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
}

class CreateOrderCommand implements Command {
  constructor(private orderData: OrderData, private orderService: OrderService) {}

  async execute(): Promise<void> {
    this.createdOrder = await this.orderService.create(this.orderData);
  }

  async undo(): Promise<void> {
    if (this.createdOrder) await this.orderService.cancel(this.createdOrder.id);
  }
}

// 使用：命令队列
const commandQueue: Command[] = [
  new CreateOrderCommand(data1, orderService),
  new CreateOrderCommand(data2, orderService),
];

for (const cmd of commandQueue) {
  await cmd.execute();
}
```

### Chain of Responsibility（责任链）

**适用场景**：WHEN 请求需要经过多个处理器，每个处理器决定是否处理或传递给下一个，THEN 使用责任链。

**示例**：
```typescript
interface Middleware {
  handle(request: Request, next: () => Response): Response;
}

class AuthMiddleware implements Middleware {
  handle(request: Request, next: () => Response): Response {
    if (!request.token) return { status: 401, body: 'Unauthorized' };
    return next();
  }
}

class ValidationMiddleware implements Middleware {
  handle(request: Request, next: () => Response): Response {
    const errors = validate(request.body);
    if (errors.length > 0) return { status: 400, body: errors };
    return next();
  }
}

class RateLimitMiddleware implements Middleware {
  handle(request: Request, next: () => Response): Response {
    if (rateLimiter.isExceeded(request.ip)) return { status: 429 };
    return next();
  }
}

// 使用：责任链按顺序执行
function buildChain(middlewares: Middleware[]): (req: Request) => Response {
  return (req: Request) => {
    let index = 0;
    const next = () => {
      if (index < middlewares.length) return middlewares[index++].handle(req, next);
      return finalHandler(req);
    };
    return next();
  };
}
```

### State（状态模式）

**适用场景**：WHEN 对象的行为根据其内部状态而变化，THEN 使用状态模式将状态相关行为提取为独立类。

**示例**：
```typescript
interface OrderState {
  next(order: Order): void;
  cancel(order: Order): void;
}

class PendingState implements OrderState {
  next(order: Order): void { order.state = new PaidState(); }
  cancel(order: Order): void { order.state = new CancelledState(); }
}

class PaidState implements OrderState {
  next(order: Order): void { order.state = new ShippedState(); }
  cancel(order: Order): void { order.state = new RefundState(); }
}

class Order {
  state: OrderState = new PendingState();

  next(): void { this.state.next(this); }
  cancel(): void { this.state.cancel(this); }
}
```

---

## 模式选择速查

| 问题 | 推荐模式 |
|------|---------|
| 需要根据条件创建不同类型对象？ | Factory Method |
| 对象构建步骤复杂、有多个可选参数？ | Builder |
| 同一操作有多种实现需要运行时切换？ | Strategy |
| 两个不兼容的接口需要协作？ | Adapter |
| 复杂子系统需要简化对外接口？ | Facade |
| 对象状态变化需要通知多个观察者？ | Observer |
| 需要动态给对象添加职责？ | Decorator |
| 算法骨架固定但某些步骤可变？ | Template Method |
| 请求需要封装为对象（撤销/排队/日志）？ | Command |
| 请求需要经过多个处理器依次处理？ | Chain of Responsibility |
| 对象行为根据内部状态而变化？ | State |
