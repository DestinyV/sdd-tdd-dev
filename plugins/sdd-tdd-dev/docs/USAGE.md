# 使用指南

## 快速开始（5分钟）

需求SDD工作流包括5个阶段：

### 第1步：需求规范化

```bash
/spec-creation [需求描述]
```

AI会进行需求分析，通过交互式确认生成规范文档。

**输入**：
- 需求标题
- 详细需求描述
- 相关背景说明

**输出**：`spec-dev/{requirement_desc_abstract}/spec/`
- `README.md` - 规范总览
- `scenarios/*.md` - BDD格式场景（WHEN-THEN）
- `data-models.md` - 数据模型
- `business-rules.md` - 业务规则
- `glossary.md` - 术语表

### 第2步：代码设计规划

```bash
/code-designer
```

AI会基于规范进行设计分析，生成详细的设计方案。

**输出**：`spec-dev/{requirement_desc_abstract}/design/design.md`
- 需求分析
- 设计方案（架构、组件、状态管理、数据流、样式、交互）
- 与参考设计的对比
- 技术方案和API设计
- 设计决策记录

### 第3步：任务列表生成

```bash
/code-task
```

AI会将设计方案转换为详细的代码级别任务列表。

**输出**：`spec-dev/{requirement_desc_abstract}/tasks/tasks.md`
- 任务依赖关系
- 每个Task的详细定义（目标、交付物、验收标准）
- 质量标准

### 第4步：代码执行实现

```bash
/code-execute
```

AI会为每个Task分配子代理进行实现，并进行多阶段审查。

**输出**：
- `src/` - 生成的源代码
- `spec-dev/{requirement_desc_abstract}/execution/execution-report.md` - 执行报告

**执行过程**：
- 实现子代理：编码实现
- 规范审查：验证代码符合设计
- 质量审查：检查代码质量
- 修复循环：问题修复后重新审查

### 第5步：测试和验证

```bash
/code-test
```

AI会进行全面的测试和闭环验证。

**输出**：
- `tests/` - 生成的测试文件
- `spec-dev/{requirement_desc_abstract}/testing/testing-report.md` - 测试报告

**包含**：
- 代码质量审查（Lint、TypeScript检查）
- TDD实现单元测试（RED-GREEN-REFACTOR-REVIEW）：由code-execute通过TDD流程完成
- 高层测试（集成、E2E、性能）：由code-test执行
- 闭环验证（TEST-VERIFY→Test→Code→Result）
- 覆盖率分析（单元测试≥80%，完整的集成/E2E/性能验证）

---

## 详细使用流程

### 场景1：React 项目 - 订单管理表单

#### 1.1 需求规范化
```bash
/spec-creation 需要实现订单管理表单，支持搜索、排序、分页、批量操作

# AI会进行交互式分析，通过多轮确认生成规范
# - 拆解需求为不同的业务场景
# - 对每个场景进行细化和确认
# - 生成BDD格式的规范文档
```

**输出**：`spec-dev/REQ-001/spec/`
- README.md - 规范总览
- scenarios/scenario-1.md 等 - 场景详情（WHEN-THEN格式）
- data-models.md - 数据模型
- business-rules.md - 业务规则
- glossary.md - 术语表

#### 1.2 代码设计规划
```bash
/code-designer

# AI会：
# 1. 读取spec-dev/REQ-001/spec/中的规范
# 2. 分析用户需求，识别对应的设计模式
# 3. 进行需求-参考差异分析
# 4. 生成详细的设计方案
```

**输出**：`spec-dev/REQ-001/design/design.md`
- 需求分析（核心功能、用户场景）
- 设计方案（架构、组件设计、状态管理、数据流、样式、交互）
- 与参考设计的对比
- 技术方案和API设计
- 设计决策记录

#### 1.3 任务列表生成
```bash
/code-task

# AI会：
# 1. 分析design.md中的设计方案
# 2. 分解为具体的编码任务
# 3. 定义每个Task的目标、交付物、验收标准
```

**输出**：`spec-dev/REQ-001/tasks/tasks.md`
- 任务依赖关系图
- Task 1: 实现 OrderTable 组件
- Task 2: 实现搜索和排序功能
- Task 3: 实现分页功能
- Task 4: 实现批量操作功能
- Task 5: 编写测试

#### 1.4 代码执行实现（TDD流程）
```bash
# 用户审查设计和任务，批准进入执行

/code-execute

# AI会：
# 1. 为每个Task分配独立实现子代理
# 2. 执行TDD流程：
#    🔴 RED阶段：编写失败的单元测试（test-spec.md生成的测试框架）
#    🟢 GREEN阶段：实现最少代码让测试通过
#    🔵 REFACTOR阶段：在测试通过的前提下优化代码
#    ✅ REVIEW阶段：质量和规范审查
# 3. 进行规范审查（确保符合design.md）
# 4. 进行质量审查（确保代码质量）
# 5. 发现问题则修复并重新审查

# 结果：单元测试覆盖率≥85%，代码经过TDD验证
```

**输出**：
- `src/components/OrderTable.tsx` - 订单表格组件
- `src/components/OrderForm.tsx` - 订单表单组件
- `src/hooks/useOrderManagement.ts` - 业务逻辑Hook
- `src/types/order.ts` - 类型定义
- `spec-dev/REQ-001/execution/execution-report.md` - 执行报告

#### 1.5 高层测试和验证（Phase 3）
```bash
# 编译验证通过

/code-test

# AI会：
# 1. 进行全面代码审查（Lint、TypeScript检查）
# 2. 进行单元测试验证（TDD流程确保的单元测试）
# 3. 执行高层测试：
#    - 集成测试：验证多个Task的协作
#    - E2E测试：验证完整的业务流程
#    - 性能测试：建立性能基准
# 4. 闭环验证：TEST-VERIFY → Test → Code → Result
# 5. 生成详细的测试报告和验证矩阵
# 4. 分析测试覆盖率
```

**输出**：
- `tests/components/OrderTable.test.tsx` - 表格测试
- `tests/hooks/useOrderManagement.test.ts` - Hook测试
- `spec-dev/REQ-001/testing/testing-report.md` - 测试报告

---

### 场景2：Vue 项目 - 数据仪表板

```bash
# 1. 需求规范化
/spec-creation 需要创建数据仪表板，支持实时数据、多图表展示、自定义面板

# 2. 代码设计
/code-designer
# AI分析规范并生成设计方案

# 3. 任务分解
/code-task
# AI将设计转换为具体任务

# 4. 代码执行
/code-execute
# AI执行所有Task，生成代码

# 5. 测试验证
/code-test
# AI进行测试和闭环验证
```

---

### 场景3：整个工作流完整示例

```bash
# Step 1: 需求规范化（首次使用）
/spec-creation 需要实现用户认证模块，支持邮箱注册、密码重置、社交登录

# [AI会进行交互分析，确认需求...]
# 输出：spec-dev/REQ-002/spec/

# Step 2: 设计规划
/code-designer
# [AI读取规范，进行设计分析...]
# 输出：spec-dev/REQ-002/design/design.md

# Step 3: 任务分解
/code-task
# [AI分解任务...]
# 输出：spec-dev/REQ-002/tasks/tasks.md

# Step 4: 代码执行
/code-execute
# [AI执行所有Task，多阶段审查...]
# 输出：src/ + spec-dev/REQ-002/execution/execution-report.md

# Step 5: 测试验证
/code-test
# [AI进行测试和闭环验证...]
# 输出：tests/ + spec-dev/REQ-002/testing/testing-report.md

# 完成！代码质量达标，可以上线
```

---

## 常见问题和技巧

### Q1: 首次使用应该运行哪个命令？

**A:** 首次使用应该运行 `/spec-creation` 生成项目规范。这是规范驱动开发工作流的开始第一步，将需求进行规范化描述。

### Q2: spec-creation 做什么？

**A:** `/spec-creation` 接收需求描述，通过交互式确认将其拆解为具体的业务场景，生成BDD格式的规范文档（WHEN-THEN格式）。规范是后续所有设计和开发的基础。

### Q3: code-designer 如何使用？

**A:** `/code-designer` 需要在 spec-creation 完成后运行。它会读取规范文档，进行代码设计分析，生成详细的设计方案（design.md），包含架构、组件设计、状态管理、技术方案等。

### Q4: code-task 和 code-designer 的输出有什么区别？

**A:**
- **code-designer 输出 design.md** - 架构级别的设计方案，说明"应该怎样设计"
- **code-task 输出 tasks.md** - 代码级别的任务清单，说明"具体要做什么编码任务"

### Q5: code-execute 中两道审查是什么？

**A:**
1. **规范审查** - 确保生成的代码符合 design.md 的设计方案
2. **质量审查** - 确保代码质量达标（无Lint错误、TypeScript检查通过等）

两道审查都要通过，发现问题会自动修复并重新审查。

### Q6: 能否跳过某个阶段？

**A:** 不建议跳过。每个阶段都有特定目的：
- 跳过规范化会导致设计偏离需求
- 跳过设计会导致后续频繁修改
- 跳过任务分解会导致执行混乱
- 跳过审查会降低代码质量
- 跳过测试无法验证功能完整性

### Q7: 测试覆盖率不足 80% 怎么办？

**A:** 需要继续编写测试用例，直到覆盖率达到 80%。这是质量的最低保证。code-test 会给出具体的未覆盖部分。

### Q8: 测试发现问题怎么处理？

**A:** 回到源代码修复问题，然后重新运行测试。不要修改测试用例来适应代码。

### Q9: 如何处理需求变更？

**A:**
1. 更新 spec-dev/{requirement_desc_abstract}/spec/ 中的规范文档
2. 在 code-designer 阶段确认影响范围
3. 更新 design.md 和 tasks.md
4. 重新执行受影响的 Task
5. 重新进行测试

### Q10: 工作流出错了怎么办？

**A:**
1. 检查执行报告中的错误记录
2. 确认错误的原因（通常是设计不清晰或需求理解偏差）
3. 修正规范或设计文档
4. 重新执行相应的阶段

---

## 最佳实践

### 1. 充分进行需求规范化
在 spec-creation 阶段，充分进行交互分析，确保规范完整。

```bash
# ✅ 好的做法
通过多轮确认，拆解出所有业务场景，确认WHEN-THEN条件清晰

# ❌ 不好的做法
仓促结束，规范内容模糊，后续需要频繁修正
```

### 2. 设计要足够详细
在 code-designer 阶段，生成的设计方案要足够详细，能直接指导编码。

```bash
# ✅ 好的做法
- Props接口明确
- 状态管理方式明确
- 数据流清晰
- 交互流程详细

# ❌ 不好的做法
设计笼统，编码时还要反复确认细节
```

### 3. 任务分解要清晰
在 code-task 阶段，任务要分解清楚，每个Task都有明确的交付物和验收标准。

```bash
# ✅ 好的做法
- 每个Task在4小时内可完成
- 依赖关系清晰
- 验收标准可衡量

# ❌ 不好的做法
任务过大或过小，或验收标准模糊
```

### 4. 审查很关键
在 code-execute 阶段，规范审查和质量审查都要通过。

```bash
# ✅ 规范审查要检查：
# - Props设计与design.md一致
# - 功能实现与design.md相符
# - 交互流程与设计一致

# ✅ 质量审查要检查：
# - 代码风格统一
# - 类型完整、无any
# - 测试覆盖率≥80%
```

### 5. 闭环验证很重要
在 code-test 阶段，确保Task→代码→测试的完全对应。

```bash
# 验证项：
# 1. 功能点完整性 - 设计中的每个功能都被实现并测试了吗？
# 2. 接口一致性 - 代码接口与设计相符吗？
# 3. 数据完整性 - 数据模型与设计相符吗？
```

---

## 工作流程总结

```
┌──────────────────────────────────────────────┐
│ 1. /spec-creation [需求描述]                 │
│    输出 spec-dev/{requirement_desc_abstract}/spec/             │
│    规范化需求、拆解场景、确认细节          │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 2. /code-designer                            │
│    输出 spec-dev/{requirement_desc_abstract}/design/design.md  │
│    设计分析、架构规划、用户确认             │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 3. /code-task                                │
│    输出 spec-dev/{requirement_desc_abstract}/tasks/tasks.md    │
│    任务分解、交付物定义、用户确认           │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 4. /code-execute                             │
│    输出 src/ + execution-report.md           │
│    编码实现 + 规范审查 + 质量审查           │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 5. /code-test                                │
│    输出 tests/ + testing-report.md           │
│    代码审查 + 单元/集成/E2E测试 + 闭环验证 │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
                 ✅ 功能完成，代码质量达标
```

---

更多详情见 [README.md](../README.md) 和 [最佳实践](./BEST_PRACTICES.md)
