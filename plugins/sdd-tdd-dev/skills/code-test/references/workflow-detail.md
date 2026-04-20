# code-test 详细工作流

本文档包含 code-test SKILL.md 中省略的详细工作流步骤、检查清单和最佳实践。

## 完整工作流步骤

### 步骤1: 代码质量静态分析

#### 1.1 运行 Lint 检查

```bash
# ESLint 检查（代码规范）
npm run lint

# 输出示例：
# ✓ 0 errors
# ✓ 0 warnings
```

**检查点**：
- ❌ 0 个 Lint 错误（必须）
- ⚠️ 0 个 Lint 警告（推荐）
- 常见问题：
  - 未使用的变量
  - 多余的 console.log
  - 代码风格不一致
  - 安全问题（如 eval 使用）

**修复步骤**：
```bash
# 1. 查看具体错误
npm run lint -- --format json > lint-report.json

# 2. 自动修复可修复的问题
npm run lint -- --fix

# 3. 手动修复不可自动修复的问题
# 根据 Lint 规则说明调整代码

# 4. 重新运行检查
npm run lint
```

#### 1.2 运行 TypeScript 检查

```bash
# TypeScript strict 模式检查
npm run type-check

# 输出示例：
# ✓ 0 errors
# ✓ 编译完成
```

**检查点**：
- ❌ 0 个 TypeScript 错误（必须）
- ❌ 无 any 类型（strict 模式）
- ❌ 无隐式 any
- 常见问题：
  - 函数参数没有类型
  - 返回值没有类型
  - 使用了 any
  - 类型不匹配

**修复步骤**：
```bash
# 1. 查看具体错误
npm run type-check 2>&1 | head -50

# 2. 根据错误信息添加类型注解
# 示例：
// ❌ 错误
function processData(data) {
  return data.value;
}

// ✅ 正确
function processData(data: DataType): string {
  return data.value;
}

# 3. 重新运行检查
npm run type-check
```

#### 1.3 运行和验证单元测试

```bash
# 运行所有单元测试
npm run test:unit

# 输出示例：
# PASS src/__tests__/UserAuth.test.ts
#   ✓ should login successfully (145ms)
#   ✓ should show error on invalid email (32ms)
#
# Test Suites: 1 passed, 1 total
# Tests: 15 passed, 15 total
# Statements: 87%
```

**覆盖率检查**：
- ✅ 所有测试都通过
- ✅ 语句覆盖率 ≥ 85%
- ✅ 分支覆盖率 ≥ 80%
- ✅ 函数覆盖率 ≥ 85%
- 常见问题：
  - 测试失败
  - 覆盖率不足
  - 不稳定的测试（flaky tests）

**单元测试真实性检查（关键）**：

这一步不仅要检查覆盖率数字，还要验证测试的真实性。虚假的测试会导致代码质量问题被隐藏。

检查清单：
```
□ 测试是否验证了真实的业务逻辑？
  □ 是否有 always-true 的断言（expect(true).toBe(true)）？
  □ 是否有模糊的断言（toBeTruthy、toBeFalsy）？
  □ 每个测试是否都验证了具体的期望值？

□ 测试覆盖是否完整？
  □ 是否覆盖了正常路径（正确输入 → 正确输出）？
  □ 是否覆盖了边界情况（null、undefined、0、空字符串、最大值）？
  □ 是否覆盖了所有 if/else 分支？
  □ 是否覆盖了错误处理（异常抛出、验证失败）？
  □ 是否覆盖了所有业务规则（spec 中的 TEST-VERIFY）？

□ 测试质量是否达标？
  □ 测试命名是否清晰描述预期行为？
  □ 每个测试是否只验证一个行为（单一责任）？
  □ 测试是否相互独立（可任意顺序执行）？
  □ 测试是否确定性（同输入必同输出）？

□ 是否存在虚假测试的迹象？
  □ 修改源代码后，某些测试仍然通过？
  □ 有测试只验证工具函数但忽视业务逻辑？
  □ 跳过了边界和错误场景的测试？
  □ 使用了不合适的 mock（mock了业务逻辑）？
```

如发现虚假测试，需要返回 code-execute 重新编写。

**覆盖率详情**：
```bash
# 查看详细的覆盖率报告
npm run test:unit -- --coverage

# 输出示例：
# -------|---------|---------|---------|---------|
# File   |  Stmts  | Branch  | Funcs   | Lines   |
# -------|---------|---------|---------|---------|
# Total  |   87%   |  82%    |  85%    |  87%    |
# -------|---------|---------|---------|---------|
```

**低覆盖率修复和虚假测试改正**：
```bash
# 1. 找出未覆盖或虚假的代码
npm run test:unit -- --coverage --verbose

# 2. 判断是否是虚假覆盖（测试通过但逻辑不验证）
# 修改源代码逻辑，看测试是否失败
# 如果测试仍然通过，说明是虚假测试

# 3. 为真正未覆盖的代码编写真实测试
# 通常是：
# - 边界条件（null、undefined、0、""等）
# - 错误处理路径（异常、验证失败）
# - 条件分支（所有 if/else）
# - 业务规则（来自 spec 的 TEST-VERIFY）

# 4. 重新运行覆盖率检查
npm run test:unit -- --coverage
```

### 步骤2: 代码审查

#### 2.1 功能完整性检查

**检查清单**：

```
□ 是否实现了 tasks.md 中定义的所有功能？
□ 是否处理了所有的边界情况（空值、0、负数等）？
□ 是否处理了所有的错误情况？
□ Props/参数是否与任务定义一致？
□ 返回值是否与任务定义一致？
□ 是否有遗漏的功能点？
```

**审查方法**：
```
1. 对比 tasks.md 中的验收标准
2. 逐项检查是否都已实现
3. 运行功能测试验证
4. 查看是否有 TODO 注释（不应该有）
```

#### 2.2 代码质量检查

**检查清单**：

```
□ 代码是否易读易维护？
□ 函数/组件是否职责单一？
□ 是否有适当的抽象和代码复用？
□ 是否有重复代码需要提取？
□ 复杂逻辑是否有注释说明？
□ 是否有明显的性能问题？
□ 是否遵循项目的代码风格？
```

**具体审查**：

| 维度 | 检查项 | 示例 |
|------|--------|------|
| **可读性** | 变量名清晰 | ❌ `const a = getData()` / ✅ `const userData = getData()` |
| **职责单一** | 单一功能 | ❌ 一个函数处理登录、验证、存储 / ✅ 三个独立函数 |
| **代码复用** | 避免重复 | ❌ 三处相同的验证逻辑 / ✅ 提取为 `validate()` 函数 |
| **注释** | 解释复杂逻辑 | ❌ 没有注释 / ✅ "使用指数退避重试策略，避免频繁请求" |
| **性能** | 没有不必要的操作 | ❌ 循环中发送 API 请求 / ✅ 批量发送 |

#### 2.3 TypeScript 类型检查

**检查清单**：

```
□ 所有函数参数都有类型注解吗？
□ 所有返回值都有类型注解吗？
□ 是否避免了 any 类型？
□ 是否正确使用了泛型？
□ 接口设计是否合理？
□ Union 类型是否清晰？
```

**常见问题和修复**：

```typescript
// ❌ 问题1：缺少参数类型
function processUser(user) {
  return user.name;
}

// ✅ 修复
function processUser(user: User): string {
  return user.name;
}

// ❌ 问题2：使用 any
function handleData(data: any) {
  return data.value;
}

// ✅ 修复
interface DataType {
  value: string;
}
function handleData(data: DataType): string {
  return data.value;
}

// ❌ 问题3：缺少返回类型
const fetchUser = async (id: string) => {
  const response = await api.get(`/users/${id}`);
  return response.data;
};

// ✅ 修复
const fetchUser = async (id: string): Promise<User> => {
  const response = await api.get(`/users/${id}`);
  return response.data;
};
```

#### 2.4 性能考虑

**检查清单**：

```
□ 组件是否有不必要的重新渲染？
□ 是否正确使用了 React.memo、useMemo、useCallback？
□ 是否有 N+1 查询问题？
□ 是否有内存泄漏的风险？
□ 列表渲染是否有 key？
□ 是否有懒加载需求？
```

**常见问题和修复**：

```typescript
// ❌ 问题：每次渲染都创建新对象
function UserList({ users }: Props) {
  const handleClick = (id: string) => {
    console.log(id);
  };
  return <ListItem onClick={handleClick} />;
}

// ✅ 修复：使用 useCallback 缓存函数
function UserList({ users }: Props) {
  const handleClick = useCallback((id: string) => {
    console.log(id);
  }, []);
  return <ListItem onClick={handleClick} />;
}

// ❌ 问题：每次渲染都计算复杂值
function Dashboard({ data }: Props) {
  const expensiveValue = computeExpensive(data);
  return <div>{expensiveValue}</div>;
}

// ✅ 修复：使用 useMemo 缓存计算结果
function Dashboard({ data }: Props) {
  const expensiveValue = useMemo(
    () => computeExpensive(data),
    [data]
  );
  return <div>{expensiveValue}</div>;
}
```

### 步骤3: 集成测试

集成测试验证多个组件/模块的协作。

#### 3.1 设计集成测试

**测试场景**：
```
1. 登录流程：输入 → 表单验证 → API 调用 → token 保存 → 页面重定向
2. 数据加载：API 请求 → 数据转换 → 状态更新 → UI 渲染
3. 表单提交：输入验证 → 数据转换 → API 调用 → 成功/错误提示
```

#### 3.2 运行集成测试

```bash
npm run test:integration

# 输出示例：
# PASS src/__tests__/integration/auth.test.ts
#   Auth Flow
#     ✓ should complete login flow successfully (234ms)
#     ✓ should handle login errors gracefully (156ms)
```

**集成测试检查清单**：
```
□ 所有集成测试都通过
□ 覆盖了主要业务流程
□ 覆盖了错误场景
□ 使用了合适的 Mock（API、localStorage 等）
```

### 步骤4: E2E 测试

E2E 测试在真实浏览器环境中验证用户流程。

#### 4.1 设计 E2E 测试

**E2E 测试框架**：Cypress 或 Playwright

**测试场景**：
```
1. 用户能完整地完成登录吗？
   - 打开登录页
   - 输入邮箱和密码
   - 点击登录按钮
   - 验证重定向到首页
   - 验证显示欢迎信息

2. 用户能正确看到错误消息吗？
   - 输入错误的邮箱格式
   - 提交表单
   - 验证显示邮箱错误提示
```

#### 4.2 运行 E2E 测试

```bash
npm run test:e2e

# 输出示例：
# PASS src/e2e/auth.cy.ts
#   User Authentication
#     ✓ should successfully login (1234ms)
#     ✓ should show validation errors (856ms)
```

**E2E 测试检查清单**：
```
□ 所有 E2E 测试都通过
□ 覆盖了关键的用户流程
□ 没有不稳定的测试（flaky）
□ 运行时间在可接受范围内
```

### 步骤5: 性能测试

性能测试验证应用是否满足性能指标。

#### 5.1 性能指标

**前端性能**：
```
- 首屏加载时间：< 3s
- 交互响应时间：< 100ms
- 内存占用：< 100MB
```

**后端性能**：
```
- API 响应时间：< 200ms
- 数据库查询：< 100ms
- 并发处理：> 1000 RPS
```

#### 5.2 运行性能测试

```bash
# 前端性能测试
npm run test:performance:frontend

# 后端性能测试
npm run test:performance:backend
```

**性能测试检查清单**：
```
□ 首屏加载时间 < 3s
□ API 响应时间 < 200ms
□ 内存占用正常
□ 没有内存泄漏
□ 并发性能满足要求
```

### 步骤6: 闭环验证

生成验证矩阵，确保 TEST-VERIFY → Test → Code 完全对应。

#### 6.1 闭环验证矩阵

```markdown
## 闭环验证矩阵

| 验收标准 | 单元测试 | 集成测试 | E2E 测试 | 性能测试 | 结果 |
|---------|---------|---------|---------|---------|------|
| TV-Auth-1: 登录成功返回 token | ✅ | ✅ | ✅ | - | ✅ |
| TV-Auth-2: 正确保存 token | ✅ | ✅ | ✅ | - | ✅ |
| TV-Auth-3: 重定向到首页 | ❌ | ✅ | ✅ | - | ✅ |
| TV-Auth-4: 邮箱验证 | ✅ | ✅ | ✅ | - | ✅ |
| TV-Perf-1: 登录响应 < 200ms | - | - | ❌ | ✅ | ✅ |

总计：5/5 验收标准通过 ✅
```

#### 6.2 覆盖率分析

```
单元测试覆盖率：87%
集成测试：12 个测试，全部通过
E2E 测试：8 个场景，全部通过
性能测试：所有指标达标

总体：✅ 所有测试通过，可以发布！
```

### 步骤7: 生成测试报告

生成 `testing-report.md`：

```markdown
# 测试报告

**项目**: [项目名]
**日期**: 2026-03-26
**执行人**: Code Test Agent

---

## 测试执行总览

### 静态分析
- ESLint: ✅ 0 errors, 0 warnings
- TypeScript: ✅ 0 errors
- 代码审查: ✅ 通过

### 单元测试
- 测试数: 45
- 通过数: 45
- 通过率: 100%
- 覆盖率: 87%

### 集成测试
- 测试数: 12
- 通过数: 12
- 通过率: 100%

### E2E 测试
- 场景数: 8
- 通过数: 8
- 通过率: 100%

### 性能测试
- 首屏加载: 2.1s (< 3s ✅)
- API 响应: 145ms (< 200ms ✅)
- 内存占用: 85MB (< 100MB ✅)

---

## 闭环验证

✅ 所有 5 个验收标准都有对应的测试
✅ 所有测试都通过
✅ 覆盖率满足要求
✅ 性能指标达标

---

## 总结

代码质量达标，所有测试通过，可以安全发布！🚀
```

---

## 最佳实践

### 1. 测试层级金字塔

```
        E2E 测试（少）
       /          \
     集成测试（中）
    /            \
 单元测试（多）

比例：70% 单元测试，20% 集成测试，10% E2E 测试
```

### 2. 测试覆盖率目标

| 覆盖类型 | 目标 | 含义 |
|---------|------|------|
| **语句覆盖** | ≥ 85% | 代码行都被执行过 |
| **分支覆盖** | ≥ 80% | if/else 的各个分支都被执行 |
| **函数覆盖** | ≥ 85% | 所有函数都被调用过 |
| **行覆盖** | ≥ 85% | 所有代码行都被执行 |

### 3. 测试数据隔离

**问题**：
```typescript
// ❌ 测试互相污染
describe('User', () => {
  let user: User;

  it('should create user', () => {
    user = createUser('john');
  });

  it('should get user', () => {
    // 依赖前一个测试的副作用
    expect(user.name).toBe('john');
  });
});
```

**修复**：
```typescript
// ✅ 每个测试独立
describe('User', () => {
  it('should create user', () => {
    const user = createUser('john');
    expect(user.name).toBe('john');
  });

  it('should get user', () => {
    const user = createUser('john');
    expect(user.name).toBe('john');
  });
});
```

### 4. Mock 策略

**原则**：
- Mock 外部依赖（API、数据库、文件系统）
- 不 Mock 业务逻辑
- 不 Mock 代码库内的模块

```typescript
// ✅ 正确的 Mock
jest.mock('axios');
const mockAxios = axios as jest.Mocked<typeof axios>;

it('should fetch user', async () => {
  mockAxios.get.mockResolvedValue({
    data: { id: '1', name: 'John' }
  });

  const user = await fetchUser('1');
  expect(user.name).toBe('John');
});
```

### 5. 不稳定测试（Flaky Tests）排查

**症状**：
- 有时通过，有时失败
- 与执行顺序相关

**常见原因**：
- 异步操作没有等待
- 时间相关的逻辑
- 全局状态污染
- 随机生成的数据

**排查方法**：
```bash
# 运行多次同一个测试
for i in {1..10}; do npm run test -- --testNamePattern="should fetch user"; done

# 运行测试多次，查看是否总是通过
npm run test -- --maxWorkers=1  # 单线程运行
```

---

**关键理念**：全面的代码质量检查和多层级测试，确保代码在生产环境的稳定性和性能。
