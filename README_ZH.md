# DestinyV Marketplace

[English](./README_EN.md) | 中文

企业级 Claude Code 插件市场，汇集高质量的开发工作流插件，帮助团队建立规范、提升效率、保证质量。

## 🎯 市场概述

DestinyV Marketplace 是一个开源的 Claude Code 插件市场，为企业开发团队提供经过验证的、生产级别的插件和技能（Skills）。我们的目标是：

- 📦 集中管理高质量 Claude Code 插件
- 🚀 加速企业级 AI 辅助开发工作流
- 🔧 提供即插即用的开发规范和最佳实践
- 🤝 建立 AI 驱动的开发工具生态

---

## 🚀 快速开始

### 1️⃣ 在 Claude Code 中添加 Marketplace

#### 方式 A：自动安装（推荐）

```bash
# 在你的项目中运行
/plugin install https://github.com/DestinyV/sdd-tdd-dev.git
```

#### 方式 B：手动配置

1. 打开项目的 `.claude/config.json`（如果不存在则创建）

2. 添加 marketplace 源：
   ```json
   {
     "marketplaces": [
       {
         "name": "DestinyV Marketplace",
         "url": "https://github.com/DestinyV/sdd-tdd-dev.git",
         "type": "git"
       }
     ]
   }
   ```

3. 在 Claude Code 中执行：
   ```bash
   /marketplace list    # 查看可用插件
   /marketplace install sdd-tdd-dev  # 安装插件
   ```

---

### 2️⃣ 创建新的 Plugin

#### Plugin 标准结构

所有 Plugin 必须遵循以下目录结构，放在本市场的 `/plugins` 目录中：

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json                  # 插件元数据（必需）
├── skills/                          # Skills 目录（可选，如果有多个 skill）
│   ├── skill-1/
│   │   └── SKILL.md                # Skill 定义文件
│   ├── skill-2/
│   │   └── SKILL.md
│   └── package.json                # Dependencies（如果有）
├── agents/                          # Agents 目录（可选）
│   ├── agent-1.md
│   └── agent-2.md
├── docs/                           # 文档目录
│   ├── USAGE.md                    # 使用指南
│   ├── INSTALLATION.md             # 安装说明
│   ├── BEST_PRACTICES.md           # 最佳实践
│   ├── FAQ.md                      # 常见问题
│   └── examples/                   # 使用示例
├── README.md                       # 插件概述（必需）
├── CHANGELOG.md                    # 更新日志
├── CLAUDE.md                       # 开发指南
├── LICENSE                         # 许可证（必需）
└── package.json                    # npm 包配置（如果需要）
```

#### 最小化 Plugin 示例

**plugin.json** - 插件元数据
```json
{
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": "Your Name",
  "license": "MIT",
  "skills": [
    {
      "name": "skill-name",
      "path": "./skills/skill-name/SKILL.md"
    }
  ],
  "commands": [],
  "hooks": [],
  "agents": []
}
```

**README.md** - 插件介绍
```markdown
# Your Plugin Name

简要描述插件功能

## 特性

- Feature 1
- Feature 2

## 安装

```bash
/plugin install your-plugin@DestinyV-marketplace
```

## 使用

```bash
/your-command
```

**skills/your-skill/SKILL.md** - Skill 定义
```markdown
---
name: your-skill
description: What this skill does
---

Skill 内容和实现说明
```

#### 创建 Plugin 的步骤

1. **在市场中创建 Plugin 目录**
   ```bash
   mkdir plugins/your-plugin-name
   cd plugins/your-plugin-name
   ```

2. **初始化必要文件**
   ```bash
   mkdir -p .claude-plugin skills/your-skill docs agents
   touch .claude-plugin/plugin.json README.md LICENSE CLAUDE.md
   touch skills/your-skill/SKILL.md
   ```

3. **编写插件代码和文档**
   - 完善 `plugin.json` 配置
   - 实现 Skill 功能
   - 编写完整的 README 和文档

4. **测试插件**
   ```bash
   # 在本地项目中测试
   npx skills add ./plugins/your-plugin-name
   ```

5. **提交到市场（见下方贡献指南）**

---

## 📦 Marketplace 中已有的 Plugin

### 🏆 sdd-tdd-dev - 企业级全栈 SDD+TDD 开发工作流

**最新版本**: v2.3.1 (2026-03-23)

一个企业级 Claude Code 插件，提供完整的规范驱动开发（SDD）和测试驱动开发（TDD）工作流。支持前端、后端、数据库、微服务等全类型项目的 AI 辅助开发。

#### 核心特性

- ✅ **规范驱动设计（SDD）** - 定义规范，AI 按规范生成代码
- ✅ **完整工作流** - Spec → Design → Task → Execute → Test → Archive（6个阶段）
- ✅ **TDD 实现体系** - RED-GREEN-REFACTOR-REVIEW 四阶段 TDD 流程
- ✅ **智能子代理** - 5 个专业角色（架构师、探索者、执行者、审查者、归档专家）并行协作
- ✅ **质量保证** - 规范审查、代码质量审查、单元测试≥80%、集成/E2E/性能测试
- ✅ **隔离工作环境** - Git-Worktrees 隔离，修复失败可安全重做
- ✅ **规范沉淀** - 自动归档已验证的规范到企业级规范库
- ✅ **即插即用** - 交互式问卷、自动化流程、详尽文档

#### 工作流

```
需求 → /spec-creation (规范) 
      → /code-designer (设计) 
      → /code-task (任务) 
      → /code-execute (编码+TDD) 
      → /code-test (测试) 
      → /spec-archive (归档) 
      → 完成 ✅
```

#### 包含 6 个核心 Skills

| Skill | 阶段 | 功能 | 输出 |
|-------|------|------|------|
| **spec-creation** | Spec | 需求分析和规范生成（BDD格式） | spec-dev/{req}/spec/ |
| **code-designer** | Design | 架构和技术设计规划 | spec-dev/{req}/design/design.md |
| **code-task** | Task | 任务分解和清单定义 | spec-dev/{req}/tasks/tasks.md |
| **code-execute** | Execute | 代码生成 + TDD流程 + Git-Worktrees隔离 | src/ + 执行报告 |
| **code-test** | Test | 集成/E2E/性能测试 + 闭环验证 | tests/ + 测试报告 |
| **spec-archive** | Archive | 规范归档和沉淀到企业级规范库 | spec-dev/spec/ |

#### 包含 5 个核心 Agents

- **code-architect** - 架构设计专家，分析项目模式、规划架构
- **code-explorer** - 代码探索专家，深度分析相似功能实现
- **code-executor** - 代码执行专家，逐个编写高质量代码
- **code-reviewer** - 代码审查专家，检查规范和质量问题
- **spec-archiver** - 规范归档专家，对标分析和智能合并

#### 快速开始

```bash
# 0. 进入插件
/sdd-tdd-dev:sdd-dev
# 输入需求内容

# 1. 生成规范（交互式）
/spec-creation

# 2-6. 自动执行后续步骤
# - /code-designer 设计阶段（自动）
# - /code-task 任务分解（自动）
# - /code-execute 代码执行+TDD（自动，使用Git-Worktrees隔离）
# - /code-test 测试验证（自动）
# - /spec-archive 规范归档（自动）
```

#### 📖 完整文档

- 📌 [完整 README](./plugins/sdd-tdd-dev/README.md) - 插件详细介绍和工作流
- 🚀 [快速开始](./plugins/sdd-tdd-dev/docs/USAGE.md) - 5分钟上手指南
- 📋 [安装说明](./plugins/sdd-tdd-dev/docs/INSTALLATION.md) - 详细的安装和故障排查
- 💡 [最佳实践](./plugins/sdd-tdd-dev/docs/BEST_PRACTICES.md) - 全栈 SDD 最佳实践
- 🎓 [完整案例](./plugins/sdd-tdd-dev/docs/使用案例.md) - 前端/后端/全栈真实项目案例
- 🔀 **[Git-Worktrees 工作流](./plugins/sdd-tdd-dev/skills/code-execute/git-worktrees-guide.md)** - 隔离工作环境详细指南（v2.3.1+）
- ⚡ **[Worktrees 快速参考](./plugins/sdd-tdd-dev/skills/code-execute/QUICK_REFERENCE.md)** - 日常使用快速查询卡片
- 📚 **[完整 TDD 工作流](./plugins/sdd-tdd-dev/docs/TDD_COMPLETE_WORKFLOW.md)** - Phase 1-3 端到端 TDD 指南（5000+行）
- 🧪 **[TDD Phase 3 示例](./plugins/sdd-tdd-dev/docs/TDD_PHASE3_EXAMPLE.md)** - 集成、E2E、性能测试实例（6000+行）
- 🛠️ [Skills 详解](./plugins/sdd-tdd-dev/skills/README.md) - 6个Skills的详细工作流
- 🏗️ [架构设计](./plugins/sdd-tdd-dev/docs/ARCHITECTURE.md) - 技术架构和设计原则
- 📝 [更新日志](./plugins/sdd-tdd-dev/CHANGELOG.md) - 版本更新历史

---

## 🎯 使用场景

### 场景 1：新增功能模块

适合：React/Vue 等前端项目新增功能、页面、组件

```bash
# 1. 初始化规范（首次）
/spec-creation

# 2. 设计新功能
/code-designer 需要新增用户管理模块

# 3. 任务分解
/code-task design.md

# 4. 代码执行（含 TDD + Git-Worktrees）
/code-execute tasks.md

# 5. 测试验证
/code-test tasks.md
```

### 场景 2：后端 API 开发

适合：Node.js/Python/Go 等后端项目新增 API 和服务

```bash
# 1. 规范化需求
/spec-creation 需要新增订单 API，支持增删改查和复杂查询

# 2-5. 自动执行设计、任务、执行、测试流程
```

### 场景 3：数据库和微服务

适合：数据库设计、微服务架构、事件驱动等复杂场景

```bash
# 支持多服务协作、事件驱动、分布式事务等高级特性
```

### 场景 4：现有代码重构

适合：优化现有代码结构、提升代码质量、性能优化

```bash
/code-designer 需要重构登录认证模块，改用 OAuth2 + JWT
# 后续自动执行设计、任务、执行、测试、归档流程
```

---

## 🏆 核心优势

### 对比传统 AI 辅助开发

| 特性 | sdd-tdd-dev | 传统 AI 辅助 |
|------|-------------|-----------|
| **规范驱动** | ✅ 基于定义的规范生成代码 | ❌ 无统一规范 |
| **工作流完整** | ✅ Spec-Design-Task-Execute-Test-Archive | ⚠️ 代码生成 + 人工测试 |
| **TDD 实现** | ✅ RED-GREEN-REFACTOR-REVIEW完整流程 | ❌ 无 TDD 保证 |
| **质量保证** | ✅ 规范审查 + 质量审查 + 高层测试 | ⚠️ 单一审查 |
| **可追踪性** | ✅ 完整的规范-设计-任务-代码-测试链 | ⚠️ 难以追踪 |
| **规范沉淀** | ✅ 自动归档验证规范到规范库 | ❌ 规范无法沉淀 |
| **工作隔离** | ✅ Git-Worktrees隔离每个Task | ⚠️ 修改混乱 |
| **团队一致性** | ✅ 确保所有代码符合规范 | ⚠️ 每个人可能不同 |

---

## 📊 版本信息

### sdd-tdd-dev

- **当前版本**: v2.3.1
- **发布日期**: 2026-03-23
- **最新特性**:
  - ✨ Git-Worktrees 隔离工作环境
  - ✨ 完整 TDD 实现（Phase 1-3）
  - ✨ 规范自动归档和沉淀机制
  - ✨ 5个专业 Agents 并行协作

- **包含 Skills**: 6 个
  - spec-creation - 规范生成
  - code-designer - 代码设计
  - code-task - 任务分解
  - code-execute - 代码实现 + TDD
  - code-test - 测试验证
  - spec-archive - 规范归档

- **支持框架**: 
  - 前端：React, Vue, Angular, Svelte
  - 后端：Node.js, Python, Go, Java
  - 数据库：PostgreSQL, MongoDB, MySQL
  - 测试：Jest, Vitest, Pytest, Cypress, k6

- **文档**: 完整（15+ 文档）
- **测试覆盖率**: 单元测试≥80% + 集成/E2E/性能测试

---

## 🚀 开始使用

### 前置要求

- Claude Code 已安装
- Node.js 14+ 环境
- 项目已初始化

### 安装步骤

1. **在 Claude 中添加 Marketplace**
   ```bash
   /plugin -> Marketplaces -> +Add Marketplace
   https://github.com/DestinyV/sdd-tdd-dev.git
   ```

2. **安装 sdd-tdd-dev 插件**
   ```bash
   /plugin -> Marketplaces -> DestinyV-marketplace -> sdd-tdd-dev -> 安装
   ```

3. **开始使用工作流**
   ```bash
   /sdd-tdd-dev:sdd-dev
   # 根据提示输入需求
   ```

详见 [sdd-tdd-dev 快速开始](./plugins/sdd-tdd-dev/docs/USAGE.md)

---

## 📖 文档导航

### 市场级文档

- [创建新的 Plugin](#2️⃣-创建新的-plugin) - Plugin 标准结构
- [贡献指南](#🤝-如何贡献) - 如何贡献新插件

### sdd-tdd-dev 用户文档

- [完整 README](./plugins/sdd-tdd-dev/README.md) - 插件全面介绍
- [快速开始](./plugins/sdd-tdd-dev/docs/USAGE.md) - 5分钟上手
- [安装说明](./plugins/sdd-tdd-dev/docs/INSTALLATION.md) - 安装和故障排查
- [最佳实践](./plugins/sdd-tdd-dev/docs/BEST_PRACTICES.md) - 全栈 SDD 最佳实践
- [完整案例](./plugins/sdd-tdd-dev/docs/使用案例.md) - 真实项目案例

### sdd-tdd-dev 技术文档

- [Skills 详解](./plugins/sdd-tdd-dev/skills/README.md) - 6个Skills详细说明
- [架构设计](./plugins/sdd-tdd-dev/docs/ARCHITECTURE.md) - 技术架构和原则
- [Git-Worktrees 工作流](./plugins/sdd-tdd-dev/skills/code-execute/git-worktrees-guide.md) - 隔离工作环境指南
- [完整 TDD 工作流](./plugins/sdd-tdd-dev/docs/TDD_COMPLETE_WORKFLOW.md) - Phase 1-3 TDD 详解
- [TDD Phase 3 示例](./plugins/sdd-tdd-dev/docs/TDD_PHASE3_EXAMPLE.md) - 高层测试实例
- [Claude 工作指南](./plugins/sdd-tdd-dev/CLAUDE.md) - Claude 的架构和设计指南
- [更新日志](./plugins/sdd-tdd-dev/CHANGELOG.md) - 版本更新历史

---

## 💡 工作流示例

### React 项目 - 新增订单管理模块

```bash
# Step 1: 进入插件和输入需求
$ /sdd-tdd-dev:sdd-dev
输入需求：需要新增订单管理模块，支持搜索、排序、分页、批量操作

# Step 2: 规范化需求（交互式）
$ /spec-creation
输出: spec-dev/order-management/spec/ 
  - requirement.md (规范入口)
  - scenarios/*.md (BDD场景)
  - data-models.md (数据模型)
  - business-rules.md (业务规则)
  - glossary.md (术语表)

# Step 3-7: 自动执行（无需用户干预）
  ✓ /code-designer 设计架构
    输出: spec-dev/order-management/design/design.md
  
  ✓ /code-task 分解任务
    输出: spec-dev/order-management/tasks/tasks.md
  
  ✓ /code-execute 代码实现 + TDD（使用Git-Worktrees隔离）
    输出: src/ + 执行报告
    特色：
      - RED-GREEN-REFACTOR-REVIEW四阶段TDD
      - 每个Task独立worktree，修复失败可安全重做
      - 规范审查 + 代码质量审查两阶段
  
  ✓ /code-test 高层测试
    输出: tests/ + 测试报告
    包含：集成测试、E2E测试、性能测试
  
  ✓ /spec-archive 规范归档
    输出: spec-dev/spec/ （已合并到企业级规范库）
    作用：沉淀规范知识，供后续需求对标

✅ 完成！代码质量达标，规范已沉淀
```

---

## 🔧 定制和扩展

### 定制 sdd-tdd-dev 工作流

sdd-tdd-dev 的工作流可根据项目需求定制：

1. **定制规范维度**
   - 编辑 `skills/spec-creation/SKILL.md`
   - 修改问卷和规范生成逻辑

2. **定制代码质量标准**
   - 编辑 `skills/code-execute/` 中的审查 Prompt
   - 调整代码检查规则和风格要求

3. **定制测试策略**
   - 编辑 `skills/code-test/SKILL.md`
   - 修改测试框架、覆盖率要求

4. **定制 TDD 流程**
   - 编辑 `skills/code-execute/tdd-implementer-prompt.md`
   - 调整 RED-GREEN-REFACTOR-REVIEW 各阶段的标准

详见 [sdd-tdd-dev - 如何定制工作流](./plugins/sdd-tdd-dev/README.md#如何定制工作流)

### 贡献新插件到市场

如果你有好的插件想贡献到市场：

1. 按照市场规范组织你的插件
2. 编写完整的文档和使用指南
3. 提交 Pull Request
4. 社区审核通过后即可发布

详见 [贡献指南](#🤝-如何贡献)

---

## 🤝 如何贡献

### 贡献新插件

我们欢迎社区贡献高质量的插件！

#### 贡献流程

1. **Fork 本仓库**
   ```bash
   git clone https://github.com/DestinyV/sdd-tdd-dev.git
   cd DestinyV-marketplace
   ```

2. **创建你的插件**

   按照本 README 中「**2️⃣ 创建新的 Plugin**」部分的标准结构创建你的插件，放在 `/plugins` 目录中。

3. **本地测试**
   ```bash
   cd your-project
   npx skills add ../DestinyV-marketplace/plugins/your-plugin-name
   # 测试插件功能
   ```

4. **提交审核**
   ```bash
   git push origin feature/your-plugin
   # 创建 Pull Request
   ```

5. **通过审核后发布**
   - 社区管理员审核代码质量
   - 验证功能完整性
   - 合并到 main 分支
   - 更新市场目录

#### 审核标准

- ✅ 功能完整且经过测试
- ✅ 文档清晰且示例充分
- ✅ 代码质量高（无明显 bug）
- ✅ 遵循 Claude Code 插件规范
- ✅ 包含使用协议和许可证

### 改进现有插件

如果你发现 bug 或有改进建议：

1. **报告 Issue**
   - 描述问题或建议
   - 提供复现步骤（如果是 bug）

2. **提交 Pull Request**
   - Fork 仓库
   - 创建特性分支
   - 提交改进
   - 创建 Pull Request

3. **参与讨论**
   - 在 Discussions 中分享想法
   - 帮助其他用户解答问题

---

## 📈 市场统计

- **总插件数**: 1
- **总 Skills 数**: 6
- **支持的框架**: React, Vue, Angular, Svelte, Node.js, Python, Go, Java
- **支持的数据库**: PostgreSQL, MongoDB, MySQL
- **支持的测试框架**: Jest, Vitest, Pytest, Cypress, k6
- **月活跃用户**: 持续增长中
- **社区贡献**: 欢迎加入

---

## 🎓 学习资源

### 新手入门

1. 阅读 [sdd-tdd-dev README](./plugins/sdd-tdd-dev/README.md)
2. 跟随 [快速开始指南](./plugins/sdd-tdd-dev/docs/USAGE.md)
3. 执行 `/sdd-tdd-dev:sdd-dev` 进入插件
4. 根据提示输入需求内容
5. 选择一个小功能进行完整流程试运行

### 深入学习

1. 阅读 [最佳实践](./plugins/sdd-tdd-dev/docs/BEST_PRACTICES.md) 了解每阶段的最佳做法
2. 研究 [完整案例](./plugins/sdd-tdd-dev/docs/使用案例.md) 学习真实项目
3. 学习 [完整 TDD 工作流](./plugins/sdd-tdd-dev/docs/TDD_COMPLETE_WORKFLOW.md) 理解 Phase 1-3
4. 掌握 [Git-Worktrees 工作流](./plugins/sdd-tdd-dev/skills/code-execute/git-worktrees-guide.md) 的隔离机制
5. 理解 [架构设计](./plugins/sdd-tdd-dev/docs/ARCHITECTURE.md) 的技术细节
6. 学习如何 [定制工作流](./plugins/sdd-tdd-dev/README.md#如何定制工作流)

### 团队推广

1. 确保团队理解 SDD+TDD 的核心原则
2. 为项目编写定制化的设计规范
3. 培训团队成员使用整个工作流
4. 建立基于规范的代码审查流程
5. 定期总结和沉淀团队最佳实践

---

## 🔗 相关资源

- [Claude Code 官方文档](https://claude.com/claude-code)
- [Claude Code Plugins 开发指南](https://docs.anthropic.com/claude-code)
- [规范驱动开发（SDD）理念](./plugins/sdd-tdd-dev/docs/ARCHITECTURE.md)
- [测试驱动开发（TDD）原则](./plugins/sdd-tdd-dev/docs/TDD_COMPLETE_WORKFLOW.md)

---

## 📄 许可证

本市场和所有包含的插件均采用 **MIT License**

详见各插件目录下的 [LICENSE](./LICENSE) 文件

---

## 🙏 致谢

感谢所有贡献者和用户的支持！特别感谢：

- Claude 团队提供的 Claude Code 平台
- 开源社区的灵感和最佳实践
- 所有参与测试和反馈的用户

---

**让 AI 辅助的规范驱动、测试驱动开发变得高效、可信赖！** 🚀
