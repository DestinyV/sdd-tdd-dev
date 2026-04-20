---
name: code-task
description: |
  第3步：任务列表生成（Task阶段）

  输入：spec-dev/{requirement_desc_abstract}/design/design.md
  输出：spec-dev/{requirement_desc_abstract}/tasks/tasks.md

  功能：将设计方案转换为详细的代码级别任务列表，为开发者定义具体的实现任务和交付物。
  支持全栈开发的各类任务拆解（前端、后端、数据库、微服务等）。

  核心特性：
  - ✅ 任务拆解和优先级规划
  - ✅ 并行执行计划（基于依赖关系）
  - ✅ 测试用例映射（Test Case Mapping）
  - ✅ Mock数据关联（来自spec规范）
  - ✅ 支持多种任务类型（前端、后端、数据库、微服务等）

  前置Skill：code-designer ✓
  下一步：/code-execute 或 /test-design (TDD流程)
---

# code-task

## 职责

根据设计方案，进行代码级别的任务拆解和定义。

**核心流程**：
1. 读取 `design.md` 中的设计方案
2. 分解设计为具体的编码任务
3. 定义每个任务的交付物、依赖、验收标准
4. 规划任务的并行执行计划
5. 关联测试验收标准（TEST-VERIFY → Task → Test Case）
6. 生成任务列表文档

**输出路径**：`spec-dev/{requirement_desc_abstract}/tasks/tasks.md`

## 何时使用

**前置条件**：
- ✅ code-designer 已完成，design.md 已生成且确认
- ✅ 所有设计决策已明确
- ✅ 关键的 TODO 项都已澄清
- ⚠️ **快速模式**：code-task 跳过。任务拆解由 code-execute 在内存中完成。

### 快速模式下的行为

当 spec 标注为快速模式时：
- code-task **不执行**
- code-execute 直接读取 spec，在内存中自主拆解实现步骤
- 不生成 tasks.md 文档
- 任务拆解的逻辑记录在执行报告的"自主拆解"章节中

---

## 工作流程

### 步骤1: 分析设计方案

读取 `design.md`，理解：
- 整体的功能目标
- 组件/服务架构
- 数据流和状态管理
- 技术选型和依赖

### 步骤2: 分解为编码任务

根据设计方案，分解为具体的、可独立实现的任务。

**分解策略**：
1. **按功能模块** - 每个主功能为一个 Task
2. **按层级** - 组件 → 样式 → 工具函数 → 测试
3. **考虑依赖** - 标识 Task 之间的依赖关系

**任务类型**：
- **组件实现**：UI 组件、Props、状态、事件处理
- **Hooks/工具**：可复用的业务逻辑
- **样式**：组件样式、主题配置
- **API 集成**：HTTP 调用、数据转换
- **浏览器 E2E 测试** ⭐：编写 Playwright E2E 测试，覆盖完整用户流程
- **视觉回归测试** ⭐：配置和执行截图对比测试
- **组件 UI 测试** ⭐：编写组件级别的交互测试
- **构建配置** ⭐（前端/全栈）：Webpack/Vite配置优化、代码分割、压缩
- **CDN部署** ⭐（前端/全栈）：静态资源上传CDN、缓存策略配置
- **前端监控** ⭐（前端/全栈）：错误追踪SDK接入、性能指标采集、用户行为埋点
- **数据库**：表结构、迁移脚本、索引
- **中间件配置** ⭐：Redis缓存、Nacos配置、消息队列
- **日志实现** ⭐：业务日志记录、错误日志、链路追踪
- **监控埋点** ⭐：API监控指标、业务埋点、告警规则
- **部署配置** ⭐：Dockerfile、K8s配置、环境变量
- **上线脚本** ⭐：数据库迁移脚本、配置变更脚本

**任务粒度**：推荐 4 小时内完成，保持相对独立性。

### 步骤3: 定义任务详情

对每个 Task 定义：
- **目标**：这个任务实现什么
- **交付物**：需要完成的文件
- **依赖**：依赖哪些其他 Task
- **验收标准**：如何判断完成
- **估时**：预计耗时
- **详细说明**：实现思路（可选）

**Task 模板**：详见 [`templates/task-template.md`](./templates/task-template.md)

### 步骤4: 关联测试验收标准（TDD支持）

对每个 Task，关联对应的验收标准（TEST-VERIFY）：

**Test Case Mapping 表格**：
```
| Task ID | Task 名称 | TEST-VERIFY | Test Case ID | Browser Test ID | Mock Data |
|---------|---------|-------------|-------------|----------------|-----------|
| T1      | UserAuth | TV-1.1 | TC-1.1.1 | BT-1.1.1 | user_001.json |
| T2      | FormValidation | TV-2.1, TV-2.2 | TC-2.1.1, TC-2.2.1 | BT-2.1.1, BT-2.2.1 | form_*.json |
```

映射说明：
- **Task ID** → 任务编号
- **TEST-VERIFY** → 来自 spec 的验收标准
- **Test Case ID** → 单元测试用例编号
- **Browser Test ID** → 浏览器测试用例编号（前端/全栈场景）
- **Mock Data** → 来自 spec 的 Mock 数据位置

详见 [`templates/test-case-mapping-template.md`](./templates/test-case-mapping-template.md)

### 步骤5: 规划并行执行

根据 Task 依赖关系，规划并行执行计划：

```
Batch 1（无依赖）：T1, T2, T3 → 并行执行
  ↓ (等待全部完成)
Batch 2（依赖Batch1）：T4, T5 → 并行执行
  ↓ (等待全部完成)
Batch 3：T6 → 执行
```

并行执行可以：
- 加快整体开发进度
- 充分利用多个 implementer agent
- 确保依赖关系不被违反

### 步骤5.5: 规划基础设施和运维任务 ⭐（后端/全栈场景）

对于后端/全栈项目，除了代码实现任务外，还需规划基础设施和运维相关任务。

详见：[`templates/backend-tasks-guide.md`](./templates/backend-tasks-guide.md)

### 步骤5.6: 规划前端构建和运维任务 ⭐（前端/全栈场景）

对于前端/全栈项目，除了UI组件开发外，还需规划构建部署和监控相关任务。

详见：[`templates/frontend-tasks-guide.md`](./templates/frontend-tasks-guide.md)

### 步骤6: 生成任务列表文档

生成 `tasks.md` 包含任务总览、并行执行计划、Test Case Mapping 和 Task 详情。

详见：[`templates/tasks-document-template.md`](./templates/tasks-document-template.md)

---

## 关键约束

✅ **必须做**：
- 任务分解要清晰，每个 Task 要能独立实现
- 依赖关系要准确，便于并行执行
- 验收标准要明确，不能模糊
- Test Case Mapping 要完整（100% 覆盖 TEST-VERIFY）
- 没有 TODO 项（所有设计都已澄清）

❌ **禁止做**：
- 任务过大（超过 8 小时）或过小（管理开销大）
- 依赖关系不清或有环形依赖
- 验收标准不可验证
- Test Case Mapping 不完整
- 保留 TODO 项（应在设计阶段澄清）

---

## 相关资源

| 资源 | 说明 |
|------|------|
| [`templates/task-template.md`](./templates/task-template.md) | 单个Task定义模板 |
| [`templates/tasks-document-template.md`](./templates/tasks-document-template.md) | 完整tasks.md文档模板 |
| [`templates/backend-tasks-guide.md`](./templates/backend-tasks-guide.md) | 后端基础设施任务指南 |
| [`templates/frontend-tasks-guide.md`](./templates/frontend-tasks-guide.md) | 前端基础设施任务指南 |
| [`templates/test-case-mapping-template.md`](./templates/test-case-mapping-template.md) | Test Case Mapping 模板 |
| `prompts/implementer-prompt.md` | 实现子代理提示词 |
| `prompts/spec-reviewer-prompt.md` | 规范审查提示词 |
| `prompts/code-quality-reviewer-prompt.md` | 质量审查提示词 |

## 相关示例

| 类型 | 说明 |
|------|------|
| 前端组件 Task | React/Vue 组件实现，包括 Props、状态、事件 |
| 后端 API Task | API 端点、业务逻辑、数据处理 |
| 数据库 Task | 表结构、迁移脚本、索引优化 |
| 全栈 Task | 跨越前后端的功能实现 |

详见 [`references/workflow-detail.md`](./references/workflow-detail.md) 中的完整示例。

---

**关键理念**：清晰的任务定义和完整的 Test Case Mapping，使得后续的实现和测试可以高效并行执行。
