---
name: spec-creation
description: |
  第1步：需求分析和规范生成（Spec阶段）
  
  输入：需求描述
  输出：spec-dev/{requirement_desc_abstract}/spec/ 目录
  
  功能：接收需求内容，通过交互式分析进行多次确认，输出BDD格式的规范文档（WHEN-THEN格式）。
  支持全栈开发（前端、后端、移动端、全栈应用）的需求规范化。
  
  输出内容：
  - requirement.md (需求文档总览)
  - scenarios/*.md (BDD场景 - WHEN-THEN格式 + TEST-VERIFY验收标准 + BROWSER-TESTABLE浏览器可执行断言，如 scenarios/tab-visibility.md)
  - data-models.md (数据模型定义，支持SQL/NoSQL)
  - business-rules.md (业务规则和约束)
  - glossary.md (术语表)
  
  新增：TDD支持
  - 每个场景包含TEST-VERIFY - 定义可测试的验收标准
  - 包含Mock数据示例 - 为后续测试设计提供数据
  
  支持的场景：
  - 前端应用：UI组件、页面交互、状态管理
  - 后端服务：API设计、数据处理、业务逻辑
  - 数据库：表结构、索引策略、数据一致性
  - 微服务：服务边界、通信协议、数据流
  - 移动应用：跨平台兼容、离线支持、同步机制
  
  下一步：/code-designer (读取spec-dev/{requirement_desc_abstract}/spec/)
---

# spec-creation

## 职责

根据用户需求，进行深入分析和拆解，生成完整的、可执行的规范文档。支持全栈开发的各类需求：

1. 接收并理解需求内容（前端UI、后端API、数据库、微服务等）
2. 拆解需求为不同的业务场景和技术场景
3. 对每个场景进行多轮确认和细化
4. 生成BDD格式的规范文档（WHEN-THEN描述）
5. 输出为多个场景文件到项目的 `spec-dev/[需求ID]/spec/` 目录
6. 定义数据模型（支持关系型和非关系型数据库）
7. 明确业务规则和约束条件

**输出路径**：当前项目的 `spec-dev/[需求ID]/spec/` 目录

---

## 工作流程

### 工作模式选择

在开始工作前，与用户确认工作模式：

**标准模式（默认）**：
```
需求 → spec → design → task → execute(TDD+审查) → test → archive
```
适用于：新功能开发、复杂修改、涉及多个组件的变更

**快速模式（用户明确要求时启用）**：
```
需求 → spec（精简版）→ execute（精简版）→ test（精简版）
```
适用于：Bug 修复、简单配置变更、小型 UI 调整

快速模式下的变更：
- ✅ 保留：BDD 场景定义、TEST-VERIFY、BROWSER-TESTABLE（前端）
- ✅ 保留：HARD-GATE、完成验证
- ⚡ 跳过：独立 design 文档（设计决策内联到 spec 的 scenarios 中）
- ⚡ 跳过：独立 task 文档（直接读取 spec 进入 code-execute）
- ⚡ 精简：测试只保留单元测试 + E2E（跳过视觉回归、组件UI、性能）

<HARD-GATE>
快速模式只在用户明确要求时启用。
触发词包括但不限于：
  - "用快速模式做..."
  - "这个很简单，跳过设计..."
  - "快速修复..."
  - "别走完整流程..."
不要自行决定使用快速模式。当不确定时，使用标准模式并询问用户。
</HARD-GATE>

### 模式确认输出

在生成的 requirement.md 中，必须明确标注工作模式及所有约束标签：

```markdown
## 工作模式
- 模式：[standard | quick]
- TDD要求：[必须执行 | 可跳过]
- 子代理要求：[必须使用 | 可选]
- 测试覆盖要求：[≥85% | ≥60% | 跳过]
- 触发原因：[用户明确要求 / 默认标准模式]
- 影响范围：[跳过的步骤清单]

## 全栈模式判定 ⭐🆕
- 开发模式：[fullstack | frontend-only | backend-only]
- 是否同时涉及前端和后端改动：[是 | 否]
- 判定依据：[列出前端和后端的具体改动内容]

## 数据库设计需求 ⭐🆕
- 需要数据库设计：[是 | 否]
- 目标 SQL 方言：[mysql | postgresql | sqlite | sqlserver | 未确定]（needs-database=是 时必填）
- 判定依据：
  - [ ] 新增数据表/集合：[列出]
  - [ ] 修改现有表结构：[列出]
  - [ ] 新增/修改索引：[列出]
  - [ ] 数据迁移需求：[列出]
  - [ ] 纯展示层调整（无表结构变更）
- 说明：[如不需要，说明原因]

## 接口契约需求 ⭐🆕
- 需要接口契约：[是 | 否]
- 判定依据：
  - [ ] 新增 API 端点
  - [ ] 修改现有 API
  - [ ] 前后端数据交互需求
  - [ ] 已有 API 的纯前端展示调整
- 说明：[如不需要，说明原因]
```

**模式标签作用说明**：

| 标签 | standard 模式 | quick 模式 |
|------|--------------|-----------|
| 模式 | standard | quick |
| TDD要求 | 必须执行（RED→GREEN→REFACTOR→REVIEW） | 可跳过 |
| 子代理要求 | 必须使用 Agent 工具 | 可选 |
| 测试覆盖要求 | ≥85% | ≥60% |
| code-designer | 必须执行 | 跳过 |
| code-task | 必须执行 | 跳过 |
| 审查阶段 | 两阶段（规范+质量） | 单阶段（仅质量） |
| 测试类型 | 单元+E2E+视觉+组件+性能 | 单元+E2E |
| TDD日志 | 必须完整记录四阶段 | 可精简 |

**全栈模式标签作用说明** ⭐🆕：

| 开发模式 | code-designer 阶段 | code-task 阶段 |
|---------|-------------------|---------------|
| fullstack | 数据库（按需）→ 接口契约 → 前后端设计 | Batch 0: 数据库+接口 → Batch 1: 前后端并行 → Batch 2: 联调 |
| frontend-only | 仅前端组件设计 | 前端任务独立执行 |
| backend-only | 数据库（按需）+ 后端服务设计 | 后端任务独立执行 |

**后续阶段必须读取并遵循这些模式标签**，不得自行降级或变更。

### 阶段1: 需求理解和初步分析

接收用户的需求输入，进行初步分析。用户可以提供需求描述，也可以提供本地需求文档（如 README、需求规格说明等）。

**输入选项**：
```
方式1 - 直接描述：
需求标题：[需求名称]
需求描述：[详细的需求描述]
相关背景：[产品背景、用户场景等]

方式2 - 本地需求文档：
用户上传本地需求文档（如 README.md、需求说明.md 等）
系统读取并分析文档内容
```

**活动**：
- 阅读和理解需求（直接描述或从本地文档读取）
- 识别核心功能点
- 初步提取主要场景
- 提出澄清问题
- 如需要，分配 subagent 探索项目结构（查看现有代码、架构、相关组件）以增强对项目的理解

**输出**：
- 需求核心功能点清单
- 初步场景列表
- 待确认的问题清单
- 项目理解笔记（如果进行了项目探索）

**SQL 方言确认**（当 needs-database=是 时）：
- 使用 AskUserQuestion 确认目标 SQL 方言：
  ```
  question: "请确认本项目使用的数据库方言？"
  header: "SQL方言"
  multiSelect: false
  options:
    - { label: "MySQL", description: "MySQL 5.7+/8.0+" }
    - { label: "PostgreSQL", description: "PostgreSQL 12+" }
    - { label: "SQLite", description: "SQLite 3.x" }
    - { label: "SQL Server", description: "SQL Server 2017+" }
  ```
- 如用户不确定，标注"未确定"，由 code-designer 阶段自动检测

**多项目协作检测**（可选）：
- 当前需求是否涉及多个项目/仓库？
- 如是，逐个确认：
  1. 项目名称和路径
  2. 本项目在此需求中的角色（提供方/消费方/两者）
  3. 与哪些其他项目有接口依赖？
  4. 协作链路类型（上下游服务/前后端分离/微服务集群/Monorepo多Package）
- 确认后写入 requirement.md 的「多项目协作」章节

**项目探索**（标准流程必须执行）：
- 分配 **code-explorer agent** 进行项目探索
- 读取最近 3 个已完成需求的 spec 和 design，提取成功模式
- 识别项目中已有的相似功能，确认技术实现路径
- 输出"项目理解报告"作为场景拆解的输入
- code-explorer 会追踪项目的现有功能实现，帮助理解：
  - 现有的类似功能实现方式
  - 当前的代码结构和技术栈
  - 项目使用的设计模式和架构
  - 关键的代码路径和依赖关系
- 将 code-explorer 的分析结果用于指导需求分析和规范设计，确保新需求与现有架构一致

---

### 阶段1.5: 需求深度追问（标准化）⭐🆕

场景确认后、规范细化前，执行系统化追问流程。

**追问维度**（逐个确认，每次一个维度）：

1. **用户意图** - "这个功能最终要解决什么问题？谁受益最大？"
2. **成功标准** - "怎么判断这个功能做对了？可量化的指标？"
3. **范围边界** - "哪些在这个需求范围内？哪些不在？"
4. **技术约束** - "有哪些技术限制或已有系统依赖？"
5. **时间优先级** - "核心MVP是什么？哪些可以后续迭代？"

使用 AskUserQuestion 工具逐个确认，避免一次性抛出所有问题。

---

### 阶段2.5: 方案对比选择（复杂需求）⭐🆕

当需求存在多种实现路径时，执行方案对比。

**触发条件**（满足任一）：
- 需求涉及新的技术栈或架构模式
- 存在多种数据库设计方案
- 存在多种服务间通信方式
- 前端存在多种状态管理或组件化方案

**执行步骤**：
1. 提出 2-3 种可行方案
2. 列出每种方案的 trade-offs
3. 给出推荐方案和理由
4. 与用户确认后，再进入规范细化

**隔离设计原则**：
- 每个场景应聚焦单一用户行为或系统事件
- 场景间的依赖关系应显式声明（depends-on / provides-to）
- 避免场景嵌套或职责重叠
- 场景应能被独立测试和验证

---

### 阶段2: 场景拆解和确认

将需求拆解为具体的业务场景，通过交互确认每个场景。

**场景拆解策略**：

根据以下维度拆解场景：
1. **用户类型** - 不同用户角色的操作
2. **业务流程** - 不同的操作流程或阶段
3. **数据状态** - 不同的数据状态下的行为
4. **异常情况** - 错误、边界、异常等情况

**场景模板**：

```markdown
## 场景: [场景名称]

**描述**：[场景的简要描述]

**前置条件**：
- [条件1]
- [条件2]

### Case 1: [case名称]
**WHEN** [触发条件]
**THEN** [预期结果]

### Case 2: [case名称]
**WHEN** [触发条件]
**THEN** [预期结果]

**备注**：[任何需要说明的地方]
```

**确认过程**：
- 逐个场景进行确认
- 针对每个Case的WHEN和THEN进行确认
- 识别遗漏的场景
- 确认场景之间的关系

---

### 阶段3: 规范细化和验证

进行多轮确认，细化规范细节。

**细化维度**：

1. **功能完整性** - 覆盖所有主要场景、用户类型、异常情况
2. **数据一致性** - 数据定义一致、字段类型明确、有效范围定义
3. **交互流程** - 场景依赖关系、状态转换、取消/返回处理
4. **约束和边界** - 业务约束、边界条件、不允许的操作
5. **性能和可用性** - 大数据量表现、响应时间、离线处理
6. **服务上下游** ⭐ - 上游调用方、下游依赖服务、接口契约变更、跨服务数据流
7. **数据库设计** ⭐ - 新增/变更表结构、索引策略、数据迁移方案、事务一致性
8. **中间件** ⭐ - Redis缓存策略、Nacos配置管理、Docker/K8s部署变更
9. **项目配置** ⭐ - 环境变量、依赖管理、构建配置、CI/CD变更
10. **日志规范** ⭐ - 日志框架、关键业务日志、错误上下文、链路追踪
11. **上线方案** ⭐ - 上线执行顺序、灰度策略、回滚方案、依赖服务上线顺序
12. **监控方案** ⭐ - API监控指标、告警规则、业务埋点、分布式追踪
13. **前端构建和部署** ⭐（前端/全栈） - 构建优化、CDN策略、HTTP缓存、回滚方案
14. **前端监控** ⭐（前端/全栈） - JS错误追踪、Core Web Vitals、用户埋点、兼容性监控
15. **前端可测试性** ⭐（前端/全栈） - BROWSER-TESTABLE 验收标准（浏览器可执行的DOM/URL/交互断言）
16. **多项目协作** ⭐🆕 - 是否涉及多个项目/仓库、项目间依赖关系、协作链路类型、跨项目接口契约

**验证问题**：
```
多轮确认检查清单：
- [ ] 所有场景的WHEN条件都明确吗？
- [ ] 所有THEN结果都可验证吗？
- [ ] 是否遗漏了异常或边界情况？
- [ ] 场景之间是否有冲突或矛盾？
- [ ] 所有的业务规则都已明确？
- [ ] 所有的约束条件都已说明？
```

---

### 阶段4: 生成规范文档

将确认的规范转换为结构化的文档。

**输出结构**（遵循**单一入口 + 内部导航**策略）：

```
spec-dev/
└── [需求ID]/                         # 需求目录（如 REQ-2024-001）
    └── spec/                         # 规范目录
        ├── requirement.md            # 【入口文件】规范总览和导航索引
        ├── scenarios/                # 场景目录
        │   ├── tab-visibility.md     # 场景（使用功能名，如 tab-visibility）
        │   ├── form-validation.md    # 场景
        │   └── [scenario-name].md    # 场景N
        ├── data-models.md            # 数据模型定义
        ├── business-rules.md         # 业务规则和约束
        ├── glossary.md               # 术语表
        ├── infrastructure.md         # 【后端/全栈】基础设施分析（服务依赖、数据库、中间件、配置、日志、上线、监控）
        ├── deployment.md             # 【后端/全栈】上线和部署方案（执行顺序、灰度、回滚、监控点）
        ├── frontend-deployment.md    # 【前端/全栈】前端上线和部署方案（构建优化、CDN、缓存策略、回滚）
        ├── frontend-monitoring.md    # 【前端/全栈】前端监控和可观测性（错误追踪、性能指标、埋点、用户体验）
        └── sql-ddl.md                # 【后端/全栈】SQL DDL/DML 可执行脚本（needs-database=是 时生成）
```

**关键说明**：
- `infrastructure.md` 和 `deployment.md` 在后端/全栈场景下必须生成
- `frontend-deployment.md` 和 `frontend-monitoring.md` 在前端/全栈场景下必须生成
- `sql-ddl.md` 在 needs-database=是 时必须生成，包含完整的可执行 SQL 脚本
  - 第 1 节 DDL：新增表（CREATE TABLE）、现有表变更（ALTER TABLE）、索引、约束
  - 第 2 节 DML：种子数据（INSERT）、数据迁移（UPDATE）、数据清理（DELETE）
  - 第 3 节：执行顺序说明
  - 第 4 节：回滚 SQL（每条正向变更对应回滚语句）
- 纯后端场景可省略前端相关文档
- requirement.md 作为**入口文件**，提供快速导航链接到各部分
- 每个文件中使用 `// TODO：{描述}` 标记待确认事项
- 按一致的格式和导航方式组织文档

**模板导航**：
所有规范文档模板位于 [`templates/`](./templates/) 目录：

| 模板 | 用途 | 场景 |
|------|------|------|
| [`templates/spec-requirement-template.md`](./templates/spec-requirement-template.md) | 需求文档总览 | 所有场景 |
| [`templates/spec-scenario-template.md`](./templates/spec-scenario-template.md) | BDD场景定义 | 所有场景 |
| [`templates/spec-data-models-template.md`](./templates/spec-data-models-template.md) | 数据模型定义 | 所有场景 |
| [`templates/spec-business-rules-template.md`](./templates/spec-business-rules-template.md) | 业务规则和约束 | 所有场景 |
| [`templates/spec-glossary-template.md`](./templates/spec-glossary-template.md) | 术语表 | 所有场景 |
| [`templates/spec-infrastructure-template.md`](./templates/spec-infrastructure-template.md) | 基础设施分析（后端） | 后端/全栈场景 |
| [`templates/spec-deployment-template.md`](./templates/spec-deployment-template.md) | 上线和部署方案（后端） | 后端/全栈场景 |
| [`templates/spec-frontend-deployment-template.md`](./templates/spec-frontend-deployment-template.md) | 前端上线和部署方案 | 前端/全栈场景 |
| [`templates/spec-frontend-monitoring-template.md`](./templates/spec-frontend-monitoring-template.md) | 前端监控和可观测性 | 前端/全栈场景 |

---

### 阶段5: 规范交付和确认

<HARD-GATE>
在前端/全栈场景中，未为每个 UI 交互场景定义 BROWSER-TESTABLE 验收标准
之前，不生成最终的规范文档。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>

最后的审查和确认。

---

### 阶段4.5: 规范自审查 ⭐🆕

生成所有规范文档后、交付前，执行以下自审查：

1. **占位符扫描**：检查是否有未完成的 TODO、TBD、"[待确认]" 标记
2. **内部一致性**：数据模型与场景中的字段引用是否一致
   - scenarios 中的字段是否在 data-models.md 中定义
   - business-rules 中的规则是否在场景中体现
   - infrastructure 中的服务依赖是否与场景中的数据流对应
3. **范围检查**：当前 spec 是否聚焦于单一需求，还是包含了多个需求？
4. **歧义检查**：是否有可被两种解读的需求描述？
5. **完整性**：每个 WHEN-THEN 的 THEN 是否都有对应的数据模型支撑
6. **SQL 一致性** ⭐🆕（needs-database=是 时）：
   - sql-ddl.md 中的表/字段是否在 data-models.md 中定义
   - 每个 DML 操作是否有对应的业务场景支撑
7. **多项目一致性** ⭐🆕（多项目场景）：
   - 跨项目接口是否在 collaboration-plan.md 和 api-contract 中一致定义
   - 项目依赖链是否完整无遗漏

发现问题立即修复，不进入交付环节。

---

**交付清单**：
- [ ] spec/requirement.md 已生成（需求文档完整）
- [ ] 所有scenarios/*.md 已生成（所有场景都有）
- [ ] spec/data-models.md 已生成（所有数据模型都定义了）
- [ ] spec/business-rules.md 已生成（所有规则都记录了）
- [ ] spec/glossary.md 已生成（术语都定义了）
- [ ] spec/infrastructure.md 已生成（后端/全栈场景：服务依赖、数据库、中间件、配置、日志）
- [ ] spec/deployment.md 已生成（后端/全栈场景：上线顺序、灰度、回滚、监控）
- [ ] spec/frontend-deployment.md 已生成（前端/全栈场景：构建、CDN、缓存、上线顺序、回滚）
- [ ] spec/frontend-monitoring.md 已生成（前端/全栈场景：错误追踪、性能指标、埋点、告警）
- [ ] spec/sql-ddl.md 已生成（needs-database=是 时：DDL+DML 可执行脚本完整）
- [ ] 项目理解文档已生成（如进行了项目探索）
- [ ] 所有文件都经过多轮确认
- [ ] 无遗漏和矛盾

**最终确认**：
- 产品经理确认规范完整正确
- 技术负责人确认规范可实现
- 所有涉众确认规范符合要求

---

## BDD格式说明

详见：[`references/bdd-format.md`](./references/bdd-format.md)

---

## 规范质量检查清单

详见：[`references/quality-checklist.md`](./references/quality-checklist.md)

---

## 关键约束

- **BDD格式严格** - 所有场景必须使用WHEN-THEN格式
- **多轮确认** - 规范必须经过至少3轮确认
- **完整性优先** - 宁可详细也不要遗漏
- **可测试性** - 所有规范都应该能自动化测试
- **无歧义** - 避免模糊的词语，明确所有细节
- **输出路径** - 必须输出到 `spec-dev/[需求ID]/spec/` 目录

---

## 最佳实践

1. **充分沟通** - 与产品、业务、技术多轮沟通确认
2. **场景优先** - 先定义场景，再细化细节
3. **例子优先** - 用具体的例子说明，避免抽象的描述
4. **边界意识** - 特别关注异常、边界、特殊情况
5. **前向参考** - 考虑未来的可能扩展和演进

---

**规范的质量决定了后续开发的效率和质量。花足够的时间在规范阶段，可以大幅减少后续的变更和返工。** 🎯
