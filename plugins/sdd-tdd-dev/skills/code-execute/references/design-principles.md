# 软件设计原则综合参考

> code-execute 阶段 REFACTOR 阶段和代码审查时参考

## SOLID 原则

### S - 单一职责原则 (Single Responsibility Principle)

**一句话总结**：一个类/函数/模块只应该有一个变化的理由。

**❌ 违反示例**：
```typescript
// ❌ 一个函数做了太多事情
class UserService {
  async registerUser(data: UserRegistrationDto): Promise<User> {
    // 1. 验证输入
    if (!data.email) throw new Error('Email required');
    if (!data.password || data.password.length < 8) throw new Error('Password too short');
    // 2. 检查重复
    const existing = await db.users.findByEmail(data.email);
    if (existing) throw new Error('Email exists');
    // 3. 加密密码
    const hash = await bcrypt.hash(data.password, 10);
    // 4. 创建用户
    const user = await db.users.create({ ...data, password: hash });
    // 5. 发送欢迎邮件
    await emailService.sendWelcomeEmail(user.email, user.name);
    // 6. 创建默认配置
    await db.configs.create({ userId: user.id, theme: 'light' });
    // 7. 记录审计日志
    await auditLog.log('USER_REGISTERED', { userId: user.id });
    return user;
  }
}
```

**✅ 正确示例**：
```typescript
// ✅ 每个方法只负责一件事
class UserService {
  async registerUser(data: UserRegistrationDto): Promise<User> {
    const validated = this.validateRegistration(data);
    const hash = this.hashPassword(validated.password);
    const user = await this.createUser({ ...validated, password: hash });
    this.sendWelcomeEmail(user);
    this.createDefaultConfig(user.id);
    this.logAudit(user);
    return user;
  }
  private validateRegistration(data: UserRegistrationDto): UserRegistrationDto { ... }
  private hashPassword(password: string): Promise<string> { ... }
  private async createUser(data: CreateUserDto): Promise<User> { ... }
  private sendWelcomeEmail(user: User): Promise<void> { ... }
  private createDefaultConfig(userId: number): Promise<void> { ... }
  private logAudit(user: User): Promise<void> { ... }
}
```

**检查清单**：
- [ ] 函数名是否暗示了多个职责？（如 `validateAndProcessAndSave`）
- [ ] 函数超过 30 行？考虑拆分
- [ ] 函数内的代码可以分为逻辑组？每组拆分为私有方法
- [ ] 类同时处理 UI + 业务逻辑 + 数据库？拆分

---

### O - 开闭原则 (Open-Closed Principle)

**一句话总结**：对扩展开放，对修改关闭。

**❌ 违反示例**：
```typescript
// ❌ 每次新增通知类型都要修改这个函数
function sendNotification(type: string, message: string) {
  if (type === 'email') {
    // 发送邮件逻辑
  } else if (type === 'sms') {
    // 发送短信逻辑
  } else if (type === 'push') {
    // 发送推送逻辑
  } else if (type === 'webhook') {
    // 发送 webhook 逻辑
  }
}
```

**✅ 正确示例（策略模式）**：
```typescript
// ✅ 新增通知类型只需添加新类，不修改现有代码
interface NotificationSender {
  send(message: string): Promise<void>;
}

class EmailSender implements NotificationSender {
  async send(message: string) { /* 发送邮件 */ }
}

class SmsSender implements NotificationSender {
  async send(message: string) { /* 发送短信 */ }
}

// 新增 WebhookSender 只需实现接口，无需修改任何已有代码
class WebhookSender implements NotificationSender {
  async send(message: string) { /* 发送 webhook */ }
}

const senders: Record<string, NotificationSender> = {
  email: new EmailSender(),
  sms: new SmsSender(),
  push: new PushSender(),
  webhook: new WebhookSender(),
};

async function sendNotification(type: string, message: string) {
  const sender = senders[type];
  if (!sender) throw new Error(`Unknown notification type: ${type}`);
  await sender.send(message);
}
```

---

### L - 里氏替换原则 (Liskov Substitution Principle)

**一句话总结**：子类必须能完全替代父类而不改变程序的正确性。

**❌ 违反示例**：
```typescript
// ❌ 子类改变了父类的行为契约
class Bird {
  fly(): void { console.log('flying'); }
}

class Penguin extends Bird {
  fly(): void {
    // 企鹅不会飞！这违反了 LSP
    throw new Error('Penguins cannot fly');
  }
}
```

**✅ 正确示例**：
```typescript
// ✅ 按行为能力分类
class Bird {
  layEggs(): void { console.log('laying eggs'); }
}

interface Flyable {
  fly(): void;
}

class Sparrow extends Bird implements Flyable {
  layEggs(): void { /* ... */ }
  fly(): void { console.log('flying'); }
}

class Penguin extends Bird {
  layEggs(): void { /* ... */ }
  // 不实现 Flyable，企鹅本来就不会飞
}
```

---

### I - 接口隔离原则 (Interface Segregation Principle)

**一句话总结**：客户端不应该被迫依赖它不使用的方法。

**❌ 违反示例**：
```typescript
// ❌ 胖接口：每个实现类都要实现不需要的方法
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Robot implements Worker {
  work(): void { /* 工作 */ }
  eat(): void { /* 机器人不需要吃东西，但必须实现 */ throw new Error('N/A'); }
  sleep(): void { /* 机器人不需要睡觉，但必须实现 */ throw new Error('N/A'); }
}
```

**✅ 正确示例**：
```typescript
// ✅ 拆分接口
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work(): void { /* ... */ }
  eat(): void { /* ... */ }
  sleep(): void { /* ... */ }
}

class Robot implements Workable {
  work(): void { /* ... */ }
  // 只实现需要的接口
}
```

---

### D - 依赖倒置原则 (Dependency Inversion Principle)

**一句话总结**：高层模块不应该依赖低层模块，两者都应该依赖抽象。

**❌ 违反示例**：
```typescript
// ❌ 高层直接依赖具体实现
class OrderService {
  private mysqlRepo = new MySQLUserRepository();  // 直接依赖具体实现

  async processOrder(order: Order): Promise<void> {
    const user = await this.mysqlRepo.findById(order.userId);
    // ...
  }
}
```

**✅ 正确示例**：
```typescript
// ✅ 依赖抽象接口
interface UserRepository {
  findById(id: number): Promise<User>;
}

class OrderService {
  private repo: UserRepository;  // 依赖抽象

  constructor(repo: UserRepository) {  // 依赖注入
    this.repo = repo;
  }

  async processOrder(order: Order): Promise<void> {
    const user = await this.repo.findById(order.userId);
    // ...
  }
}

// 具体实现在组合根注入
const service = new OrderService(new MySQLUserRepository());
```

---

## DRY 原则 (Don't Repeat Yourself)

**一句话总结**：系统中每一部分都必须有一个单一的、明确的、权威的代表。

**❌ 违反示例**：
```typescript
// ❌ 相同的验证逻辑复制多处
function createUser(data: CreateUserDto) {
  if (!data.email || !data.email.includes('@')) throw new Error('Invalid email');
  if (!data.phone || !/^\d{11}$/.test(data.phone)) throw new Error('Invalid phone');
}

function updateUser(id: number, data: UpdateUserDto) {
  if (!data.email || !data.email.includes('@')) throw new Error('Invalid email');
  if (!data.phone || !/^\d{11}$/.test(data.phone)) throw new Error('Invalid phone');
}
```

**✅ 正确示例**：
```typescript
// ✅ 提取公共验证逻辑
class UserValidator {
  static validateEmail(email: string): void {
    if (!email || !email.includes('@')) throw new Error('Invalid email');
  }
  static validatePhone(phone: string): void {
    if (!phone || !/^\d{11}$/.test(phone)) throw new Error('Invalid phone');
  }
}

function createUser(data: CreateUserDto) {
  UserValidator.validateEmail(data.email);
  UserValidator.validatePhone(data.phone);
}

function updateUser(id: number, data: UpdateUserDto) {
  if (data.email) UserValidator.validateEmail(data.email);
  if (data.phone) UserValidator.validatePhone(data.phone);
}
```

**检查清单**：
- [ ] 相同或相似的代码出现超过 2 次？提取为公共函数
- [ ] 相同的 SQL 查询在多处？提取为 Repository 方法
- [ ] 相同的样式代码块？提取为 mixin/class
- [ ] 相同的配置值？提取为常量/环境变量

---

## KISS 原则 (Keep It Simple, Stupid)

**一句话总结**：大多数系统如果保持简单就会工作得最好。

**❌ 违反示例**：
```typescript
// ❌ 过度设计：一个简单查询用了 5 层抽象
class UserQueryExecutor extends BaseQueryExecutor<UserQuery> {
  private readonly queryBuilderFactory = new QueryBuilderFactory();
  private readonly queryOptimizer = new QueryOptimizer();

  async execute(query: UserQuery): Promise<User[]> {
    const builder = this.queryBuilderFactory.create(query);
    const optimized = this.queryOptimizer.optimize(builder.build());
    const result = await this.executeOptimizedQuery(optimized);
    return this.transformResult(result);
  }
}
```

**✅ 正确示例**：
```typescript
// ✅ 简单直接
async function findUsers(filters: UserFilters): Promise<User[]> {
  const query = db.users.where(filters);
  return query.limit(filters.pageSize).offset(filters.page * filters.pageSize);
}
```

**何时可以复杂**：
- 性能瓶颈确凿存在（有数据支撑）
- 需求明确要求可扩展性
- 团队规模确实需要分层
- 已有明确的、可验证的复杂度需求

---

## YAGNI 原则 (You Aren't Gonna Need It)

**一句话总结**：不要写现在不需要的代码，因为你以后很可能不需要它。

**❌ 违反示例**：
```typescript
// ❌ 提前实现了"以后可能用到"的功能
class UserService {
  // 当前只需要根据 ID 查询，但提前实现了所有查询方式
  async findById(id: number): Promise<User> { /* 需要 */ }
  async findByName(name: string): Promise<User> { /* 不需要 */ }
  async findByEmail(email: string): Promise<User> { /* 不需要 */ }
  async findByPhone(phone: string): Promise<User> { /* 不需要 */ }
  async findByRole(role: string): Promise<User[]> { /* 不需要 */ }
  async findByStatus(status: string): Promise<User[]> { /* 不需要 */ }
  async search(query: string, filters: SearchFilters): Promise<User[]> { /* 不需要 */ }
}
```

**✅ 正确做法**：
```typescript
// ✅ 只实现当前需要的，后续根据需求扩展
class UserService {
  async findById(id: number): Promise<User> { /* 当前唯一需要的方法 */ }
}
```

---

## GRASP 原则（通用职责分配软件模式）

| 模式 | 一句话总结 | 适用场景 |
|------|-----------|---------|
| **信息专家** | 职责分配给拥有完成该职责所需信息的类 | 设计类的方法时 |
| **创建者** | 类 B 创建类 A 的实例：B 聚合 A、B 记录 A、B 使用 A | 对象创建逻辑放在哪里 |
| **控制器** | 第一个接收系统事件并协调处理的非 UI 类 | API 控制器、事件处理器 |
| **低耦合** | 减少类之间的依赖 | 设计模块边界时 |
| **高内聚** | 类内的元素彼此关联、共同完成任务 | 评估类的职责是否合理 |
| **纯虚构** | 引入一个不存在但合理的类来降低耦合 | 多个类之间的中介 |
| **间接性** | 通过中介解耦两个直接依赖的类 | 需要插件化/可替换的实现 |
| **多态** | 基于类型的条件行为通过多态实现 | 有 if-else 判断类型时 |
| **保护变化** | 将不稳定的点封装在稳定接口后 | 外部 API 变化、数据库变化 |

---

## 组合优于继承

**原则**：优先使用组合（has-a）而非继承（is-a）。

**❌ 违反示例**：
```typescript
// ❌ 深层继承链导致脆弱设计
class Entity -> Model -> BaseModel -> AuditableModel -> SoftDeletableModel -> UserModel
```

**✅ 正确示例**：
```typescript
// ✅ 组合实现功能复用
class UserModel {
  private auditing = new AuditingMixin();
  private softDelete = new SoftDeleteMixin();
  // 按需组合，不强制继承所有功能
}
```

**检查清单**：
- [ ] 继承链超过 3 层？考虑改为组合
- [ ] 子类只使用了父类的少量方法？考虑组合
- [ ] 子类需要 override 多个父类方法？考虑组合
- [ ] 类的职责是否过于依赖继承层级？考虑接口 + 组合
