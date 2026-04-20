# TDD实现提示词

用于指导实现子代理遵循TDD（测试驱动开发）的红-绿-重构循环进行编码。

---

## 实现子代理的TDD工作流程

### 前置要求

在开始TDD之前，确保有：

1. ✅ **test-spec.md** - 测试规范
   - 完整的test case列表
   - 每个test的期望行为
   - Mock和Fixture的定义

2. ✅ **fixtures.json** - 测试数据
   - 有效的测试数据
   - 边界值
   - 特殊值

3. ✅ **test-*.template** - 测试框架
   - 所有test case的骨架
   - Mock初始化代码
   - 可直接运行

4. ✅ **design.md 中的测试性设计**（前端/全栈场景）
   - data-testid 选择器定义
   - 可测试交互路径

5. ✅ **spec 中的 BROWSER-TESTABLE**（前端/全栈场景）
   - 浏览器级别的可验证验收标准

---

## 关键指导

### 🔴 RED阶段 - 写失败的测试

#### 任务

将test-*.template中的test框架**转换为可运行的test代码**。

#### 步骤

1. **选择一个test case**
   - 从test-spec.md中选一个test
   - 或按TC-ID顺序处理

2. **在test-*.template中找到对应的框架**
   ```typescript
   // test-*.template.ts中：
   describe('TC-1.1.1: 应该支持username输入', () => {
     test('应该...', () => {
       // Arrange
       // Act
       // Assert
     });
   });
   ```

3. **填充test的具体内容**
   ```typescript
   describe('TC-1.1.1: 应该支持username输入', () => {
     test('表单应该包含username输入框', () => {
       // Arrange
       const form = render(<UserForm />);

       // Act
       // (没有特殊的act，只是验证)

       // Assert
       expect(form.getByTestId('username-input')).toBeInTheDocument();
     });
   });
   ```

4. **运行这个test，确保失败**
   ```bash
   npm test -- TC-1.1.1
   # 预期结果：❌ FAIL
   # 因为<UserForm>还没有username-input
   ```

5. **重复**：对每个test case都填充框架并运行，确保都失败

#### 关键点

- ✅ **一次一个test** - 不要一次写所有test，要逐个来
- ✅ **确保失败** - 如果test意外通过，说明test没有真正测试什么
- ✅ **使用fixtures** - 从fixtures.json中复制测试数据
- ✅ **使用Mock** - 使用test-*.template中定义的Mock
- ✅ **清晰的失败** - test失败时，failure message要清楚表达期望

#### 失败示例

```
● TC-1.1.1: 应该支持username输入

    Unable to find an element with the test id of `username-input`

    Tip: Use screen.logTestingLibrary() to see the current DOM
```

这是好的失败 - 清楚地说明了期望什么。

---

### 🟢 GREEN阶段 - 写最小化实现

#### 任务

写**最少的代码**让该test通过。

#### 原则

- ✅ **最小化** - 只写test需要的
- ✅ **快速** - 不追求完美，目标是通过test
- ✅ **暂时** - 可以hardcode、复制粘贴，之后会重构
- ✅ **完全** - 但是必须让test通过

#### 步骤

1. **理解test的期望**
   ```typescript
   // test期望什么？
   expect(form.getByTestId('username-input')).toBeInTheDocument();
   // → 期望：存在一个data-testid="username-input"的input元素
   ```

2. **实现最小化代码**
   ```typescript
   export function UserForm() {
     return (
       <form>
         <input data-testid="username-input" />
       </form>
     );
   }
   // → 最小化：只添加test需要的input，其他都先不管
   ```

3. **运行test，验证通过**
   ```bash
   npm test -- TC-1.1.1
   # 预期结果：✅ PASS
   ```

4. **如果test还是失败，调整实现**
   - 不是改test，而是调整实现
   - 确认test的期望，让实现满足
   - 重新运行test

5. **重复**：对每个test逐个实现

#### 实现技巧

- 🔷 可以hardcode值（如果test允许）
  ```typescript
  const username = 'john_doe'; // hardcode
  ```

- 🔷 可以复制粘贴代码（重构阶段再提取）
  ```typescript
  function validate1() { return !!input1; }
  function validate2() { return !!input2; }  // 暂时复制，之后提取
  ```

- 🔷 可以跳过错误处理（test没有要求就不写）
  ```typescript
  // 暂时不写try/catch，test通过再补充
  ```

- 🔷 可以跳过注释（之后会补充）

#### 关键点

- ❌ **不要过度工程化** - 不要考虑可扩展性、性能等
- ❌ **不要一次全做** - 写完一个test的实现，再做下一个
- ❌ **不要猜测** - 不要为可能的需求提前实现
- ✅ **只做test需要的** - test需要什么就实现什么

---

### 🔵 REFACTOR阶段 - 改进代码质量

#### 任务

在**所有test都通过的前提下**，改进代码质量。

#### 原则

- ✅ **test先行** - 重构前，所有test都要通过
- ✅ **小步快走** - 每次重构一个小点
- ✅ **不改行为** - 只改结构，不改逻辑
- ✅ **保持通过** - 重构后，test仍然要通过

#### 可做的重构

1. **提取重复代码**
   ```typescript
   // 重构前：
   if (username && username.length > 0 && username.length <= 20) {}
   if (email && email.length > 0 && email.length <= 100) {}

   // 重构后：
   function isValidLength(value, min, max) {
     return value && value.length >= min && value.length <= max;
   }
   if (isValidLength(username, 0, 20)) {}
   if (isValidLength(email, 0, 100)) {}
   ```

2. **改进变量和函数命名**
   ```typescript
   // 重构前：
   const x = data.map(i => i.name);
   function v(d) { return d > 0; }

   // 重构后：
   const usernames = data.map(user => user.name);
   function isPositive(value) { return value > 0; }
   ```

3. **提取工具函数**
   ```typescript
   // 重构前：
   const isValid = username && username.trim().length > 0;

   // 重构后：
   function isValidUsername(username) {
     return username && username.trim().length > 0;
   }
   const isValid = isValidUsername(username);
   ```

4. **添加类型注解**
   ```typescript
   // 重构前：
   function validate(data) { return !!data.username; }

   // 重构后：
   function validate(data: { username: string }): boolean {
     return !!data.username;
   }
   ```

5. **添加注释和文档**
   ```typescript
   // 重构后：
   /**
    * 验证用户名是否有效
    * @param username 用户名字符串，长度1-20字符
    * @returns 如果有效返回true，否则返回false
    */
   function isValidUsername(username: string): boolean {
     return username && username.length > 0 && username.length <= 20;
   }
   ```

#### 不应该做的重构

- ❌ **添加新功能** - 这会违反test的期望，test会失败
- ❌ **改变函数签名** - 除非更新对应的test
- ❌ **改变行为** - 重构应该保持行为不变
- ❌ **性能优化** - 如果不是test要求的

#### 重构步骤

1. **做一个小的重构**
   ```typescript
   // 例如：提取一个函数
   function isValidUsername(username) {
     return !!username;
   }
   ```

2. **立即运行test，确保还是通过**
   ```bash
   npm test
   # 结果：✅ 所有test仍然通过
   ```

3. **如果test失败，回滚重构**
   - 说明重构改变了行为
   - 重新审视重构方式

4. **重复**：继续小步重构

#### 关键点

- ✅ **运行test** - 每次重构后立即运行test
- ✅ **小步** - 一次重构一个点，不要大改
- ✅ **保持通过** - 重构的目的不是改变行为，只是改进结构

---

### ✅ REVIEW阶段 - 质量审查

#### 任务

验证重构后的代码仍然满足质量要求。

#### 检查清单

- [ ] **所有test都通过**
  ```bash
  npm test
  ```

- [ ] **Lint检查通过**
  ```bash
  npm run lint
  ```

- [ ] **TypeScript检查通过**
  ```bash
  npm run type-check
  ```

- [ ] **覆盖率达标**
  ```bash
  npm test -- --coverage
  # 应该 ≥ 80%
  ```

- [ ] **没有hardcoded值**（除了在fixtures中）

- [ ] **没有TODO注释**（应该都实现了）

- [ ] **代码风格一致**

- [ ] **命名清晰准确**

- [ ] **注释完整（复杂逻辑）**

#### 审查重点

1. **功能正确性**
   - 所有test都通过？
   - 是否满足设计规范？
   - 是否满足验收标准？

2. **代码质量**
   - TypeScript strict模式通过？
   - 没有any类型？
   - 没有Lint错误？

3. **可维护性**
   - 命名清晰吗？
   - 逻辑清楚吗？
   - 有必要的注释吗？

4. **性能**
   - 有性能问题吗？
   - 有内存泄漏吗？
   - 有不必要的重渲染吗？（如果是React）

#### 如果发现问题

- 🔴 **严重问题**（如逻辑错误）
  - 回到GREEN阶段修复
  - 添加或修改test以覆盖这个问题
  - 重新运行所有test

- 🟡 **中度问题**（如命名不清晰）
  - 在REFACTOR阶段修复
  - 运行test确保仍然通过

- 🟢 **轻微问题**（如注释不完整）
  - 添加注释或进行轻微调整
  - 确保test仍然通过

---

## 完整的TDD循环示例

### 场景

实现一个验证用户名的函数

### RED阶段

```typescript
describe('isValidUsername', () => {
  test('TC-1.1.1: 应该在username为空时返回false', () => {
    expect(isValidUsername('')).toBe(false);
  });

  test('TC-1.1.2: 应该在username有效时返回true', () => {
    expect(isValidUsername('john_doe')).toBe(true);
  });

  test('TC-1.2.1: 应该在username超过20字符时返回false', () => {
    expect(isValidUsername('a'.repeat(21))).toBe(false);
  });
});

// npm test → ❌ FAIL: isValidUsername is not defined
```

### GREEN阶段

```typescript
function isValidUsername(username) {
  return !!username && username.length <= 20;
}

// npm test
// ✅ TC-1.1.1 PASS
// ✅ TC-1.1.2 PASS
// ✅ TC-1.2.1 PASS
```

### REFACTOR阶段

```typescript
/**
 * 验证用户名是否有效
 * 用户名必须非空，长度1-20字符
 * @param username 用户名
 * @returns 如果有效返回true
 */
function isValidUsername(username: string): boolean {
  if (!username) return false;
  if (username.length > 20) return false;
  return true;
}

// 或更简洁的版本：
function isValidUsername(username: string): boolean {
  return username && username.length > 0 && username.length <= 20;
}

// npm test
// ✅ 所有test仍然通过
```

### REVIEW阶段

```bash
npm test                 # ✅ 所有test通过
npm run lint            # ✅ 通过
npm run type-check      # ✅ 通过
```

---

## 常见问题和解答

### Q: 写test会不会太慢？

A: 初期会慢，但：
- 使用test-*.template加速
- fixtures.json已经定义好Mock
- 习惯后会很快
- 长期看bug修复时间减少，总体更快

### Q: 是否应该一次全写所有test？

A: **不应该**。TDD的核心是：
1. 写一个test
2. 让它失败
3. 写实现使其通过
4. 重构
5. 重复

一次全写所有test会失去"驱动"的效果。

### Q: 如果test很复杂怎么办？

A: 说明：
- 代码设计可能有问题（test难以测试说明接口设计不好）
- 可以改进代码设计以简化test
- 或使用Mock来简化test

### Q: 如何处理async/await？

A: 使用async test：
```typescript
test('应该异步加载数据', async () => {
  const result = await fetchData();
  expect(result).toBeDefined();
});
```

或使用Jest提供的utilities：
```typescript
test('应该处理Promise', () => {
  return fetchData().then(data => {
    expect(data).toBeDefined();
  });
});
```

### Q: 如何Mock API调用？

A: 使用fixtures.json中的Mock定义：
```typescript
beforeEach(() => {
  jest.mock('../api', () => ({
    fetchUsers: jest.fn().mockResolvedValue(fixtures.mocks.API.users.success)
  }));
});
```

或使用MSW（Mock Service Worker）：
```typescript
import { setupServer } from 'msw/node';
import { rest } from 'msw';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json(fixtures.mocks.API.users.success));
  })
);

beforeAll(() => server.listen());
```

---

## TDD最佳实践总结

| 实践 | 说明 |
|------|------|
| 🔴 一个test一个实现 | 写一个失败的test，实现它，再写下一个 |
| 🟢 最小化实现 | 只写test需要的，可以hardcode |
| 🔵 持续重构 | 在test通过的前提下改进代码 |
| ✅ 频繁运行test | 每次改动后立即运行test |
| 📊 保持覆盖率 | target ≥ 80% |
| 📝 清晰的test名称 | test名称应该清楚描述期望行为 |
| 🔧 使用fixtures | 复用test-*.template和fixtures.json |
| 📋 逐个处理test | 不要一次处理所有test |

---

## 进度跟踪

在实现过程中，可以用以下表格追踪进度：

| TC-ID | 描述 | RED | GREEN | REFACTOR | REVIEW | 状态 |
|-------|------|-----|-------|----------|--------|------|
| TC-1.1.1 | 验证非空 | ✅ | ✅ | ✅ | ✅ | ✅ 完成 |
| TC-1.1.2 | 验证长度 | ✅ | ✅ | ✅ | ⏳ | 审查中 |
| TC-1.2.1 | 边界值 | ✅ | ⏳ | - | - | 实现中 |
| TC-1.3.1 | 错误处理 | - | - | - | - | 待开始 |

---

## 🆕 浏览器测试 TDD 循环（前端/全栈 Task）

对于涉及 UI 组件或用户交互的前端 Task，在完成单元测试 TDD 循环后，还需进行浏览器测试 TDD 循环：

### RED-BROWSER 阶段

编写 Playwright 测试，验证 spec 中的 BROWSER-TESTABLE 验收标准：

```typescript
import { test, expect } from '@playwright/test';

test.describe('@e2e 登录流程', () => {
  test('应该能用正确的凭据登录并跳转到Dashboard', async ({ page }) => {
    // 导航到登录页
    await page.goto('/login');

    // 填写表单（使用 design.md 中定义的 data-testid）
    await page.getByTestId('login-username').fill('test@example.com');
    await page.getByTestId('login-password').fill('password123');
    await page.getByTestId('login-submit').click();

    // 验证 BROWSER-TESTABLE 标准
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.getByTestId('welcome-message')).toContainText('欢迎');
    await expect(page.getByTestId('login-form')).not.toBeVisible();
  });
});

// npx playwright test → ❌ FAIL: 元素不存在（UI 还未实现）
```

### GREEN-BROWSER 阶段

实现 UI 组件，使浏览器测试通过：

```typescript
// 在组件中添加对应的 data-testid 和交互逻辑
// 确保 Playwright 测试通过
// npx playwright test → ✅ PASS
```

### 浏览器测试编写要求

1. **使用 data-testid**：所有元素选择器使用 design.md 中定义的 data-testid
2. **覆盖 BROWSER-TESTABLE**：每个 spec 中的 BROWSER-TESTABLE 都要有对应的浏览器测试
3. **使用工具函数**：优先使用 `browser-test-helpers.ts` 中的工具函数
4. **测试类型标签**：使用 `@e2e`、`@visual`、`@component` 标签分类

### 模板参考

- E2E 测试模板：`skills/code-test/templates/frontend-e2e-test.template.ts`
- 视觉回归模板：`skills/code-test/templates/visual-regression.template.ts`
- 组件 UI 模板：`skills/code-test/templates/frontend-component-ui.template.ts`
- 完整指南：`skills/code-test/references/frontend-browser-testing.md`

---

## 🆕 代码完整性检查（在GREEN和REFACTOR阶段）

在编写代码时，**必须确保所有代码都是完整的生产级代码**，不能遗留框架代码或示例代码。

### 禁止的代码模式 ❌

**1. 样式块只有注释**
```vue
<!-- ❌ 错误：样式块只有示例注释 -->
<style lang="less" scoped>
  // 样式规则遵循现有审核项的风格
</style>
```

**2. 函数只有 TODO**
```typescript
// ❌ 错误：函数体只有TODO
function validateForm(data) {
  // TODO: 实现表单验证逻辑
}
```

**3. 事件处理器是空的**
```typescript
// ❌ 错误：事件处理器是空函数
const handleSubmit = () => {};
```

**4. 条件分支只有一个实现**
```typescript
// ❌ 错误：else分支只有注释
if (isValid) {
  return processData(data);
} else {
  // TODO: 处理无效数据
}
```

**5. 对象属性都是空的**
```typescript
// ❌ 错误：配置对象的属性都是空的或占位符
const config = {
  title: 'Form',
  fields: [],  // 应该填充
  onSubmit: () => {},  // 空函数
};
```

### 正确的做法 ✅

**1. 完整的样式实现或删除样式块**
```vue
<!-- ✅ 选项 A：完整的样式 -->
<style lang="less" scoped>
  .form {
    display: flex;
    flex-direction: column;
    gap: 12px;
    padding: 16px;
  }

  .form-input {
    padding: 8px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
</style>

<!-- ✅ 选项 B：或根本不写样式块 -->
```

**2. 完整的函数实现**
```typescript
// ✅ 正确：函数有完整的实现
function validateForm(data: FormData): ValidationResult {
  const errors: Record<string, string> = {};

  if (!data.username || data.username.trim().length === 0) {
    errors.username = '用户名不能为空';
  } else if (data.username.length > 20) {
    errors.username = '用户名最多20个字符';
  }

  if (!data.password || data.password.length < 8) {
    errors.password = '密码至少8个字符';
  }

  return {
    valid: Object.keys(errors).length === 0,
    errors
  };
}
```

**3. 完整的事件处理器**
```typescript
// ✅ 正确：事件处理器有完整的实现
const handleSubmit = async () => {
  const result = validateForm(formData);
  if (!result.valid) {
    setErrors(result.errors);
    return;
  }

  try {
    await api.submitForm(formData);
    showSuccess('提交成功');
    resetForm();
  } catch (error) {
    showError('提交失败，请重试');
  }
};
```

**4. 所有分支都有实现**
```typescript
// ✅ 正确：所有分支都有完整实现
if (isValid) {
  return processData(data);
} else {
  return {
    success: false,
    error: '数据验证失败',
    details: getValidationErrors(data)
  };
}
```

**5. 对象属性都有值**
```typescript
// ✅ 正确：配置对象的所有属性都有实际值
const config = {
  title: 'User Registration Form',
  fields: [
    {
      name: 'username',
      label: '用户名',
      type: 'text',
      required: true,
      validation: /^[a-zA-Z0-9_]+$/
    },
    {
      name: 'password',
      label: '密码',
      type: 'password',
      required: true,
      minLength: 8
    }
  ],
  onSubmit: handleFormSubmit,
};
```

### 检查清单（每个阶段都要检查）

在 **GREEN 阶段**和 **REFACTOR 阶段**中，完成代码后请检查：

- [ ] 所有样式块都有完整实现（或完全删除）
- [ ] 所有函数都有完整实现（不能只有 TODO 或 throw NotImplemented）
- [ ] 所有事件处理器都有实现（不能是空函数）
- [ ] 所有条件分支都有实现（if/else 都不能只有注释）
- [ ] 所有对象属性都有实际值（不能是空值或占位符）
- [ ] 没有被注释掉的代码块
- [ ] 没有 TODO 注释在最终代码中

如果发现任何不完整代码：
1. 要么 **立即完成它**
2. 要么 **完全删除它**

不能留下"半成品"代码给质量审查子代理！

---

## 验证模式

在任何声称"通过"或"完成"之前，必须运行实际验证命令并展示证据。

### 测试验证
```
✅ [运行测试命令] [看到: 34/34 通过] "所有测试通过"
❌ "现在应该通过了" / "看起来正确"
```

### 回归验证（TDD 红-绿）
```
✅ 编写测试 → 运行(失败) → 实现代码 → 运行(通过) → 恢复修复前 → 运行(必须失败) → 恢复修复 → 运行(通过)
❌ "我写了回归测试"（没有红-绿验证）
```

### 浏览器测试验证
```
✅ [npx playwright test] [看到: 8/8 通过] "浏览器 E2E 测试全部通过"
❌ "代码逻辑正确，测试应该没问题"
```

### 构建验证
```
✅ [运行构建] [看到: exit 0] "构建成功"
❌ "Linter 通过了"（Linter ≠ 编译器）
```

### 子代理验证
```
✅ 子代理报告成功 → 检查 git diff → 验证改动 → 报告实际状态
❌ 信任子代理报告
```

### 常见借口反制

| 借口 | 现实 |
|------|------|
| "应该可以了" | 运行验证命令 |
| "我有信心" | 信心 ≠ 证据 |
| "就这一次" | 没有例外 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "子代理说成功" | 必须独立验证 |
| "我累了" | 疲惫不是借口 |
| "部分检查就够了" | 部分证明不了什么 |

---

**开始TDD，写出更好的代码！** 🚀

