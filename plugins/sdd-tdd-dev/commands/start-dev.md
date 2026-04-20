---
description: SDD+TDD规范驱动开发工作流 - 从需求规范化到测试验证的完整流程
argument-hint: 可选：需求描述
---

# SDD 规范驱动开发工作流

你正在使用 **sdd-dev-plugin**，这是一个企业级的规范驱动开发（SDD + TDD）Claude Code 插件。通过6个专业的 Skills，帮助你快速进行需求规范化、设计规划、任务分解、TDD实现、高层测试、规范归档。

## 🎯 工作流概览

```
需求 → 规范化 → 设计 → 任务 → 执行 → 测试 → 完成
```

### SDD完整工作流的6个阶段

| 阶段 | Skill命令 | 对应Agent | 输入 | 输出 | TDD阶段 | 说明 |
|------|-----------|----------|------|------|---------|------|
| **Phase 0** | `/spec-creation` | 主Agent | 需求描述 | `spec-dev/{req_id}/spec/` | - | 🎯 **规范定义**：生成BDD规范 + TEST-VERIFY验收标准 |
| **Phase 1** | `/code-designer` | 主Agent | spec/ | `spec-dev/{req_id}/design/design.md` | - | 🎨 **架构设计**：设计规范和技术方案 |
| **Phase 2** | `/code-task` | 主Agent | design.md | `spec-dev/{req_id}/tasks/tasks.md` | - | 📋 **任务分解**：Task清单 + Test Case Mapping |
| **Phase 3** | `/code-execute` | 主Agent + ImplementerAgent + SpecReviewerAgent + QualityReviewerAgent | tasks.md | `src/` + `execution-report.md` | **RED-GREEN-REFACTOR-REVIEW** | ⚙️ **TDD编码**：单元测试≥85%覆盖 |
| **Phase 4** | `/code-test` | 主Agent + IntegrationTestDesigner + E2ETestDesigner + PerformanceTestDesigner | src/ + tasks.md | `tests/` + `testing-report.md` | **集成/E2E/性能** | 🧪 **高层测试**：闭环验证TEST-VERIFY |
| **Phase 5** | `/spec-archive` | 主Agent | spec/ + tests/ | `spec-dev/spec/` | - | 📚 **规范沉淀**：归档到企业规范库 |

---

## 🚀 第一阶段：需求规范化（首次使用）

**目标**：将需求进行规范化描述，生成BDD格式的规范文档

### 操作步骤

```bash
/spec-creation
```

或带有需求描述：

```bash
/spec-creation 需要实现订单管理表单，支持搜索、排序、分页、批量操作
```

### AI工作流程

1. **需求理解** - 阅读需求描述，提出澄清问题
2. **场景拆解** - 将需求拆解为不同的业务场景
3. **规范细化** - 通过多轮交互确认，细化规范细节
4. **文档生成** - 生成BDD格式的规范文档

### 输出成果

生成 `spec-dev/{requirement_desc_abstract}/spec/` 目录，包含：

- **README.md** - 规范总览
- **scenarios/*.md** - BDD格式场景（WHEN-THEN格式 + TEST-VERIFY验收标准）✨新增
- **data-models.md** - 数据模型定义
- **business-rules.md** - 业务规则和约束
- **glossary.md** - 术语表

### 新增：TDD支持

规范阶段现已支持**TEST-VERIFY**格式，用于定义可测试的验收标准：

```markdown
### TEST-VERIFY (可测试的验收标准)
- [ ] 应该[动作]，[期望结果]
- [ ] 应该验证[字段/属性]为[值]
- [ ] 应该[行为]不应该[禁止行为]

### Mock Data (测试数据和边界值)

**有效输入示例**：
{
  "field1": "valid_value",
  "field2": 100
}

**边界值**：
- field2 最小值：0（应接受）
- field2 最大值：10000（应接受）
```

详见：[test-verify-template.md](../spec-creation/test-verify-template.md)

### 核心原则

- ✅ **规范优先** - 规范是设计和开发的基础
- ✅ **多轮确认** - 通过交互确认，确保规范完整
- ✅ **BDD格式** - 使用WHEN-THEN格式便于验证
- ✅ **可测试性** - 通过TEST-VERIFY定义可验证的验收标准（TDD支持）✨新增

---

## 🎨 第二阶段：代码设计规划

**目标**：根据规范进行代码设计分析，生成设计方案

### 操作步骤

```bash
/code-designer
```

前置条件：
- ✅ `/spec-creation` 已完成

### AI工作流程

1. **需求确认** - 读取规范，确认用户需求
2. **参考分析** - 分析参考组件的完整实现
3. **差异分析** - 比对需求和参考设计，识别差异
4. **方案生成** - 生成详细的设计方案

### 输出成果

生成 `spec-dev/{requirement_desc_abstract}/design/design.md`，包含：

- **需求分析** - 核心功能、用户场景
- **设计方案** - 架构、组件设计、状态管理、数据流、样式、交互
- **与参考设计的对比** - 相同点和差异点
- **技术方案** - 依赖库、API设计、性能考虑
- **设计决策记录** - 每个决策的原因

### 重要提示

- 🔴 **不要跳过设计阶段** - 设计是后续执行的基础
- ✅ **等待用户批准** - 设计方案需要用户确认后再进入Task阶段

---

## 📋 第三阶段：任务列表生成

**目标**：将设计方案转换为详细的代码级别任务列表，关联规范中的验收标准

### 操作步骤

```bash
/code-task
```

前置条件：
- ✅ `/code-designer` 已完成
- ✅ 用户已批准设计方案

### AI工作流程

1. **分析设计** - 读取design.md，理解设计方案
2. **任务分解** - 将设计分解为具体的编码任务
3. **任务定义** - 定义每个Task的目标、交付物、验收标准
4. **关联验收标准** - 为每个Task关联规范中的TEST-VERIFY ✨新增
5. **文档生成** - 生成完整的任务列表文档

### 输出成果

生成 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`，包含：

- **任务依赖关系** - 任务之间的依赖关系图
- **Task详情** - 每个Task的：
  - 目标和说明
  - 交付物清单
  - 依赖关系
  - 验收标准
  - 技术栈
  - 估时
  - **Test Case Mapping** - 验收标准 → 实现 → 测试用例的映射 ✨新增
  - **Mock数据关联** - 来自规范的测试数据 ✨新增
- **质量标准** - 代码质量、功能测试、设计一致性标准

### 新增：TDD支持

每个Task现已关联规范中的TEST-VERIFY验收标准，形成验证链：

```
规范的TEST-VERIFY (验收标准)
    ↓
Task的Test Case Mapping (映射表)
    ↓
后续test-design生成的测试用例
    ↓
code-execute执行时的验证
```

**Test Case Mapping 表格示例**：

```markdown
| # | 验收标准 | 来源 | 对应实现 | 测试ID |
|----|---------|------|--------|--------|
| 1 | 应该支持查询 | Case1#1 | src/api.ts:fetch() | TC-1.1.1 |
| 2 | 应该验证参数 | Case1#2 | src/api.ts:validate() | TC-1.1.2 |
| 3 | 边界：limit最小值0 | Case2 | src/api.ts:fetch() | TC-1.2.1 |
```

详见：[test-case-mapping-template.md](../code-task/test-case-mapping-template.md)

### 重要提示

- ✅ **任务要清晰** - 每个Task的目标、交付物、验收标准要明确
- ✅ **粒度要合适** - 每个Task在4小时内可完成
- ✅ **关联规范** - 每个Task都要关联对应的规范验收标准（TDD支持）✨新增
- ✅ **等待用户确认** - Task列表需要用户确认后再进入执行

---

## ⚙️ 第四阶段：代码执行实现（TDD流程）

**目标**：根据任务列表，通过TDD流程执行代码，进行多阶段审查，完成单元测试

### 操作步骤

```bash
/code-execute
```

前置条件：
- ✅ `/code-task` 已完成
- ✅ 用户已确认Task列表
- ✅ `/code-designer` 和 `/spec-creation` 已完成

### TDD流程（红-绿-重构-审查）

对每个Task，执行完整的TDD循环：

#### 🔴 RED阶段 - 编写失败的单元测试
1. 读取规范中的TEST-VERIFY验收标准
2. 基于test-spec.md生成的测试框架编写测试
3. 运行测试确保失败（Test fails）

#### 🟢 GREEN阶段 - 实现最少代码
1. 编写最少量的代码让测试通过
2. 运行测试确保通过（Test passes）
3. 确保其他测试也继续通过

#### 🔵 REFACTOR阶段 - 改进代码质量
1. 在测试通过的前提下重构代码
2. 提取公共逻辑、改进命名、增强可读性
3. 添加类型注解、错误处理
4. 重新运行测试确保仍然通过

#### ✅ REVIEW阶段 - 多阶段审查
1. **规范审查** - 检查代码是否符合 design.md 规范
2. **质量审查** - 检查代码质量（Lint、TypeScript、覆盖率）
3. 发现问题则修复并重新审查

### AI工作流程和Agent调用

**主Agent（code-execute）的职责**：
1. 分析tasks.md中的并行执行计划
2. 根据Task依赖关系，分批并行执行
3. 为每个Task分配专用的 **ImplementerAgent**
4. 执行后分配 **SpecReviewerAgent**（规范审查）
5. 执行后分配 **CodeQualityReviewerAgent**（质量审查）
6. 协调修复循环，直到Task通过两道审查
7. 生成执行报告

**Batch并行执行示例**：
```
Batch 1 (T1, T2, T3) - 无依赖，可并行
├── ImplementerAgent 1 处理 T1 (UserAuth)
├── ImplementerAgent 2 处理 T2 (FormValidation)
└── ImplementerAgent 3 处理 T3 (DataFetching)
    ↓ (等待Batch 1全部完成)
Batch 2 (T4, T5) - 依赖Batch 1，可并行
├── ImplementerAgent 4 处理 T4 (AuthenticatedFormSubmit)
└── ImplementerAgent 5 处理 T5 (DataDisplay)
    ↓ (等待Batch 2全部完成)
Batch 3 (T6, T7) - 依赖Batch 2，可并行
├── ImplementerAgent 6 处理 T6 (ErrorHandling)
└── ImplementerAgent 7 处理 T7 (Performance)
```

**对每个Task的执行流程**：

1. **ImplementerAgent处理**：
   - 执行RED阶段：编写单元测试 → 确保失败
   - 执行GREEN阶段：实现代码 → 确保测试通过
   - 执行REFACTOR阶段：改进代码 → 确保测试仍通过
   - 在worktree中提交各阶段代码

2. **SpecReviewerAgent处理（规范审查）**：
   - 检查代码是否符合 design.md 的架构规范
   - 检查Props/接口是否与设计一致
   - 检查功能实现是否完整
   - 输出规范审查报告
   - 如果失败 → ImplementerAgent修复 → 重新审查

3. **CodeQualityReviewerAgent处理（质量审查）**：
   - 运行ESLint检查（0个错误）
   - 运行TypeScript strict检查（0个any）
   - 检查代码复杂度（CC < 15）
   - 检查测试覆盖率（≥85%）
   - 输出质量审查报告
   - 如果失败 → ImplementerAgent改进 → 重新审查

4. **主Agent标记完成**：
   - 两道审查都通过 → Task完成
   - 将worktree的修改合并回main分支
   - 更新TodoList中的Task状态

### 执行流程细节

每个Task都要通过两道审查关卡：

#### 规范审查（Specification Review）
- 代码是否符合 design.md 的设计方案？
- 代码是否遵循规范中的技术方案？
- Props/接口是否与设计一致？
- 功能实现是否完整？

#### 质量审查（Code Quality Review）
- 代码是否符合项目编码规范？
- TypeScript strict模式是否通过（无any）？
- 是否有Lint错误或警告？
- 代码复杂度是否过高？
- 是否有性能问题？
- 测试覆盖率是否达到 ≥80%？

### 输出成果

生成以下文件：

- **源代码** - `src/` 目录下的所有实现文件
  - 组件文件
  - Hook/工具函数
  - 类型定义
  - **单元测试**（由TDD流程生成，覆盖率≥85%）
- **执行报告** - `spec-dev/{requirement_desc_abstract}/execution/execution-report.md`
  - 执行摘要（总Task数、完成数、迭代次数等）
  - 每个Task的执行情况和TDD流程结果
  - 测试覆盖率统计
  - 问题和修复记录
  - 最终审查结果

### 核心原则

- ✅ **执行严谨** - 两道审查都要通过
- ✅ **修复必须** - 发现问题必须修复，不能跳过
- ✅ **追踪记录** - 详细记录每个问题和修复

### 重要提示

- 🔴 **不能跳过规范审查** - 必须确保符合设计规范
- 🔴 **不能跳过质量审查** - 必须确保代码质量
- 🔴 **不能跳过修复循环** - 发现问题必须修复后重新审查
- ✅ **等待编译验证通过** - 代码要编译通过后再进入测试

---

## 🧪 第五阶段：高层测试和闭环验证（Phase 3）

**目标**：进行高层测试（集成、E2E、性能）和完整的闭环验证

### 操作步骤

```bash
/code-test
```

前置条件：
- ✅ `/code-execute` 已完成
- ✅ 代码已编译验证通过
- ✅ 单元测试已由code-execute的TDD流程完成（≥85%覆盖）

### AI工作流程和Agent调用

**主Agent（code-test）的职责**：
1. 读取code-execute生成的代码和执行报告
2. 进行代码质量审查（ESLint、TypeScript）
3. 验证code-execute生成的单元测试（不重复执行）
4. 分配专用的 **IntegrationTestDesigner**、**E2ETestDesigner**、**PerformanceTestDesigner** 分别设计和执行高层测试
5. 生成闭环验证矩阵
6. 生成完整的测试报告

**Agent分工**：

1. **主Agent（code-test）处理**：
   - 运行ESLint检查：`npm run lint` (0个错误)
   - 运行TypeScript检查：`npm run type-check` (0个错误)
   - 验证覆盖率：`npm run test:coverage` (≥85%)
   - 与design.md对比，检查代码架构一致性

2. **IntegrationTestDesigner处理（集成测试）**：
   - 分析Task之间的协作点
   - 设计集成测试用例
   - 实现集成测试代码（tests/integration/）
   - 运行集成测试，生成结果报告

3. **E2ETestDesigner处理（E2E测试）**：
   - 分析完整的业务流程（用户场景）
   - 使用Cypress/Playwright设计E2E测试
   - 实现E2E测试代码（tests/e2e/）
   - 运行E2E测试，生成结果报告

4. **PerformanceTestDesigner处理（性能测试）**：
   - 定义性能基准（P95、RPS等）
   - 设计性能测试用例
   - 使用K6/Artillery实现性能测试
   - 运行性能测试，建立基准报告

5. **主Agent生成闭环验证矩阵**：
   - 将规范的TEST-VERIFY映射到：
     - ✅ 代码实现（Task中的对应函数/组件）
     - ✅ 单元测试（code-execute生成）
     - ✅ 集成/E2E/性能测试（code-test生成）
     - ✅ 测试结果（全部通过）
   - 输出验证矩阵表格
   - 确保TEST-VERIFY → Test → Code → Result的完整对应

### 闭环验证详解

闭环验证是确保完整的追踪链条：

```
规范(TEST-VERIFY)
    ↓
测试设计(test-spec.md)
    ↓
单元测试(code-execute TDD完成)
    ↓
集成测试(code-test执行)
    ↓
E2E测试(code-test执行)
    ↓
性能测试(code-test执行)
    ↓
验证矩阵(TEST-VERIFY→Test→Code→Result)
    ↓
✅ 闭环完成
```

### 输出成果

生成以下文件：

- **测试文件** - `tests/` 目录下的所有测试文件
  - `unit/` - 单元测试（由code-execute生成）
  - `integration/` - 集成测试（code-test生成）
  - `e2e/` - E2E测试（code-test生成）
  - `performance/` - 性能测试（code-test生成）
- **测试报告** - `spec-dev/{requirement_desc_abstract}/testing/testing-report.md`
  - 代码审查结果
  - 单元测试结果和覆盖率（≥85%）
  - 集成测试结果
  - E2E测试结果
  - 性能测试结果和基准
  - **闭环验证矩阵** - TEST-VERIFY映射到Test→Code→Result
  - 发现的问题和建议

### 核心原则

- ✅ **单元测试完成** - 由code-execute的TDD流程保证（≥85%覆盖）
- ✅ **高层测试完整** - 集成 + E2E + 性能 全覆盖
- ✅ **覆盖充分** - 单元测试≥85% + 完整的集成/E2E/性能验证
- ✅ **验证闭环** - 确保TEST-VERIFY→Test→Code→Result完全对应
- ✅ **文档完善** - 详细的测试报告和验证矩阵便于后续维护

### 重要提示

- 🔴 **不能忽视单元测试** - 由code-execute的TDD流程完成，覆盖率≥85%
- 🔴 **不能忽视高层测试** - 集成、E2E、性能都要完成
- 🔴 **不能跳过闭环验证** - 必须确保TEST-VERIFY、Test、Code、Result四者对应
- 🔴 **不能为了通过测试而改代码逻辑** - 应该修复测试用例或完善代码
- ✅ **所有测试通过后才能上线** - 确保代码质量和系统可靠性

---

## ✅ 工作流完成

**恭喜！功能已完成** ✅

### 最终检查清单

- ✅ 规范生成完成（`spec-dev/{requirement_desc_abstract}/spec/`）
- ✅ 设计方案完成（`spec-dev/{requirement_desc_abstract}/design/design.md`）
- ✅ 任务列表完成（`spec-dev/{requirement_desc_abstract}/tasks/tasks.md`）
- ✅ 代码实现完成（`src/`）
  - ✅ 通过TDD流程（RED-GREEN-REFACTOR-REVIEW）
  - ✅ 单元测试覆盖率≥85%
  - ✅ 通过规范审查
  - ✅ 通过质量审查
- ✅ 测试完成（`tests/`）
  - ✅ 单元测试通过（由code-execute完成）
  - ✅ 集成测试通过
  - ✅ E2E测试通过
  - ✅ 性能测试通过
- ✅ 闭环验证通过（TEST-VERIFY→Test→Code→Result）

### 后续步骤

1. 📝 生成项目文档
2. 👥 进行人工代码审查（可选）
3. 🚀 部署到测试环境
4. 🧪 进行功能测试
5. 📦 发布到生产环境

---

## 📚 完整文档导航

- [README.md](../README.md) - 项目概述和快速开始
- [docs/USAGE.md](../docs/USAGE.md) - 详细使用指南和常见问题
- [docs/INSTALLATION.md](../docs/INSTALLATION.md) - 安装说明和故障排查
- [docs/BEST_PRACTICES.md](../docs/BEST_PRACTICES.md) - SDD最佳实践（包含Phase 3）
- [skills/README.md](../skills/README.md) - Skills详细说明
- [docs/使用案例.md](../docs/使用案例.md) - 真实项目案例研究
- **🆕 [docs/TDD_COMPLETE_WORKFLOW.md](../docs/TDD_COMPLETE_WORKFLOW.md)** - Phase 1-3完整TDD工作流 (5000+行)
- **🆕 [docs/TDD_PHASE3_EXAMPLE.md](../docs/TDD_PHASE3_EXAMPLE.md)** - Phase 3集成、E2E、性能实例 (6000+行)

---

## 🎓 使用建议

### 首次使用
1. 阅读 [README.md](../README.md) 了解整个工作流
2. 阅读 [docs/USAGE.md](../docs/USAGE.md) 学习详细使用方式
3. 执行 `/spec-creation` 开始你的第一个需求

### 深入学习（TDD工作流）
1. 阅读 [docs/TDD_COMPLETE_WORKFLOW.md](../docs/TDD_COMPLETE_WORKFLOW.md) 了解Phase 1-3完整工作流
2. 查看 [docs/TDD_PHASE3_EXAMPLE.md](../docs/TDD_PHASE3_EXAMPLE.md) 学习集成、E2E、性能测试

### 日常开发
1. 需求来临 → `/spec-creation` 规范化
2. 规范确认 → `/code-designer` 设计
3. 设计批准 → `/code-task` 任务分解
4. 任务确认 → `/code-execute` TDD实现（单元测试≥85%）
5. 编译通过 → `/code-test` 高层测试和闭环验证
6. ✅ 完成！代码质量达标，可以上线

---

## 🔑 核心原则总结

### ✅ 必须做

1. **首次使用前** - 运行 `/spec-creation` 生成项目规范
2. **充分规范化** - 通过多轮确认确保规范完整，定义TEST-VERIFY验收标准
3. **详细设计** - 生成足够详细的设计方案指导编码
4. **清晰任务** - 任务要有明确的目标和验收标准，关联规范的TEST-VERIFY
5. **TDD实现** - 所有Task都要通过RED-GREEN-REFACTOR-REVIEW的TDD流程，单元测试≥85%
6. **两道审查** - 所有Task都要通过规范审查和质量审查
7. **修复循环** - 发现问题必须修复后重新审查
8. **高层测试** - 完成集成、E2E、性能测试
9. **闭环验证** - 确保TEST-VERIFY、Test、Code、Result的完全对应

### ❌ 不能做

1. **跳过规范** - 不能在不初始化规范的情况下开始设计
2. **跳过设计** - 不能跳过 `/code-designer` 直接进行编码
3. **跳过TDD** - 不能跳过code-execute的TDD流程
4. **忽视单元测试** - 单元测试由TDD流程完成，覆盖率必须≥85%
5. **跳过审查** - 不能在 `/code-execute` 中跳过规范或质量审查
6. **忽视问题** - 不能发现问题后不修复就继续下一个Task
7. **改代码通过测试** - 不能为了通过测试而修改源代码逻辑
8. **跳过高层测试** - 集成、E2E、性能测试都不能跳过
9. **跳过闭环验证** - 不能忽视TEST-VERIFY、Test、Code、Result的对应关系

---

## 💡 快速参考

### 常用命令

| 命令 | 功能 | 前置条件 |
|------|------|---------|
| `/spec-creation [需求]` | 需求规范化 | 无 |
| `/code-designer` | 代码设计规划 | spec-creation ✓ |
| `/code-task` | 任务列表生成 | code-designer ✓ |
| `/code-execute` | **TDD实现**（RED-GREEN-REFACTOR-REVIEW + 单元测试≥85%） | code-task ✓ |
| `/code-test` | **Phase 3高层测试**（集成/E2E/性能 + 闭环验证） | code-execute ✓ |

### 输出文件位置

| 阶段 | 输出位置 |
|------|---------|
| Spec | `spec-dev/{requirement_desc_abstract}/spec/` |
| Design | `spec-dev/{requirement_desc_abstract}/design/design.md` |
| Task | `spec-dev/{requirement_desc_abstract}/tasks/tasks.md` |
| Execute（TDD） | `src/` + `spec-dev/{requirement_desc_abstract}/execution/execution-report.md` |
| Test（Phase 3） | `tests/` + `spec-dev/{requirement_desc_abstract}/testing/testing-report.md` |

---

**让AI辅助的全栈开发变得规范、高效、可信赖！通过Phase 1-3 TDD工作流，确保代码质量！** 🚀
