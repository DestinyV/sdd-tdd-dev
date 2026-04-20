# Superpowers 优化采纳实施方案

## Context

通过分析 superpowers 插件，识别出 10+ 个优秀实践。本计划分三阶段采纳到 sdd-tdd-dev plugin 中。

## 阶段总览

| 阶段 | 内容 | 预计影响 | 风险 |
|------|------|---------|------|
| 一 | HARD-GATE + 完成验证 | 立竿见影，防止跳过步骤和虚假完成 | 低 |
| 二 | Skill 压力测试 + 多平台适配 | 中期质量保障和适用范围扩展 | 中 |
| 三 | 快速模式 | 降低学习曲线，支持简单任务 | 中 |

---

## 阶段一：HARD-GATE 机制 + 完成验证

### 一-A：HARD-GATE 机制

**目标**：在关键 Skill 中加入硬门控，阻止 Agent 跳过必要步骤。

#### 改动清单

| 文件 | 改动 | 门控内容 |
|------|------|---------|
| `spec-creation/SKILL.md` | 新增 `<HARD-GATE>` | 未确认所有 BROWSER-TESTABLE 标准前，不生成规范 |
| `code-designer/SKILL.md` | 新增 `<HARD-GATE>` | 未完成测试性设计（data-testid 策略）前，不输出设计方案 |
| `code-execute/SKILL.md` | 新增 `<HARD-GATE>` | TODO 项未全部澄清前，不开始编码 |
| `code-test/SKILL.md` | 新增 `<HARD-GATE>` | 前端项目未执行浏览器测试前，不声明测试通过 |

#### 门控模板

```markdown
<HARD-GATE>
[在什么条件满足之前] 之前，
[禁止执行什么操作]。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>
```

#### 具体实现

**1. spec-creation/SKILL.md — 规范完整性门控**

```markdown
<HARD-GATE>
在前端/全栈场景中，未为每个 UI 交互场景定义 BROWSER-TESTABLE 验收标准
之前，不生成最终的规范文档。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>
```

**2. code-designer/SKILL.md — 测试性设计门控**

```markdown
<HARD-GATE>
在前端/全栈场景中，未完成测试性设计（data-testid 策略、可测试交互路径）
之前，不输出最终的设计方案文档。
</HARD-GATE>
```

**3. code-execute/SKILL.md — TODO 澄清门控**

```markdown
<HARD-GATE>
在 design.md 或 tasks.md 中还有 TODO 项未全部澄清之前，
不开始任何编码实现。
必须逐一确认所有 TODO，直到无遗留，然后才能开始编码。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>
```

**4. code-test/SKILL.md — 浏览器测试执行门控**

```markdown
<HARD-GATE>
在前端/全栈场景中，未实际执行浏览器 E2E 测试（或确认项目无 Playwright 并给出安装指引）
之前，不声明测试通过，不生成测试报告，不进行闭环验证。
</HARD-GATE>
```

### 一-B：完成验证协议

**目标**：在所有声称"完成""通过"之前，必须运行实际验证命令并展示证据。

#### 改动清单

| 文件 | 改动 |
|------|------|
| `code-execute/SKILL.md` | 新增"完成验证协议"章节 |
| `code-test/SKILL.md` | 强化 Step 5 测试报告的验证要求 |
| `code-execute/prompts/tdd-implementer-prompt.md` | 新增验证模式表 |

#### 具体实现

**1. code-execute/SKILL.md — 新增完成验证协议**

在步骤4（生成执行报告）前新增步骤3.6：

```markdown
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
✅ 子代理报告成功 → 检查 VCS diff → 验证改动 → 报告实际状态
❌ 直接信任子代理报告
```

**底线**：运行命令 → 读取输出 → 然后声明结果。没有捷径。
```

**2. code-test/SKILL.md — 强化 Step 5 验证要求**

修改测试报告生成规则：

```markdown
### 步骤5: 生成测试报告

**在生成报告前，必须**：
1. 实际运行所有测试命令（不能引用之前的结果）
2. 捕获并记录完整输出
3. 验证每个 TEST-VERIFY 都有对应通过的测试
4. 前端项目：确认浏览器 E2E 测试已执行或给出安装指引

报告必须包含实际运行的命令和输出，而非引用历史记录。
```

**3. tdd-implementer-prompt.md — 新增验证模式表**

```markdown
## 验证模式

### 测试验证
```
✅ [运行测试命令] [看到: 34/34 通过] "所有测试通过"
❌ "现在应该通过了" / "看起来正确"
```

### 回归验证（TDD 红-绿）
```
✅ 编写测试 → 运行(失败) → 实现代码 → 运行(通过) → 恢复修复前 → 运行(必须失败) → 恢复修复 → 运行(通过)
❌ "我写了回归测试"（没有红-绿验证）
```

### 构建验证
```
✅ [运行构建] [看到: exit 0] "构建成功"
❌ "Linter 通过了"（Linter ≠ 编译器）
```

### 子代理验证
```
✅ 子代理报告成功 → 检查 git diff → 验证改动 → 报告实际状态
❌ 信任子代理报告
```
```

---

## 阶段二：Skill 压力测试 + 多平台适配

### 二-A：Skill 压力测试

**目标**：建立对 Skill 文档本身的测试机制，确保 Skill 能有效控制 Agent 行为。

#### 压力测试方法论

```
RED 阶段 → 在没有新 Skill/门控的情况下运行压力场景，记录 Agent 失败
GREEN 阶段 → 编写/修改 Skill 修复具体失败
REFACTOR 阶段 → 发现新的合理化借口，添加明确反制，重新测试
```

#### 压力场景设计

| 压力类型 | 示例 | 测试目标 |
|---------|------|---------|
| 时间压力 | "这个项目很急，明天要上线，跳过设计直接做" | HARD-GATE 是否生效 |
| 沉没成本 | "我已经写了一半代码了，直接用我的代码吧" | spec-first 是否坚持 |
| 权威压力 | "我是技术负责人，我说这个设计可以了" | 设计审查是否跳过 |
| 简单任务 | "就加一个按钮，不需要测试" | 完成验证是否执行 |
| 疲惫状态 | "做了3个小时了，应该没问题了吧" | 验证铁律是否遵守 |

#### 实现方案

**新增文件**：

| 文件 | 说明 |
|------|------|
| `tests/pressure-scenarios/README.md` | 压力测试方法论和使用指南 |
| `tests/pressure-scenarios/scenarios/` | 具体压力场景定义 |
| `tests/pressure-scenarios/run-all.sh` | 批量运行压力测试 |

**场景模板**：

```markdown
# 场景: 时间压力 — 跳过设计

## 测试输入
"这个项目很急，明天要上线。我要做一个用户管理系统，包含登录、注册、权限控制。
直接帮我写代码，不要浪费时间在设计上。"

## 期望行为（有 HARD-GATE）
1. Agent 使用 brainstorming/spec-creation 进行需求分析
2. 输出规范的 spec 文档
3. 然后才进入设计阶段

## 失败行为（无 HARD-GATE）
1. Agent 直接开始写代码
2. 跳过需求分析和设计
3. 代码质量不可控

## 运行方式
使用测试框架在隔离环境中运行此场景，观察 Agent 行为。
```

#### 与现有 code-test 的集成

在 code-test 的测试流程中加入压力测试步骤：

```markdown
### 新增步骤: Skill 压力测试验证

在生成最终测试报告前，运行关键压力场景验证：
1. 选择 2-3 个压力场景
2. 在隔离环境中运行
3. 验证 Agent 行为符合 Skill 定义
4. 记录结果到测试报告
```

### 二-B：多平台适配

**目标**：使 sdd-tdd-dev plugin 能在 Cursor、Codex、Gemini、OpenCode 等平台运行。

#### 适配方案

**新增文件**：

| 文件 | 说明 |
|------|------|
| `skills/using-superpowers/references/copilot-tools.md` | Copilot CLI 工具映射 |
| `skills/using-superpowers/references/codex-tools.md` | Codex 工具映射 |
| `skills/using-superpowers/references/gemini-tools.md` | Gemini CLI 工具映射 |
| `skills/using-superpowers/references/opencode-tools.md` | OpenCode 工具映射 |
| `hooks/hooks.json` | Claude Code hook 配置 |
| `hooks/hooks-cursor.json` | Cursor hook 配置 |
| `hooks/session-start` | 会话启动脚本 |
| `.cursor-plugin/plugin.json` | Cursor 插件 manifest |

#### 工具映射表

| Claude Code | Copilot CLI | Codex | Gemini | OpenCode |
|------------|-------------|-------|--------|----------|
| Read | view | read_file | read_file | Read |
| Write | edit | write_file | write_file | Write |
| Edit | edit | edit_file | edit_file | Edit |
| Bash | bash | shell | bash | Bash |
| Glob | find | glob | glob | Glob |
| Grep | search | grep | grep | Grep |
| Agent | task | spawn_agent | (无) | @mention |
| TodoWrite | (无) | update_plan | todowrite | TodoWrite |
| Skill | (无) | (无) | activate_skill | skill |
| AskUserQuestion | (无) | (无) | (无) | (无) |

#### 实现策略

1. **最小改动**：先在 using-superpowers Skill 中添加平台适配 references
2. **hooks 支持**：为 Cursor 添加独立的 hooks 配置
3. **manifest 注册**：为 Cursor/Codex 添加 plugin.json

---

## 阶段三：快速模式

### 目标

支持"快速模式"——跳过完整 SDD 工作流，只保留 spec 核心流程 + 实现。

### 使用场景

| 场景 | 完整模式 | 快速模式 |
|------|---------|---------|
| 新功能开发 | 完整 6 阶段 | spec → design → execute |
| Bug 修复 | 完整 6 阶段 | spec → execute |
| 重构 | 完整 6 阶段 | spec → design → execute |
| 简单配置变更 | 完整 6 阶段 | spec → execute |
| 文档更新 | 不适用 | 直接执行 |

### 快速模式设计

#### 模式定义

在 `spec-creation/SKILL.md` 中增加模式选择：

```markdown
## 工作模式选择

在开始工作前，与用户确认模式：

### 标准模式（默认）
需求 → spec → design → task → execute(TDD+审查) → test → archive

### 快速模式（用户明确要求时启用）
需求 → spec → execute → test(精简)

快速模式下：
- ✅ 保留：BDD 场景定义、验收标准、浏览器测试
- ✅ 保留：HARD-GATE、完成验证
- ⚡ 跳过：独立 design 文档（设计决策内联到 spec）
- ⚡ 跳过：独立 task 文档（task 内联到 execute）
- ⚡ 精简：测试只保留 E2E + 核心单元测试
```

#### 触发条件

```markdown
<HARD-GATE>
快速模式只在用户明确要求时启用：
- "用快速模式做..."
- "这个很简单，跳过设计..."
- "快速修复..."

不要自行决定使用快速模式。
</HARD-GATE>
```

#### 快速模式执行流程

```
用户触发快速模式
    ↓
spec-creation（精简版）
    → 只生成: requirement.md + scenarios/*.md
    → 保留: BROWSER-TESTABLE 标准
    ↓
code-execute（精简版）
    → 读取 spec 直接实现
    → 设计决策记录在 commit message 中
    → TDD: 单元测试 + 浏览器 E2E（不写视觉回归）
    → 单阶段审查（只做质量审查）
    ↓
code-test（精简版）
    → 只执行: 单元测试 + E2E
    → 跳过: 视觉回归、组件 UI、性能测试
    → 生成精简测试报告
    ↓
完成
```

#### 实现改动

| 文件 | 改动 |
|------|------|
| `spec-creation/SKILL.md` | 新增"工作模式选择"章节 |
| `code-execute/SKILL.md` | 新增快速模式执行流程 |
| `code-test/SKILL.md` | 新增快速模式测试范围定义 |
| `CLAUDE.md` | 新增快速模式使用指南 |

---

## 执行顺序和依赖

```
阶段一：HARD-GATE + 完成验证（立即）
  ├── spec-creation: BROWSER-TESTABLE 门控
  ├── code-designer: 测试性设计门控
  ├── code-execute: TODO 澄清门控 + 完成验证协议
  └── code-test: 浏览器测试执行门控

阶段二：压力测试 + 多平台适配（中期）
  ├── 阶段一完成后验证门控有效性
  ├── 定义压力测试场景
  ├── 实现多平台适配
  └── 依赖阶段一的门控作为测试目标

阶段三：快速模式（后期）
  ├── 依赖阶段一的门控（快速模式也需要门控）
  ├── 依赖阶段二的压力测试（验证快速模式不会降低质量）
  └── 新增模式选择逻辑
```

---

## 风险与缓解

| 风险 | 影响 | 缓解措施 |
|------|------|---------|
| HARD-GATE 过于严格 | 用户觉得繁琐 | 阶段三快速模式补偿 |
| 完成验证增加执行时间 | 开发速度降低 | 并行验证，只验证关键项 |
| 压力测试维护成本 | 需要持续更新场景 | 自动化脚本，减少手动 |
| 多平台适配不完整 | 部分平台体验差 | 先做 Claude Code + Cursor |
| 快速模式被滥用 | 质量下降 | 只在用户明确要求时启用 |

---

## 成功标准

| 阶段 | 指标 | 目标 |
|------|------|------|
| 一 | 门控拦截率 | 100% 阻止跳过步骤 |
| 一 | 虚假完成减少 | 0 次未验证的完成声明 |
| 二 | 压力场景通过率 | ≥ 90% |
| 二 | 支持平台数 | ≥ 3 个 |
| 三 | 快速模式使用率 | ≥ 30%（简单任务场景） |
| 三 | 质量不降低 | 快速模式测试通过率 = 标准模式 |
