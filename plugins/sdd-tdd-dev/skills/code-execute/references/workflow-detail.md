# code-execute 详细工作流

本文档包含 code-execute SKILL.md 中省略的详细工作流步骤和示例。

## 完整工作流步骤

### 步骤1: 分析任务列表

读取 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`，提取：

```
- Task 1: UserAuth (优先级1，无依赖)
- Task 2: FormValidation (优先级1，无依赖)
- Task 3: ApiIntegration (优先级2，依赖 Task 1)
- Task 4: Styling (优先级3，依赖 Task 1, 2)
```

### 步骤2: 制定并行执行计划

根据依赖关系，分批执行：

```
批次1（无依赖）：Task 1, Task 2 → 并行执行
  ↓ (等待都完成)
批次2（依赖Task1）：Task 3 → 执行
  ↓ (等待完成)
批次3（依赖Task1,2）：Task 4 → 执行
```

### 步骤3: 并行执行Task批次

#### 执行第1批Task（无依赖）

为每个Task分配独立的子代理实例并行执行：
- code-executor agent 实例1 执行 Task 1
- code-executor agent 实例2 执行 Task 2

这些Task可以**完全并行执行**，互不影响。

#### 执行单个Task

对于每个分配的Task，按以下流程：

**1. 准备Task上下文**

从tasks.md中提取该Task的完整信息：
- Task名称、目标、交付物
- 验收标准和检查清单
- 依赖关系
- 涉及的文件和技术栈
- 详细的Task说明

**2. 创建隔离工作环境（git-worktree）**

```bash
# 创建worktree
git worktree add .claude/worktrees/T1-UserAuth HEAD
cd .claude/worktrees/T1-UserAuth

# 后续所有编码、测试、审查、修复都在此worktree中进行
```

**3. 分配实现子代理**

传递以下内容给子代理：
- Task的完整详情
- 技术背景和框架信息
- 项目上下文和已有代码
- 关键要求和验收标准

**4. 实现子代理：编码和自审查**

实现子代理进行以下活动：

a. **理解任务**
   - 阅读Task详情
   - 检查是否有未明确的TODO项
   - 如有TODO，提问以澄清（等待回答）

b. **编码实现**
   - 遵循项目的代码规范
   - TypeScript strict模式
   - 清晰的命名和注释
   - 模块化设计

c. **编写单元测试**（遵循TDD流程：RED→GREEN→REFACTOR）
   - 覆盖率要求 ≥ 85%：语句覆盖 ≥85%, 分支覆盖 ≥80%, 函数覆盖 ≥85%
   - 测试完整性：正常路径 + 边界情况 + 错误处理 + 业务规则验证
   - 测试质量：单一责任 + 清晰命名 + 明确断言 + 隔离独立 + 确定性
   - 真实性约束：
     * ✅ 测试真实业务逻辑（Mock外部依赖不Mock业务）
     * ✅ 测试数据来自spec示例（避免虚假数据）
     * ✅ 每个测试验证一个具体预期（不用always-true的断言）
     * ❌ 禁止修改源代码使其适配测试
     * ❌ 禁止虚假测试（看起来通过但没验证任何东西）

d. **自审查**
   - 代码是否满足Task的验收标准
   - 是否符合设计方案（design.md）
   - 是否符合项目规范
   - 是否有可优化的部分

e. **提交和反馈**
   - 将代码commit到worktree
   - 提供自审查报告

**5. 规范审查（子代理）**

规范审查子代理验证：

a. **设计一致性**
   - 代码架构是否符合design.md？
   - 关键组件是否按设计实现？
   - Props/参数是否与设计一致？

b. **命名和结构**
   - 是否遵循项目的命名规范？
   - 文件组织是否符合项目结构？
   - 导出和导入是否合理？

c. **依赖关系**
   - 是否正确引用了其他组件？
   - 是否避免了不必要的依赖？

如果未通过，提供详细的审查意见。

**6. 代码质量审查（子代理）**

代码质量审查子代理检查：

a. **TypeScript类型**
   - 所有函数参数都有类型吗？
   - 所有返回值都有类型吗？
   - 是否避免了any？
   - 是否正确使用了泛型？

b. **代码规范**
   - ESLint规则是否全部通过？
   - 是否有未使用的变量或导入？
   - 代码风格是否一致？

c. **最佳实践**
   - 复杂度是否合理？
   - 是否有重复代码可以提取？
   - 是否有潜在的性能问题？
   - 是否正确处理错误？

e. **完整性检查（🆕 新增）** - 防止不完整的框架代码被遗留
   - ❌ **不完整的样式块**：`<style lang="..." scoped>` 后面只有示例注释或空白？
     * ✅ 正确：要么有完整的样式实现，要么根本不要这个块
   - ❌ **不完整的方法体**：函数签名已定义但实现只有注释？
     * 例如：`function createForm() { // TODO: 实现表单 }`
   - ❌ **不完整的组件模板**：`<template>` 中只有占位符或注释，没有实际内容？
   - ❌ **不完整的导入语句**：导入了但没有使用？
   - ❌ **不完整的if/else分支**：只实现了一个分支，另一个分支只有注释？
   - ❌ **不完整的配置对象**：定义了属性但值都是注释或占位符？
   - ❌ **代码留白问题**：大段被注释掉的代码应该完全删除而不是保留？

   **检查方式**：
   - 逐个文件扫描是否有"框架代码"（有结构但无实现）
   - 特别关注 Vue/React 组件的 `<style>`、`<template>`、`<script>` 三个部分
   - 检查是否有以下模式：
     ```vue
     <!-- 不完整示例 -->
     <style lang="less" scoped>
       // 样式规则遵循现有审核项的风格
     </style>
     ```
     或
     ```typescript
     // 不完整示例
     function validateForm(data) {
       // TODO: 实现表单验证逻辑
     }
     ```

   如果发现不完整代码，**必须全部完成或删除**，不能留半成品

d. **测试质量检查**
   - 单元测试是否充分？覆盖率 ≥ 85%（≥80% 分支、≥85% 函数、≥85% 语句）
   - 测试是否真实可用？
     * 测试名称清晰描述预期行为（describe + it 组合清晰）
     * 每个测试只验证一个行为（单一责任）
     * 断言明确具体（不用 toBeTruthy / toBeFalsy 这种模糊断言）
     * 测试独立隔离（可任意顺序执行，互不依赖）
     * 测试确定性（同输入必同输出，避免时间或随机依赖）
   - 测试是否覆盖了所有关键场景？
     * ✅ 正常路径：正确输入产生正确输出
     * ✅ 边界情况：null、undefined、0、空字符串、空数组、最大值、最小值
     * ✅ 错误处理：异常抛出、验证失败、API错误、业务规则冲突
     * ✅ 所有if/else分支都被执行过（分支覆盖）
     * ✅ spec中提到的所有验收标准都有测试
   - 是否避免虚假测试？
     * ❌ 不要使用 always-true 的断言（如 expect(true).toBe(true)）
     * ❌ 不要仅测试工具函数而忽视业务逻辑
     * ❌ 不要为了让测试通过而修改源代码
     * ❌ 不要mock业务逻辑（只mock外部依赖）

如果未通过，提供详细的审查意见。

**7. 修复循环**

如果某个审查未通过：

1. 主Agent总结所有问题
2. 实现子代理进行修复
3. 修复后的代码再次提交commit
4. 重新进行规范审查或质量审查

重复直到**所有审查都通过**。

### 步骤4: 执行第2批及后续Task

当第1批所有Task完成后，开始第2批Task（按同样流程）。

### 步骤5: 生成执行报告

所有Task完成后，生成报告到 `spec-dev/{requirement_desc_abstract}/execution/execution-report.md`：

```markdown
# 执行报告

## 执行总览
- 总Task数：4
- 成功完成：4
- 修复循环数：3

## Task执行详情

### Task 1: UserAuth
- 状态：✅ 完成
- 修复次数：0
- 代码行数：245
- 测试覆盖率：85%
- 时间：2h

### Task 2: FormValidation
- 状态：✅ 完成
- 修复次数：2
  - 修复1：类型检查失败
  - 修复2：测试覆盖率不足
- 代码行数：312
- 测试覆盖率：82%
- 时间：3h

...

## 质量指标
- 平均覆盖率：83%
- Lint错误：0
- TypeScript错误：0
- 修复循环平均次数：1.25

## 总结
所有Task成功完成，代码质量达标，可进入code-test阶段。
```

---

## 关键概念说明

### Git-worktree的隔离性

每个Task有独立的worktree：
```
repo/
├── .claude/worktrees/T1-UserAuth/          # Task 1的worktree
├── .claude/worktrees/T2-FormValidation/   # Task 2的worktree
├── .claude/worktrees/T3-ApiIntegration/   # Task 3的worktree
└── main branch                              # 主分支保持干净
```

**优势**：
- Task修改互不影响
- 修复失败可直接删除worktree重新开始
- 提交历史清晰记录每个Task的修复过程
- 并行执行时不产生git冲突

### 两阶段审查的目的

- **规范审查** → 确保设计一致性（从design.md的角度）
- **质量审查** → 确保代码质量（从代码规范的角度）

两个审查维度不同，都必须通过，保证设计准确和代码高质量。

### TODO处理原则

Task中如果有TODO项：
1. **不能跳过** - 必须在开始编码前澄清
2. **等待确认** - 等待主Agent或用户的回答
3. **最终移除** - 澄清后，代码中不应再有TODO

这确保最终交付的代码是**完全明确的，没有待定项**。

---

## 单元测试实践指南

### 编写真实可用的单元测试

**问题**：如何区分"真实可用的测试"和"虚假测试"？

虚假测试的表现：
- 看起来通过（绿色），但实际没验证任何东西
- 修改源代码后，测试仍然通过（说明测试没有真正验证逻辑）
- 测试用例太简单（只测试工具函数或always-true的路径）
- 使用模糊的断言（toBeTruthy、toBeFalsy 等）
- 跳过边界和错误场景

### 示例：真实单元测试 vs 虚假测试

#### ❌ 虚假测试（反面例子）

```typescript
// 问题：这个测试什么都没验证
describe('UserService', () => {
  it('should create user', () => {
    const user = createUser('John');
    expect(user).toBeTruthy();  // 模糊的断言！
  });

  it('should validate email', () => {
    const result = validateEmail('test@example.com');
    expect(true).toBe(true);    // always-true，无意义！
  });

  it('should handle users', () => {
    const users = [];
    expect(users).toEqual([]);  // 没有测试真实逻辑
  });
});
```

问题：
1. 断言太模糊，不验证具体值
2. always-true 的断言
3. 没有测试真实业务逻辑（验证email格式、处理错误等）

#### ✅ 真实可用的测试（正面例子）

```typescript
// 来自spec的验收标准：
// - TV-Auth-1: 邮箱验证失败返回特定错误
// - TV-Auth-2: 创建用户需要邮箱唯一性

describe('UserService', () => {
  describe('validateEmail', () => {
    // 正常路径
    it('should return true for valid email format', () => {
      const result = validateEmail('john@example.com');
      expect(result).toBe(true);
    });

    // 边界情况
    it('should return false for email without @', () => {
      const result = validateEmail('johnexample.com');
      expect(result).toBe(false);
    });

    it('should return false for email with multiple @', () => {
      const result = validateEmail('john@@example.com');
      expect(result).toBe(false);
    });

    it('should return false for empty email', () => {
      const result = validateEmail('');
      expect(result).toBe(false);
    });

    it('should return false for null or undefined', () => {
      expect(validateEmail(null)).toBe(false);
      expect(validateEmail(undefined)).toBe(false);
    });
  });

  describe('createUser', () => {
    // 正常路径
    it('should create user with valid data', () => {
      const user = createUser('john@example.com', 'John Doe');
      expect(user).toEqual({
        id: expect.any(String),
        email: 'john@example.com',
        name: 'John Doe',
        createdAt: expect.any(Date),
      });
    });

    // 错误处理：邮箱验证失败（来自spec验收标准 TV-Auth-1）
    it('should throw error for invalid email format', () => {
      expect(() => {
        createUser('invalid-email', 'John');
      }).toThrow(new ValidationError('Invalid email format'));
    });

    // 错误处理：邮箱唯一性（来自spec验收标准 TV-Auth-2）
    it('should throw error when email already exists', () => {
      createUser('john@example.com', 'John');

      expect(() => {
        createUser('john@example.com', 'Jane');
      }).toThrow(new DuplicateError('Email already exists'));
    });

    // 边界情况
    it('should create user with minimal data', () => {
      const user = createUser('minimal@example.com');
      expect(user.email).toBe('minimal@example.com');
    });

    it('should trim whitespace from email', () => {
      const user = createUser('  john@example.com  ');
      expect(user.email).toBe('john@example.com');
    });
  });

  describe('getUserById', () => {
    beforeEach(() => {
      // setup：创建测试数据
      createUser('test@example.com', 'Test User');
    });

    // 正常路径
    it('should return user when exists', () => {
      const user = getUserById('1');
      expect(user).toEqual({
        id: '1',
        email: 'test@example.com',
        name: 'Test User',
      });
    });

    // 错误处理
    it('should return null when user not found', () => {
      const user = getUserById('nonexistent-id');
      expect(user).toBeNull();
    });

    // 边界情况
    it('should return null for empty id', () => {
      const user = getUserById('');
      expect(user).toBeNull();
    });
  });
});
```

### 测试质量检查清单

编写完测试后，检查以下项目：

```
□ 覆盖率检查
  □ 语句覆盖率 ≥ 85% （npm run test:unit -- --coverage）
  □ 分支覆盖率 ≥ 80% （所有 if/else 分支都执行过）
  □ 函数覆盖率 ≥ 85% （所有函数都被调用过）
  □ 优先覆盖业务逻辑，而非工具函数

□ 测试完整性
  □ 正常路径：有测试验证正确输入产生正确输出
  □ 边界情况：有测试验证 null、undefined、0、空字符串、最大值、最小值等
  □ 错误处理：有测试验证异常抛出、错误返回等
  □ 业务规则：spec 中的所有验收标准（TEST-VERIFY）都有对应测试
  □ 依赖相关：有测试验证与其他模块的交互

□ 测试质量
  □ 命名清晰：describe + it 明确描述预期行为
  □ 单一责任：每个 it 只验证一个行为
  □ 断言具体：使用 toEqual、toBe、toThrow 等，避免 toBeTruthy
  □ 隔离独立：测试间互不依赖，可任意顺序执行
  □ 确定性：同样输入必产生同样输出（避免时间、随机依赖）

□ 真实性检查
  □ 是否真的在测试业务逻辑（而不仅仅是工具函数）？
  □ 是否修改过源代码使其适配测试？（如果是，说明测试有问题）
  □ 是否有 always-true 的断言？（如 expect(true).toBe(true)）
  □ 是否正确使用了 mock（只 mock 外部依赖，不 mock 业务逻辑）？
  □ 测试数据是否来自 spec 示例（避免虚构的测试数据）？

□ Mock 策略
  □ ✅ Mock 了外部 API、数据库、文件系统
  □ ✅ Mock 了第三方库（HTTP 客户端、日期库等）
  □ ✅ Mock 了付费服务（支付、短信、邮件等）
  □ ❌ 没有 mock 业务逻辑
  □ ❌ 没有 mock 内部函数
  □ 测试数据具体且有代表性（来自 spec 示例）
```

### 常见的虚假测试陷阱

1. **Always-true 断言**
   ```typescript
   // ❌ 虚假
   it('should process data', () => {
     expect(true).toBe(true);
   });

   // ✅ 真实
   it('should return processed data', () => {
     const result = processData({ name: 'John' });
     expect(result).toEqual({ processed: true, name: 'John' });
   });
   ```

2. **模糊的断言**
   ```typescript
   // ❌ 虚假（不知道实际值是什么）
   it('should create user', () => {
     const user = createUser('john@example.com');
     expect(user).toBeTruthy();
   });

   // ✅ 真实（清晰的期望值）
   it('should create user with email', () => {
     const user = createUser('john@example.com');
     expect(user.email).toBe('john@example.com');
     expect(user.id).toBeDefined();
   });
   ```

3. **为了让测试通过修改源代码**
   ```typescript
   // ❌ 虚假：测试通过但违反了源代码的原意
   // 源代码被改为: return user || defaultUser
   it('should return default user for invalid id', () => {
     const user = getUserById('');
     expect(user).toBeDefined();  // 测试通过但没验证真实逻辑
   });

   // ✅ 真实：源代码保持不变，测试验证真实行为
   it('should return null for invalid id', () => {
     const user = getUserById('');
     expect(user).toBeNull();  // 验证了真实行为
   });
   ```

4. **跳过边界和错误场景**
   ```typescript
   // ❌ 虚假（只测试正常路径）
   it('should validate user', () => {
     const result = validateUser({ email: 'test@example.com', age: 25 });
     expect(result).toBe(true);
   });

   // ✅ 真实（完整的测试套件）
   describe('validateUser', () => {
     it('should return true for valid user', () => {
       const result = validateUser({ email: 'test@example.com', age: 25 });
       expect(result).toBe(true);
     });

     it('should return false for invalid email', () => {
       const result = validateUser({ email: 'invalid', age: 25 });
       expect(result).toBe(false);
     });

     it('should return false for age < 18', () => {
       const result = validateUser({ email: 'test@example.com', age: 17 });
       expect(result).toBe(false);
     });

     it('should throw error for missing email', () => {
       expect(() => {
         validateUser({ age: 25 });
       }).toThrow();
     });
   });
   ```

### TDD 流程中的单元测试

遵循 RED-GREEN-REFACTOR 流程：

```typescript
// RED: 先写测试（会失败）
describe('calculateDiscount', () => {
  it('should apply 10% discount for purchase > 100', () => {
    const price = 150;
    const discount = calculateDiscount(price);
    expect(discount).toBe(15);  // 预期值
  });
});

// GREEN: 写最少代码使测试通过
function calculateDiscount(price: number): number {
  if (price > 100) {
    return price * 0.1;
  }
  return 0;
}

// REFACTOR: 代码审查和优化（测试保持通过）
// - 添加更多边界情况的测试
// - 重构代码使其更清晰
// - 确保所有测试都通过
```

---


### Q: Task修复失败，如何恢复？

A: 因为使用了worktree隔离，修复失败可以直接删除worktree重新开始：
```bash
git worktree remove .claude/worktrees/T1-UserAuth
git worktree add .claude/worktrees/T1-UserAuth HEAD
cd .claude/worktrees/T1-UserAuth
# 重新开始
```

### Q: 如何确保并行Task不冲突？

A: worktree天生支持并行：
- 每个Task的worktree是独立的git树
- 修改完全隔离
- 并行执行时不产生git冲突

### Q: 修复后如何更新主分支？

A: 当所有Task都通过审查后，在code-test阶段会统一合并所有worktree的修改到主分支。

### Q: 如何跟踪修复过程？

A: worktree中的commit清晰记录：
```bash
cd .claude/worktrees/T1-UserAuth
git log                      # 查看该Task的修复历史
git show <commit>            # 查看具体修复内容
```

---

**规范执行 = 设计准确 + 代码高质量**
