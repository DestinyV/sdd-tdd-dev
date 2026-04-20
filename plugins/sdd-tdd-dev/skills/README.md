# 研发SDD工作流 - Skills集合（全栈开发版）

这是 **sdd-tdd-dev-plugin** 项目的6个Claude Code Skills，实现**Spec-Design-Task-Execute-Test-Archive**工作流，支持全栈开发（前端、后端、移动端、全栈应用）。

## 📋 工作流概览

```
用户需求
   ↓
[1] /spec-creation → 生成设计规范和参考组件清单
   ↓
[2] /code-designer → 根据需求进行代码设计规划
   ↓
[3] /code-task → 将设计转换为详细任务列表
   ↓
[4] /code-execute → 执行任务，生成代码实现
   ↓
[5] /code-test → 代码审核、测试、闭环验证
   ↓
[6] /spec-archive → 规范归档，沉淀到企业级规范库 ✨新增
   ↓
✅ 功能完成，代码上线，规范沉淀
```

---

## 🎯 Skills详细说明

| Skill | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **spec-creation** | 需求描述 | spec-dev/{requirement_desc_abstract}/spec/ | BDD格式规范生成，包含场景、数据模型、业务规则 |
| **code-designer** | spec/ | spec-dev/{requirement_desc_abstract}/design/design.md | 根据规范进行代码设计规划，生成架构和技术方案 |
| **code-task** | design.md | spec-dev/{requirement_desc_abstract}/tasks/tasks.md | 将设计分解为代码级别任务列表 |
| **code-execute** | tasks.md | src/ + spec-dev/{requirement_desc_abstract}/execution/execution-report.md | 执行任务，规范+质量两阶段审查。**v2.3.1+**：为每个Task创建git-worktree，隔离工作环境，支持安全的修复循环和并行执行 |
| **code-test** | src/ + tasks.md | tests/ + spec-dev/{requirement_desc_abstract}/testing/testing-report.md | 集成/E2E/性能测试、闭环验证（单元测试由code-execute的TDD流程完成） |
| **spec-archive** | spec/ | spec-dev/spec/ | 规范对标分析、冲突检测、智能合并，沉淀到主规范库 |

---

## 🚀 快速使用

### 第1步：需求规范化
```bash
/spec-creation [需求描述]
```
生成 `spec-dev/{requirement_desc_abstract}/spec/`
- README.md - 规范总览
- scenarios/*.md - BDD格式场景
- data-models.md - 数据模型
- business-rules.md - 业务规则
- glossary.md - 术语表

### 第2步：代码设计
```bash
/code-designer
```
生成 `spec-dev/{requirement_desc_abstract}/design/design.md`

### 第3步：生成任务列表
```bash
/code-task
```
生成 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`

### 第4步：执行代码
```bash
/code-execute
```
生成 `src/` + `spec-dev/{requirement_desc_abstract}/execution/execution-report.md`

**v2.3.1+ Git-Worktrees支持**：
- 为每个Task创建独立的git-worktree（`.claude/worktrees/{task-id}-{name}`）
- 编码、修复都在worktree中进行，每次修复作为独立commit
- 修复完成后cherry-pick或squash merge到main分支
- 修复失败可删除worktree重新开始，不污染main分支
- 详见 [Git-Worktrees指南](./code-execute/git-worktrees-guide.md) 和 [快速参考](./code-execute/QUICK_REFERENCE.md)

### 第5步：测试和验证
```bash
/code-test
```
生成 `tests/` + `spec-dev/{requirement_desc_abstract}/testing/testing-report.md`

**新增Phase 3高层测试**（code-test现在专注于集成、E2E、性能测试）：
- 集成测试：参考 `code-test/integration-test-prompt.md`
- E2E测试：参考 `code-test/e2e-test-prompt.md`
- 性能测试：参考 `code-test/performance-test-prompt.md`

**重要提示**：单元测试由 `/code-execute` 的TDD流程（RED-GREEN-REFACTOR-REVIEW）完成，`code-test` 不重复审查单元测试

### 第6步：规范归档 ✨新增
```bash
/spec-archive
```
（在code-test所有测试通过后自动触发）

生成 `spec-dev/spec/` + `archive-report.md`

---

## 📁 Skills目录结构

```
skills/
├── spec-creation/
│   ├── SKILL.md                     # 规范生成工作流
│   └── README.md
├── code-designer/
│   ├── SKILL.md                     # 设计规划工作流
│   └── README.md
├── code-task/
│   ├── SKILL.md                     # 任务分解工作流
│   └── README.md
├── code-execute/
│   ├── SKILL.md                     # 代码执行工作流（含git-worktrees支持）
│   ├── git-worktrees-guide.md       # Git-Worktrees隔离工作环境详细指南 ✨v2.3.1新增
│   ├── QUICK_REFERENCE.md           # Git-Worktrees快速参考卡片 ✨v2.3.1新增
│   ├── implementer-prompt.md        # 实现子代理提示词
│   ├── spec-reviewer-prompt.md      # 规范审查提示词
│   ├── code-quality-reviewer-prompt.md  # 质量审查提示词
│   └── README.md
├── code-test/
│   ├── SKILL.md                     # 测试工作流（集成/E2E/性能）
│   ├── integration-test-prompt.md   # 集成测试设计指南 ✨Phase 3新增
│   ├── e2e-test-prompt.md           # E2E测试设计指南 ✨Phase 3新增
│   ├── performance-test-prompt.md   # 性能测试设计指南 ✨Phase 3新增
│   └── README.md
├── spec-archive/                    # ✨新增
│   ├── SKILL.md                     # 规范归档工作流
│   └── README.md
├── package.json                     # Skills声明
└── README.md                        # 本文件
```

---

## 🔑 核心设计原则

### 1. **规范优先（Spec-First）**
- 项目首先通过 `/spec-creation` 生成 `design-spec/README.md`
- 所有后续设计和开发都基于此规范
- 规范定义了参考组件和设计模式

### 2. **设计驱动（Design-Driven）**
- 通过 `/code-designer` 深入分析设计
- 生成详细的设计方案，包括架构、技术方案、设计决策
- 设计审批通过后才能进入Task阶段

### 3. **任务清晰（Task-Clear）**
- 通过 `/code-task` 将设计转换为可执行的任务列表
- 每个Task有清晰的目标、交付物、验收标准
- 任务列表为后续执行提供清晰指导

### 4. **执行严谨（Execute-Rigorous）**
- 通过 `/code-execute` 进行多阶段审查
  - 规范审查：确保代码符合design.md
  - 质量审查：确保代码质量达标
  - 修复循环：发现问题必须修复后重新审查
- 每个Task都要通过两道审查关卡

### 5. **测试完整（Test-Complete）**
- 通过 `/code-test` 进行高层测试和闭环验证
- 集成测试、E2E测试、性能测试全覆盖
- **注意**：单元测试由 `/code-execute` 的TDD流程保证（不重复）
- 闭环验证：确保Task-代码-测试的完全对应

### 6. **规范沉淀（Spec-Archive）** ✨新增
- 通过 `/spec-archive` 将验证通过的规范归档到主规范库
- 对标分析和冲突检测，确保规范库的一致性和完整性
- 建立企业级规范库，便于后续需求的参考和对标

---

## ✅ 工作流特点

### 灵活性
- **模式自定义** - spec-creation中用户自己定义业务模式，不预设
- **设计驱动** - 每个需求都通过code-designer详细设计后才执行
- **任务清晰** - code-task确保任务定义完整清晰，避免歧义
- **全栈支持** - 支持前端、后端、移动端、全栈应用的需求和设计

### 质量保证
- **多阶段审查** - code-execute中的规范审查+质量审查
- **修复循环** - 问题发现后必须修复并重新审查
- **测试完整** - code-test中的单元、集成、E2E全覆盖
- **技术栈适配** - 根据项目技术栈（React/Vue/Node.js/Java/Python/Go等）调整设计和实现规范

### 可追踪
- **文档完整** - 每个阶段都生成详细的设计/任务/报告文档
- **决策记录** - 设计阶段记录所有设计决策和原因
- **问题记录** - 执行和测试阶段记录所有发现的问题和修复

---

## 📖 详细文档

- **项目主文档** → [`../README.md`](../README.md)
- **安装说明** → [`../docs/INSTALLATION.md`](../docs/INSTALLATION.md)
- **使用指南** → [`../docs/USAGE.md`](../docs/USAGE.md)
- **最佳实践** → [`../docs/BEST_PRACTICES.md`](../docs/BEST_PRACTICES.md)

---

## 🔧 Skill详细文档

每个Skill在自己的目录下都有完整的`SKILL.md`文档，包含：
- 职责说明
- 工作流程（分阶段说明）
- 关键约束
- 危险信号

例如：
- `spec-creation/SKILL.md` - 规范生成工作流
- `code-designer/SKILL.md` - 设计规划工作流
- `code-task/SKILL.md` - 任务分解工作流
- `code-execute/SKILL.md` - 代码执行工作流
- `code-test/SKILL.md` - 测试工作流（集成/E2E/性能，不重复单元测试）✨Phase 3优化版
- `spec-archive/SKILL.md` - 规范归档工作流 ✨新增

---

## 🎓 使用建议

### 首次使用
1. 阅读 `../README.md` 了解整个流程
2. 执行 `/spec-creation` 生成项目规范
3. 选择一个需求试运行完整的 Spec → Design → Task → Execute → Test → Archive 流程

### 日常开发
1. 需求来临 → `/code-designer` 设计
2. 设计审批通过 → `/code-task` 任务列表
3. `/code-execute` 执行代码生成
4. `/code-test` 测试验证
5. `/spec-archive` 规范归档 ✨新增
6. 完成！

### 定制工作流
- 修改各个`SKILL.md`中的流程和原则以适应项目特点
- 调整执行/审查提示词以适应特定需求
- 参见 `../docs/CUSTOMIZATION.md` 学习如何定制

---

## 🔑 核心约束

### ✅ 必须做
- 首次使用前运行 `/spec-creation` 生成规范
- 所有需求都通过 `/code-designer` 设计
- 设计审批通过后才进入 `/code-task`
- 所有Task都通过 `/code-execute` 的两道审查 + TDD流程（确保单元测试覆盖）
- 所有代码都通过 `/code-test` 的集成/E2E/性能测试和闭环验证
- 测试通过后执行 `/spec-archive` 将规范沉淀到主规范库 ✨新增

### ❌ 不能做
- 跳过设计阶段直接进行编码
- 在code-execute中跳过规范或质量审查
- 发现问题后不修复就继续下一个Task
- 为了让测试通过而修改源代码逻辑
- 忽视闭环验证（Task-代码-测试的对应）
- 跳过规范归档流程，导致规范库无法沉淀和完善 ✨新增

---

## 🚀 架构特点

### 分离关注点
- **spec-creation**: 定义规范
- **code-designer**: 设计方案
- **code-task**: 分解任务
- **code-execute**: 执行编码
- **code-test**: 验证质量

### 质量门禁
```
规范 ✅ → 设计 ✅ → 任务 ✅ → 执行审查 ✅ → 测试审查 ✅ → 上线
```

每个阶段都有明确的输出和验收标准。

---

## 💡 最佳实践

1. **设计优先** - 充分的设计能避免后续修改
2. **任务清晰** - 清晰的Task定义能提高执行效率
3. **审查严格** - 多阶段审查能及早发现问题
4. **文档完整** - 详细的文档便于追踪和未来维护
5. **测试完整** - 高覆盖率的测试是质量保证

---

## 📝 更新日志

### v2.2.0 (2026-03-20) ✨新增
- ✅ **规范归档流程** - 新增spec-archive技能和spec-archiver Agent
- ✅ **规范沉淀机制** - 将验证通过的需求规范自动归档到企业级主规范库
- ✅ **场景拆分合并** - 支持通过场景拆分和智能合并策略集成新规范
- ✅ **冲突检测机制** - 自动检测和处理规范冲突，提供决策建议
- ✅ **规范对标分析** - 新增场景、数据模型、业务规则、术语的对标分析
- ✅ **版本管理** - 支持规范版本追踪和演进历史记录

### v2.1.0 (2026-03-10)
- ✅ 全栈开发能力支持：前端、后端、移动端、全栈应用
- ✅ 技术栈扩展：Node.js、Python、Go、Java 等后端框架
- ✅ 数据库设计支持：SQL、NoSQL 数据模型设计
- ✅ API设计规范：REST/GraphQL API 设计和验证
- ✅ 部署和配置：容器化、环境配置、CI/CD 管道设计
- ✅ 微服务架构：多个服务的协调设计和任务分解

### v2.0.0 (2026-03-09)
- ✅ 重构工作流为 Spec-Design-Task-Execute-Test-Archive 6个阶段
- ✅ spec-creation: 研发规范生成（取代原spec-generator）
- ✅ code-designer: 代码设计规划（取代原ai-planning）
- ✅ code-task: 任务列表生成（新增）
- ✅ code-execute: 代码执行实现（取代原ai-code-execution）
- ✅ code-test: 代码审核和测试（取代原ai-test-creation）
- ✅ spec-archive: 规范归档和沉淀（新增）
- ✅ 支持全栈开发，现阶段重点支持前端（React/Vue/Angular/Svelte）

---

**让AI辅助的研发开发变得规范、高效、可信赖！** 🚀
