# SDD-TDD完整架构说明

## 📋 概述

本文档总结了sdd-dev-plugin中**TDD（测试驱动开发）执行的完整架构**，包括：
- 6个Phase的完整链条
- Phase 3（代码执行）的Agent分工和并行调度
- Phase 4（高层测试）的Agent分工和闭环验证

---

## 🎯 完整的TDD链条（6个Phase）

### 从规范到验证的完整流程

```
Phase 0             Phase 1             Phase 2             Phase 3              Phase 4              Phase 5
规范定义         架构设计          任务分解          TDD编码            高层测试           规范沉淀
(Spec)           (Design)          (Task)            (Execute)           (Test)             (Archive)
   ↓                ↓                  ↓                 ↓                  ↓                  ↓
/spec-creation  /code-designer  /code-task      /code-execute        /code-test       /spec-archive
   ↓                ↓                  ↓                 ↓                  ↓                  ↓
TEST-VERIFY     design.md       tasks.md        代码+单元测试≥85%   集成/E2E/性能    企业规范库
                                              +规范审查+质量审查   +闭环验证
```

---

## 📊 Phase对照表

| Phase | 名称 | Skill | 关键职责 | Agent类型 | 主要输出 | 核心指标 |
|-------|------|-------|--------|----------|---------|---------|
| **0** | 规范定义 | `/spec-creation` | 需求→BDD规范 | MainAgent | TEST-VERIFY验收标准 | 所有场景 |
| **1** | 架构设计 | `/code-designer` | 规范→设计方案 | MainAgent | design.md | 完整设计 |
| **2** | 任务分解 | `/code-task` | 设计→Task清单 | MainAgent | tasks.md + Test Case Mapping | 清晰依赖 |
| **3** | TDD编码 | `/code-execute` | Task→代码+单元测试 | 4类Agent | src/ + 单元测试 | 覆盖率≥85% |
| **4** | 高层测试 | `/code-test` | 代码→集成/E2E/性能 | 4类Agent | tests/ + 闭环矩阵 | 验证覆盖100% |
| **5** | 规范沉淀 | `/spec-archive` | 规范→企业库 | MainAgent | 企业规范库 | 规范入库 |

---

## ⚙️ Phase 3: TDD编码（核心阶段）

### 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                    /code-execute（主Skill）                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  MainAgent (1个)                                                    │
│  ├─ 分析tasks.md的并行执行计划                                      │
│  ├─ 根据Task依赖关系分批：Batch1 → Batch2 → Batch3                 │
│  ├─ 为每个Batch中的Task分配ImplementerAgent                        │
│  ├─ 协调规范审查和质量审查                                          │
│  └─ 生成执行报告                                                   │
│                                                                       │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ Batch 1: T1, T2, T3（无依赖，并行）                 │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ ImplementerAgent-1 (T1)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  │ ImplementerAgent-2 (T2)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  │ ImplementerAgent-3 (T3)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  └──────────────────────────────────────────────────────┘          │
│                         ↓                                           │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ REVIEW阶段（可并行）                                 │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ SpecReviewerAgent      检查T1, T2, T3 ↔ design.md  │          │
│  │ CodeQualityReviewerAgent 检查T1, T2, T3代码质量    │          │
│  │ 如果FAIL → ImplementerAgent修复 → 重新REVIEW       │          │
│  └──────────────────────────────────────────────────────┘          │
│                         ↓                                           │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ Batch 2: T4, T5（依赖Batch 1，并行）                │          │
│  │ ImplementerAgent-4 (T4)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  │ ImplementerAgent-5 (T5)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  └──────────────────────────────────────────────────────┘          │
│                         ↓                                           │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ Batch 3: T6（依赖Batch 2）                          │          │
│  │ ImplementerAgent-6 (T6)    RED → GREEN → REFACTOR   │ ⟲ ⟲ ⟲   │
│  └──────────────────────────────────────────────────────┘          │
│                         ↓                                           │
│                   所有Task完成 ✅                                   │
│                   生成执行报告                                      │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Agent分工详表

| Agent | 职责 | 工作内容 | 数量 | 协作方式 |
|-------|------|--------|------|---------|
| **MainAgent** | 任务调度、审查协调 | 分析并行计划、分配ImplementerAgent、协调两次审查、生成报告 | 1 | 顺序调度其他Agent |
| **ImplementerAgent** | 编码实现、TDD流程 | RED→GREEN→REFACTOR，在worktree中提交代码 | N（每Batch Task数） | 并行执行 |
| **SpecReviewerAgent** | 规范审查 | 检查design.md一致性、Props一致性、功能完整性 | 1 | 待ImplementerAgent完成后执行 |
| **CodeQualityReviewerAgent** | 质量审查 | 检查ESLint、TypeScript、覆盖率 | 1 | 待规范审查通过后执行 |

### TDD循环详解

#### 第1个Task的完整TDD循环

```
Task T1: UserAuth
│
├─ 1️⃣ RED阶段 (ImplementerAgent-1)
│  ├─ 读取规范中的TEST-VERIFY
│  ├─ 读取Test Case Mapping表
│  ├─ 编写单元测试代码
│  ├─ 运行测试 → ❌ 所有测试失败
│  └─ 提交: git commit -m "test(T1): 编写T1的单元测试"
│
├─ 2️⃣ GREEN阶段 (ImplementerAgent-1)
│  ├─ 编写最少代码让测试通过
│  ├─ 运行测试 → ✅ 所有测试通过
│  └─ 提交: git commit -m "feat(T1): 实现T1的基本功能"
│
├─ 3️⃣ REFACTOR阶段 (ImplementerAgent-1)
│  ├─ 改进代码可读性
│  ├─ 完善类型注解和错误处理
│  ├─ 提取公共逻辑
│  ├─ 运行测试 → ✅ 所有测试仍通过
│  └─ 提交: git commit -m "refactor(T1): 改进T1的代码质量"
│
├─ 4️⃣ REVIEW阶段
│  │
│  ├─ 规范审查 (SpecReviewerAgent)
│  │  ├─ 检查: 代码是否符合design.md?
│  │  ├─ 检查: Props/接口是否一致?
│  │  ├─ 检查: 功能实现是否完整?
│  │  ├─ 结果: ✅ PASS 或 ❌ FAIL
│  │  └─ 如果FAIL → ImplementerAgent修复 → 重新审查
│  │
│  ├─ 质量审查 (CodeQualityReviewerAgent)
│  │  ├─ 检查: ESLint = 0个错误
│  │  ├─ 检查: TypeScript strict = 0个any
│  │  ├─ 检查: 覆盖率 ≥ 85%
│  │  ├─ 结果: ✅ PASS 或 ❌ FAIL
│  │  └─ 如果FAIL → ImplementerAgent改进 → 重新审查
│  │
│  └─ 修复循环（如果审查失败）
│     ├─ ImplementerAgent修复问题
│     ├─ 重新运行测试
│     ├─ 提交: git commit -m "fix(T1): 修复xxx问题"
│     └─ ↻ 直到两次审查都通过 ✅
│
└─ 🏁 Task T1完成
   ├─ 合并到main分支
   ├─ 输出：src/auth.ts + tests/auth.test.ts
   └─ 覆盖率：92%（> 85%要求）
```

---

## 🧪 Phase 4: 高层测试（验证阶段）

### 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                      /code-test（主Skill）                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  MainAgent (1个)                                                    │
│  ├─ 代码质量检查：ESLint、TypeScript、覆盖率                         │
│  ├─ 协调3类测试Designer                                             │
│  ├─ 生成闭环验证矩阵                                                │
│  └─ 生成完整的测试报告                                              │
│                                                                       │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ IntegrationTestDesigner                              │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ 设计集成测试：                                        │          │
│  │ ├─ IntegrationTest-1: T1(UserAuth) + T2(FormVal)   │          │
│  │ ├─ IntegrationTest-2: T3(DataFetch) + T5(DataDisp) │          │
│  │ └─ 生成tests/integration/*.test.ts                 │          │
│  │ 运行集成测试 → ✅ 全部通过                          │          │
│  └──────────────────────────────────────────────────────┘          │
│                                                                       │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ E2ETestDesigner                                      │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ 设计E2E测试：                                        │          │
│  │ ├─ E2ETest-1: 用户登录→仪表板加载                   │          │
│  │ ├─ E2ETest-2: 表单提交→数据保存                     │          │
│  │ └─ 生成tests/e2e/*.e2e.ts (Cypress/Playwright)     │          │
│  │ 运行E2E测试 → ✅ 全部通过                           │          │
│  └──────────────────────────────────────────────────────┘          │
│                                                                       │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ PerformanceTestDesigner                              │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ 设计性能测试：                                        │          │
│  │ ├─ 基准1: 认证响应时间 P95 < 500ms                 │          │
│  │ ├─ 基准2: 数据加载时间 < 2s                        │          │
│  │ └─ 生成tests/performance/*.k6.js                   │          │
│  │ 运行性能测试 → ✅ 所有基准达标                      │          │
│  └──────────────────────────────────────────────────────┘          │
│                                                                       │
│  ┌──────────────────────────────────────────────────────┐          │
│  │ 闭环验证矩阵 (MainAgent)                            │          │
│  ├──────────────────────────────────────────────────────┤          │
│  │ TEST-VERIFY → Code → Test → Result 的完整对应      │          │
│  │ ├─ "应该支持登录" → T1 → TC-1.1 → 单元+E2E ✅    │          │
│  │ ├─ "应该验证表单" → T2 → TC-2.1 → 单元+集成 ✅    │          │
│  │ └─ ... (18个验收标准全覆盖)                         │          │
│  │ 验证覆盖率: 100% (18/18) ✅                         │          │
│  └──────────────────────────────────────────────────────┘          │
│                                                                       │
│                  生成完整的测试报告                                  │
│                  ✅ 所有测试通过，可以上线！                        │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### 闭环验证矩阵示例

```markdown
从规范到验证的完整链条：

| # | 规范TEST-VERIFY | Task | 代码实现 | 单元测试 | 集成测试 | E2E测试 | 性能测试 | 结果 |
|----|-----------------|------|--------|---------|---------|---------|---------|------|
| 1 | 应该支持用户名密码认证 | T1 | src/auth.ts:authenticate() | TC-1.1 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ |
| 2 | 应该验证用户名不为空 | T1 | src/auth.ts:validateCredentials() | TC-1.2 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ |
| 3 | 应该验证表单数据 | T2 | src/form.ts:validateForm() | TC-2.1 ✅ | IntegrationTest-2 ✅ | E2ETest-2 ✅ | - | ✅ |
| 4 | 应该显示错误消息 | T3 | src/components/ErrorDisplay.tsx | TC-3.1 ✅ | IntegrationTest-2 ✅ | E2ETest-2 ✅ | - | ✅ |
| 5 | 认证响应时间 < 500ms | T1 | src/auth.ts | - | - | - | PerfTest-1 ✅ | ✅ |

验证覆盖率：18/18 验收标准 = 100% ✅
```

---

## 🔄 关键概念

### Test Case Mapping（Phase 2中建立）

```
规范TEST-VERIFY (Phase 0)
    ↓ (Phase 2中)
Task T1 → Test Case Mapping 表格
    ↓ (映射)
TC-1.1: "应该支持登录" → src/auth.ts:authenticate()
TC-1.2: "应该验证参数" → src/auth.ts:validate()
TC-1.3: "边界：最小值0" → src/auth.ts:validate()
    ↓ (Phase 3中)
编写单元测试、实现代码、运行测试 ✅
    ↓ (Phase 4中)
验证矩阵：TC-1.1 通过 ✅、TC-1.2 通过 ✅、...
```

### Git-Worktree隔离（Phase 3中）

```
每个Task在独立的worktree中工作
    ↓
T1在 .claude/worktrees/T1-UserAuth 中
    ├─ 提交1: test(T1): 编写单元测试
    ├─ 提交2: feat(T1): 实现功能
    ├─ 提交3: refactor(T1): 改进质量
    └─ （修复阶段如果有问题）
        ├─ 提交4: fix(T1): 修复问题1
        ├─ 提交5: fix(T1): 修复问题2
        └─ ...
    ↓ (Task完成后)
合并回main分支
    ├─ cherry-pick: 将所有commit合并
    ├─ 或 squash merge: 将所有commit合并为1个
    └─ 清理worktree
```

---

## 📍 核心约束

### Phase 3（TDD编码）的约束

✅ **必须做**：
- 严格遵循RED-GREEN-REFACTOR-REVIEW流程
- 两道审查（规范 + 质量）都要通过
- 发现问题必须修复后重新审查
- 单元测试覆盖率≥85%
- 在worktree中工作，保持隔离

❌ **禁止做**：
- 跳过RED阶段直接实现
- 跳过REFACTOR阶段
- 跳过任何一道审查
- 发现问题不修复就标记完成
- 为了让测试通过而修改源代码逻辑

### Phase 4（高层测试）的约束

✅ **必须做**：
- 验证code-execute的单元测试覆盖率
- 完成集成、E2E、性能测试
- 生成闭环验证矩阵
- 确保TEST-VERIFY → Code → Test → Result完整对应

❌ **禁止做**：
- 不重复执行单元测试（由code-execute保证）
- 跳过高层测试
- 忽视闭环验证

---

## 📚 相关文档导航

| 文档 | 重点 | 行数 |
|------|------|------|
| [TDD_EXECUTION_PHASES.md](./TDD_EXECUTION_PHASES.md) | Phase 0-5完整说明、工作流细节 | 2000+ |
| [TDD_QUICK_REFERENCE.md](./TDD_QUICK_REFERENCE.md) | 快速查询、图示、流程图 | 400+ |
| [TDD_COMPLETE_WORKFLOW.md](./TDD_COMPLETE_WORKFLOW.md) | Phase 1-3完整示例 | 5000+ |
| [TDD_PHASE3_EXAMPLE.md](./TDD_PHASE3_EXAMPLE.md) | Phase 3详细实例 | 6000+ |
| [../commands/sdd-dev.md](../commands/sdd-dev.md) | 命令行操作指南 | 500+ |

---

**让AI辅助的研发开发变得规范、高效、可信赖！通过Phase 0-5的完整TDD工作流，确保代码质量和交付物的一致性！** 🚀
