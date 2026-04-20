# code-test README

快速导航和常用操作指南。

---

## 📌 快速开始（2分钟）

### 这个 Skill 做什么？

对 code-execute 生成的代码进行全面验证和测试。

**核心流程**：
```
静态分析 → 代码审查 → 高层测试 → 闭环验证 → 测试报告
```

**前置条件**：
- ✅ code-execute 已完成，所有Task都通过了审查
- ✅ src/ 目录中有生成的源代码
- ✅ 单元测试已由 code-execute 完成

### 基本命令

```bash
# 1. 代码质量检查
npm run lint                   # ESLint检查
npm run type-check            # TypeScript检查
npm run test:unit             # 单元测试（验证真实性）

# 2. 高层测试
npm run test:integration      # 集成测试
npm run test:e2e              # E2E测试
npm run test:performance      # 性能测试

# 3. 生成报告
npm run test:coverage         # 覆盖率报告
```

---

## 📚 文档导航

### 核心概念（5分钟）
👉 **从这里开始**：阅读 [SKILL.md](./SKILL.md)
- Skill的职责和核心流程
- 何时使用
- 关键约束

### 完整工作流（20-30分钟）
👉 **深入理解**：阅读 [references/workflow-detail.md](./references/workflow-detail.md)
- 5个完整工作步骤
- 代码质量静态分析
- 代码审查维度和检查清单
- 高层测试（集成、E2E、性能）
- 闭环验证矩阵
- 测试报告生成
- 最佳实践和常见问题

### 代码审查指南（10-15分钟）
👉 **代码审查方法**：阅读 [references/code-reviewer.md](./references/code-reviewer.md)
- 功能完整性检查
- 代码质量维度
- TypeScript类型检查
- 性能考虑
- 审查清单

### E2E测试提示（10分钟）
👉 **E2E测试设计**：阅读 [references/e2e-test-prompt.md](./references/e2e-test-prompt.md)

### 集成测试提示（10分钟）
👉 **集成测试设计**：阅读 [references/integration-test-prompt.md](./references/integration-test-prompt.md)

### 性能测试提示（10分钟）
👉 **性能测试设计**：阅读 [references/performance-test-prompt.md](./references/performance-test-prompt.md)

---

## 🔑 关键概念

### 1. 代码质量静态分析

运行以下检查：

```bash
# ESLint - 代码规范检查
npm run lint
# 检查点：
# - 0个Lint错误（必须）
# - 0个Lint警告（推荐）

# TypeScript - 类型检查
npm run type-check
# 检查点：
# - 0个TypeScript错误
# - 无any类型（strict模式）
# - 无隐式any

# 单元测试 - 覆盖率检查
npm run test:unit -- --coverage
# 检查点：
# - 覆盖率 ≥ 85%（语句、分支、函数）
# - 单元测试是否真实（不是虚假的"绿色"）
```

### 2. 单元测试真实性验证

这是 code-test 的新增职责——不仅要检查测试是否通过，还要验证测试是否真实：

**虚假测试的表现**：
- always-true断言：`expect(true).toBe(true)`
- 模糊的断言：`expect(data).toBeTruthy()`
- 修改源代码以适配测试
- 跳过边界和错误场景

**真实性检查方法**：
1. 修改源代码逻辑
2. 重新运行测试
3. 如果测试仍然通过 → 虚假测试
4. 需要返回 code-execute 重新编写

详见：[references/workflow-detail.md](./references/workflow-detail.md#单元测试真实性检查关键) 中的"单元测试真实性检查"部分

### 3. 代码审查五个维度

| 维度 | 检查点 | 示例 |
|------|--------|------|
| **功能完整性** | 是否实现了所有功能？ | Task定义的所有功能都有吗？ |
| **代码质量** | 是否易读易维护？ | 函数职责单一？有重复代码？ |
| **类型安全** | 所有参数都有类型？ | 是否避免了any？ |
| **性能** | 是否有不必要的操作？ | 是否有N+1查询？ |
| **一致性** | 是否符合规范？ | 符合design.md？符合项目规范？ |

### 4. 闭环验证矩阵

确保 TEST-VERIFY → 代码 → 测试 → 结果 完全对应：

```markdown
| 验收标准 | 单元测试 | 集成测试 | E2E测试 | 性能测试 | 结果 |
|---------|---------|---------|--------|---------|------|
| TV-1    | ✅      | ✅      | ✅     | -       | ✅   |
| TV-2    | ✅      | -       | -      | -       | ✅   |
| TV-3    | ✅      | ✅      | -      | ✅      | ✅   |

总计：所有验收标准都有对应的测试 ✅
```

---

## ⚠️ 常见陷阱

### 陷阱1: 接受虚假的单元测试

```typescript
// ❌ 虚假（不应该接受）
it('should process data', () => {
  expect(true).toBe(true);
});

// ✅ 真实（应该接受）
it('should transform data correctly', () => {
  const result = processData({ name: 'John' });
  expect(result).toEqual({ processed: true, name: 'John' });
});
```

👉 详见：[references/workflow-detail.md](./references/workflow-detail.md#单元测试真实性检查关键) 中的"单元测试真实性检查"部分

### 陷阱2: 为了提高覆盖率而添加无意义的测试

```typescript
// ❌ 无意义（只是凑覆盖率）
it('should have name property', () => {
  const user = { name: 'John' };
  expect(user.name).toBeDefined();
});

// ✅ 有意义（验证业务逻辑）
it('should create user with name', () => {
  const user = createUser('John', 'john@example.com');
  expect(user.name).toBe('John');
  expect(user.email).toBe('john@example.com');
});
```

### 陷阱3: 跳过E2E和性能测试

```bash
# ❌ 不完整
npm run test:unit              # 只运行单元测试

# ✅ 完整
npm run test:unit              # 单元测试
npm run test:integration       # 集成测试
npm run test:e2e               # E2E测试
npm run test:performance       # 性能测试
```

---

## 🔄 工作流检查清单

在使用 code-test 时，遵循以下步骤：

```
□ 步骤1: 代码质量静态分析
  □ npm run lint → 0个错误
  □ npm run type-check → 0个错误
  □ npm run test:unit -- --coverage → ≥85%覆盖率
  □ 验证单元测试真实性（无虚假测试）

□ 步骤2: 代码审查
  □ 功能完整性检查
    □ 所有Task的功能都实现了吗？
    □ 所有边界情况都处理了吗？
  □ 代码质量检查
    □ 易读易维护吗？
    □ 职责单一吗？
    □ 有重复代码吗？
  □ TypeScript类型检查
    □ 所有参数都有类型吗？
    □ 避免了any吗？
  □ 性能考虑
    □ 有N+1查询吗？
    □ 有不必要的重新渲染吗？

□ 步骤3: 高层测试
  □ 集成测试
    □ 多个组件协作正确吗？
    □ 数据流正确吗？
  □ E2E测试
    □ 关键用户流程都通过吗？
    □ UI交互正确吗？
  □ 性能测试
    □ 加载时间达标吗？
    □ 内存占用正常吗？

□ 步骤4: 闭环验证
  □ 生成验证矩阵
  □ 所有TEST-VERIFY都有对应测试
  □ 所有测试都通过

□ 步骤5: 生成测试报告
  □ 生成 spec-dev/{req_id}/testing/testing-report.md
  □ 包含所有测试结果
  □ 包含质量指标
  □ 包含闭环验证矩阵
```

---

## 💡 最佳实践

### 1. 单元测试真实性要严格

不要让虚假的单元测试通过 code-test 阶段：

```typescript
// 检验方法：修改源代码逻辑
// 如果修改后测试仍然通过 → 虚假测试 → 要求返回code-execute重新编写

// 原始源代码
function validateEmail(email: string): boolean {
  return email.includes('@');
}

// 修改为虚假实现
function validateEmail(email: string): boolean {
  return true;  // 总是返回true
}

// 如果测试仍然通过 → 虚假测试！
// if (test.passed) return REJECT;
```

### 2. 代码审查要全面

不要只关注代码功能，还要考虑：
- 易读性
- 可维护性
- 性能
- 一致性

### 3. 闭环验证要完整

确保 100% 的 TEST-VERIFY 都有对应的测试：

```markdown
# 闭环验证矩阵

| TEST-VERIFY | 单元测试 | 集成测试 | E2E | 结果 |
|------------|--------|---------|-----|------|
| TV-Auth-1  | ✅     | ✅      | ✅  | ✅   |
| TV-Auth-2  | ✅     | -       | -   | ✅   |
| TV-Auth-3  | ✅     | ✅      | -   | ✅   |

总计：3/3 TEST-VERIFY都有对应测试 ✅
```

### 4. 性能测试要有基准

性能测试应该有明确的目标：

```markdown
# 性能目标

前端：
- 首屏加载时间：< 3s
- 交互响应：< 100ms
- 内存占用：< 100MB

后端：
- API响应时间：< 200ms
- 数据库查询：< 100ms
- 并发处理：> 1000 RPS
```

---

## 🔗 相关Skills

- ← **code-execute** 提供生成的代码和单元测试
- ← **code-task** 提供任务清单和验收标准
- ← **spec-creation** 提供规范和TEST-VERIFY
- → **spec-archive** 接收测试通过的规范进行归档

---

## 📖 推荐阅读顺序

**快速上手（15分钟）**：
1. SKILL.md（职责和流程）
2. references/code-reviewer.md（代码审查）
3. references/workflow-detail.md 的"步骤1-2"部分

**深入学习（1小时）**：
1. references/workflow-detail.md（完整工作流）
2. references/code-reviewer.md（详细的审查维度）
3. references/e2e-test-prompt.md、integration-test-prompt.md、performance-test-prompt.md

**需要时查阅**：
- references/code-reviewer.md（审查维度速查）
- references/workflow-detail.md（各类测试的详细说明）

---

## ❓ 常见问题

**Q: 如何判断单元测试是否真实？**

A: 修改源代码逻辑，看测试是否失败：
```bash
# 1. 修改源代码（如改为总是返回true）
# 2. 重新运行测试
# 3. 如果测试仍然通过 → 虚假测试
# 4. 如果测试失败 → 真实测试
```

**Q: E2E测试应该覆盖哪些场景？**

A: 关键的用户流程和业务场景：
- 用户登录 → 操作 → 登出的完整流程
- 常见的错误场景（无权限、数据不存在等）
- 边界情况（超大输入、并发请求等）

**Q: 集成测试和E2E测试的区别？**

A:
- **集成测试**：验证多个组件/模块的协作（在代码层面）
- **E2E测试**：验证完整的用户流程（从UI到数据库）

**Q: 性能测试应该包含哪些指标？**

A: 根据应用类型而定，但通常包括：
- 加载时间（首屏、全页）
- 响应时间（用户操作 → UI更新）
- 内存占用
- CPU占用
- 并发处理能力

---

**更新时间**：2026-03-26
**对应版本**：sdd-tdd-dev v2.1.0
