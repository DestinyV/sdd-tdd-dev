# 设计文档：全栈项目数据库优先 + 接口对齐驱动流程

**日期**: 2026-04-23
**状态**: 待审核
**范围**: sdd-tdd-dev 插件增强

## 目标

优化 sdd-tdd-dev 插件处理全栈项目（前后端同时改动）的流程，确保：

1. **数据库优先**：涉及数据持久化时，先设计数据库再制定接口
2. **接口对齐驱动**：前后端基于同一套接口契约各自开发，避免最后发现两套方案不一致
3. **遵循现有约定**：新接口设计最大化参考和沿用原有项目的架构、模式、风格

## 当前问题

现有流程中，前后端任务在 code-task 阶段被拆分为独立 Task，各自进入 code-execute 独立开发。由于缺乏前置的接口对齐机制，开发完成后可能发现：

- 前后端接口路径不一致
- 请求/响应字段名或类型不匹配
- 错误码体系不统一
- 分页、认证等公共约定各自实现

## 整体架构

```
spec-creation → code-designer → api-contract(NEW) → code-task → code-execute → code-test
```

### 全链路增强点

| 阶段 | 增强内容 | 类型 |
|------|---------|------|
| code-architect agent | 全栈场景自动识别 + 现有代码库约定扫描 | 增强 |
| code-designer | 条件触发数据库设计 + 接口清单初稿 | 增强 |
| api-contract (NEW) | 接口契约定义、审查、版本管理 | 新增 |
| code-task | 接口对齐强制检查 + provides/consumes 依赖关系 | 增强 |
| code-execute | Task 执行前接口契约验证 + 版本锁定 | 增强 |
| code-test | 接口一致性端到端测试 + 契约测试 | 增强 |

## Section 1：全栈场景识别与触发机制

### 1.1 识别规则

由 `code-architect` agent 在分析阶段自动判断：

| 条件 | 判定 |
|------|------|
| spec 场景中包含 UI/前端交互（页面、组件、表单、按钮等）且同时包含后端逻辑/数据存储 | `project-mode: fullstack` |
| 仅包含 UI/前端交互，不涉及后端 | `project-mode: frontend` |
| 仅包含后端逻辑/数据存储，不涉及 UI | `project-mode: backend` |

### 1.2 数据库需求判定

| 条件 | 判定 |
|------|------|
| 需求涉及数据持久化（新建/修改数据库表、存储业务数据） | `needs-database: true` |
| 需求仅涉及前端 + 已有第三方 API 对接，无新数据存储 | `needs-database: false` |

判定结果写入 `requirement.md`：

```yaml
project-mode: fullstack
needs-database: true
```

### 1.3 触发逻辑

```
if project-mode == fullstack && needs-database == true:
    → 数据库设计必须完成
    → 接口契约必须完成
elif project-mode == fullstack && needs-database == false:
    → 数据库设计可跳过
    → 接口契约仍必须完成
```

## Section 2：新增 api-contract 阶段

### 2.1 位置

位于 `code-designer` 和 `code-task` 之间。

### 2.2 输入输出

| 输入 | 输出 |
|------|------|
| `design.md`（含数据库设计或接口清单初稿） | `spec-dev/{req}/api-contract/api-contract.md` |
| 现有代码库接口风格（由 code-architect 扫描） | `spec-dev/{req}/api-contract/review-report.md` |

### 2.3 api-contract.md 结构

```markdown
# API Contract - {需求名称}

**版本**: v1
**生成日期**: YYYY-MM-DD
**关联设计**: spec-dev/{req}/design/design.md

## 现有项目约定

### URL 风格
- 基础路径前缀：/api/v1
- 命名规范：复数小写

### 响应格式
- 成功：{ code: 0, data: {...}, message: "success" }
- 失败：{ code: xxx, message: "错误描述", data: null }

### 认证方式
- Header: Authorization: Bearer {token}

### 已有中间件/工具
- 全局异常处理中间件
- 请求日志中间件
- Redis 缓存策略

## 接口列表

### {METHOD} {path}

**描述**: {接口功能描述}
**认证**: 是/否
**性能要求**: 响应时间 < Xms

**请求参数**:
| 参数名 | 位置 | 类型 | 必填 | 校验规则 | 示例值 | 说明 |
|--------|------|------|------|---------|--------|------|

**成功响应**:
```json
{ ... }
```

**错误响应**:
| 错误码 | HTTP 状态码 | 说明 | 触发条件 |
|--------|------------|------|---------|
```

## 2.4 接口审查（review-report.md）

审查维度：

1. **完整性**：每个前端交互是否都有对应的后端接口？
2. **命名规范**：RESTful 风格一致性、URL 命名规范
3. **类型一致性**：前端期望类型与后端返回类型匹配
4. **错误处理**：每个接口是否定义了错误码和错误信息
5. **项目约定遵循**：新接口是否符合现有项目的接口风格约定？

审查通过标准：以上 5 项全部通过，否则拒绝进入 code-task 阶段。

### 2.5 版本管理

- api-contract.md 初始版本号为 `v1`
- 后续如有接口变更，必须更新版本号（v2, v3...）并记录变更日志
- 变更日志格式：

```markdown
## 版本历史

| 版本 | 日期 | 变更内容 | 变更原因 |
|------|------|---------|--------|
| v1   | YYYY-MM-DD | 初始版本 | - |
| v2   | YYYY-MM-DD | 新增 GET /api/xxx | 需求变更 |
```

## Section 3：code-designer 增强

### 3.1 数据库设计（条件触发）

当 `project-mode: fullstack && needs-database: true` 时，design.md 中必须包含独立的「数据库设计」章节：

- 表结构（表名、字段、类型、约束、默认值）
- 索引策略（主键、唯一索引、复合索引）
- 关系图（表之间的关联关系）
- 迁移策略（首次创建 / 增量迁移、回滚方案）

当 `needs-database: false` 时，该章节可跳过。

### 3.2 接口清单初稿

无论是否需要数据库，只要是 `project-mode: fullstack`，design.md 中必须包含接口清单初稿：

- 接口路径和 HTTP 方法
- 简要的请求/响应描述
- 与数据库表的映射关系（如果需要数据库）

此初稿作为 api-contract 阶段的输入基础。

### 3.3 code-architect agent 增强

**新增能力**：

1. **全栈场景自动识别**：分析 spec 文档，自动判断 project-mode 和 needs-database
2. **现有代码库约定扫描**：
   - 扫描已有接口的 URL 命名风格
   - 识别响应格式约定
   - 识别认证/授权模式
   - 识别已有中间件、工具、公共函数
   - 识别已有的数据库 ORM 配置和迁移工具
3. **数据库设计推荐**：基于业务规则推荐数据库设计模式

**输出**：约定扫描结果写入 design.md 的「现有项目约定」章节。

### 3.4 HARD-GATE

```
# code-designer 输出检查
if project-mode == fullstack:
    → 必须包含「现有项目约定」章节
    → 必须包含「接口清单初稿」章节
    → if needs-database == true:
        → 必须包含「数据库设计」章节
```

## Section 4：code-task 增强

### 4.1 前置检查（新增 HARD-GATE）

生成 tasks.md 前，必须验证：

```
1. api-contract.md 已存在于 spec-dev/{req}/api-contract/
2. review-report.md 已存在且审查通过
3. 每个前端 Task 的接口依赖（consumes）都在 api-contract.md 中定义
4. 每个后端 Task 的接口实现（provides）都在 api-contract.md 中定义
5. 同一接口的前端依赖定义与后端实现定义完全一致（路径、参数、响应结构）
```

### 4.2 对齐验证失败处理

如果前置检查不通过：

1. 输出差异报告，列出不一致项：
   - 路径不一致（前端调用 /api/user，后端实现 /api/users）
   - 参数类型不匹配（前端传 string，后端期望 number）
   - 缺失字段（前端需要 avatar 字段，后端未返回）
   - 错误码不匹配
2. **拒绝生成 tasks.md**
3. 提示回到 api-contract 阶段修复

### 4.3 Task 依赖关系增强

每个 Task 新增标记：

| 标记 | 说明 | 示例 |
|------|------|------|
| `provides` | 该 Task 实现的接口 | `provides: [GET /api/users, POST /api/users]` |
| `consumes` | 该 Task 依赖的接口 | `consumes: [GET /api/users]` |

系统自动生成依赖图：

- 消费方 Task 必须在提供方 Task 完成后才能执行
- 即使支持并行执行，也必须遵守此依赖顺序
- 依赖图写入 tasks.md 的「Task 依赖关系」章节

### 4.4 项目约定遵循检查

新增检查项：

- 新接口是否符合现有项目的 URL 命名风格？
- 新接口使用的中间件、工具是否与已有代码一致？
- 新接口的错误码是否遵循已有错误码体系？

如发现风格不一致，提示并询问是否沿用已有约定。

## Section 5：code-execute 增强

### 5.1 Task 执行前接口契约验证

每个 Task 启动时：

1. 检查其 `provides` 或 `consumes` 标记是否在 api-contract.md 中存在
2. 如果标记的接口不存在，该 Task **拒绝执行并报错**
3. 记录当前 api-contract.md 版本号到执行日志

### 5.2 实现约束

**后端 Task**：
- 必须严格按照 api-contract.md 中定义的请求参数结构编码
- 必须严格按照 api-contract.md 中定义的响应格式返回
- 必须实现 api-contract.md 中定义的所有错误码

**前端 Task**：
- 必须严格按照 api-contract.md 中定义的格式发送请求
- 必须严格按照 api-contract.md 中定义的响应结构解析数据
- 必须处理 api-contract.md 中定义的所有错误情况

### 5.3 版本锁定

- Task 执行期间锁定 api-contract.md 的版本号
- 如需接口变更，必须先：
  1. 更新 api-contract.md 版本号
  2. 重新触发接口审查
  3. 通知所有相关 Task 重新审查

## Section 6：code-test 增强

### 6.1 接口一致性端到端测试

| 测试类型 | 验证内容 |
|---------|---------|
| 契约测试（后端） | 后端实际返回的字段、类型、结构与 api-contract.md 定义一致 |
| 契约测试（前端） | 前端发送的请求格式与 api-contract.md 定义一致 |
| 错误码覆盖测试 | api-contract.md 中定义的所有错误码在后端都有对应实现 |
| 边界条件测试 | 空值、超长字符串、特殊字符等边界情况的接口行为符合契约定义 |

### 6.2 测试生成

基于 api-contract.md 自动生成：

- 后端契约测试用例：针对每个接口，验证响应字段、类型、错误码
- 前端集成测试用例：针对每个接口调用，验证请求格式、响应解析
- 端到端流程测试：验证前端到后端的完整数据流

### 6.3 失败处理

- 契约测试失败意味着实现与接口契约不一致
- 必须修复后重新运行测试
- 不允许跳过契约测试

## Section 7：文件结构变更

```
spec-dev/{req}/
├── spec/                          # 不变
│   ├── requirement.md
│   ├── scenarios/
│   ├── data-models.md
│   └── business-rules.md
├── design/                        # 增强
│   └── design.md                  # 新增：现有项目约定、数据库设计、接口清单初稿
├── api-contract/                  # 新增
│   ├── api-contract.md            # 接口契约定义
│   └── review-report.md           # 接口审查报告
├── tasks/                         # 增强
│   └── tasks.md                   # 新增：provides/consumes 标记、接口对齐验证结果
├── execution/                     # 增强
│   └── execution-report.md        # 新增：接口契约版本号、验证日志
└── testing/                       # 增强
    └── testing-report.md          # 新增：接口一致性测试结果
```

## Section 8：快速模式（Quick Mode）适配

当 `mode: quick` 时：

- code-designer 阶段跳过（设计决策内联到场景）
- api-contract 阶段简化为：从 spec 场景中提取接口清单，生成精简版 api-contract.md
- code-task 阶段的接口对齐检查仍然执行（但审查范围缩小）
- code-execute 阶段的接口契约验证仍然执行
- code-test 阶段的契约测试仍然执行（但用例数量可缩减）

## Section 9：向后兼容

- 对于 `project-mode: frontend` 或 `project-mode: backend` 的项目，api-contract 阶段自动跳过
- 对于已有的不涉及前后端协作的项目，流程保持不变
- 新增的 api-contract 目录和检查仅在 fullstack 模式下触发
