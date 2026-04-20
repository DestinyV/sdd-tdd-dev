# TDD流程（测试驱动开发）

在code-execute阶段，支持遵循**TDD（测试驱动开发）**的红-绿-重构循环。

---

## 概述

TDD流程将原本的编码实现分解为4个清晰的阶段：

```
     🔴                  🟢               🔵              ✅
   RED                GREEN            REFACTOR        REVIEW
（测试失败）         （测试通过）       （代码优化）      （质量审查）
    ↓                   ↓                 ↓               ↓
写失败的测试  →  写最小化实现  →  改进代码质量  →  通过审查
   │                   │                 │               │
   └─────────────────────────────────────┴───────────────┘
           (可能多次循环，直到功能完整)
```

---

## 何时使用TDD流程

### ✅ **推荐使用TDD**

- ✅ 复杂的业务逻辑（需要精细的测试驱动）
- ✅ 核心功能模块（需要高质量和强的信心）
- ✅ 团队有TDD经验（已培训，理解最佳实践）
- ✅ 有test-designer提供的test-spec.md（清晰的测试需求）

### ❌ **可考虑跳过TDD**

- ❌ 简单的UI界面（直接编码+测试可能更快）
- ❌ 原型开发（快速迭代，质量要求不高）
- ❌ 脚本工具（简单逻辑，自动化脚本）

### 📋 **前置条件**

要使用TDD流程，需要有：

1. **test-spec.md** ✅
   - 来自test-designer的完整测试规范
   - 包含test case清单和Mock定义
   - 包含fixtures.json

2. **test-*.template** ✅
   - 来自test-designer的测试框架代码
   - 可直接运行（或少量填充）

3. **清晰的需求** ✅
   - tasks.md中的验收标准
   - test-case-mapping映射关系

---

## 阶段1：RED - 编写失败的测试

**目标**：根据测试规范写出能表达需求的失败的test

### 活动

1. **理解测试需求**
   ```
   阅读 test-spec.md，理解：
   - 该Task的所有test case
   - 每个test的期望行为
   - Mock和Fixture的定义
   ```

2. **选择一个test case并写成代码**
   ```typescript
   // 例子：
   describe('Task1: UserForm', () => {
     test('TC-1.1.1: 应该支持username输入', () => {
       // 使用test-*.template中的框架
       const form = render(<UserForm />);
       expect(form.getByTestId('username-input')).toBeInTheDocument();
       // ← 此时这个test会失败（因为代码还没写）
     });
   });
   ```

3. **运行test，确认失败**
   ```bash
   npm test -- TC-1.1.1
   # 结果：❌ FAIL
   ```

4. **重复**：对每个test case都写出对应的失败的test

### 关键点

- ✅ **一次一个test** - 先写一个test，让它失败
- ✅ **清晰的失败信息** - test失败时，要清楚地表达期望的行为
- ✅ **不要跳跃** - 不要一次写所有test，要逐个来
- ✅ **使用Mock** - 使用fixtures.json中定义的Mock和测试数据

### 时间估算

- 简单Task（3-5个test）：15-20分钟
- 复杂Task（10+个test）：30-45分钟

---

## 阶段2：GREEN - 写最小化实现

**目标**：写**最少的代码**，让test通过

### 活动

1. **根据test的期望，实现功能**
   ```typescript
   // 例子：
   export function UserForm() {
     return (
       <form>
         <input data-testid="username-input" />
         {/* 最少化实现：只添加test需要的input */}
       </form>
     );
   }
   ```

2. **运行test**
   ```bash
   npm test -- TC-1.1.1
   # 结果：✅ PASS
   ```

3. **如果test还是失败，调整实现**
   - 不是重写test，而是调整实现
   - 确保实现满足test的期望

4. **重复**：对每个test逐个实现功能

### 关键点

- ✅ **最小化实现** - 只写test需要的，不要过度工程化
- ✅ **一个test一个实现** - 完成一个test的实现，再处理下一个
- ✅ **快速反馈** - 立即运行test，验证实现正确
- ✅ **不提前优化** - 不要在这个阶段考虑性能或美观，只关注功能

### 实现技巧

- 🔷 使用hardcoded值（如果test允许）
- 🔷 复制粘贴代码（之后的重构阶段再提取）
- 🔷 不完整的错误处理（test没有要求就不写）
- 🔷 目的是快速让test通过

### 时间估算

- 简单Task（3-5个test）：20-30分钟
- 复杂Task（10+个test）：40-60分钟

---

## 阶段3：REFACTOR - 改进代码质量

**目标**：在test仍然通过的前提下，改进代码质量

### 活动

1. **识别重复代码**
   ```typescript
   // 重构前：
   function validateEmail(email) { return email.includes('@'); }
   function validateEmail2(email) { return email.includes('@'); }

   // 重构后：
   function validateEmail(email) { return email.includes('@'); }
   ```

2. **提取工具函数**
   ```typescript
   // 重构前：
   if (username && username.length > 0 && username.length <= 20) {}

   // 重构后：
   function isValidUsername(username) {
     return username && username.length > 0 && username.length <= 20;
   }
   if (isValidUsername(username)) {}
   ```

3. **改进变量和函数命名**
   ```typescript
   // 重构前：
   const x = data.map(item => item.name);

   // 重构后：
   const usernames = data.map(user => user.name);
   ```

4. **添加注释和类型**
   ```typescript
   // 重构后：
   /**
    * 验证用户名是否有效
    * @param username - 用户名，长度1-20字符
    * @returns 如果有效返回true，否则返回false
    */
   function isValidUsername(username: string): boolean {
     return username.length > 0 && username.length <= 20;
   }
   ```

5. **运行所有test，确保都通过**
   ```bash
   npm test
   # 结果：✅ ALL PASS（和之前一样的test通过）
   ```

### 关键点

- ✅ **test先行** - 所有重构前，test都要通过
- ✅ **小步重构** - 每次重构一个小点，立即运行test
- ✅ **不改变行为** - 重构只改代码结构，不改逻辑
- ✅ **保持test通过** - 如果test失败，说明重构出了问题

### 可做的重构

- 🔷 提取工具函数
- 🔷 提取公共逻辑
- 🔷 改进命名
- 🔷 添加类型和注释
- 🔷 优化算法（在test仍然通过的前提下）
- 🔷 改进错误处理

### 不应该做的重构

- ❌ 添加新功能（违反test的期望）
- ❌ 改变函数签名（除非更新了对应的test）
- ❌ 改变行为（重构应该是结构改进，不是逻辑改进）

### 时间估算

- 简单Task：10-15分钟
- 复杂Task：20-30分钟

---

## 阶段4：REVIEW - 质量审查

**目标**：确保重构后的代码仍然满足验收标准和质量要求

### 活动

1. **运行所有test**
   ```bash
   npm test
   # 确保所有test都通过
   ```

2. **检查代码质量**
   ```bash
   npm run lint
   npm run type-check
   ```

3. **验证test覆盖率**
   ```bash
   npm test -- --coverage
   # 确保覆盖率 ≥ 80%（对于该Task）
   ```

4. **进行规范审查**
   - 代码是否符合design.md规范？
   - 是否遵循项目的编码规范？
   - 是否满足验收标准？

5. **最后一遍自审查**
   - 代码可读性如何？
   - 有逻辑错误吗？
   - 有性能问题吗？
   - 有安全问题吗？

### 质量检查清单

- [ ] 所有test都通过
- [ ] Lint检查通过（0个警告或错误）
- [ ] TypeScript strict模式通过
- [ ] 覆盖率 ≥ 80%
- [ ] 没有hardcoded值（除非必要）
- [ ] 没有TODO注释（都应该解决了）
- [ ] 代码符合design.md规范
- [ ] 命名清晰准确
- [ ] 注释完整

### 时间估算

- 简单Task：5-10分钟
- 复杂Task：10-20分钟

---

## 完整例子

假设Task是：实现表单验证

### RED阶段

```typescript
describe('Task 1: UserForm验证', () => {
  test('TC-1.1.1: 应该在username为空时验证失败', () => {
    const form = new UserForm({username: ''});
    expect(form.validate()).toBe(false);
  });

  test('TC-1.1.2: 应该在username有效时验证通过', () => {
    const form = new UserForm({username: 'john_doe'});
    expect(form.validate()).toBe(true);
  });
});

// 运行test → ❌ FAIL（因为UserForm还没写）
```

### GREEN阶段

```typescript
class UserForm {
  constructor(data) {
    this.data = data;
  }

  validate() {
    // 最小化实现：检查username是否非空
    return !!this.data.username;
  }
}

// 运行test → ✅ PASS
```

### REFACTOR阶段

```typescript
class UserForm {
  constructor(data) {
    this.data = data;
  }

  validate() {
    return this.isValidUsername(this.data.username);
  }

  /**
   * 验证用户名是否有效
   */
  private isValidUsername(username: string): boolean {
    return !!username && username.length > 0 && username.length <= 20;
  }
}

// 运行test → ✅ PASS（和之前一样）
```

### REVIEW阶段

```bash
npm test                    # ✅ 所有test通过
npm run lint               # ✅ lint检查通过
npm run type-check         # ✅ TypeScript类型检查通过
npm test -- --coverage     # ✅ 覆盖率 85%
```

---

## TDD的好处

### 💡 为什么要用TDD？

1. **高质量代码** - test强制你考虑所有场景
2. **更少的bug** - 问题在早期就被发现
3. **更易维护** - test文档化了代码的行为
4. **设计更清晰** - test写出好的接口设计
5. **重构有信心** - test保证改动没有破坏功能
6. **快速反馈** - 立即知道改动是否有效

### ⚠️ TDD的常见困难

- **学习曲线** - 需要时间习惯"test先行"的思维
- **test写得慢** - 刚开始会比较慢，需要时间提速
- **过度设计** - 有时test会导致过度的代码结构
- **维护成本** - 修改代码时也要修改test

### 📈 提速技巧

1. **使用test模板** - test-*.template大大加快test编写
2. **复用Mock** - fixtures.json中已经定义了所有Mock
3. **小步快走** - 一次一个test，快速循环
4. **工具支持** - 使用IDE的test运行器快速反馈
5. **熟能生巧** - 写得多了，自然会更快

---

## TDD vs 传统流程对比

### 传统流程（设计→编码→测试）

```
[设计] → [编码] → [测试] → [修复]
  ↑                         │
  └─────────────────────────┘
           (可能多次修复)
```

**问题**：测试在最后，问题发现得晚

### TDD流程（红-绿-重构）

```
[RED: 写失败test] → [GREEN: 写实现] → [REFACTOR: 改进] → [REVIEW: 审查]
         ↑                                                      │
         └──────────────────────────────────────────────────────┘
              (对每个test重复，确保功能完整)
```

**优点**：test驱动设计，问题发现得早

---

## TDD进度追踪

在执行阶段，可以使用以下进度表追踪TDD的进展：

| Test Case ID | 描述 | RED | GREEN | REFACTOR | REVIEW | 状态 |
|-------------|------|-----|-------|----------|--------|------|
| TC-1.1.1 | 验证username非空 | ✅ | ✅ | ✅ | ✅ | ✅ |
| TC-1.1.2 | 验证username有效 | ✅ | ✅ | ✅ | ✅ | ✅ |
| TC-1.2.1 | 边界值测试 | ✅ | ⏳ | - | - | 进行中 |
| TC-1.3.1 | 错误处理 | - | - | - | - | 待开始 |

---

## 与code-execute阶段的整合

TDD流程**集成在code-execute阶段的实现子代理**中。

### 流程

```
code-execute分配实现子代理
    ↓
实现子代理遵循TDD流程：
    RED → GREEN → REFACTOR → REVIEW
    ↓
所有test通过
    ↓
进入规范审查阶段（code-reviewer）
    ↓
进入代码质量审查阶段
    ↓
Task完成
```

### 与传统实现的区别

**传统实现**：编码 → 写test → 修复

**TDD实现**：写test → 实现 → 优化 → 验证

---

## 常见问题

**Q: TDD会让实现变慢吗？**

A: 初期会慢，但长期来看：
- 修复bug的时间减少（前面就发现了）
- 维护时间减少（test文档化了行为）
- 重构更有信心（不怕改坏）
- 总体时间反而更快

**Q: 是否应该为所有代码都写test？**

A: 理想是100%，但实际可以：
- 关键逻辑：必须写test（TDD）
- 简单逻辑：可以简化test
- UI代码：可以用集成test代替单元test
- 目标覆盖率：≥ 80%

**Q: test写多了会不会太复杂？**

A: 如果test复杂，说明代码设计有问题。
- test应该简洁清晰
- 如果test难写，改进代码设计
- 使用test框架和Mock简化test

**Q: 如何处理test中的flaky（不稳定）问题？**

A:
1. 避免time-dependent logic（如Date.now()）
2. Mock所有外部依赖（API、时间等）
3. 使用fixtures.json中的固定数据
4. 避免顺序依赖（各test应独立）

---

## 总结

TDD的红-绿-重构循环确保：

✅ **质量** - test驱动设计，高质量代码
✅ **速度** - 快速反馈，快速迭代
✅ **信心** - test为修改提供保障
✅ **维护** - test文档化代码行为

**开始TDD，写出更好的代码！** 🚀

