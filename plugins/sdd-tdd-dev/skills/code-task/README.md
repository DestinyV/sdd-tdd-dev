# code-task README

快速导航和常用操作指南。

---

## 📌 快速开始（2分钟）

### 这个 Skill 做什么？

根据设计方案，将其分解为代码级别的具体任务清单，定义每个任务的交付物、依赖关系和验收标准。

**核心流程**：
```
分析设计 → 任务分解 → 依赖规划 → Test Case映射 → 任务清单
```

**前置条件**：
- ✅ spec-creation 已完成，规范文档已生成
- ✅ code-designer 已完成，设计方案已确认

### 基本输出

生成 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`，包含：
- 总Task数和并行计划
- 快速导航
- 并行执行计划
- Test Case Mapping
- Task详情

---

## 📚 文档导航

### 核心概念（5分钟）
👉 **从这里开始**：阅读 [SKILL.md](./SKILL.md)
- Skill的职责和工作流程
- 何时使用
- 关键约束

### 完整工作流（20-30分钟）
👉 **深入理解**：阅读 [references/workflow-detail.md](./references/workflow-detail.md)
- 6个完整工作步骤
- Task分解策略
- Task模板和示例
- Test Case Mapping完整说明
- 并行执行计划规划
- 最佳实践

### Task模板
👉 **Task定义方法**：查看 [templates/test-case-mapping-template.md](./templates/test-case-mapping-template.md)

### 相关提示词
👉 **理解各个Agent**：
- [prompts/implementer-prompt.md](./prompts/implementer-prompt.md) - 实现子代理
- [prompts/spec-reviewer-prompt.md](./prompts/spec-reviewer-prompt.md) - 规范审查
- [prompts/code-quality-reviewer-prompt.md](./prompts/code-quality-reviewer-prompt.md) - 质量审查

---

## 🔑 关键概念

### 1. Task分解原则

根据以下维度分解：

**方式1：按功能模块**
```
订单管理功能
├── Task 1: 订单创建
├── Task 2: 订单编辑
├── Task 3: 订单查询
└── Task 4: 订单删除
```

**方式2：按层级**
```
用户认证
├── Task 1: 后端API实现
├── Task 2: 前端表单组件
├── Task 3: 样式设计
└── Task 4: 单元测试
```

**方式3：按依赖关系**
```
Batch 1（无依赖）: Task 1, Task 2, Task 3 → 并行
  ↓
Batch 2（依赖Task1）: Task 4, Task 5 → 并行
  ↓
Batch 3（依赖Task2）: Task 6 → 单独
```

### 2. Task粒度

**推荐粒度**：4小时内完成的任务

```markdown
# 太大（❌ 不好）
Task: "实现整个用户认证系统"
- 包含：用户注册、登录、忘记密码、MFA等
- 问题：太复杂，难以并行，修复成本高

# 刚好（✅ 推荐）
Task: "实现用户登录功能"
- 包含：表单验证、密码验证、Session创建
- 好处：易于并行，修复成本低，自包含

# 太小（❌ 浪费）
Task: "实现LoginForm组件的输入框"
- 问题：管理开销大，任务太多
```

### 3. Task依赖关系

明确标识Task间的依赖：

```markdown
# Task依赖关系

| Task | 依赖 | 理由 |
|------|------|------|
| Task 1: 数据库设计 | 无 | 基础任务 |
| Task 2: API实现 | Task 1 | 需要数据库结构 |
| Task 3: 前端组件 | Task 2 | 需要API契约 |
| Task 4: 样式 | Task 3 | 样式依赖组件结构 |
| Task 5: E2E测试 | Task 2, 3 | 需要API和UI |

# 执行计划
Batch 1: Task 1 → 完成
Batch 2: Task 2, 3 → 并行（Task 3可以mock Task 2的API）
Batch 3: Task 4 → 开始
Batch 4: Task 5 → 最后
```

### 4. Test Case Mapping

将验收标准（TEST-VERIFY）映射到具体的测试用例：

```markdown
## Test Case Mapping

| Task | TEST-VERIFY | TC-ID | Mock数据 | 备注 |
|------|------------|-------|---------|------|
| Task 1: 订单创建 | TV-Order-1 | TC-1.1.1 | fixtures/product-valid.json | 创建有效订单 |
| Task 1: 订单创建 | TV-Order-2 | TC-1.1.2 | fixtures/product-invalid.json | 验证商品ID |
| Task 1: 订单创建 | TV-Order-3 | TC-1.1.3 | fixtures/quantity-boundary.json | 验证数量范围 |
| Task 2: 订单编辑 | TV-Order-4 | TC-1.2.1 | fixtures/order-owned.json | 编辑自己的订单 |
| Task 2: 订单编辑 | TV-Order-5 | TC-1.2.2 | fixtures/order-other.json | 拒绝编辑他人订单 |

# 验证
- 总TEST-VERIFY: 5个
- 总TC: 5个
- 映射完整性: 100% ✅
```

### 5. 验收标准定义

每个Task应该有清晰的验收标准：

```markdown
## Task 1: 用户登录

### 交付物
- LoginForm.tsx（React组件）
- LoginService.ts（业务逻辑）
- 单元测试（coverage ≥80%）

### 验收标准
- [ ] 用户能输入邮箱和密码
- [ ] 系统能验证邮箱格式
- [ ] 系统能验证密码长度（≥8字符）
- [ ] 登录成功后返回token
- [ ] 登录失败显示错误信息
- [ ] 单元测试覆盖率≥80%
- [ ] 代码通过ESLint和TypeScript检查

### 依赖
- Task 0: 数据库用户表设计（已完成）

### 估时
- 4小时
```

---

## ⚠️ 常见陷阱

### 陷阱1: Task过大或过小

```markdown
# ❌ 太大（无法并行）
Task: "实现整个订单模块"
- 包括：创建、编辑、查询、删除、支付、发货
- 问题：8小时以上，难以分配

# ✅ 合适
Task: "实现订单创建功能"
- 包括：表单验证、数据库保存、发送邮件
- 工期：3-4小时

# ❌ 太小（管理开销大）
Task: "实现订单表的id字段"
- 问题：1个Task = 1个字段，太琐碎
```

### 陷阱2: Test Case Mapping不完整

```markdown
# ❌ 不完整
Task 1: 订单创建
（没有与TV的映射）

# ✅ 完整
Task 1: 订单创建
映射：
- TV-Order-1 → TC-1.1.1 (创建有效订单)
- TV-Order-2 → TC-1.1.2 (验证商品ID)
- TV-Order-3 → TC-1.1.3 (验证数量范围)

确保: 所有TV都有对应TC
```

### 陷阱3: 遗漏重要的依赖

```markdown
# ❌ 缺少依赖
Task 3: 前端表单组件
（没有标记依赖Task 2: API实现）

# ✅ 明确依赖
Task 3: 前端表单组件
依赖：Task 2 (API实现)
理由：需要了解API的输入输出格式

可以并行做法：Task 2和3可以同时开始
但Task 3需要使用Task 2的API mock
```

---

## 📋 工作流检查清单

在进行任务分解时：

```
□ 步骤1: 分析设计方案
  □ 理解架构设计
  □ 理解技术方案
  □ 理解关键组件

□ 步骤2: 分解为编码任务
  □ 按功能模块或层级分解
  □ 确保Task独立性
  □ 控制粒度（3-4小时）

□ 步骤3: 定义Task详情
  □ 目标清晰
  □ 交付物明确
  □ 依赖关系清楚
  □ 验收标准完整
  □ 估时合理

□ 步骤4: 关联测试验收标准
  □ 将每个TV映射到相应TC
  □ 关联Mock数据
  □ 确保100%映射完整性

□ 步骤5: 规划并行执行
  □ 识别无依赖的Task（可并行）
  □ 按依赖关系分批
  □ 合理安排并行度

□ 步骤6: 生成任务列表文档
  □ 生成tasks.md
  □ 包含所有Task和并行计划
  □ 包含完整的Test Case Mapping
```

---

## 💡 最佳实践

### 1. Task要自包含

每个Task应该能独立完成：

```markdown
# ❌ 不够自包含
Task: "实现用户注册表单"
- 缺少：后端API、数据库设计

# ✅ 自包含
Task: "实现用户注册功能"
- 包括：表单（前端）+ API + 数据库 + 单元测试
- 或明确依赖：Task 1（后端）→ Task 2（前端）
```

### 2. 依赖关系要清晰

```markdown
# Task依赖关系

Task 1: 数据库设计
  ↓ 依赖
Task 2: 后端API实现
  ↓ 依赖
Task 3: 前端组件
  ↑ 也依赖
Task 4: E2E测试

# 执行计划
Phase 1: Task 1 (完成后)
Phase 2: Task 2, Task 4 (并行)
Phase 3: Task 3, Task 4 (并行，Task 4需要Task 2和3都完成)
```

### 3. 验收标准要可测试

```markdown
# ❌ 不可测试
验收标准：
- "功能正常工作"
- "代码质量好"

# ✅ 可测试
验收标准：
- "创建订单成功返回orderID"
- "单元测试覆盖率≥85%"
- "ESLint和TypeScript检查都通过"
- "API响应时间<200ms"
```

### 4. Test Case Mapping要完整

```markdown
# 检查清单
□ 所有TV项都有对应的TC
□ 每个TC都有唯一的TC-ID
□ 每个TC都有关联的Mock数据
□ Mock数据都来自spec
□ 映射100%完整（没有遗漏的TV）
```

---

## 🔗 相关Skills

- ← **code-designer** 提供设计方案
- ← **spec-creation** 提供规范和TEST-VERIFY
- → **code-execute** 接收任务列表进行编码实现
- → **test-design** 接收Task和TEST-VERIFY进行测试设计

---

## 📖 推荐阅读顺序

**快速上手（15分钟）**：
1. SKILL.md（职责和流程）
2. SKILL.md 中的"工作流程"部分前两个步骤

**深入学习（30分钟）**：
1. SKILL.md 的完整内容
2. references/workflow-detail.md 的前四个步骤
3. templates/test-case-mapping-template.md

**需要时查阅**：
- references/workflow-detail.md（详细工作流）
- templates/test-case-mapping-template.md（映射模板）

---

## ❓ 常见问题

**Q: Task应该多大？**

A: 推荐4小时内完成。太大（>8h）难以并行和修复，太小（<1h）管理开销大。

**Q: 如何处理有复杂依赖关系的Task？**

A: 明确标记依赖，但尽可能减少依赖数量。可以使用Mock或接口契约来"解耦"：
```markdown
Task 2: 后端API
Task 3: 前端组件（依赖Task 2）

可以并行做法：
- Task 2和3同时开始
- Task 3使用Task 2的API mock进行开发
- 最后再集成真实API
```

**Q: Test Case Mapping应该多细？**

A: 足以清楚地定义每个测试是什么。通常到TC-ID级别就够了，不需要编写完整的测试代码（那是code-test的职责）。

**Q: 如何检查Task分解是否合理？**

A:
- [ ] 每个Task能在4小时内完成吗？
- [ ] Task之间的依赖清晰吗？
- [ ] 每个Task的验收标准都可测试吗？
- [ ] 所有TEST-VERIFY都有对应的TC吗？
- [ ] 没有Task跨越多个模块吗？

---

**更新时间**：2026-03-26
**对应版本**：sdd-tdd-dev v2.1.0
