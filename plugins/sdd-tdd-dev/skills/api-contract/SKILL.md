---
name: api-contract
description: |
  第2.5步：接口契约定义与审查（Api-Contract 阶段）

  输入：spec-dev/{requirement_desc_abstract}/design/design.md
  输出：spec-dev/{requirement_desc_abstract}/api-contract/api-contract.md + review-report.md

  功能：将设计阶段的接口清单初稿转化为正式的接口契约文档，进行接口审查，
  确保前后端接口定义一致、命名规范、类型匹配、错误处理完整。
  仅在 project-mode: fullstack 时触发。

  核心特性：
  - ✅ 接口契约定义（路径、方法、参数、响应、错误码）
  - ✅ 五维度接口审查（完整性、命名规范、类型一致性、错误处理、项目约定遵循）
  - ✅ 版本管理（版本号 + 变更日志）
  - ✅ fullstack 模式自动触发
  - ✅ 快速模式适配

  前置Skill：code-designer ✓
  下一步：/code-task
---

# API Contract 阶段

## 职责

读取 `design.md` 中的接口清单初稿和现有项目约定，生成正式的 `api-contract.md`（接口契约定义）和 `review-report.md`（接口审查报告）。

本阶段位于 code-designer 和 code-task 之间，是 fullstack 项目的强制环节。

**输出路径**：`spec-dev/{requirement_desc_abstract}/api-contract/`

## 何时使用

**前置条件**：
- ✅ code-designer 已完成，design.md 已生成
- ✅ requirement.md 中 `project-mode: fullstack`
- ✅ design.md 中包含「现有项目约定」和「接口清单初稿」章节

### 快速模式适配

当 `mode: quick` 时：
- api-contract 阶段简化为：从 spec 场景中提取接口清单，生成精简版 api-contract.md（仅包含路径、方法、核心参数）
- 审查范围缩小为完整性和错误处理两项
- 审查通过标准：2 项全部通过

---

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

基于以下 6 个维度进行审查，使用 `templates/review-report-template.md` 生成 `review-report.md`：

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

6. **🆕 数据库依赖验证**：接口涉及的表/字段是否在 sql-ddl.md 中定义
   - 每个接口返回的字段是否都有对应的数据库表/字段定义？
   - 接口中的查询条件是否有对应的数据库索引支持？
   - 接口的事务需求是否与数据库设计匹配？

### Phase 4：审查判定

- **通过**：6 项全部通过 → 输出 `review-report.md`（状态：通过），允许进入 code-task 阶段
- **不通过**：任一项不通过 → 输出 `review-report.md`（状态：不通过），列出不通过项和修复建议，拒绝进入 code-task 阶段

### Phase 5：跨项目契约验证 ⭐🆕（多项目场景）

当 requirement.md 中协作模式非 single 时，额外执行：

1. **跨项目接口清单验证**：
   - 遍历 collaboration-plan.md 中的跨项目接口清单
   - 确认每个接口都在 api-contract.md 中有完整定义
   - 确认提供方和消费方项目对同一接口的定义一致

2. **版本兼容性验证**：
   - 跨项目调用的接口版本号是否兼容
   - 消费方项目的版本要求是否在提供方支持范围内

3. **契约变更影响分析**：
   - 接口变更对下游项目的影响范围
   - 是否需要通知其他项目负责人

<HARD-GATE>接口审查不通过 → 不生成 review-report.md 的通过状态，不允许进入 code-task 阶段</HARD-GATE>

---

## 关键约束

✅ **必须做**：
- 每个接口必须有明确的请求参数和响应结构定义
- 每个接口必须定义错误码和错误信息
- 必须遵循现有项目的接口风格约定
- fullstack 模式下必须执行接口审查

❌ **禁止做**：
- 定义未在 spec 场景中出现的接口
- 跳过接口审查步骤
- 在审查不通过的情况下进入 code-task 阶段
- 修改接口契约而不更新版本号

---

## 相关资源

| 资源 | 说明 |
|------|------|
| `templates/api-contract-template.md` | 接口契约文档模板 |
| `templates/review-report-template.md` | 接口审查报告模板 |
