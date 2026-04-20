# code-execute README

快速导航和常用操作指南。

---

## 📌 快速开始（2分钟）

### 这个 Skill 做什么？

根据任务列表（tasks.md），逐个实现代码任务，并通过两阶段审查（规范 + 质量）。

**核心流程**：
```
分析任务 → 并行执行Task → 两阶段审查 → 修复循环 → 执行报告
```

**前置条件**：
- ✅ spec-creation 已完成
- ✅ code-designer 已完成
- ✅ code-task 已完成（任务列表已定义）

### 基本命令

```bash
# 1. 为Task创建隔离工作环境
git worktree add .claude/worktrees/T1-TaskName HEAD
cd .claude/worktrees/T1-TaskName

# 2. 编码实现 + 单元测试
npm run test:unit              # 运行单元测试
npm run lint                   # ESLint检查
npm run type-check            # TypeScript检查

# 3. 完成后删除worktree
git worktree remove .claude/worktrees/T1-TaskName
```

---

## 📚 文档导航

### 核心概念（5分钟）
👉 **从这里开始**：阅读 [SKILL.md](./SKILL.md)
- Skill的职责和核心流程
- 何时使用
- 关键约束

### 完整工作流（15-20分钟）
👉 **深入理解**：阅读 [references/workflow-detail.md](./references/workflow-detail.md)
- 6个完整工作步骤
- 实际示例和操作指南
- git-worktree的详细使用
- 修复循环的实践

### 单元测试指南（10-15分钟）
👉 **编写真实可用的测试**：阅读 [references/unit-test-real-practices.md](./references/unit-test-real-practices.md)
- 什么是虚假测试（常见陷阱）
- 如何编写真实的单元测试
- TDD流程（RED-GREEN-REFACTOR）
- 完整的测试覆盖示例
- 测试质量检查清单

### TDD流程说明（5分钟）
👉 **了解RED-GREEN-REFACTOR**：阅读 [references/tdd-flow.md](./references/tdd-flow.md)

### git-worktree使用指南（10分钟）
👉 **掌握Task隔离**：阅读 [references/git-worktrees-guide.md](./references/git-worktrees-guide.md)

### 快速参考（1分钟）
👉 **速查表**：阅读 [references/QUICK_REFERENCE.md](./references/QUICK_REFERENCE.md)

---

## 🔑 关键概念

### 1. git-worktree 隔离

每个 Task 使用独立的 worktree：

```
repo/
├── main branch （保持干净）
└── .claude/worktrees/
    ├── T1-UserAuth/        ← Task 1 的隔离环境
    ├── T2-FormValidation/  ← Task 2 的隔离环境
    └── T3-ApiIntegration/  ← Task 3 的隔离环境
```

**优势**：
- ✅ Task间完全隔离，互不污染
- ✅ 修复失败可直接删除worktree重新开始
- ✅ 提交历史清晰记录修复过程
- ✅ 多Task并行执行不产生git冲突

### 2. 两阶段审查

每个 Task 都必须通过两道关卡：

| 阶段 | 检查点 | 审查者 |
|------|--------|--------|
| **规范审查** | 代码是否符合 design.md？ | spec-reviewer-prompt |
| **质量审查** | 代码质量是否达标？ | code-quality-reviewer-prompt |

两个审查都必须通过，才能进入下一个Task。

### 3. 单元测试约束

编写的单元测试必须是**真实可用的**，不是虚假的"绿色"：

**必须包含**：
- ✅ 正常路径测试
- ✅ 边界值测试
- ✅ 错误处理测试
- ✅ 所有业务规则验证（来自spec的TEST-VERIFY）

**禁止**：
- ❌ always-true断言（expect(true).toBe(true)）
- ❌ 修改源代码使其适配测试
- ❌ Mock业务逻辑（只Mock外部依赖）
- ❌ 跳过边界和错误场景

**覆盖率要求**：
- ≥ 85% 语句覆盖
- ≥ 80% 分支覆盖
- ≥ 85% 函数覆盖

### 4. 修复循环

如果审查未通过，执行修复循环：

```
Task实现 → 规范审查 ❌ 问题 → 修复 → 重新审查 ✅ 通过
                   ↓
                质量审查 ❌ 问题 → 修复 → 重新审查 ✅ 通过
```

**重复直到所有审查都通过**。

---

## ⚠️ 常见陷阱

### 陷阱1: 虚假的单元测试

```typescript
// ❌ 虚假
it('should create user', () => {
  expect(true).toBe(true);  // 无意义！
});

// ✅ 真实
it('should create user with email', () => {
  const user = createUser('john@example.com');
  expect(user.email).toBe('john@example.com');
});
```

👉 详见：[references/unit-test-real-practices.md](./references/unit-test-real-practices.md) 中的"问题识别"部分

### 陷阱2: worktree未清理

```bash
# ❌ 容易遗忘
git worktree add .claude/worktrees/T1-UserAuth HEAD
# ... 编码、测试、修复 ...
# 忘记删除！

# ✅ 正确做法
git worktree list              # 查看所有worktree
git worktree remove <path>     # 删除完成的worktree
```

### 陷阱3: 跳过边界和错误测试

```typescript
// ❌ 不完整（只测试正常情况）
it('should validate email', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBe(true);
});

// ✅ 完整
describe('validateEmail', () => {
  it('should accept valid email', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });

  it('should reject empty email', () => {
    expect(validateEmail('')).toBe(false);
  });

  it('should reject email without @', () => {
    expect(validateEmail('invalid')).toBe(false);
  });
});
```

---

## 🔄 工作流检查清单

在使用 code-execute 时，遵循以下步骤：

```
□ 步骤1: 分析任务列表
  □ 读取 spec-dev/{req_id}/tasks/tasks.md
  □ 理解所有Task及其依赖关系
  □ 制定并行执行计划

□ 步骤2: 执行Task批次
  □ 为每个Task创建独立worktree
  □ 分配实现子代理
  □ 在worktree中进行编码和自审查
  □ 编写单元测试（覆盖率 ≥85%）

□ 步骤3: 规范审查
  □ 检查代码是否符合design.md
  □ 检查命名、结构是否符合规范
  □ 问题修复后重新审查

□ 步骤4: 质量审查
  □ TypeScript检查（npm run type-check）
  □ ESLint检查（npm run lint）
  □ 单元测试覆盖率检查（≥85%）
  □ 单元测试真实性检查（是否有虚假测试）
  □ 问题修复后重新审查

□ 步骤5: 删除worktree并记录
  □ 所有Task都通过两阶段审查
  □ 删除worktree（git worktree remove）
  □ 生成执行报告

□ 步骤6: 输出报告
  □ 生成 spec-dev/{req_id}/execution/execution-report.md
  □ 包含所有Task的完成状态
  □ 包含修复循环的次数和内容
  □ 包含质量指标
```

---

## 💡 最佳实践

### 1. 及时清理worktree

```bash
# 检查当前worktree
git worktree list

# 删除已完成的worktree
git worktree remove .claude/worktrees/T1-UserAuth

# 不要长期保留worktree
```

### 2. 单元测试要覆盖业务规则

如果 spec 中有 TEST-VERIFY，对应的业务规则必须有单元测试：

```
spec/TEST-VERIFY-1: 邮箱必须唯一
  → 必须有单元测试验证邮箱唯一性
  → 测试数据：重复邮箱应该被拒绝
```

### 3. 修复前先记录问题

在修复前，清晰地记录：
- 什么通过了检查
- 什么没有通过
- 为什么失败

这样便于追踪和学习。

### 4. 使用小commit，便于追踪

在worktree中，每次修复都创建一个新commit：

```bash
# 初始实现
git commit -m "feat: implement UserAuth"

# 修复问题1
git commit -m "fix: add missing type annotation in UserAuth"

# 修复问题2
git commit -m "fix: increase test coverage for error handling"
```

---

## 🔗 相关Skills

- ← **code-task** 提供任务列表（tasks.md）
- ← **code-designer** 提供设计方案（design.md）
- ← **spec-creation** 提供规范和TEST-VERIFY
- → **code-test** 接收生成的代码进行高层测试

---

## 📖 推荐阅读顺序

**快速上手（15分钟）**：
1. SKILL.md（职责和流程）
2. references/QUICK_REFERENCE.md（速查表）
3. references/git-worktrees-guide.md（worktree使用）

**深入学习（1小时）**：
1. references/workflow-detail.md（完整工作流）
2. references/unit-test-real-practices.md（单元测试）
3. references/tdd-flow.md（TDD流程）

**需要时查阅**：
- references/QUICK_REFERENCE.md（遗忘时查询）
- references/git-worktrees-guide.md（worktree问题排查）

---

## ❓ 常见问题

**Q: worktree中的修改丢失了怎么办？**

A: 使用 `git reflog` 恢复：
```bash
cd .claude/worktrees/T1-UserAuth
git reflog
git checkout <lost-commit>
```

**Q: 怎样判断单元测试是否真实？**

A: 修改源代码逻辑，看测试是否失败。如果源代码改变后测试仍然通过，说明是虚假测试。

**Q: 应该Mock什么？**

A: 只Mock外部依赖（API、数据库、第三方服务），不Mock业务逻辑。详见 [references/unit-test-real-practices.md](./references/unit-test-real-practices.md#mock-策略)。

**Q: 单元测试覆盖率应该多少？**

A: ≥ 85%（语句、分支、函数都要达标）。优先覆盖业务逻辑，而非工具函数。

---

**更新时间**：2026-03-26
**对应版本**：sdd-tdd-dev v2.1.0
