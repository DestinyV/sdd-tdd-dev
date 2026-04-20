# CLAUDE.md

本文件为 Claude Code（claude.ai/code）在 **sdd-dev-plugin** 代码库中工作时提供指导。

## 📋 项目概述

**sdd-dev-plugin** 是一个企业级 Claude Code 插件，提供 AI 辅助开发的**完整全栈工作流**。通过 5 个专业的 Skills，帮助团队快速建立规范框架、进行设计规划、分解任务、生成代码、验证功能。支持前端、后端、移动端、全栈应用等多类型项目。

**核心工作流**：
```
需求 → /spec-creation (规范) → /code-designer (设计) → /code-task (任务)
       → /code-execute (TDD编码) → /code-test (高层测试) → 完成
```

**工作流特色**：
- Phase 1: 规范定义（TEST-VERIFY格式）
- Phase 2: TDD实现（RED-GREEN-REFACTOR-REVIEW）+ 单元测试
- Phase 3: 高层测试（集成、E2E、性能）+ 闭环验证

**版本**：v2.1.0 (2026-03-10) - 全栈开发版，支持前端/后端/移动端/全栈应用

## 🏗️ 架构与文件结构

### 关键目录

```
sdd-dev-plugin/
├── skills/                                  # 核心 Skills 目录
│   ├── spec-creation/SKILL.md               # 规范生成（交互问卷）
│   ├── code-designer/SKILL.md               # 设计规划工作流
│   ├── code-task/SKILL.md                   # 任务列表生成
│   ├── code-execute/SKILL.md                # 代码执行和多阶段审查
│   │   ├── implementer-prompt.md            # 实现子代理提示词
│   │   ├── spec-reviewer-prompt.md          # 规范审查提示词
│   │   └── code-quality-reviewer-prompt.md  # 质量审查提示词
│   ├── code-test/SKILL.md                   # 测试生成和闭环验证
│   ├── package.json                         # Skills 声明文件
│   └── README.md                            # Skills 导航指南
├── docs/
│   ├── USAGE.md                             # 使用指南
│   ├── INSTALLATION.md                      # 安装说明
│   ├── BEST_PRACTICES.md                    # 最佳实践
│   ├── 使用案例.md                          # 真实项目案例
│   └── img/                                 # 演示图片
├── agents/                                  # 代理配置（可选）
├── .claude-plugin/plugin.json               # 插件元数据
├── README.md                                # 项目高层概述
└── CLAUDE.md                                # 本文件
```

### 文档层级

- **README.md** → 项目概述、核心特性、6 个 Skills、快速开始、工作流图
- **docs/USAGE.md** → 详细使用场景和步骤（包含TDD流程说明）
- **docs/INSTALLATION.md** → 安装和故障排查
- **docs/BEST_PRACTICES.md** → 最佳实践和检查清单（包含Phase 3最佳实践）
- **docs/使用案例.md** → 真实项目案例（React/Vue/Node.js/Python 等）
- **🆕 docs/TDD_COMPLETE_WORKFLOW.md** → 完整的Phase 1-3 TDD工作流指南 (5000+行)
- **🆕 docs/TDD_PHASE3_EXAMPLE.md** → Phase 3集成、E2E、性能测试实例 (6000+行)
- **skills/README.md** → 6 个 Skills 导航指南和原则
- **skills/*/SKILL.md** → 每个 Skill 的完整工作流文档

### 6 个 Skills 简介

| Skill | 阶段 | 职责 | 输出 |
|-------|------|------|------|
| **spec-creation** | Spec | 需求分析和规范生成，BDD 格式规范 | spec-dev/{req_id}/spec/ |
| **code-designer** | Design | 代码设计规划，生成架构和技术方案 | spec-dev/{req_id}/design/design.md |
| **code-task** | Task | 任务分解，生成代码级任务清单 | spec-dev/{req_id}/tasks/tasks.md |
| **code-execute** | Execute | TDD实现，规范+质量两阶段审查+单元测试 | src/ + spec-dev/{req_id}/execution/report.md |
| **code-test** | Test | 高层测试（集成/E2E/性能）和闭环验证 | tests/ + spec-dev/{req_id}/testing/report.md |
| **spec-archive** | Archive | 规范归档和沉淀到企业级规范库 | spec-dev/spec/ |

## 🔑 核心设计原则

### 1. **规范优先（Spec-First）**
- 项目首先通过 `/spec-creation` 生成规范文档
- 所有后续设计和开发都基于此规范
- 规范包含业务场景、数据模型、业务规则

### 2. **设计驱动（Design-Driven）**
- 通过 `/code-designer` 分析需求并制定设计方案
- 生成详细的架构设计、技术方案、设计决策
- 设计审批后才能进入 Task 阶段

### 3. **任务清晰（Task-Clear）**
- 通过 `/code-task` 将设计转换为可执行的任务列表
- 每个 Task 有清晰的目标、交付物、验收标准
- 任务列表为后续执行提供精确指导

### 4. **执行严谨（Execute-Rigorous）**
- 通过 `/code-execute` 进行多阶段审查
- **规范审查**：确保代码符合 design.md
- **质量审查**：确保代码质量达标（TypeScript、Lint、最佳实践）
- **TDD流程**：RED-GREEN-REFACTOR-REVIEW，单元测试覆盖率≥85%
- **修复循环**：发现问题必须修复后重新审查

### 5. **测试完整（Test-Complete）** ✨ Phase 3
- 通过 `/code-test` 进行高层测试（集成、E2E、性能）
- **重要**：单元测试由 `/code-execute` 的TDD流程保证（不重复审查）
- 集成测试、E2E测试、性能测试全覆盖
- **闭环验证**：确保 TEST-VERIFY→Test→Code→Result 完全对应

## 🎯 常见开发工作流

### 用户调用 `/spec-creation <需求描述>`

1. Skill 进行交互式问卷：
   - 项目名称、描述、技术栈
   - 业务场景和设计模式
2. 生成 `spec-dev/{requirement_id}/spec/` 包含：
   - `README.md` - 规范总览
   - `scenarios/` - BDD 格式场景定义
   - `data-models.md` - 数据模型
   - `business-rules.md` - 业务规则
   - `glossary.md` - 术语表

### 用户调用 `/code-designer <需求>`

1. 读取最新的 spec-dev 规范
2. 分析需求，确认关键设计决策
3. 生成 `spec-dev/{requirement_id}/design/design.md` 包含：
   - 需求确认
   - 架构设计
   - 技术方案
   - 关键组件设计
   - 设计决策记录

### 用户调用 `/code-task`

1. 读取 design.md
2. 将设计分解为具体的代码 Task
3. 生成 `spec-dev/{requirement_id}/tasks/tasks.md` 包含：
   - Task 列表（按优先级排序）
   - 每个 Task 的目标、交付物、验收标准
   - Task 之间的依赖关系

### 用户调用 `/code-execute`

1. 读取 tasks.md
2. 为每个 Task 分配独立的实现子代理
3. 对每个 Task 进行：
   - **规范审查**：检查代码是否符合 design.md 的架构和规范
   - **质量审查**：检查代码质量（TypeScript、Lint、最佳实践）
   - **修复循环**：问题修复后重新审查（直到通过）
4. 输出：
   - `src/` - 生成的源代码
   - `spec-dev/{requirement_id}/execution/execution-report.md` - 执行报告

### 用户调用 `/code-test`

1. 读取 tasks.md 和 src/
2. 进行全面的代码质量审查
3. 为每个 Task 生成对应的测试（单元/集成/E2E）
4. 执行测试，验证功能完整性
5. 输出：
   - `tests/` - 测试代码
   - `spec-dev/{requirement_id}/testing/testing-report.md` - 测试报告
   - 包含验证矩阵（Task ↔ 测试 ↔ 结果）

## ⚙️ 关键实现细节

### spec-creation 的规范生成

规范生成采用交互式问卷方式：
- 收集项目基础信息（名称、描述、技术栈）
- 用户定义业务场景和设计模式
- AI 生成 BDD 格式的规范文档

输出目录结构：
```
spec-dev/{requirement_id}/spec/
├── README.md                    # 规范总览
├── scenarios/
│   ├── scenario-1.md
│   ├── scenario-2.md
│   └── ...
├── data-models.md               # 数据模型
├── business-rules.md            # 业务规则
└── glossary.md                  # 术语表
```

### code-designer 的设计规划

设计规划包含以下关键部分：
- **需求确认**：确保理解需求
- **架构设计**：整体架构、模块划分、关键组件
- **技术方案**：技术选型、框架使用、库依赖
- **核心算法**：关键业务逻辑的伪代码或流程图
- **设计决策**：为什么这样设计，权衡考虑

### code-execute 中的多阶段审查

每个 Task 通过两道关卡：

1. **规范审查子代理**（spec-reviewer-prompt.md）
   - 检查代码是否符合 design.md 的架构规范
   - 检查关键组件是否按设计实现
   - 检查命名、结构是否符合规范

2. **质量审查子代理**（code-quality-reviewer-prompt.md）
   - TypeScript 类型检查
   - ESLint/代码规范检查
   - 最佳实践检查
   - 可读性、可维护性检查

### code-test 的闭环验证

生成的验证矩阵展示：
- Task ID → 生成的代码 → 对应的测试 → 测试结果（通过/失败）
- 覆盖率分析（单元、集成、E2E）
- 测试和 design.md 的映射关系

## 📊 核心约束

### ✅ 必须做

- ✅ 首次使用前运行 `/spec-creation` 生成规范
- ✅ 所有需求都通过 `/code-designer` 进行详细设计
- ✅ 设计审批通过后才能进入 `/code-task`
- ✅ 所有 Task 都通过 `/code-execute` 的规范+质量两阶段审查
- ✅ 所有代码都通过 `/code-test` 的测试和闭环验证（≥80% 覆盖率）
- ✅ 遵循 Task 清单严格执行，避免范围蔓延

### ❌ 禁止做

- ❌ 跳过设计阶段直接进行编码
- ❌ 在 code-execute 中跳过规范或质量审查
- ❌ 发现问题后不修复就继续下一个 Task
- ❌ 为了让测试通过而修改源代码逻辑
- ❌ 忽视闭环验证（Task-代码-测试的对应）
- ❌ 改变参考组件的既定设计模式

## 🔄 Skills 之间的关系

```
用户需求
    ↓
[spec-creation] （首次或新需求）
    ↓ 生成 spec-dev/{req_id}/spec/
    ↓
[code-designer] （需求分析和设计）
    ↓ 生成 spec-dev/{req_id}/design/design.md
    ↓ （用户审查设计）
    ↓
[code-task] （任务分解）
    ↓ 生成 spec-dev/{req_id}/tasks/tasks.md
    ↓ （用户确认任务清单）
    ↓
[code-execute] （代码执行）
    ↓ 包含规范审查 + 质量审查 + 修复循环
    ↓ 生成 src/ + execution-report.md
    ↓ （编译验证通过）
    ↓
[code-test] （测试和验证）
    ↓ 包含单元/集成/E2E测试
    ↓ 生成 tests/ + testing-report.md
    ↓ 包含闭环验证矩阵
    ↓
✅ 功能完成，代码质量达标，可以上线！
```

## 🌐 支持的技术栈

### 前端
- **框架**：React、Vue 3、Angular、Svelte
- **类型**：TypeScript
- **样式**：Tailwind CSS、UNO CSS、CSS-in-JS
- **测试**：Jest、Vitest、Cypress、Playwright

### 后端
- **语言**：Node.js、Python、Go、Java
- **框架**：Express、Fastify、Django、Flask、Spring Boot、Gin
- **API**：REST、GraphQL
- **测试**：Jest、Pytest、JUnit、Go testing

### 数据库
- **SQL**：PostgreSQL、MySQL、SQLite
- **NoSQL**：MongoDB、Redis
- **迁移**：数据库架构设计和迁移脚本

### 移动端
- **框架**：React Native、Flutter
- **测试**：Jest、Flutter testing

### 全栈应用
- 支持前端+后端+数据库的完整应用设计
- 支持微服务架构的多服务协调设计

## 📝 代码规范与注释

### TODO 注释格式

在代码和文档中使用统一的 TODO 注释格式，便于追踪和管理待做事项：

```
// TODO：{描述}
```

**示例**：
```typescript
// TODO：从后端API获取商品列表数据结构定义
// TODO：实现搜索和筛选功能
// TODO：验证API返回的错误处理逻辑
```

**使用场景**：
- **需求澄清阶段**：标记需要确认的信息或决策
  - `// TODO：确认后端API的分页参数`
  - `// TODO：确认错误返回格式`
- **设计阶段**：标记设计中的待定决策
  - `// TODO：确定状态管理方案`
  - `// TODO：定义数据模型结构`
- **实现阶段**：标记代码中的待完成部分
  - `// TODO：实现表单验证`
  - `// TODO：添加错误提示`
- **文档中**：标记文档中的待补充内容
  - `// TODO：补充API文档`
  - `// TODO：添加使用示例`

---

## 🚀 修改 Skills 时的注意事项

Skills 可以针对特定项目进行定制：

### 1. 最重要的：修改 Skill SKILL.md 文件

调整每个 Skill 的工作流和原则：
- `skills/spec-creation/SKILL.md` - 问卷形式、规范格式
- `skills/code-designer/SKILL.md` - 设计深度、输出格式
- `skills/code-task/SKILL.md` - 任务分解粒度、验收标准
- `skills/code-execute/SKILL.md` - 执行策略、审查流程
- `skills/code-test/SKILL.md` - 测试框架、覆盖率要求

### 2. 调整执行和审查提示词（code-execute 中）

- `implementer-prompt.md` - 控制代码实现方式和风格
- `spec-reviewer-prompt.md` - 控制规范审查维度
- `code-quality-reviewer-prompt.md` - 控制代码质量标准

### 3. 定制测试策略（code-test 中）

- 调整测试框架和工具
- 调整测试覆盖率要求
- 调整闭环验证的标准

## 📚 文档参考

- **项目概述** → [`README.md`](./README.md)
- **详细使用指南** → [`docs/USAGE.md`](./docs/USAGE.md)
- **安装和故障排查** → [`docs/INSTALLATION.md`](./docs/INSTALLATION.md)
- **最佳实践** → [`docs/BEST_PRACTICES.md`](./docs/BEST_PRACTICES.md)
- **真实项目案例** → [`docs/使用案例.md`](./docs/使用案例.md)
- **Skills 导航** → [`skills/README.md`](./skills/README.md)
- **每个 Skill 详细文档** → `skills/*/SKILL.md`

## 🎓 辅助用户时的建议

### 首次使用的用户
1. 指引阅读 `README.md` 了解整个工作流
2. 引导执行 `/spec-creation` 生成规范
3. 选择一个小功能进行试运行

### 需要理解工作流
1. 查看 `docs/USAGE.md` 的详细场景说明
2. 参考 `skills/README.md` 的 Skills 导航
3. 查看具体 `skills/*/SKILL.md` 了解每个 Skill

### 定制工作流
1. 阅读相应的 `SKILL.md` 文件
2. 理解工作流的各个阶段和输出
3. 修改 SKILL.md 或提示词以适应项目需求
4. 测试修改效果

### 调试工作流问题
1. 查看 `docs/INSTALLATION.md` 中的常见问题
2. 检查 `spec-dev/` 中的各个阶段输出
3. 验证输出文档是否完整和准确
4. 检查前一个阶段的输出是否作为后一个阶段的输入

## 🔍 项目结构快速导航

```
plugins/sdd-dev-plugin/
├── README.md                    # 项目主文档 ← 从这里开始
├── CLAUDE.md                    # Claude 工作指南（本文件）
│
├── skills/                      # 5个 Skills
│   ├── README.md               # Skills 导航指南
│   ├── spec-creation/
│   ├── code-designer/
│   ├── code-task/
│   ├── code-execute/          # 包含 3 个提示词文件
│   └── code-test/
│
├── docs/                        # 完整文档
│   ├── USAGE.md                # 使用指南（必读）
│   ├── INSTALLATION.md         # 安装说明
│   ├── BEST_PRACTICES.md       # 最佳实践
│   ├── 使用案例.md             # 真实案例
│   └── img/                    # 演示图片
│
└── .claude-plugin/
    └── plugin.json             # 插件元数据
```

## 💡 关键要点总结

1. **规范驱动** - 一切从 `/spec-creation` 开始
2. **设计优先** - `/code-designer` 生成的设计是后续的指南
3. **任务清晰** - `/code-task` 确保任务定义完整明确
4. **多阶段审查** - `/code-execute` 的规范+质量两道关卡保证质量
5. **闭环验证** - `/code-test` 的验证矩阵确保 Task-代码-测试对应

记住：**整个工作流是规范驱动且任务清晰的**。规范和设计确定后，所有其他 Skill 都严格按照规范和任务清单执行，确保交付物的一致性和质量。

---

**最后更新**：2026-03-10 (v2.1.0 全栈开发版)
**让 AI 辅助的研发开发变得规范、高效、可信赖！** 🚀
