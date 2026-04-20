# test-design Skill - 使用指南

---

## 概述

**test-design** 是SDD工作流中的关键Skill，负责将规范中的验收标准转换为可执行的测试规范和测试代码。

### 核心价值

- ✅ **100%覆盖验收标准** - 所有TEST-VERIFY都有对应测试
- ✅ **测试驱动开发** - 为code-execute提供清晰的测试需求
- ✅ **完整的追踪链** - 从规范 → 任务 → 测试的完整映射
- ✅ **可执行的框架** - 生成可直接使用的测试代码

---

## 工作流程

### 1️⃣ 前置条件

在使用test-design之前，需要完成：

- ✅ `/spec-creation` - 已生成规范和TEST-VERIFY
- ✅ `/code-designer` - 已生成设计方案
- ✅ `/code-task` - 已生成Task和test-case-mapping

### 2️⃣ 调用test-design

```bash
/test-design
```

或指定项目：

```bash
/test-design user-registration
```

### 3️⃣ test-designer Agent的工作

1. **分析需求** (5分钟)
   - 读取规范中的TEST-VERIFY
   - 读取任务中的test-case-mapping
   - 理解技术栈和框架

2. **设计测试分层** (10分钟)
   - 确定Unit、Integration、E2E的分布
   - 规划Mock策略
   - 优化测试金字塔

3. **生成测试规范** (15分钟)
   - 为每个Task设计test case
   - 创建test case清单
   - 设计Mock和Fixture

4. **生成框架代码** (15分钟)
   - 生成test-*.template文件
   - 包括所有test的骨架
   - 包括Mock初始化代码

5. **验证覆盖率** (10分钟)
   - 验证TEST-VERIFY 100%覆盖
   - 生成覆盖率报告
   - 生成test-spec.md

### 4️⃣ 输出成果

```
spec-dev/{project}/tests/
├── test-spec.md              # 完整的测试规范
├── fixtures.json             # Mock数据和Fixture定义
├── test-Task1.template.ts    # Task 1的测试框架
├── test-Task2.template.ts    # Task 2的测试框架
└── ...
```

---

## 关键文件说明

### test-spec.md（测试规范文档）

**用途**：
- 记录项目的完整测试计划
- 记录所有test case和映射关系
- 记录Mock和Fixture定义
- 记录覆盖率分析

**内容**：
```markdown
# 测试规范

## 项目信息
对应规范、设计、任务的链接

## 测试框架
使用的框架、版本、配置

## 测试分层
Unit、Integration、E2E的分布

## Test Case总览
完整的test case表格，包括：
- TC-ID
- Task
- TEST-VERIFY
- 实现文件
- 测试类型
- 优先级

## Mock和Fixture
JSON格式的Mock定义

## 覆盖率分析
TEST-VERIFY覆盖矩阵
代码覆盖率目标

## 执行指南
如何运行测试、查看覆盖率等
```

### fixtures.json（测试数据）

**用途**：
- 定义所有Mock数据
- 定义测试Fixture
- 便于复用和维护

**结构**：
```json
{
  "metadata": {...},
  "fixtures": {
    "Task1": {
      "validInputs": [...],
      "boundaryValues": [...],
      "specialValues": [...]
    }
  },
  "mocks": {
    "API": {...},
    "Database": {...}
  }
}
```

### test-*.template.ts（测试框架）

**用途**：
- 提供test的骨架代码
- 可直接复制使用
- 包括所有test case的结构

**特点**：
```typescript
describe('Task X: [name]', () => {
  beforeEach(() => {
    // Mock和setup
  });

  describe('TC-X.Y.Z: [说明]', () => {
    test('应该[期望行为]', () => {
      // Arrange
      // Act
      // Assert
    });
  });

  // 更多test cases...

  afterEach(() => {
    // 清理
  });
});
```

---

## TEST-VERIFY到Test的映射

### 映射过程

```
规范（spec/）
  ↓ TEST-VERIFY
  ├─ 应该[功能A]
  ├─ 应该[功能B]
  ├─ 边界值：X最小值为0
  └─ 特殊值：X为空时

任务（tasks/）
  ↓ test-case-mapping
  ├─ TC-1.1.1 → TEST-VERIFY#1 → src/component.ts
  ├─ TC-1.1.2 → TEST-VERIFY#2 → src/component.ts
  ├─ TC-1.2.1 → 边界值 → src/component.ts
  └─ TC-1.3.1 → 特殊值 → src/component.ts

测试规范（test-spec.md）
  ↓ test case表格
  ├─ TC-1.1.1: 应该[功能A]
  ├─ TC-1.1.2: 应该[功能B]
  ├─ TC-1.2.1: 边界值[X最小值]
  └─ TC-1.3.1: 特殊值[X为空]

框架代码（test-*.template.ts）
  ↓ 实现test
  ├─ describe('TC-1.1.1')
  ├─ describe('TC-1.1.2')
  ├─ describe('TC-1.2.1')
  └─ describe('TC-1.3.1')
```

### 完整例子

**规范中的TEST-VERIFY**：
```markdown
## Case 1: 表单验证

### TEST-VERIFY
- [ ] 应该支持username输入
- [ ] 应该验证username非空
- [ ] 应该验证username长度1-20
- [ ] 应该验证email格式

### Mock Data
**有效值**：username: "john_doe", email: "john@example.com"
**边界值**：username min=1, max=20
**特殊值**：username="", email invalid
```

**任务中的test-case-mapping**：
```markdown
| 验收标准 | 来源 | 实现 | TC-ID |
|---------|------|------|-------|
| 应该支持username | Case1#1 | Form.render() | TC-1.1.1 |
| 应该验证username非空 | Case1#2 | validate() | TC-1.1.2 |
| username边界值min | Mock | validate() | TC-1.2.1 |
```

**测试规范中的test case**：
```typescript
describe('TC-1.1.1: 应该支持username输入', () => {
  test('表单包含username字段', () => {
    const form = render(<UserForm />);
    expect(form.getByTestId('username-input')).toBeInTheDocument();
  });
});

describe('TC-1.1.2: 应该验证username非空', () => {
  test('username为空时应该显示错误', () => {
    const form = new Form({username: ''});
    expect(form.validate()).toBe(false);
    expect(form.getError('username')).toBe('用户名不能为空');
  });
});

describe('TC-1.2.1: 边界值min', () => {
  test('username最小1个字符', () => {
    const form = new Form({username: 'a'});
    expect(form.validate()).toBe(true);
  });
});
```

---

## Mock策略

### 何时使用Mock

✅ **应该Mock**：
- 外部API调用（HTTP请求）
- 数据库操作
- 第三方服务（邮件、支付等）
- 系统功能（时间、随机数等）
- 复杂的依赖

❌ **不应该Mock**：
- 应用内部的业务逻辑
- 用户交互（直接触发事件）
- 验证逻辑
- 简单的工具函数

### Mock方式

#### 1. API Mock（使用msw）

```typescript
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.post('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({ id: '123', username: 'john_doe' })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

#### 2. 函数Mock（使用jest.mock）

```typescript
jest.mock('../api/userApi');
const mockCreateUser = jest.fn().mockResolvedValue({id: '123'});
userApi.createUser = mockCreateUser;

test('应该调用API', () => {
  // ...
  expect(mockCreateUser).toHaveBeenCalledWith({username: 'john'});
});
```

#### 3. 数据库Mock（使用内存数据库）

```typescript
beforeEach(() => {
  // 初始化内存SQLite
  db = new Database(':memory:');
  // 加载seed数据
  db.exec('INSERT INTO users VALUES (1, "admin", "admin@example.com")');
});

afterEach(() => {
  db.close();
});
```

---

## 测试分层指南

### 单元测试（Unit Tests）

**什么时候写**：
- 测试单个函数或组件
- 需要验证输入输出

**如何写**：
```typescript
// 隔离依赖，只测试函数逻辑
describe('validateEmail', () => {
  test('应该接受有效email', () => {
    expect(validateEmail('john@example.com')).toBe(true);
  });

  test('应该拒绝无效email', () => {
    expect(validateEmail('invalid-email')).toBe(false);
  });
});
```

**覆盖范围**：
- 正常输入
- 边界值
- 特殊值
- 错误情况

### 集成测试（Integration Tests）

**什么时候写**：
- 多个模块协作
- API调用和数据库操作
- 状态管理的流程

**如何写**：
```typescript
describe('用户注册流程', () => {
  test('应该成功注册用户', async () => {
    const form = render(<RegisterForm />);

    // 填表单
    fireEvent.change(form.getByLabelText('Username'), {target: {value: 'john'}});
    fireEvent.change(form.getByLabelText('Email'), {target: {value: 'john@example.com'}});

    // 提交
    fireEvent.click(form.getByText('Register'));

    // 验证API调用
    await waitFor(() => {
      expect(mockAPI.createUser).toHaveBeenCalledWith({...});
    });
  });
});
```

**Mock策略**：
- Mock外部API
- 不Mock应用内部逻辑
- 使用真实或内存数据库

### E2E测试（E2E Tests）

**什么时候写**：
- 完整的用户场景
- 关键业务流程

**如何写（使用Cypress）**：
```typescript
describe('用户注册E2E', () => {
  it('应该成功注册并登录', () => {
    cy.visit('/register');
    cy.get('[data-testid="username"]').type('john');
    cy.get('[data-testid="email"]').type('john@example.com');
    cy.get('[data-testid="password"]').type('SecurePass123!');
    cy.get('button:contains("Register")').click();

    cy.location('pathname').should('eq', '/login');
    cy.contains('Registration successful').should('be.visible');
  });
});
```

---

## 常见问题

### Q: Test Case太多怎么办？

A: 可以分组组织，优化测试金字塔：
- 减少重复的单元测试
- 用集成测试覆盖组合场景
- 用E2E测试覆盖关键流程

### Q: 如何处理异步操作？

A: 使用async/await和waitFor：
```typescript
test('应该加载数据', async () => {
  const result = await fetchData();
  await waitFor(() => {
    expect(screen.getByText('Data')).toBeInTheDocument();
  });
});
```

### Q: Mock不work怎么办？

A: 检查：
1. Mock的路径是否正确
2. Mock是否在import前定义
3. Mock的return值是否正确
4. 是否正确restore mock

### Q: 如何测试错误情况？

A: Mock API返回错误，或直接触发错误：
```typescript
test('应该处理错误', async () => {
  mockAPI.reject(new Error('Network error'));

  // 触发代码

  expect(screen.getByText('Network error')).toBeInTheDocument();
});
```

### Q: 测试覆盖率目标是多少？

A:
- Unit：≥ 85%
- Integration：≥ 70%
- 整体：≥ 80%
- TEST-VERIFY：100%

---

## 检查清单

在使用test-design之前，确保：

- [ ] 规范（spec/）中的TEST-VERIFY已完成
- [ ] 任务（tasks/）中的test-case-mapping已完成
- [ ] 设计（design/）中的技术栈已确定

在test-design完成后，应该有：

- [ ] test-spec.md 完成
- [ ] fixtures.json 完成
- [ ] test-*.template 完成
- [ ] TEST-VERIFY覆盖率 = 100%
- [ ] 没有broken links

在code-execute前，检查：

- [ ] 所有test都能理解（注释清晰）
- [ ] 所有fixtures都能使用（格式正确）
- [ ] 所有框架都能复制（代码完整）

---

## 下一步

### 立即使用

1. 在你的项目中调用 `/test-design`
2. 检查输出的test-spec.md
3. 审查fixtures.json和test-*.template
4. 进入code-execute阶段进行TDD开发

### 后续优化

1. 根据实际开发反馈优化test case
2. 补充缺失的测试
3. 优化Mock策略
4. 提高覆盖率

---

**准备好进行TDD开发了吗？** 👉 [/code-execute](../code-execute/)

