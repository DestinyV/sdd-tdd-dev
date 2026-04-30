# 项目文档发现指南

> 本指南适用于 sdd-tdd-dev 工作流中的 **spec-creation**、**code-designer** 和 **code-task** 阶段。
> 在所有涉及项目理解的设计和分析环节，都应先执行项目文档发现。

## 1. 为什么要发现项目文档

项目中的自描述文档（CLAUDE.md、AGENTS.md、ARCHITECTURE.md 等）包含了项目创建者和维护者**明确声明**的架构约定、编码规范和开发流程。这些文档是项目意图的权威表达，其优先级应高于从代码中推断的约定。

**核心原则**：文档声明的约定 > 代码推断的约定

## 2. 文档扫描路径清单

按优先级顺序扫描，找到即读，最多读取 10 个文档：

### P0 — AI 编码指南（最高优先级）

| 路径 | 内容特点 |
|------|---------|
| `CLAUDE.md` / `.claude/CLAUDE.md` | 项目对 AI 编码的明确要求，包括架构原则、开发流程、禁止事项 |
| `AGENTS.md` | AI Agent 专用指南，通常包含详细的代码生成规则 |
| `GEMINI.md` | Gemini CLI 项目指南 |

### P1 — 项目核心文档

| 路径 | 内容特点 |
|------|---------|
| `README.md` | 项目概述、技术栈、目录结构、快速开始 |
| `ARCHITECTURE.md` / `docs/ARCHITECTURE.md` | 架构描述、模块划分、技术决策（ADR） |
| `CONTRIBUTING.md` / `docs/CONTRIBUTING.md` | 贡献指南、编码规范、提交流程、测试要求 |

### P2 — 项目扩展文档

| 路径 | 内容特点 |
|------|---------|
| `docs/**/*.md` | 取前 3 个最核心的文档（如架构设计、开发规范、API 文档） |
| `wiki/**/*.md` | 取前 3 个最核心的 Wiki 文档 |
| `.cursor/rules/**` | Cursor 编辑器的规则文件 |
| `.github/CONTRIBUTING.md` | GitHub 工程规范 |

### P3 — 技术栈声明

| 路径 | 内容特点 |
|------|---------|
| `package.json` / `pyproject.toml` / `pom.xml` / `go.mod` | 依赖声明、脚本命令、技术栈信息 |

## 3. 从各类型文档中提取什么

| 文档类型 | 提取目标 |
|---------|---------|
| CLAUDE.md / AGENTS.md | 架构原则、开发流程约束、编码禁止事项、测试要求、提交规范 |
| README.md | 项目定位、技术栈、目录结构说明、核心功能模块 |
| ARCHITECTURE.md | 架构模式选择、模块边界、分层定义、技术决策及原因 |
| CONTRIBUTING.md | 命名规范、代码风格、分支策略、审查流程 |
| docs/*.md | 具体模块设计、API 规范、数据库规范、部署流程 |
| package.json 等 | 框架版本、关键依赖、构建工具、测试框架 |

## 4. 冲突处理规则

当文档声明的约定与实际代码不一致时：

1. **文档优先级最高**：以文档声明的约定为准进行新设计
2. **记录冲突**：在输出的「项目文档摘要」中标注"⚠️ 文档声明 X，实际代码使用 Y"
3. **选择策略**：
   - 如果文档是最新的（有近期更新痕迹），严格遵循文档
   - 如果文档明显过时（引用的技术栈已不再使用），以代码为准但明确标注
   - 如果不确定，提示使用者确认

## 5. 项目文档摘要输出格式

所有阶段的 agent 在执行项目分析时，应在输出最前面附加以下摘要：

```markdown
## 项目文档摘要

### 已发现文档
| 文件 | 状态 | 关键内容摘要 |
|------|------|-------------|
| CLAUDE.md | 已读取 | 架构原则：Clean Architecture，禁止直接操作 DB，必须通过 Repository 层 |
| README.md | 已读取 | 项目为 Vue 3 + Node.js 全栈应用，前后端分离部署 |
| ARCHITECTURE.md | 未发现 | - |

### 项目自述架构
- 架构模式：Clean Architecture（domain/application/infrastructure/presentation 四层）
- 模块划分：按业务领域划分（order/payment/user/notification）
- 技术栈：Vue 3 + Pinia + TypeScript（前端），Node.js + Express + Prisma（后端）
- 关键约定：
  - 领域层零外部依赖
  - 所有 API 响应格式 {code, data, message}
  - 所有服务通过依赖注入
  - 提交信息遵循 Conventional Commits
```

## 6. Token 控制策略

- 最多读取 10 个文档
- P0 文档优先读取（最多 3 个）
- `docs/**/*.md` 和 `wiki/**/*.md` 只取前 3 个最相关的文件
- 每个文档读取后输出 1-2 句话摘要，不全文引用
