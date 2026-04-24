---
name: code-execute
description: |
  第4步：代码执行实现（Execute阶段）

  输入：spec-dev/{requirement_desc_abstract}/tasks/tasks.md
  输出：src/ + spec-dev/{requirement_desc_abstract}/execution/execution-report.md

  功能：根据任务列表和设计，通过子代理逐个实现代码，并进行多阶段审查（规范/质量）。
  采用git-worktree隔离每个Task，支持全栈开发的各类代码实现。

  核心特性：
  - ✅ 每Task独立git-worktree（隔离、可追踪、可恢复）
  - ✅ 模式标签读取和严格遵循（standard/quick）
  - ✅ 测试环境预检（HARD-GATE拦截）
  - ✅ 子代理强制派遣（standard模式必须）
  - ✅ TDD四阶段状态追踪（RED→GREEN→REFACTOR→REVIEW）
  - ✅ 两阶段审查：规范审查 + 质量审查
  - ✅ 执行完成前自检清单
  - ✅ 详细执行报告和修复循环
  - ✅ 支持多种技术栈（前后端、数据库、微服务）

  前置Skill：code-task ✓ + code-designer ✓ + spec-creation ✓
  下一步：/code-test
---

# code-execute

## 职责

根据任务列表，通过分配独立子代理逐个实现代码任务，并进行多阶段质量审查。

**核心流程**：
1. 分析tasks.md，制定并行执行计划
2. 验证测试基础设施和环境 ⭐ **新增**
3. 逐批执行Task（每Task分配独立子代理 + 隔离worktree）
4. TDD实现：单元测试 + 浏览器测试（前端Task）
5. 两阶段审查：规范审查 → 质量审查
6. 修复问题后重新审查，直到通过
7. 执行完成前自检 ⭐ **新增**
8. 输出执行报告

**输出路径**：`spec-dev/{requirement_desc_abstract}/execution/execution-report.md`

## 何时使用

**前置条件**：
- ✅ spec-creation 已完成，设计规范已生成
- ✅ code-designer 已完成，设计方案已确认（标准模式）
- ✅ code-task 已完成，任务列表已定义（标准模式）
- ✅ 所有关键决策已在前面阶段确认
- ✅ **快速模式**：spec 已生成，可直接读取 spec 开始编码（跳过 design 和 task）

### 快速模式执行（当 spec 标注为快速模式时）

**前提**：模式标签必须在 `requirement.md` 中明确标注为 `quick`。

在快速模式下，code-execute 执行以下精简流程：

1. **直接读取 spec**：从 `spec-dev/{req_id}/spec/` 读取场景和验收标准
2. **自主 Task 拆解**：在内存中拆解实现步骤（不生成 tasks.md）
3. **单阶段审查**：只做质量审查（跳过规范审查，因为 design 不存在）
4. **精简测试**：
   - ✅ 必须：单元测试（覆盖率 ≥ 60%）
   - ✅ 必须：浏览器 E2E 测试（前端场景）
   - ⚡ 跳过：视觉回归测试、组件 UI 测试、性能测试
5. **子代理可选**：用户可选择使用或直接在主上下文执行
6. **TDD 日志可选**：可简化为精简版记录

```
快速模式执行流程：
读取模式标签 → 读取 spec → 自主拆解 → 编码（可选TDD）→ 质量审查 → 修复循环 → 自检 → 生成报告
```

**注意**：快速模式仍需执行步骤1.5（测试环境验证）和执行完成前自检。

---

## 工作流程

### 步骤0: 读取模式标签 ⭐ **新增**

读取 `spec-dev/{requirement_desc_abstract}/spec/requirement.md` 头部的工作模式标签：

```markdown
## 工作模式
- 模式：[standard | quick]
- TDD要求：[必须执行 | 跳过]
- 子代理要求：[必须使用 | 可选]
- 测试覆盖要求：[≥85% | ≥60% | 跳过]
```

根据模式标签执行不同流程：

| 字段 | standard 模式 | quick 模式 |
|------|--------------|-----------|
| TDD要求 | 必须执行 | 可跳过 |
| 子代理要求 | 必须使用 | 可选 |
| 测试覆盖要求 | ≥85% | ≥60% |
| 审查阶段 | 两阶段（规范+质量） | 单阶段（仅质量） |
| 测试类型 | 单元+E2E+视觉+组件+性能 | 单元+E2E |

<HARD-GATE>
未读取到模式标签 → 暂停，使用 AskUserQuestion 确认工作模式
模式标签为 standard → 必须严格执行TDD和子代理，不允许降级
</HARD-GATE>

---

### 步骤1: 分析任务列表和并行计划

从tasks.md中提取：
- 所有Task信息（名称、目标、交付物、验收标准）
- Task之间的依赖关系
- 并行执行计划（按批次分组）

创建TodoList，列出所有Task和其完成状态。

---

### 步骤1.5: 测试环境验证 ⭐ **新增**

<HARD-GATE>
未确认测试框架状态 → 不允许开始编码
标准模式下未编写测试文件 → 不允许编写实现代码
</HARD-GATE>

在开始编码前，**必须**执行以下检查：

#### 1.5.1 测试框架可用性检查

```bash
# 检查项目是否有测试依赖
npm list jest vitest @vue/test-utils @playwright/test 2>&1
# 或
grep -E '"jest"|"vitest"|"@vue/test-utils"' package.json
# 或 Python 项目
pip list | grep -E 'pytest|unittest'
# 或 Go 项目
go list -m all | grep testify
```

**检查结果处理**：

| 结果 | 处理 |
|------|------|
| 有测试框架 | 继续下一步 |
| 无测试框架 | ⚠️ 暂停，使用 AskUserQuestion 确认 |

**AskUserQuestion 模板**：

```
question: "项目当前无测试基础设施，是否要新增测试框架？"
header: "测试框架"
multiSelect: false
options:
  - { label: "新增 Jest", description: "适合React/Vue项目的成熟测试框架" }
  - { label: "新增 Vitest", description: "基于Vite的现代测试框架（推荐）" }
  - { label: "新增 Pytest", description: "Python项目标准测试框架" }
  - { label: "跳过测试", description: "⚠️ 仅quick模式可用，需确认" }
```

#### 1.5.2 测试文件存在性检查

```bash
# 检查是否存在测试文件或测试模板
ls **/*.test.{ts,js,tsx,jsx} **/*.spec.{ts,js,tsx,jsx} 2>&1
ls spec-dev/{requirement_desc_abstract}/tests/test-*.template 2>&1
```

**检查结果处理**：

| 模式 | 测试文件不存在 |
|------|---------------|
| standard | ❌ 暂停，必须先写测试（RED阶段） |
| quick | ⚠️ 使用 AskUserQuestion 确认是否跳过 |

#### 1.5.3 工作模式最终确认

在开始编码前，**必须**使用 AskUserQuestion 确认工作模式：

```
question: "请选择code-execute执行模式？"
header: "执行模式"
multiSelect: false
options:
  - {
      label: "标准模式（推荐）",
      description: "TDD + 子代理 + 两阶段审查 + 完整测试"
    }
  - {
      label: "快速模式",
      description: "自主拆解 + 单阶段审查 + 精简测试"
    }
  - {
      label: "仅编码",
      description: "⚠️ 跳过TDD和测试，仅编写实现代码"
    }
```

**模式锁定后不可更改**。后续所有Task必须遵循所选模式的要求。

---

### 步骤2: 并行执行Task批次

根据依赖关系，逐批执行Task：
- **每个批次**：无依赖的Task可以并行执行
- **不同批次**：必须等待前一批次完成再开始
- **每个Task**：分配独立的实现子代理（见下方强制要求）

<HARD-GATE>
standard 模式下，每个 Task **必须**使用子代理执行，不允许直接在主上下文编码。
</HARD-GATE>

#### Task 执行方式（二选一，不可跳过）

**方式1 [推荐，standard模式必须]**：分配独立的 implementer 子代理在 worktree 中执行

```
必须调用: Agent(subagent_type="sdd-tdd-dev:code-executor", prompt="...", ...)
```

子代理职责：
- 在独立 worktree 中工作
- 遵循 TDD 流程（RED → GREEN → REFACTOR → REVIEW）
- 编写单元测试和浏览器测试
- 进行自审查

**方式2 [快速模式可选]**：用户明确同意后，可在主上下文执行

必须满足以下条件：
- 用户明确同意（使用 AskUserQuestion 确认）
- 仍须遵循 TDD 流程
- 在标准模式下**不允许**使用此方式

---

### 步骤3: 执行单个Task

对于每个分配的Task，按以下流程执行：

#### 3.1 创建隔离工作环境

**第1步：向用户确认worktree基础分支** ✅

在创建worktree前，**必须**使用 `AskUserQuestion` 工具向用户明确确认从哪个分支生成worktree：

```
使用 AskUserQuestion 工具确认分支选择：

question: "请选择Worktree创建时的基础分支？"
header: "Worktree基础分支"
multiSelect: false
options:
  - {
      label: "当前分支（推荐）",
      description: "feature/fix_sdd_dev_plugin_0320 - 包含当前分支的所有改动"
    }
  - {
      label: "Master分支",
      description: "master - 基于生产稳定版本"
    }
  - {
      label: "Develop分支",
      description: "develop - 基于最新开发版本"
    }
  - {
      label: "其他分支",
      description: "输入自定义分支名称"
    }
```

**重要原则**：
- ✅ **必须**：使用 AskUserQuestion 工具（统一的交互方式）
- ❌ **禁止**：直接假设用户想要master分支
- ❌ **禁止**：不询问用户，直接创建worktree
- ✅ **兜底**：如果用户未明确选择，使用当前分支（HEAD）

**第2步：根据用户选择执行**

```bash
# 用户选择 [A]：当前分支（推荐）
$ git worktree add .claude/worktrees/{task-id}-{task-name} HEAD
$ cd .claude/worktrees/{task-id}-{task-name}

# 用户选择 [B]：Master分支
$ git worktree add .claude/worktrees/{task-id}-{task-name} master
$ cd .claude/worktrees/{task-id}-{task-name}

# 用户选择 [C]：Develop分支
$ git worktree add .claude/worktrees/{task-id}-{task-name} develop
$ cd .claude/worktrees/{task-id}-{task-name}

# 用户选择 [D]：其他分支
$ git worktree add .claude/worktrees/{task-id}-{task-name} {user-selected-branch}
$ cd .claude/worktrees/{task-id}-{task-name}

# 兜底：用户未明确选择 → 使用HEAD（当前分支）
$ git worktree add .claude/worktrees/{task-id}-{task-name} HEAD
$ cd .claude/worktrees/{task-id}-{task-name}
```

**worktree的优势**：
- ✅ 完全隔离 - Task间互不污染
- ✅ 安全修复 - 修复失败可直接删除worktree重新开始
- ✅ 清晰审计 - commit历史清晰记录修复过程
- ✅ 并行安全 - 多Task同时进行不产生git冲突

详见：`references/git-worktrees-guide.md` 和 `references/WORKTREE_CONFIRMATION_PROTOCOL.md`

#### 3.1.5 接口契约验证（fullstack 模式强制）⭐🆕

<HARD-GATE>当 project-mode 为 fullstack 时，每个 Task 启动时必须验证其 provides 或 consumes 标记的接口在 api-contract.md 中存在。验证失败则该 Task 拒绝执行并报错。</HARD-GATE>

**验证步骤**：
1. 读取 Task 的 provides/consumes 标记
2. 在 `spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md` 中查找对应接口定义
3. 如果接口不存在，拒绝执行并输出错误：`接口 [METHOD /path] 未在 api-contract.md 中定义`
4. 记录当前 api-contract.md 版本号到执行报告

**版本锁定**：
- Task 执行期间锁定 api-contract.md 的版本号
- 如需接口变更，必须先更新版本号、重新触发接口审查、通知所有相关 Task 重新审查

**接口契约遵循约束（fullstack 模式）**：

**后端 Task**：
- ❌ 禁止：返回 api-contract.md 中未定义的字段
- ❌ 禁止：缺少 api-contract.md 中定义的必填字段
- ❌ 禁止：使用与 api-contract.md 不一致的错误码
- ✅ 必须：严格按照 api-contract.md 中定义的请求参数结构编码
- ✅ 必须：严格按照 api-contract.md 中定义的响应格式返回
- ✅ 必须：实现 api-contract.md 中定义的所有错误码

**前端 Task**：
- ❌ 禁止：调用 api-contract.md 中未定义的接口
- ❌ 禁止：发送与 api-contract.md 不一致的请求格式
- ❌ 禁止：忽略 api-contract.md 中定义的错误情况
- ✅ 必须：严格按照 api-contract.md 中定义的格式发送请求
- ✅ 必须：严格按照 api-contract.md 中定义的响应结构解析数据
- ✅ 必须：处理 api-contract.md 中定义的所有错误情况

#### 3.2 实现子代理：编码和自审查

实现子代理会：
- 阅读Task详情和上下文
- **检查是否有未明确的TODO项，如有则提问以澄清**
- 进行编码实现
- 编写单元测试（覆盖率 ≥ 85%，采用TDD流程：RED→GREEN→REFACTOR）
- **（前端/全栈Task）编写浏览器测试**（使用 Playwright，TDD流程：RED-BROWSER→GREEN-BROWSER）
- 进行自审查，检查是否满足验收标准
- **代码提交前的分支确认** ⭐ **新增**：
  - 显示当前分支和待提交改动
  - 使用 AskUserQuestion 向用户确认提交
  - 执行 git commit
  - 记录提交信息到执行报告
- 返回源代码和自审查报告

**浏览器测试要求（前端/全栈Task）**：

对于涉及 UI 组件或用户交互的 Task，在单元测试之外还需编写浏览器测试：

1. **TDD 浏览器测试循环**：
   - RED-BROWSER → 编写 Playwright 测试，预期失败（UI 未实现）
   - GREEN-BROWSER → 实现 UI 组件，使浏览器测试通过
   - 验证 → 在真实浏览器中确认功能正常

2. **测试覆盖范围**：
   - spec 中的 BROWSER-TESTABLE 验收标准
   - design.md 中定义的可测试交互路径
   - 关键用户操作流程

3. **测试编写规范**：
   - 使用 design.md 中定义的 data-testid 选择器
   - 测试文件命名为 `[feature].e2e.test.ts` 或 `[feature].visual.test.ts`
   - 使用 `browser-test-helpers.ts` 中的工具函数

4. **Playwright 测试类型**：
   - `@e2e` - 端到端用户流程测试
   - `@visual` - 视觉回归截图对比测试
   - `@component` - 组件级 UI 交互测试

**代码实现要求**：
- TypeScript strict模式（所有参数和返回值都有类型）
- 遵循项目命名规范（camelCase/PascalCase）
- 为复杂逻辑添加注释
- 最终代码不应有待定的TODO注释

**单元测试约束（确保测试真实可用）**：

1. **覆盖率要求**（≥ 85%）：
   - 语句覆盖率 ≥ 85%
   - 分支覆盖率 ≥ 80%
   - 函数覆盖率 ≥ 85%
   - 优先覆盖业务逻辑，而非工具函数

2. **测试完整性**（必须包含）：
   - ✅ 正常路径：正常输入产生正常输出
   - ✅ 边界情况：空值、null、undefined、0、空字符串、最大值、最小值等
   - ✅ 错误处理：异常抛出、错误返回、验证失败场景
   - ✅ 关键业务规则：来自spec的验收标准都要有对应测试

3. **测试质量标准**：
   - 每个测试只验证一个行为（单一责任）
   - 清晰的测试命名：describe + it 清晰描述预期行为
   - 明确的断言：不使用模糊的真假判断，使用具体的期望值
   - 隔离的测试：测试间互不依赖，可任意顺序执行
   - 确定性：同样的输入必须产生同样的输出，避免时间相关或随机依赖

4. **真实性检查**（禁止的做法）：
   - ❌ 不要mock业务逻辑，应该测试真实逻辑
   - ❌ 不要修改源代码逻辑使其适配测试
   - ❌ 不要编写虚假的测试（测试通过但实际没验证任何东西）
   - ❌ 不要跳过边界和错误场景
   - ❌ 不要使用always-true的断言（如 expect(true).toBe(true)）

5. **Mock策略**：
   - ✅ Mock外部依赖（API、数据库、文件系统）
   - ✅ Mock第三方库（HTTP客户端、日期库等）
   - ✅ Mock需要真实费用的服务（支付、短信等）
   - ❌ 不要Mock业务逻辑和内部函数
   - 使用具体的mock数据（来自spec中的示例数据）

#### 3.3 规范审查

规范审查子代理验证：
- 代码是否符合design.md的架构规范
- 关键组件是否按设计实现
- 命名、结构是否符合规范

#### 3.4 质量审查

质量审查子代理检查：
- TypeScript类型检查（strict模式）
- ESLint/代码规范检查
- 最佳实践
- 可读性和可维护性
- **🆕 完整性检查**：确保没有不完整的框架代码（如空的样式块、只有TODO的函数等）
  - 禁止遗留任何框架代码（示例注释但无实现）
  - 禁止遗留任何TODO注释在最终代码中
  - 详见：`references/workflow-detail.md` 中的"完整性检查"部分

#### 3.5 修复循环

如果审查未通过：
1. 记录发现的问题
2. 实现子代理进行修复
3. 修复后重新审查（直到通过）

#### 3.6 完成验证 ⭐ 新增

在任何 Task 标记为"完成"之前，必须执行验证：

**验证铁律**：
```
未在当前消息中运行验证命令 → 不能声称通过
```

**Gate 流程**：
1. IDENTIFY → 什么命令证明此 Task 完成？
2. RUN → 执行完整命令（新鲜、完整）
3. READ → 读取完整输出，检查退出码
4. VERIFY → 输出是否确认了声明？
   - 否：声明实际状态 + 证据
   - 是：声明完成 + 附上证据
5. ONLY THEN → 做出完成声明

**常见验证对照表**：

| 声称 | 必须提供 | 不足的证据 |
|------|---------|-----------|
| 单元测试通过 | 测试命令输出: 0 failures | 之前通过的记录、"应该通过了" |
| 代码编译成功 | 编译命令: exit 0 | Linter 通过、"看起来没问题" |
| 浏览器测试通过 | Playwright 输出: 0 failures | "代码写好了"、"逻辑正确" |
| 代码质量达标 | Lint + TypeScript 检查输出 | "遵循了规范" |
| Task 完成 | 验收标准逐项核对 + 证据 | "测试通过了" |

**红旗警告**：
- 使用"应该"、"大概"、"看起来" → 停止，去验证
- 在验证前表达满意（"好的"、"完成了"、"没问题"）→ 停止
- 准备提交但未验证 → 停止
- 相信子代理的成功报告 → 必须独立验证

**子代理验证**：
```
✅ 子代理报告成功 → 检查 git diff → 验证改动 → 报告实际状态
❌ 直接信任子代理报告
```

**底线**：运行命令 → 读取输出 → 然后声明结果。没有捷径。

#### 3.7 TDD 状态追踪 ⭐ **新增**

每个 Task **必须**记录 TDD 四个阶段的执行日志，格式如下：

```markdown
## TDD 执行日志 - Task T{N}: {Task名称}

| 阶段 | 状态 | 详情 |
|------|------|------|
| RED | ✅ | 测试文件 `{test-file}.test.ts` 已创建，运行结果: FAIL (3 tests, 3 failed) |
| GREEN | ✅ | 实现文件 `{impl-file}.ts` 已创建，运行结果: PASS (3 tests, 3 passed) |
| REFACTOR | ✅ | 代码重构完成（提取工具函数、类型注解），运行结果: PASS (3 tests, 3 passed) |
| REVIEW | ✅ | 规范审查: 通过 ✅ | 质量审查: 通过 ✅ | 覆盖率: 92% |

**最终结果**: ✅ Task T{N} 完成
```

**阶段定义**：

| 阶段 | 名称 | 必须满足 | 失败处理 |
|------|------|---------|---------|
| RED | 编写失败的测试 | 测试文件存在且运行失败 | 不可跳过（standard模式） |
| GREEN | 编写最小实现使测试通过 | 测试全部通过 | 回到RED补充测试 |
| REFACTOR | 改进代码质量 | 测试仍然通过 + 代码改进 | 回滚重构 |
| REVIEW | 两阶段审查 | 规范审查✅ + 质量审查✅ | 进入修复循环 |

**执行报告中的 TDD 总览**（在最终执行报告中包含）：

```markdown
## TDD 执行总览

| Task | RED | GREEN | REFACTOR | REVIEW | 覆盖率 | 状态 |
|------|-----|-------|----------|--------|--------|------|
| T1 | ✅ | ✅ | ✅ | ✅ | 92% | ✅ 完成 |
| T2 | ✅ | ✅ | ✅ | ⏳ | 88% | 审查中 |
| T3 | ✅ | ✅ | - | - | - | 实现中 |
```

<HARD-GATE>
standard 模式下，每个 Task 必须完整记录 TDD 四阶段日志。
缺失任何阶段 → 该 Task 视为未完成。
</HARD-GATE>

---

### 步骤3.8: 执行完成前自检 ⭐ **新增**

在标记 code-execute 阶段为"完成"**之前**，**必须**执行以下自检清单：

#### 自检清单

- [ ] **TDD 流程**：每个 Task 是否都通过了 TDD 流程（RED→GREEN→REFACTOR→REVIEW）？
- [ ] **测试文件**：每个 Task 是否都创建了对应的测试文件？
- [ ] **测试实际运行**：测试是否都实际运行过（不是仅写了代码）？所有测试是否通过？
- [ ] **子代理使用**：是否有 Task 跳过了子代理？如有，原因是什么？用户是否确认？
- [ ] **RED 阶段**：是否有 Task 跳过了 RED 阶段？如有，原因是什么？用户是否确认？
- [ ] **覆盖率达标**：单元测试覆盖率是否 ≥ 85%（或模式标签指定的阈值）？
- [ ] **审查完成**：所有 Task 是否都通过了两阶段审查（或快速模式的单阶段）？
- [ ] **修复循环**：所有发现的问题是否都已修复并重新审查通过？
- [ ] **分支安全**：同一 Task 的所有改动是否都在同一分支上？
- [ ] **无伪代码**：是否检查了所有实现，确保没有使用 console/print 等伪代码？
- [ ] **TDD 日志**：每个 Task 是否都记录了完整的 TDD 四阶段日志？
- [ ] **模式标签**：是否遵循了模式标签中的所有要求？

#### 自检结果处理

| 自检结果 | 处理 |
|----------|------|
| 全部通过 | 继续生成执行报告 |
| 有未通过项 | ❌ **暂停**，回到对应 Task 重新执行，直到自检全部通过 |

**自检不通过 → 不允许生成执行报告 → 不允许进入 code-test 阶段**

---

### 步骤4: 生成执行报告

当所有Task完成后，生成执行报告：
- 所有Task的完成状态
- 修复循环的次数和内容
- 最终代码统计
- 质量指标（覆盖率、Lint结果等）

详见：`references/workflow-detail.md` 中的完整工作流说明。

---

## 关键约束

✅ **必须做**：
- 每个Task创建独立worktree（执行前向用户确认分支）
- **严禁同一Task跨分支提交** - 选定目标分支后，所有改动必须提交到该分支
- **严禁在Worktree中切换分支** - 创建worktree的分支就是目标分支，不允许中途修改
- **⚠️ 严禁使用console等伪代码代替真实实现** - 所有逻辑必须有真正的实现代码，不能用console.log/print等伪代码
- 代码提交前使用 AskUserQuestion 向用户确认
- 代码合并到主分支前向用户确认合并策略
- 所有代码通过两阶段审查（standard模式）或单阶段审查（quick模式）
- 修复问题后重新审查，直到通过
- 生成详细的执行报告
- 单元测试覆盖率 ≥ 85%（standard）或 ≥ 60%（quick），包含正常路径、边界情况、错误处理
- 测试代码质量不低于生产代码（清晰命名、明确断言、隔离独立）
- 编写真实可用的测试，而不是虚假测试
- **（前端/全栈Task）编写浏览器测试** - 每个UI相关的Task都要有对应的Playwright测试
- **浏览器测试覆盖 spec 中的 BROWSER-TESTABLE 验收标准**
- **每个 Task 必须记录 TDD 四阶段日志**（RED/GREEN/REFACTOR/REVIEW）⭐ **新增**
- **执行完成前必须自检**，不通过不允许进入下一阶段 ⭐ **新增**
- **读取并遵循模式标签**中的要求（TDD/子代理/覆盖率）⭐ **新增**
- **standard 模式下必须使用子代理**，不允许跳过 ⭐ **新增**
- **开始编码前必须验证测试环境**，无测试框架时暂停确认 ⭐ **新增**

❌ **禁止做**：
- 跳过规范审查或质量审查
- 在design.md中有TODO时开始编码
- 为了让测试通过而改变源代码逻辑
- 将未清晰说明的需求跳过
- 编写虚假测试（测试通过但不验证任何东西）
- Mock业务逻辑或内部函数
- 跳过边界情况和错误处理的测试
- 使用always-true的断言
- **遗留任何不完整的框架代码**（如 `<style>` 块只有注释、函数体只有TODO等）
- **遗留任何TODO注释在最终代码中**
- **不询问用户直接提交代码**
- **不确认就合并worktree代码到主分支**
- **同一Task的改动提交到不同分支** - 创建worktree时确定的分支是唯一目标
- **在Worktree中中途切换分支** - 禁止git checkout到其他分支
- **使用console/print等伪代码代替真实实现** - 不能用console.log、print、echo等简单输出伪装成逻辑实现
- **standard 模式下跳过子代理**（必须使用 Agent 工具）⭐ **新增**
- **未验证测试环境就开始编码**（步骤1.5）⭐ **新增**
- **standard 模式下跳过 RED 阶段** ⭐ **新增**
- **未执行自检就生成执行报告** ⭐ **新增**

---

## 相关资源

| 资源 | 说明 |
|------|------|
| `references/workflow-detail.md` | 完整的工作流说明和示例 |
| `references/unit-test-real-practices.md` | 📌 **新增：编写真实可用单元测试的详细指南** |
| `references/code-completeness-checklist.md` | 🆕 **代码完整性检查清单** - 防止框架代码遗留 |
| `references/CODE_IMPLEMENTATION_QUALITY.md` | ⚠️ **极其重要：代码实现质量约束** - 禁止使用console等伪代码代替真实实现 |
| `references/git-worktrees-guide.md` | git-worktree的详细使用指南 |
| `references/WORKTREE_CONFIRMATION_PROTOCOL.md` | ⭐ **新增：Worktree创建确认协议** - 用户明确选择基础分支 |
| `references/BRANCH_SAFETY_PROTOCOL.md` | ⭐ **新增：分支安全协议** - 提交、合并、删除时的分支确认 |
| `references/tdd-flow.md` | TDD红-绿-重构流程说明 |
| `references/QUICK_REFERENCE.md` | 快速参考检查清单 |
| `prompts/tdd-implementer-prompt.md` | 实现子代理的提示词 |

---

## 最佳实践

1. **并行执行** - 利用批次并行，加快整体进度
2. **问题提前** - 在开始编码前澄清所有TODO项
3. **小commit** - 每次修复作为独立commit，便于追踪
4. **审查效率** - 审查结果要明确，便于快速修复
5. **报告完整** - 执行报告要详细，便于后续回溯
6. **TDD 日志** - 每个 Task 都记录四阶段日志，便于追踪进度 ⭐ **新增**
7. **环境预检** - 开始编码前先验证测试环境，避免中途卡住 ⭐ **新增**
8. **模式明确** - 根据模式标签执行对应流程，避免标准/快速模式混淆 ⭐ **新增**
9. **自检习惯** - 完成前执行自检清单，确保不遗漏关键步骤 ⭐ **新增**

---

**关键理念**：规范驱动开发，通过隔离和多阶段审查确保代码质量。每个Task都是独立、可追踪、可恢复的。
