# 真实可用的单元测试实践指南

**目标**：编写能真正验证业务逻辑的单元测试，而不是虚假的、看起来通过的测试。

---

## 问题：什么是虚假测试？

虚假测试是指：
- **看起来通过**（绿色），但实际没验证任何东西
- **覆盖率高**，但只是虚假的覆盖
- **修改源代码后**，测试仍然通过（说明没有真正验证逻辑）

### 虚假测试的常见表现

#### 1. Always-true 断言

```typescript
// ❌ 虚假（断言总是真）
it('should process user data', () => {
  const user = createUser('john@example.com');
  expect(true).toBe(true);  // 无意义！无论代码如何都会通过
});

// ✅ 真实（验证具体的期望值）
it('should create user with correct email', () => {
  const user = createUser('john@example.com');
  expect(user.email).toBe('john@example.com');
  expect(user.id).toBeDefined();
});
```

#### 2. 模糊的断言

```typescript
// ❌ 虚假（太模糊，可能会误导）
it('should validate email', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBeTruthy();  // 什么都返回真都会通过
});

// ✅ 真实（明确的期望）
it('should validate valid email', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBe(true);
});

it('should reject invalid email', () => {
  const result = validateEmail('invalid-email');
  expect(result).toBe(false);
});
```

#### 3. 为了让测试通过而修改源代码

```typescript
// ❌ 虚假流程：为了让测试通过，修改了源代码的逻辑
// 原意：getUserById 找不到用户应该返回 null 或抛出错误
// 测试期望：返回 defaultUser
// 解决方案：修改源代码返回 defaultUser（违反原意！）

// 修改后的源代码：
function getUserById(id: string): User {
  return users.find(u => u.id === id) || defaultUser;  // ❌ 违反原意
}

// 测试通过了，但代码行为改变了
it('should return default user for invalid id', () => {
  const user = getUserById('nonexistent');
  expect(user).toBeDefined();  // 虽然通过，但改变了源代码逻辑
});

// ✅ 真实流程：保持源代码逻辑不变，编写真实测试
function getUserById(id: string): User | null {
  return users.find(u => u.id === id) || null;  // ✅ 保持原意
}

it('should return user for valid id', () => {
  const user = getUserById('1');
  expect(user).toEqual({ id: '1', name: 'John' });
});

it('should return null for invalid id', () => {
  const user = getUserById('nonexistent');
  expect(user).toBeNull();  // 验证了真实逻辑
});
```

#### 4. 跳过边界和错误场景

```typescript
// ❌ 虚假（只测试正常路径）
it('should validate user', () => {
  const valid = validateUser({ email: 'test@example.com', age: 25 });
  expect(valid).toBe(true);
});

// ✅ 真实（完整的测试套件）
describe('validateUser', () => {
  it('should return true for valid user', () => {
    const valid = validateUser({ email: 'test@example.com', age: 25 });
    expect(valid).toBe(true);
  });

  it('should return false for invalid email', () => {
    const valid = validateUser({ email: 'invalid', age: 25 });
    expect(valid).toBe(false);
  });

  it('should return false for age < 18', () => {
    const valid = validateUser({ email: 'test@example.com', age: 17 });
    expect(valid).toBe(false);
  });

  it('should throw error for missing email', () => {
    expect(() => {
      validateUser({ age: 25 });
    }).toThrow();
  });

  it('should return true for minimum age 18', () => {
    const valid = validateUser({ email: 'test@example.com', age: 18 });
    expect(valid).toBe(true);
  });
});
```

---

## 如何编写真实可用的测试

### 1. 遵循 TDD 流程（RED-GREEN-REFACTOR）

```typescript
// STEP 1: RED - 先写失败的测试
describe('calculateDiscount', () => {
  it('should apply 10% discount for purchase over 100', () => {
    const discount = calculateDiscount(150);
    expect(discount).toBe(15);  // 这个测试会失败，因为函数还不存在
  });
});

// STEP 2: GREEN - 写最少的代码使测试通过
function calculateDiscount(price: number): number {
  if (price > 100) {
    return price * 0.1;
  }
  return 0;
}

// STEP 3: REFACTOR - 在测试通过的情况下改进代码
// 现在添加更多测试来驱动代码改进
describe('calculateDiscount', () => {
  it('should apply 10% discount for purchase over 100', () => {
    expect(calculateDiscount(150)).toBe(15);
  });

  it('should apply 15% discount for purchase over 500', () => {
    expect(calculateDiscount(600)).toBe(90);
  });

  it('should apply no discount for purchase under 100', () => {
    expect(calculateDiscount(50)).toBe(0);
  });

  it('should apply no discount for exactly 100', () => {
    expect(calculateDiscount(100)).toBe(0);
  });

  it('should handle negative price', () => {
    expect(() => calculateDiscount(-50)).toThrow('Price cannot be negative');
  });
});

// 根据新测试重构函数
function calculateDiscount(price: number): number {
  if (price < 0) throw new Error('Price cannot be negative');
  if (price > 500) return price * 0.15;
  if (price > 100) return price * 0.1;
  return 0;
}
```

### 2. 编写完整的测试覆盖

**测试应该包括**：

#### a. 正常路径（Happy Path）

```typescript
describe('createUser', () => {
  it('should create user with valid data', () => {
    const user = createUser('john@example.com', 'John Doe', 25);
    expect(user).toEqual({
      id: expect.any(String),
      email: 'john@example.com',
      name: 'John Doe',
      age: 25,
      createdAt: expect.any(Date),
    });
  });
});
```

#### b. 边界情况（Boundary Cases）

```typescript
describe('createUser - boundary cases', () => {
  it('should handle minimum age (18)', () => {
    const user = createUser('test@example.com', 'Test', 18);
    expect(user.age).toBe(18);
  });

  it('should handle maximum age', () => {
    const user = createUser('test@example.com', 'Test', 150);
    expect(user.age).toBe(150);
  });

  it('should trim whitespace from email', () => {
    const user = createUser('  test@example.com  ', 'Test', 25);
    expect(user.email).toBe('test@example.com');
  });

  it('should handle empty name by using default', () => {
    const user = createUser('test@example.com', '', 25);
    expect(user.name).toBe('Anonymous');
  });

  it('should handle very long name', () => {
    const longName = 'A'.repeat(1000);
    const user = createUser('test@example.com', longName, 25);
    expect(user.name).toBe(longName);
  });
});
```

#### c. 错误处理（Error Cases）

```typescript
describe('createUser - error handling', () => {
  it('should throw for invalid email', () => {
    expect(() => {
      createUser('invalid-email', 'John', 25);
    }).toThrow(new ValidationError('Invalid email format'));
  });

  it('should throw for age < 18', () => {
    expect(() => {
      createUser('test@example.com', 'John', 17);
    }).toThrow(new ValidationError('Age must be at least 18'));
  });

  it('should throw for duplicate email', () => {
    createUser('taken@example.com', 'John', 25);
    expect(() => {
      createUser('taken@example.com', 'Jane', 30);
    }).toThrow(new DuplicateError('Email already exists'));
  });

  it('should throw for null or undefined input', () => {
    expect(() => createUser(null, 'John', 25)).toThrow();
    expect(() => createUser('test@example.com', undefined, 25)).toThrow();
  });
});
```

#### d. 业务规则（Business Rules）

根据 spec 中的验收标准（TEST-VERIFY）编写测试：

```typescript
// 来自 spec：
// TV-User-1: 用户创建时应自动分配唯一ID
// TV-User-2: 用户邮箱必须唯一
// TV-User-3: 创建时间应记录为当前时间

describe('createUser - business rules', () => {
  it('should assign unique ID to each user (TV-User-1)', () => {
    const user1 = createUser('user1@example.com', 'User1', 25);
    const user2 = createUser('user2@example.com', 'User2', 30);
    expect(user1.id).not.toEqual(user2.id);
  });

  it('should enforce email uniqueness (TV-User-2)', () => {
    createUser('unique@example.com', 'User1', 25);
    expect(() => {
      createUser('unique@example.com', 'User2', 30);
    }).toThrow('Email already exists');
  });

  it('should record current creation time (TV-User-3)', () => {
    const beforeCreation = new Date();
    const user = createUser('test@example.com', 'Test', 25);
    const afterCreation = new Date();
    expect(user.createdAt.getTime()).toBeGreaterThanOrEqual(beforeCreation.getTime());
    expect(user.createdAt.getTime()).toBeLessThanOrEqual(afterCreation.getTime());
  });
});
```

### 3. 测试质量清单

#### 命名清晰

```typescript
// ❌ 不清晰
it('should work', () => { ... });
it('test user', () => { ... });

// ✅ 清晰
it('should create user with valid email and return user object', () => { ... });
it('should throw ValidationError when email format is invalid', () => { ... });
```

#### 单一责任

```typescript
// ❌ 一个测试验证多个行为
it('should create user', () => {
  const user = createUser('john@example.com', 'John', 25);
  expect(user.id).toBeDefined();
  expect(user.email).toBe('john@example.com');
  expect(user.name).toBe('John');
  expect(user.age).toBe(25);
  expect(user.createdAt).toBeDefined();
  // ... 10 个断言
});

// ✅ 一个测试一个断言
it('should assign unique ID when creating user', () => {
  const user = createUser('john@example.com', 'John', 25);
  expect(user.id).toBeDefined();
});

it('should preserve email when creating user', () => {
  const user = createUser('john@example.com', 'John', 25);
  expect(user.email).toBe('john@example.com');
});

// 如果多个相关断言，保持在一个 describe 块中：
describe('createUser', () => {
  it('should create user with correct properties', () => {
    const user = createUser('john@example.com', 'John', 25);
    expect(user).toEqual(expect.objectContaining({
      email: 'john@example.com',
      name: 'John',
      age: 25,
    }));
  });
});
```

#### 隔离独立

```typescript
// ❌ 测试互相依赖
let user: User;

it('should create user', () => {
  user = createUser('john@example.com', 'John', 25);
  expect(user).toBeDefined();
});

it('should retrieve created user', () => {
  // 依赖前一个测试的副作用
  expect(user.id).toBeDefined();
});

// ✅ 每个测试独立
it('should create user', () => {
  const user = createUser('john@example.com', 'John', 25);
  expect(user).toBeDefined();
});

it('should retrieve user by ID', () => {
  // 创建自己的测试数据
  const user = createUser('jane@example.com', 'Jane', 30);
  const retrieved = getUserById(user.id);
  expect(retrieved).toEqual(user);
});
```

#### 确定性（没有时间、随机依赖）

```typescript
// ❌ 不确定性（时间依赖）
it('should record creation time', () => {
  const user = createUser('test@example.com', 'Test', 25);
  expect(user.createdAt).toEqual(new Date());  // 大多数时候失败！
});

// ✅ 确定性
it('should record creation time', () => {
  const beforeCreation = new Date();
  const user = createUser('test@example.com', 'Test', 25);
  const afterCreation = new Date();
  expect(user.createdAt.getTime()).toBeGreaterThanOrEqual(beforeCreation.getTime());
  expect(user.createdAt.getTime()).toBeLessThanOrEqual(afterCreation.getTime());
});

// ❌ 不确定性（随机ID）
it('should have unique ID', () => {
  const user = createUser('test@example.com', 'Test', 25);
  expect(user.id).toBeDefined();  // 总是通过但没验证唯一性
});

// ✅ 确定性
it('should assign unique ID to different users', () => {
  const user1 = createUser('user1@example.com', 'User1', 25);
  const user2 = createUser('user2@example.com', 'User2', 30);
  expect(user1.id).not.toEqual(user2.id);
});
```

### 4. Mock 策略

**只 Mock 外部依赖，不 Mock 业务逻辑**：

```typescript
// ❌ 错误：Mock了业务逻辑
const validateEmailMock = jest.fn().mockReturnValue(true);
it('should validate email', () => {
  const result = validateEmail('invalid');
  expect(result).toBe(true);  // 虚假！测试没有验证真实逻辑
});

// ✅ 正确：Mock外部依赖，测试真实业务逻辑
jest.mock('axios');  // Mock 外部 API
const mockAxios = axios as jest.Mocked<typeof axios>;

it('should fetch user from API', async () => {
  mockAxios.get.mockResolvedValue({
    data: { id: '1', name: 'John', email: 'john@example.com' },
  });

  const user = await fetchUserFromAPI('1');
  expect(user).toEqual({
    id: '1',
    name: 'John',
    email: 'john@example.com',
  });
  expect(mockAxios.get).toHaveBeenCalledWith('/users/1');
});

it('should handle API error', async () => {
  mockAxios.get.mockRejectedValue(new Error('API Error'));

  await expect(fetchUserFromAPI('1')).rejects.toThrow('API Error');
});
```

### 5. 检查测试真实性的方法

**改变源代码，看测试是否失败**：

```typescript
// 原源代码
function validateEmail(email: string): boolean {
  return email.includes('@');
}

// 原测试
it('should validate email with @', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBe(true);
});

// 现在故意改变源代码的逻辑
function validateEmail(email: string): boolean {
  return true;  // 总是返回 true
}

// 如果测试仍然通过，说明是虚假测试！
// ❌ 测试仍然通过（虚假！）
it('should validate email with @', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBe(true);  // 无论源代码如何都会通过
});

// ✅ 真实的测试应该在这里失败
it('should reject email without @', () => {
  const result = validateEmail('testemail.com');
  expect(result).toBe(false);  // 源代码改变后，这个会失败
});
```

---

## 测试覆盖率质量检查清单

```markdown
## 单元测试质量检查清单

### 覆盖率数字
- [ ] 语句覆盖率 ≥ 85%
- [ ] 分支覆盖率 ≥ 80%
- [ ] 函数覆盖率 ≥ 85%

### 测试完整性
- [ ] 正常路径（happy path）已测试
- [ ] 边界情况（null、undefined、0、空字符串）已测试
- [ ] 错误处理（异常、验证失败）已测试
- [ ] 所有 if/else 分支都被执行过
- [ ] spec 中的所有验收标准（TEST-VERIFY）都有测试
- [ ] 所有业务规则都有相应的测试

### 测试质量
- [ ] 测试命名清晰描述预期行为（describe + it）
- [ ] 每个测试只验证一个行为
- [ ] 断言明确具体（不使用 toBeTruthy、toBeFalsy）
- [ ] 测试相互独立（可任意顺序执行）
- [ ] 测试确定性（同输入必同输出）

### 虚假测试检查
- [ ] 没有 always-true 的断言（expect(true).toBe(true)）
- [ ] 没有模糊的断言（toBeTruthy）
- [ ] 修改源代码后，相关测试会失败（验证真实性）
- [ ] 没有仅测试工具函数而忽视业务逻辑的情况
- [ ] 没有为了让测试通过而修改源代码的情况

### Mock 策略
- [ ] 只 Mock 了外部依赖（API、数据库、文件）
- [ ] 没有 Mock 业务逻辑
- [ ] 没有 Mock 内部函数
- [ ] Mock 数据具体且有代表性（来自 spec 示例）

### 现实性验证
- [ ] 降低覆盖率后，相关测试失败吗？（验证覆盖率真实）
- [ ] 删除某个功能后，相关测试失败吗？（验证功能测试真实）
- [ ] 修改业务规则后，相关测试失败吗？（验证规则测试真实）
```

---

## 总结

真实可用的单元测试应该：

1. ✅ **验证业务逻辑** - 测试不是为了让 CI 通过，而是确保功能正确
2. ✅ **覆盖完整** - 正常路径、边界、错误都要有
3. ✅ **质量达标** - 测试代码质量不低于生产代码
4. ✅ **确定可靠** - 同输入必同输出，可重复运行
5. ✅ **可维护** - 清晰的命名、隔离的测试、明确的断言

**关键**：修改源代码时，真实的测试会失败；虚假的测试不会。这是判断测试真实性的最可靠方法。
