# 全栈项目数据库优先 + 接口对齐驱动流程 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 sdd-tdd-dev 插件中新增 api-contract 阶段，并增强 code-architect、code-designer、code-task、code-execute、code-test 六个节点，实现全栈项目的数据库优先设计、接口契约对齐、以及端到端契约测试。

**Architecture:** 在现有 code-designer 和 code-task 之间插入 api-contract 阶段；在 code-architect agent 中新增全栈识别和代码库约定扫描能力；在各 SKILL.md 中注入 HARD-GATE 约束和 provides/consumes 依赖关系；在 code-test 中新增契约测试。

**Tech Stack:** Markdown (SKILL.md 文档), YAML (frontmatter), Shell (测试脚本)

---

## 文件结构映射

### 新增文件

| 文件 | 职责 |
|------|------|
| `plugins/sdd-tdd-dev/skills/api-contract/SKILL.md` | api-contract 阶段 SKILL 定义 |
| `plugins/sdd-tdd-dev/skills/api-contract/README.md` | api-contract 阶段使用说明 |
| `plugins/sdd-tdd-dev/skills/api-contract/templates/api-contract-template.md` | api-contract.md 模板 |
| `plugins/sdd-tdd-dev/skills/api-contract/templates/review-report-template.md` | review-report.md 模板 |
| `plugins/sdd-tdd-dev/skills/code-test/templates/contract-test-template.md` | 契约测试模板 |
| `plugins/sdd-tdd-dev/docs/WORKFLOW-FULLSTACK.md` | 全栈项目完整工作流文档 |

### 修改文件

| 文件 | 改动内容 |
|------|---------|
| `plugins/sdd-tdd-dev/skills/spec-creation/templates/requirement-template.md` | 新增 project-mode 和 needs-database 字段 |
| `plugins/sdd-tdd-dev/skills/spec-creation/SKILL.md` | 新增 project-mode 和 needs-database 判定逻辑 |
| `plugins/sdd-tdd-dev/agents/code-architect.md` | 新增全栈场景识别和代码库约定扫描能力 |
| `plugins/sdd-tdd-dev/skills/code-designer/SKILL.md` | 新增 fullstack 模式下的 HARD-GATE（数据库设计、接口清单初稿） |
| `plugins/sdd-tdd-dev/skills/code-designer/templates/design-template.md` | 新增「现有项目约定」和「接口清单初稿」章节 |
| `plugins/sdd-tdd-dev/skills/code-task/SKILL.md` | 新增 api-contract 前置检查、provides/consumes 依赖关系 |
| `plugins/sdd-tdd-dev/skills/code-task/templates/tasks-document-template.md` | 新增 provides/consumes 标记、接口对齐验证结果章节 |
| `plugins/sdd-tdd-dev/skills/code-task/templates/task-template.md` | 新增 provides/consumes 字段 |
| `plugins/sdd-tdd-dev/skills/code-execute/SKILL.md` | 新增 Task 执行前接口契约验证、版本锁定 |
| `plugins/sdd-tdd-dev/skills/code-execute/prompts/tdd-implementer-prompt.md` | 新增 api-contract 遵循约束 |
| `plugins/sdd-tdd-dev/skills/code-test/SKILL.md` | 新增契约测试步骤 |
| `plugins/sdd-tdd-dev/skills/package.json` | 新增 api-contract skill 声明，更新 workflow |

---

## Task 1：新增 api-contract 阶段 SKILL

**Files:**
- Create: `plugins/sdd-tdd-dev/skills/api-contract/SKILL.md`
- Create: `plugins/sdd-tdd-dev/skills/api-contract/README.md`
- Create: `plugins/sdd-tdd-dev/skills/api-contract/templates/api-contract-template.md`
- Create: `plugins/sdd-tdd-dev/skills/api-contract/templates/review-report-template.md`

- [ ] **Step 1: 创建 SKILL.md**

```markdown
---
name: api-contract
description: >
  第2.5步：接口契约定义与审查（Api-Contract 阶段）

  输入：spec-dev/{requirement_desc_abstract}/design/design.md
  输出：spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md + review-report.md

  功能：将设计阶段的接口清单初稿转化为正式的接口契约文档，进行接口审查，
  确保前后端接口定义一致、命名规范、类型匹配、错误处理完整。
  仅在 project-mode: fullstack 时触发。
---

# API Contract 阶段

## 职责

读取 `design.md` 中的接口清单初稿和现有项目约定，生成正式的 `api-contract.md`（接口契约定义）和 `review-report.md`（接口审查报告）。

本阶段位于 code-designer 和 code-task 之间，是 fullstack 项目的强制环节。

## 工作流程

### Phase 1：读取设计文档

1. 读取 `spec-dev/{requirement_desc_abstract}/spec/requirement.md`，确认 `project-mode` 和 `needs-database` 字段
2. 读取 `spec-dev/{requirement_desc_abstract}/design/design.md`，提取：
   - 现有项目约定（URL 风格、响应格式、认证方式、中间件）
   - 接口清单初稿（接口路径、HTTP 方法、简要描述）
   - 数据库设计（如果需要数据库，提取表结构以辅助接口设计）
3. 读取 `spec-dev/{requirement_desc_abstract}/spec/scenarios/*.md`，提取所有 WHEN-THEN 场景中的接口交互需求

### Phase 2：生成接口契约文档

基于设计文档和场景，使用 `templates/api-contract-template.md` 生成 `api-contract.md`：

1. **现有项目约定**（来自 code-designer 输出）：
   - URL 风格（基础路径前缀、命名规范）
   - 响应格式（成功/失败结构）
   - 认证方式（Token/Cookie/Session）
   - 已有中间件/工具（日志、缓存、异常处理）

2. **接口列表**（每个接口包含）：
   - 接口路径和 HTTP 方法
   - 功能描述
   - 认证要求（是/否）
   - 性能要求（响应时间、并发限制）
   - 请求参数表（参数名、位置、类型、必填、校验规则、示例值、说明）
   - 成功响应 JSON 示例
   - 错误响应表（错误码、HTTP 状态码、说明、触发条件）
   - 关联数据库表（如果需要数据库）

3. **版本管理**：
   - 初始版本号为 `v1`
   - 版本历史表格

### Phase 3：接口审查

基于以下 5 个维度进行审查，使用 `templates/review-report-template.md` 生成 `review-report.md`：

1. **完整性**：每个前端交互是否都有对应的后端接口？
   - 遍历所有 spec 场景，确认每个 WHEN（用户操作）都有对应的 THEN（系统响应）背后的接口支持

2. **命名规范**：RESTful 风格一致性
   - URL 命名是否遵循现有项目的命名规范？
   - 资源名是否使用复数小写？
   - 嵌套资源是否使用 `/resource/{id}/sub-resource` 格式？

3. **类型一致性**：前端期望类型与后端返回类型匹配
   - 请求参数类型是否与后端期望类型一致？
   - 响应字段类型是否与前端期望类型一致？

4. **错误处理**：每个接口是否定义了错误码和错误信息
   - 每个接口是否定义了至少一个错误响应？
   - 错误码是否遵循现有项目的错误码体系？

5. **项目约定遵循**：新接口是否符合现有项目的接口风格约定
   - 响应格式是否一致？
   - 认证方式是否一致？
   - 分页格式是否一致？

### Phase 4：审查判定

- **通过**：5 项全部通过 → 输出 `review-report.md`（状态：通过），允许进入 code-task 阶段
- **不通过**：任一项不通过 → 输出 `review-report.md`（状态：不通过），列出不通过项和修复建议，拒绝进入 code-task 阶段

<HARD-GATE>接口审查不通过 → 不生成 review-report.md 的通过状态，不允许进入 code-task 阶段</HARD-GATE>

## 快速模式适配

当 `mode: quick` 时：
- api-contract 阶段简化为：从 spec 场景中提取接口清单，生成精简版 api-contract.md（仅包含路径、方法、核心参数）
- 审查范围缩小为完整性和错误处理两项

## 关键约束

- ✅ 必须：每个接口必须有明确的请求参数和响应结构定义
- ✅ 必须：每个接口必须定义错误码和错误信息
- ✅ 必须：必须遵循现有项目的接口风格约定
- ❌ 禁止：定义未在 spec 场景中出现的接口
- ❌ 禁止：跳过接口审查步骤
- ❌ 禁止：在审查不通过的情况下进入 code-task 阶段
```

- [ ] **Step 2: 创建 README.md**

```markdown
# API Contract 阶段

## 概述

API Contract 阶段是 sdd-tdd-dev 插件处理全栈项目的核心环节，位于 code-designer 和 code-task 之间。

## 触发条件

当 `requirement.md` 中的 `project-mode` 为 `fullstack` 时自动触发。

## 输入输出

**输入**：
- `spec-dev/{req}/design/design.md`（含现有项目约定和接口清单初稿）
- `spec-dev/{req}/spec/scenarios/*.md`（BDD 场景）

**输出**：
- `spec-dev/{req}/api-contract/api-contract.md`（接口契约定义）
- `spec-dev/{req}/api-contract/review-report.md`（接口审查报告）

## 使用方式

在 code-designer 阶段完成后，自动进入 api-contract 阶段：

```
/code-designer → /api-contract → /code-task
```

## 审查维度

1. 完整性 - 每个前端交互都有对应后端接口
2. 命名规范 - RESTful 风格一致性
3. 类型一致性 - 前后端类型匹配
4. 错误处理 - 错误码完整
5. 项目约定遵循 - 符合现有项目风格

## 版本管理

api-contract.md 带版本号（初始 v1），后续接口变更必须更新版本号并记录变更日志。
```

- [ ] **Step 3: 创建 api-contract-template.md**

```markdown
# API Contract - [需求名称]

**版本**: v1
**生成日期**: YYYY-MM-DD
**关联设计**: spec-dev/{requirement_desc_abstract}/design/design.md

## 现有项目约定

### URL 风格
- 基础路径前缀：[例如 /api/v1]
- 命名规范：[例如 复数小写 /api/v1/users]

### 响应格式
- 成功：`{ code: 0, data: {...}, message: "success" }`
- 失败：`{ code: xxx, message: "错误描述", data: null }`

### 认证方式
- [例如 Header: Authorization: Bearer {token}]

### 已有中间件/工具
- [例如 全局异常处理中间件]
- [例如 请求日志中间件]
- [例如 Redis 缓存策略]

## 接口列表

### {METHOD} {path}

**描述**: [接口功能描述]
**认证**: [是/否]
**性能要求**: [响应时间 < Xms]

**请求参数**:

| 参数名 | 位置 | 类型 | 必填 | 校验规则 | 示例值 | 说明 |
|--------|------|------|------|---------|--------|------|
| | | | | | | |

**成功响应**:

```json
{
  "code": 0,
  "data": {},
  "message": "success"
}
```

**错误响应**:

| 错误码 | HTTP 状态码 | 说明 | 触发条件 |
|--------|------------|------|---------|
| | | | |

---

（按上述格式重复定义每个接口）

## 版本历史

| 版本 | 日期 | 变更内容 | 变更原因 |
|------|------|---------|--------|
| v1   | YYYY-MM-DD | 初始版本 | - |
```

- [ ] **Step 4: 创建 review-report-template.md**

```markdown
# 接口审查报告 - [需求名称]

**审查日期**: YYYY-MM-DD
**审查状态**: [通过/不通过]
**关联契约**: spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md

## 审查维度

### 1. 完整性

**检查项**: 每个前端交互是否都有对应的后端接口？
**结果**: [通过/不通过]
**详情**:
- [ ] 场景 [场景ID] 的用户操作 [描述] → 后端接口 [路径] ✅
- [ ] 场景 [场景ID] 的用户操作 [描述] → 缺少对应接口 ❌

### 2. 命名规范

**检查项**: RESTful 风格一致性、URL 命名规范
**结果**: [通过/不通过]
**详情**:
- [ ] 接口 [路径] 遵循现有命名规范 ✅
- [ ] 接口 [路径] 命名不规范（期望: [期望路径]）❌

### 3. 类型一致性

**检查项**: 前端期望类型与后端返回类型匹配
**结果**: [通过/不通过]
**详情**:
- [ ] 接口 [路径] 的请求参数类型一致 ✅
- [ ] 接口 [路径] 的响应字段类型不匹配（前端期望 [类型A]，后端返回 [类型B]）❌

### 4. 错误处理

**检查项**: 每个接口是否定义了错误码和错误信息
**结果**: [通过/不通过]
**详情**:
- [ ] 接口 [路径] 定义了错误码 ✅
- [ ] 接口 [路径] 缺少错误码定义 ❌

### 5. 项目约定遵循

**检查项**: 新接口是否符合现有项目的接口风格约定
**结果**: [通过/不通过]
**详情**:
- [ ] 响应格式与现有项目一致 ✅
- [ ] 认证方式与现有项目一致 ✅
- [ ] 分页格式与现有项目一致 ✅

## 不通过项汇总

| 维度 | 不通过项 | 修复建议 |
|------|---------|---------|
| | | |

## 审查结论

[ ] 通过 - 允许进入 code-task 阶段
[ ] 不通过 - 请修复上述不通过项后重新审查
```

- [ ] **Step 5: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/api-contract/
git commit -m "feat(sdd-tdd-dev): 新增 api-contract 阶段 SKILL 和模板"
```

---

## Task 2：增强 code-architect agent

**Files:**
- Modify: `plugins/sdd-tdd-dev/agents/code-architect.md`

- [ ] **Step 1: 读取现有 code-architect.md**

当前文件 114 行，包含核心职责、后端/全栈场景、前端/全栈场景、核心流程、输出指导。

- [ ] **Step 2: 在核心职责后新增「全栈场景识别」章节**

在 `## 核心职责` 后、`## 核心流程` 前，插入：

```markdown
## 全栈场景识别

### 识别规则

分析 spec 文档（`spec-dev/{requirement_desc_abstract}/spec/`），自动判断项目模式：

| 条件 | 判定 |
|------|------|
| spec 场景中包含 UI/前端交互（页面、组件、表单、按钮等）且同时包含后端逻辑/数据存储 | `project-mode: fullstack` |
| 仅包含 UI/前端交互，不涉及后端 | `project-mode: frontend` |
| 仅包含后端逻辑/数据存储，不涉及 UI | `project-mode: backend` |

### 数据库需求判定

| 条件 | 判定 |
|------|------|
| 需求涉及数据持久化（新建/修改数据库表、存储业务数据） | `needs-database: true` |
| 需求仅涉及前端 + 已有第三方 API 对接，无新数据存储 | `needs-database: false` |

判定结果写入 `spec-dev/{requirement_desc_abstract}/spec/requirement.md` 的 `project-mode` 和 `needs-database` 字段。

## 现有代码库约定扫描

在 fullstack 模式下，必须执行代码库约定扫描，输出以下信息：

### 接口风格扫描

- **URL 命名风格**：扫描已有接口的 URL 模式（如 `/api/users`、`/api/v1/user/list`）
- **HTTP 方法约定**：RESTful（GET/POST/PUT/DELETE）vs 统一 POST
- **响应格式**：扫描已有接口的响应结构（如 `{ code, data, message }`）
- **分页格式**：`{ page, size, total, items }` vs cursor-based
- **错误码体系**：HTTP 状态码 vs 业务错误码

### 认证/授权模式扫描

- Token 传递方式（Header `Authorization: Bearer xxx` vs Cookie vs Query）
- 权限检查位置（中间件层 vs 控制器层 vs 服务层）

### 中间件/基础设施扫描

- 已有的日志中间件、限流中间件、缓存策略
- 已有的数据库连接池、ORM 配置、迁移工具

扫描结果写入 design.md 的「现有项目约定」章节。
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/agents/code-architect.md
git commit -m "feat(sdd-tdd-dev): code-architect agent 新增全栈场景识别和代码库约定扫描"
```

---

## Task 3：增强 spec-creation 阶段

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/spec-creation/templates/requirement-template.md`
- Modify: `plugins/sdd-tdd-dev/skills/spec-creation/SKILL.md`

- [ ] **Step 1: 修改 requirement-template.md**

在模板的 Header 部分（版本号之后）新增：

```markdown
## 项目模式

**project-mode**: [frontend/backend/fullstack]
**needs-database**: [true/false]

> 注：project-mode 和 needs-database 由 code-architect agent 在 code-designer 阶段自动识别和设置。
```

- [ ] **Step 2: 修改 SKILL.md**

在 `### 阶段3: 规范细化和验证`（第 198 行）的「细化维度」列表中，第 15 项之后新增第 16 项：

```markdown
16. **项目模式识别** - 判断 project-mode（frontend/backend/fullstack）、needs-database（true/false）
```

在 `### 阶段4: 生成规范文档`（第 233 行）的「输出结构」代码块中，在 `requirement.md` 行之后新增说明注释：

```markdown
        ├── requirement.md            # 【入口文件】规范总览和导航索引（含 project-mode 和 needs-database 字段）
```

在「模式确认输出」的代码块（第 90-98 行）中，在 `- 模式：[standard | quick]` 之后新增：

```markdown
- 项目模式：[frontend | backend | fullstack]
- 需要数据库：[true | false]
```

在 `### 阶段1: 需求理解和初步分析`（第 116 行）的「活动」列表中新增：

```markdown
- 识别项目模式（frontend/backend/fullstack）
- 识别数据库需求（needs-database: true/false）
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/spec-creation/
git commit -m "feat(sdd-tdd-dev): spec-creation 新增项目模式识别"
```

---

## Task 4：增强 code-designer 阶段

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/code-designer/SKILL.md`
- Modify: `plugins/sdd-tdd-dev/skills/code-designer/templates/design-template.md`

- [ ] **Step 1: 修改 code-designer SKILL.md**

在 SKILL.md 中找到「关键约束」或 Phase 5（输出阶段），在现有 HARD-GATE 之后新增：

```markdown
### 全栈模式约束

<HARD-GATE>当 requirement.md 中 project-mode 为 fullstack 时，design.md 中必须包含「现有项目约定」和「接口清单初稿」章节；当 needs-database 为 true 时，还必须包含「数据库设计」章节。未完成这些强制章节之前，不输出最终的设计方案文档。</HARD-GATE>
```

在 Phase 2（Agent 分析）描述中新增：

```markdown
- **全栈场景识别**（code-architect agent）：
  - 自动判断 project-mode（frontend/backend/fullstack）
  - 自动判断 needs-database（true/false）
  - 将判定结果写入 requirement.md

- **现有代码库约定扫描**（fullstack 模式必须执行）：
  - 扫描已有接口的 URL 命名风格
  - 识别响应格式约定
  - 识别认证/授权模式
  - 识别已有中间件、工具、公共函数
  - 识别已有的数据库 ORM 配置和迁移工具
  - 扫描结果写入 design.md 的「现有项目约定」章节
```

- [ ] **Step 2: 修改 design-template.md**

在 design-template.md 中，在 `## 二、设计方案` 下的 `2.5 服务上下游设计` 之前（即最前面的位置），新增两个章节：

```markdown
### 2.X 现有项目约定

> 由 code-architect agent 扫描现有代码库后填入，fullstack 模式必填。

#### URL 风格
- 基础路径前缀：
- 命名规范：

#### 响应格式
- 成功：
- 失败：

#### 认证方式
-

#### 已有中间件/工具
-

### 2.Y 接口清单初稿

> fullstack 模式必填。作为 api-contract 阶段的输入基础。

| 接口路径 | HTTP 方法 | 功能描述 | 关联数据库表 |
|---------|----------|---------|-------------|
| | | | |

简要说明：
- 请求/响应描述：
- 与数据库表的映射关系：
```

将上述章节编号调整为实际的 2.4 和 2.5（放在现有的 2.5 服务上下游设计之前，服务上下游设计顺延为 2.6）。

同时，在现有的 `### 2.6 数据库设计` 部分（会顺延为 2.7），保持其内容不变，但新增条件标记：

```markdown
### 2.7 数据库设计（needs-database: true 时必填）

> 当 needs-database 为 false 时可跳过此章节。
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/code-designer/
git commit -m "feat(sdd-tdd-dev): code-designer 新增全栈模式约束和模板章节"
```

---

## Task 5：增强 code-task 阶段

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/code-task/SKILL.md`
- Modify: `plugins/sdd-tdd-dev/skills/code-task/templates/tasks-document-template.md`
- Modify: `plugins/sdd-tdd-dev/skills/code-task/templates/task-template.md`

- [ ] **Step 1: 修改 code-task SKILL.md**

在 SKILL.md 的「工作流程」Step 1（分析设计文档）之后，新增 Step 1.5：

```markdown
### Step 1.5：接口契约前置检查（fullstack 模式强制）

<HARD-GATE>当 project-mode 为 fullstack 时，生成 tasks.md 前必须验证：1) api-contract.md 已存在于 spec-dev/{requirement_desc_abstract}/api-contract/；2) review-report.md 已存在且审查状态为「通过」。验证失败则拒绝生成 tasks.md，输出差异报告并提示回到 api-contract 阶段修复。</HARD-GATE>

**检查项**：

1. `api-contract.md` 文件存在性
2. `review-report.md` 文件存在性
3. 审查状态为「通过」
4. 每个前端 Task 的接口依赖都在 api-contract.md 中定义
5. 每个后端 Task 的接口实现都在 api-contract.md 中定义
6. 同一接口的前端依赖定义与后端实现定义完全一致

**对齐验证失败处理**：

如果检查不通过：
1. 输出差异报告（路径不一致、参数类型不匹配、缺失字段、错误码不匹配）
2. **拒绝生成 tasks.md**
3. 提示回到 api-contract 阶段修复
```

在 Step 3（定义任务详情）中，为 Task 定义新增字段：

```markdown
每个 Task 必须包含以下接口标记：

- **provides**：该 Task 实现的接口列表（后端 Task）
  - 示例：`provides: [GET /api/users, POST /api/users]`
- **consumes**：该 Task 依赖的接口列表（前端 Task）
  - 示例：`consumes: [GET /api/users]`

系统自动生成 Task 依赖图：
- 消费方 Task 必须在提供方 Task 完成后才能执行
- 依赖图写入 tasks.md 的「Task 依赖关系」章节
```

- [ ] **Step 2: 修改 tasks-document-template.md**

在现有的执行模式标签（模式/TDD要求/子代理要求/测试覆盖要求）之后，新增：

```markdown
**接口契约版本**: [v1]
**接口对齐验证**: [通过/不通过]

### Task 依赖关系

| Task ID | 接口标记 | 类型 | 依赖 Task ID | 说明 |
|---------|---------|------|-------------|------|
| T-001 | provides: [GET /api/users] | 后端 | - | 用户列表接口实现 |
| T-003 | consumes: [GET /api/users] | 前端 | T-001 | 用户列表页面展示 |
```

- [ ] **Step 3: 修改 task-template.md**

读取现有的 `plugins/sdd-tdd-dev/skills/code-task/templates/task-template.md`，在 Task 基本信息区域新增：

```markdown
**接口标记**:
- provides: [接口列表]（后端 Task）
- consumes: [接口列表]（前端 Task）
```

- [ ] **Step 4: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/code-task/
git commit -m "feat(sdd-tdd-dev): code-task 新增接口契约前置检查和 provides/consumes 依赖"
```

---

## Task 6：增强 code-execute 阶段

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/code-execute/SKILL.md`
- Modify: `plugins/sdd-tdd-dev/skills/code-execute/prompts/tdd-implementer-prompt.md`

- [ ] **Step 1: 修改 code-execute SKILL.md**

在 Step 2（并行执行Task批次）的 Step 3.1（创建 Task 独立 Worktree）之后，新增 Step 3.1.5：

```markdown
### Step 3.1.5：接口契约验证（fullstack 模式强制）

<HARD-GATE>当 project-mode 为 fullstack 时，每个 Task 启动时必须验证其 provides 或 consumes 标记的接口在 api-contract.md 中存在。验证失败则该 Task 拒绝执行并报错。</HARD-GATE>

**验证步骤**：

1. 读取 Task 的 provides/consumes 标记
2. 在 `spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md` 中查找对应接口定义
3. 如果接口不存在，拒绝执行并输出错误：`接口 [METHOD /path] 未在 api-contract.md 中定义`
4. 记录当前 api-contract.md 版本号到执行日志

**版本锁定**：

- Task 执行期间锁定 api-contract.md 的版本号
- 如需接口变更，必须先更新版本号、重新触发接口审查、通知所有相关 Task 重新审查
```

在 Step 4（生成执行报告）中，新增报告字段：

```markdown
执行报告中新增以下字段：
- **接口契约版本号**: [v1]
- **接口契约验证日志**: 每个 Task 的接口验证结果
```

- [ ] **Step 2: 修改 tdd-implementer-prompt.md**

在现有的「代码完整性检查」部分（5 forbidden patterns）之后，新增：

```markdown
### 接口契约遵循约束（fullstack 模式）

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
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/code-execute/
git commit -m "feat(sdd-tdd-dev): code-execute 新增接口契约验证和遵循约束"
```

---

## Task 7：增强 code-test 阶段

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/code-test/SKILL.md`
- Create: `plugins/sdd-tdd-dev/skills/code-test/templates/contract-test-template.md`

- [ ] **Step 1: 创建 contract-test-template.md**

```markdown
# 接口契约测试模板

> 基于 api-contract.md 自动生成，用于验证实现与接口契约的一致性。

## 后端契约测试

### {METHOD} {path}

**测试文件**: `tests/contract/{normalized-path}.test.{ext}`

#### 成功响应测试

```typescript
// 验证响应字段、类型、结构与 api-contract.md 一致
describe('{METHOD} {path} - 成功响应', () => {
  it('应返回正确的字段和类型', async () => {
    const response = await request(app)
      .{method}('{path}')
      .expect(200);

    expect(response.body).toHaveProperty('code', 0);
    expect(response.body).toHaveProperty('data');
    expect(response.body).toHaveProperty('message');
    // 按 api-contract.md 定义逐项验证字段
  });
});
```

#### 错误响应测试

// 按 api-contract.md 中定义的错误码逐一测试
// 每个错误码对应一个测试用例

#### 边界条件测试

// 空值、超长字符串、特殊字符等

---

（按上述格式重复每个接口）
```

- [ ] **Step 2: 修改 code-test SKILL.md**

在 Step 3（高层测试）中，在现有的 3.1 集成测试 之后，新增 3.1.5 契约测试：

```markdown
### 3.1.5 接口契约测试（fullstack 模式强制）

<HARD-GATE>当 project-mode 为 fullstack 时，必须执行接口契约测试，验证后端实际返回的字段、类型、结构与 api-contract.md 定义一致。契约测试失败意味着实现与接口契约不一致，必须修复后重新运行测试，不允许跳过。</HARD-GATE>

**测试内容**：

| 测试类型 | 验证内容 |
|---------|---------|
| 契约测试（后端） | 后端实际返回的字段、类型、结构与 api-contract.md 定义一致 |
| 契约测试（前端） | 前端发送的请求格式与 api-contract.md 定义一致 |
| 错误码覆盖测试 | api-contract.md 中定义的所有错误码在后端都有对应实现 |
| 边界条件测试 | 空值、超长字符串、特殊字符等边界情况的接口行为符合契约定义 |

**测试生成**：

基于 `api-contract.md` 和 `templates/contract-test-template.md` 自动生成：
- 后端契约测试用例：针对每个接口，验证响应字段、类型、错误码
- 前端集成测试用例：针对每个接口调用，验证请求格式、响应解析

**失败处理**：
- 契约测试失败 → 输出详细差异报告（期望值 vs 实际值）
- 必须修复后重新运行测试
- 不允许跳过契约测试
```

在 Step 5（生成测试报告）中，新增报告字段：

```markdown
测试报告中新增：
- **接口契约测试结果**: 每个接口的契约测试通过/失败情况
- **契约一致性评分**: 通过数 / 总接口数
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/code-test/
git commit -m "feat(sdd-tdd-dev): code-test 新增接口契约测试"
```

---

## Task 8：更新 package.json 和 workflow 文档

**Files:**
- Modify: `plugins/sdd-tdd-dev/skills/package.json`
- Create: `plugins/sdd-tdd-dev/docs/WORKFLOW-FULLSTACK.md`

- [ ] **Step 1: 修改 package.json**

在 skills 数组中，在 code-designer（order=2）和 code-task（order=3）之间新增：

```json
{
  "name": "api-contract",
  "path": "skills/api-contract",
  "order": 2.5,
  "description": "接口契约定义与审查阶段。将设计阶段的接口清单转化为正式的接口契约文档，进行完整性、命名规范、类型一致性、错误处理、项目约定遵循五个维度的审查。仅在 fullstack 模式下触发。",
  "supports": ["frontend", "backend", "fullstack"],
  "input": "spec-dev/{requirement_desc_abstract}/design/design.md",
  "output": "spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md + review-report.md",
  "dependencies": ["code-designer"],
  "key_features": [
    "接口契约定义（路径、方法、参数、响应、错误码）",
    "五维度接口审查（完整性、命名规范、类型一致性、错误处理、项目约定遵循）",
    "版本管理（版本号 + 变更日志）",
    "fullstack 模式自动触发",
    "快速模式适配"
  ]
}
```

更新 workflow 描述，将原来的 5 步流程改为 6 步：

```json
"workflow": [
  { "step": 1, "skill": "spec-creation", "description": "需求分析和规范生成" },
  { "step": 2, "skill": "code-designer", "description": "架构和技术设计（含全栈场景识别和接口清单初稿）" },
  { "step": 2.5, "skill": "api-contract", "description": "接口契约定义与审查（fullstack 强制）" },
  { "step": 3, "skill": "code-task", "description": "任务分解（含接口对齐验证和 provides/consumes 依赖）" },
  { "step": 4, "skill": "code-execute", "description": "代码执行（含接口契约验证和版本锁定）" },
  { "step": 5, "skill": "code-test", "description": "测试验证（含接口契约测试）" }
]
```

- [ ] **Step 2: 创建 WORKFLOW-FULLSTACK.md**

```markdown
# 全栈项目完整工作流

## 概述

本文档描述 sdd-tdd-dev 插件处理全栈项目（project-mode: fullstack）的完整流程。

## 流程图

```
spec-creation
  ↓ 识别 project-mode: fullstack, needs-database: true/false
code-designer
  ↓ [条件] 如需数据库：输出数据库设计
  ↓ 必须：现有项目约定扫描 + 接口清单初稿
api-contract (NEW)
  ↓ 生成 api-contract.md（接口契约定义）
  ↓ 生成 review-report.md（接口审查报告）
  ↓ HARD-GATE: 审查不通过则拒绝进入 code-task
code-task
  ↓ HARD-GATE: api-contract.md 存在性 + 审查通过验证
  ↓ 生成 provides/consumes 依赖关系
code-execute
  ↓ 每个 Task 执行前验证接口契约存在性
  ↓ 严格按 api-contract.md 实现/调用
code-test
  ↓ 接口契约测试（后端返回值 vs 契约定义）
  ↓ 契约测试不通过 → 修复 → 重测
```

## 快速模式适配

当 mode: quick 时，api-contract 阶段简化为从 spec 场景中提取接口清单生成精简版契约，审查范围缩小为完整性和错误处理。

## 向后兼容

- frontend/backend 项目自动跳过 api-contract 阶段
- 已有项目流程不受影响
```

- [ ] **Step 3: 提交**

```bash
git add plugins/sdd-tdd-dev/skills/package.json plugins/sdd-tdd-dev/docs/WORKFLOW-FULLSTACK.md
git commit -m "feat(sdd-tdd-dev): 更新 package.json 和新增全栈工作流文档"
```

---

## Task 9：自审和集成验证

**Files:**
- Read: 所有新增和修改的文件

- [ ] **Step 1: 规范覆盖自审**

逐条检查设计文档中的要求，确认每个要求都有对应的 Task 实现：

| 设计要求 | 对应 Task | 状态 |
|---------|----------|------|
| 全栈场景自动识别 | Task 2 (code-architect) | ☐ |
| 数据库需求判定 | Task 2 (code-architect) | ☐ |
| project-mode/needs-database 写入 requirement.md | Task 3 (spec-creation) | ☐ |
| code-designer 数据库设计条件触发 | Task 4 (code-designer) | ☐ |
| code-designer 接口清单初稿 | Task 4 (code-designer) | ☐ |
| code-designer 现有项目约定扫描 | Task 2 + Task 4 | ☐ |
| api-contract 阶段新增 | Task 1 | ☐ |
| api-contract.md 模板 | Task 1 | ☐ |
| review-report.md 模板 | Task 1 | ☐ |
| 五维度接口审查 | Task 1 | ☐ |
| 版本管理 | Task 1 | ☐ |
| code-task 接口对齐强制检查 | Task 5 | ☐ |
| provides/consumes 依赖关系 | Task 5 | ☐ |
| code-execute 接口契约验证 | Task 6 | ☐ |
| code-execute 版本锁定 | Task 6 | ☐ |
| code-test 契约测试 | Task 7 | ☐ |
| package.json 更新 | Task 8 | ☐ |
| 快速模式适配 | Task 1, 5, 6, 7 | ☐ |
| 向后兼容 | Task 4, 5, 6, 7, 8 | ☐ |

- [ ] **Step 2: 类型/命名一致性检查**

确认所有 Task 中使用一致的命名：
- `api-contract.md` 文件路径一致
- `review-report.md` 文件路径一致
- `provides` / `consumes` 字段名一致
- `project-mode` / `needs-database` 字段名一致
- HARD-GATE 语法格式一致（`<HARD-GATE>...</HARD-GATE>`）

- [ ] **Step 3: 最终提交**

```bash
git add -A
git commit -m "docs(sdd-tdd-dev): 全栈项目接口对齐驱动流程 - 自审和集成验证"
```
