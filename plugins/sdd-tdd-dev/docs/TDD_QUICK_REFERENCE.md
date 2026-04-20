# TDD执行阶段快速参考

## 完整的TDD链条（6个Phase）

```
Phase 0          Phase 1          Phase 2          Phase 3                 Phase 4                 Phase 5
规范定义      →  架构设计      →  任务分解      →  TDD编码             →  高层测试            →  规范沉淀
/spec-creation  /code-designer  /code-task   /code-execute         /code-test            /spec-archive
    ↓              ↓              ↓              ↓                      ↓                      ↓
TEST-VERIFY    design.md       tasks.md      代码+单元测试≥85%      集成/E2E/性能+验证矩阵  企业规范库
```

---

## Phase 3: TDD编码（最复杂阶段）

### 命令
```bash
/code-execute
```

### 关键特性
- ✅ **TDD流程**：RED → GREEN → REFACTOR → REVIEW
- ✅ **多阶段审查**：规范审查 + 质量审查
- ✅ **并行执行**：根据Task依赖关系，分批并行处理
- ✅ **Git隔离**：每个Task在独立的worktree中工作

### 4类Agent分工

| Agent类型 | 职责 | 关键输出 | 数量 |
|----------|------|--------|------|
| **MainAgent** | 并行调度、审查协调、报告生成 | 执行报告 | 1 |
| **ImplementerAgent** | 编码、RED-GREEN-REFACTOR、在worktree中工作 | 代码+测试 | N（每Batch中Task数） |
| **SpecReviewerAgent** | 规范审查（检查design.md一致性） | 规范审查报告 | 1 |
| **CodeQualityReviewerAgent** | 质量审查（ESLint、TypeScript、覆盖率） | 质量审查报告 | 1 |

### 执行流程

#### 步骤1: MainAgent分析并行计划
```
读取tasks.md的并行执行计划
    ↓
分批：
├─ Batch 1: T1, T2, T3（无依赖）
├─ Batch 2: T4, T5（依赖Batch 1）
└─ Batch 3: T6（依赖Batch 2）
```

#### 步骤2: 第1批并行执行（3个ImplementerAgent并行）
```
Batch 1开始
├─ ImplementerAgent-1 处理 T1 (UserAuth)
│   ├─ RED: 编写单元测试 → 失败
│   ├─ GREEN: 实现代码 → 通过
│   └─ REFACTOR: 改进代码 → 测试仍通过
│
├─ ImplementerAgent-2 处理 T2 (FormValidation)
│   ├─ RED: 编写单元测试 → 失败
│   ├─ GREEN: 实现代码 → 通过
│   └─ REFACTOR: 改进代码 → 测试仍通过
│
└─ ImplementerAgent-3 处理 T3 (DataFetching)
    ├─ RED: 编写单元测试 → 失败
    ├─ GREEN: 实现代码 → 通过
    └─ REFACTOR: 改进代码 → 测试仍通过

                    ↓ (Batch 1完成后)

每个Task进行REVIEW阶段 (可并行)
├─ SpecReviewerAgent 检查 T1, T2, T3 是否符合design.md
├─ CodeQualityReviewerAgent 检查 T1, T2, T3 的代码质量
└─ 如果失败 → ImplementerAgent修复 → 重新审查
```

#### 步骤3: 第2批并行执行（等待Batch 1完成后）
```
Batch 2开始（依赖Batch 1）
├─ ImplementerAgent-4 处理 T4 (AuthenticatedFormSubmit)
└─ ImplementerAgent-5 处理 T5 (DataDisplay)

                    ↓ (Batch 2完成后)

REVIEW阶段: T4, T5 的规范+质量审查
```

#### 步骤4: 第3批并行执行（等待Batch 2完成后）
```
Batch 3开始（依赖Batch 2）
└─ ImplementerAgent-6 处理 T6 (ErrorHandling)

                    ↓ (Batch 3完成后)

REVIEW阶段: T6 的规范+质量审查

                    ↓ (所有Task完成)

MainAgent生成执行报告
```

### 每个Task的TDD循环

```
1️⃣ RED: 编写失败的单元测试
   └─ 基于规范的TEST-VERIFY编写测试
   └─ 基于Test Case Mapping编写测试用例
   └─ 运行测试 → 所有测试失败 ❌

                    ↓

2️⃣ GREEN: 实现最少代码让测试通过
   └─ 编写最简单的实现代码
   └─ 运行测试 → 所有测试通过 ✅

                    ↓

3️⃣ REFACTOR: 改进代码质量（测试通过前提）
   └─ 改进代码可读性
   └─ 完善类型注解和错误处理
   └─ 提取公共逻辑
   └─ 运行测试 → 所有测试仍通过 ✅

                    ↓

4️⃣ REVIEW: 两阶段审查
   ├─ 规范审查 (SpecReviewerAgent)
   │  ├─ 检查: 代码是否符合design.md？
   │  ├─ 检查: Props/接口是否一致？
   │  ├─ 检查: 功能实现是否完整？
   │  └─ 结果: PASS ✅ 或 FAIL ❌
   │
   └─ 质量审查 (CodeQualityReviewerAgent)
      ├─ 检查: ESLint = 0个错误
      ├─ 检查: TypeScript strict = 0个any
      ├─ 检查: 覆盖率 ≥ 85%
      └─ 结果: PASS ✅ 或 FAIL ❌

   如果有FAIL → ImplementerAgent修复 → 重新REVIEW
              ↻ 直到两次审查都PASS ✅
```

### 工作上下文：Git-Worktree隔离

每个Task在独立的worktree中工作：

```bash
# ImplementerAgent为Task T1创建worktree
git worktree add .claude/worktrees/T1-UserAuth main
cd .claude/worktrees/T1-UserAuth

# 在worktree中进行RED-GREEN-REFACTOR
git commit -m "test(T1): 编写单元测试"
git commit -m "feat(T1): 实现基本功能"
git commit -m "refactor(T1): 改进代码质量"

# Task完成后，合并回main
cd <project-root>
git cherry-pick <T1-commits>
# 或
git merge --squash .claude/worktrees/T1-UserAuth
git commit -m "feat(T1): 实现用户认证功能"
```

---

## Phase 4: 高层测试（验证阶段）

### 命令
```bash
/code-test
```

### 4类Agent分工

| Agent类型 | 职责 | 关键输出 |
|----------|------|--------|
| **MainAgent** | 代码质量检查、协调、报告生成 | 测试报告 |
| **IntegrationTestDesigner** | 集成测试设计和执行 | 集成测试代码+结果 |
| **E2ETestDesigner** | E2E测试设计和执行 | E2E测试代码+结果 |
| **PerformanceTestDesigner** | 性能基准设计和执行 | 性能测试代码+基准 |

### 执行流程

```
Phase 4: 高层测试（注意：不重复单元测试）
    ↓
1️⃣ 代码质量检查 (MainAgent)
   ├─ npm run lint → 0个错误
   ├─ npm run type-check → 0个错误
   └─ npm run test:coverage → ≥85%覆盖率

                    ↓

2️⃣ 集成测试 (IntegrationTestDesigner)
   ├─ 测试Task之间的协作（如T1+T2）
   ├─ Mock外部依赖，真实运行业务逻辑
   └─ 运行集成测试 → 全部通过 ✅

                    ↓

3️⃣ E2E测试 (E2ETestDesigner)
   ├─ 测试完整的用户业务流程
   ├─ 使用Cypress或Playwright
   └─ 运行E2E测试 → 全部通过 ✅

                    ↓

4️⃣ 性能测试 (PerformanceTestDesigner)
   ├─ 建立性能基准（P95响应时间、吞吐量等）
   ├─ 使用K6或Artillery
   └─ 运行性能测试 → 所有基准达标 ✅

                    ↓

5️⃣ 闭环验证矩阵 (MainAgent)
   └─ 验证: TEST-VERIFY → Code → Test → Result
   └─ 覆盖率: 100% (所有验收标准都有验证)
   └─ 输出验证矩阵表格
```

### 闭环验证矩阵示例

```markdown
| 规范验收标准 | Task | 代码实现 | 单元测试 | 集成测试 | E2E测试 | 性能测试 | 结果 |
|-----------|------|--------|---------|---------|---------|---------|------|
| 支持用户名密码认证 | T1 | src/auth.ts | TC-1.1 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ 通过 |
| 验证用户名不为空 | T1 | src/auth.ts | TC-1.2 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ 通过 |
| 验证表单数据 | T2 | src/form.ts | TC-2.1 ✅ | IntegrationTest-2 ✅ | E2ETest-2 ✅ | - | ✅ 通过 |
| 响应时间 < 500ms | T1 | src/auth.ts | - | - | - | PerfTest-1 ✅ | ✅ 通过 |

验证覆盖率: 18/18 TEST-VERIFY ✅ 100%
```

---

## 关键约束和原则

### ✅ 必须做（Phase 3）

- ✅ 严格遵循RED-GREEN-REFACTOR-REVIEW流程
- ✅ 两道审查都要通过（规范 + 质量）
- ✅ 发现问题必须修复后重新审查
- ✅ 单元测试覆盖率≥85%
- ✅ 在worktree中工作，保持隔离和可追踪

### ❌ 禁止做（Phase 3）

- ❌ 跳过RED阶段直接实现代码
- ❌ 跳过REFACTOR阶段
- ❌ 跳过规范审查
- ❌ 跳过质量审查
- ❌ 发现问题不修复就标记完成
- ❌ 为了让测试通过而修改源代码逻辑（应该修改测试）

### ✅ 必须做（Phase 4）

- ✅ 验证code-execute的单元测试覆盖率
- ✅ 完成集成、E2E、性能测试
- ✅ 生成闭环验证矩阵
- ✅ 确保TEST-VERIFY → Code → Test → Result的完整对应

### ❌ 禁止做（Phase 4）

- ❌ 不重复执行单元测试（由code-execute保证）
- ❌ 跳过高层测试（集成/E2E/性能）
- ❌ 忽视闭环验证
- ❌ 为了通过测试而修改代码逻辑

---

## 相关文档

- **详细说明**: [TDD_EXECUTION_PHASES.md](./TDD_EXECUTION_PHASES.md) - 2000+行的完整阶段说明
- **命令文档**: [../commands/sdd-dev.md](../commands/sdd-dev.md) - 完整的工作流和操作指南
- **完整工作流**: [TDD_COMPLETE_WORKFLOW.md](./TDD_COMPLETE_WORKFLOW.md) - 5000+行的Phase 1-3完整示例
- **Phase 3实例**: [TDD_PHASE3_EXAMPLE.md](./TDD_PHASE3_EXAMPLE.md) - 6000+行的集成、E2E、性能测试实例
- **Git-Worktrees指南**: [../skills/code-execute/git-worktrees-guide.md](../skills/code-execute/git-worktrees-guide.md) - 隔离工作环境详细指南

---

**记住**: 整个TDD工作流是从规范定义（TEST-VERIFY）开始，通过代码实现→单元测试→集成/E2E/性能测试，最后通过闭环验证矩阵确保所有验收标准都被满足的完整链条。✅
