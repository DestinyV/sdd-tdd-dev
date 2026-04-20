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
  - ✅ 两阶段审查：规范审查 + 质量审查
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
2. 逐批执行Task（每Task分配独立子代理 + 隔离worktree）
3. TDD实现：单元测试 + 浏览器测试（前端Task）
4. 两阶段审查：规范审查 → 质量审查
5. 修复问题后重新审查，直到通过
6. 输出执行报告

**输出路径**：`spec-dev/{requirement_desc_abstract}/execution/execution-report.md`

## 何时使用

**前置条件**：
- ✅ spec-creation 已完成，设计规范已生成
- ✅ code-designer 已完成，设计方案已确认（标准模式）
- ✅ code-task 已完成，任务列表已定义（标准模式）
- ✅ 所有关键决策已在前面阶段确认
- ✅ **快速模式**：spec 已生成，可直接读取 spec 开始编码（跳过 design 和 task）

### 快速模式执行（当 spec 标注为快速模式时）

在快速模式下，code-execute 执行以下精简流程：

1. **直接读取 spec**：从 `spec-dev/{req_id}/spec/` 读取场景和验收标准
2. **自主 Task 拆解**：在内存中拆解实现步骤（不生成 tasks.md）
3. **单阶段审查**：只做质量审查（跳过规范审查，因为 design 不存在）
4. **精简测试**：
   - ✅ 必须：单元测试（覆盖率 ≥ 85%）
   - ✅ 必须：浏览器 E2E 测试（前端场景）
   - ⚡ 跳过：视觉回归测试、组件 UI 测试、性能测试

```
快速模式执行流程：
读取 spec → 自主拆解 → 编码（TDD）→ 质量审查 → 修复循环 → 生成报告
```

---

## 工作流程

<HARD-GATE>
在 design.md 或 tasks.md 中还有 TODO 项未全部澄清之前，
不开始任何编码实现。
必须逐一确认所有 TODO，直到无遗留，然后才能开始编码。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>

### 步骤1: 分析任务列表和并行计划

从tasks.md中提取：
- 所有Task信息（名称、目标、交付物、验收标准）
- Task之间的依赖关系
- 并行执行计划（按批次分组）

创建TodoList，列出所有Task和其完成状态。

### 步骤2: 并行执行Task批次

根据依赖关系，逐批执行Task：
- **每个批次**：无依赖的Task可以并行执行
- **不同批次**：必须等待前一批次完成再开始
- **每个Task**：分配独立的实现子代理

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
- **🆕 严禁同一Task跨分支提交** - 选定目标分支后，所有改动必须提交到该分支
- **🆕 严禁在Worktree中切换分支** - 创建worktree的分支就是目标分支，不允许中途修改
- **⚠️ 🆕 严禁使用console等伪代码代替真实实现** - 所有逻辑必须有真正的实现代码，不能用console.log/print等伪代码
- 代码提交前使用 AskUserQuestion 向用户确认 ⭐ **新增**
- 代码合并到主分支前向用户确认合并策略 ⭐ **新增**
- 所有代码通过两阶段审查
- 修复问题后重新审查，直到通过
- 生成详细的执行报告
- 单元测试覆盖率 ≥ 85%，包含正常路径、边界情况、错误处理
- 测试代码质量不低于生产代码（清晰命名、明确断言、隔离独立）
- 编写真实可用的测试，而不是虚假测试
- **（前端/全栈Task）编写浏览器测试** - 每个UI相关的Task都要有对应的Playwright测试
- **浏览器测试覆盖 spec 中的 BROWSER-TESTABLE 验收标准**

❌ **禁止做**：
- 跳过规范审查或质量审查
- 在design.md中有TODO时开始编码
- 为了让测试通过而改变源代码逻辑
- 将未清晰说明的需求跳过
- 编写虚假测试（测试通过但不验证任何东西）
- Mock业务逻辑或内部函数
- 跳过边界情况和错误处理的测试
- 使用always-true的断言
- **🆕 遗留任何不完整的框架代码**（如 `<style>` 块只有注释、函数体只有TODO等）
- **🆕 遗留任何TODO注释在最终代码中**
- **🆕 不询问用户直接提交代码**
- **🆕 不确认就合并worktree代码到主分支**
- **🆕 同一Task的改动提交到不同分支** - 创建worktree时确定的分支是唯一目标
- **🆕 在Worktree中中途切换分支** - 禁止git checkout到其他分支
- **⚠️ 🆕 使用console/print等伪代码代替真实实现** - 不能用console.log、print、echo等简单输出伪装成逻辑实现

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

---

**关键理念**：规范驱动开发，通过隔离和多阶段审查确保代码质量。每个Task都是独立、可追踪、可恢复的。
