# [需求名称] 需求文档

**版本**：1.0
**日期**：[YYYY-MM-DD]
**状态**：已确认

---

## 工作模式

<!--
  🚧 此模式标识会被后续阶段（code-designer、code-task、code-execute）读取并强制执行
  请不要手动修改，由 spec-creation 技能自动生成
-->
- 模式：[standard | quick]
- TDD要求：[必须执行 | 可简化]
- 子代理要求：[必须使用 | 可跳过]
- 测试覆盖要求：[≥85% | ≥70%]
- 触发原因：[用户明确要求 / 默认标准模式]
- 影响范围：[跳过的步骤清单]

## 项目模式

<!--
  🚧 由 code-architect agent 在 code-designer 阶段自动识别和设置
-->
- **project-mode**: [frontend | backend | fullstack]
- **needs-database**: [true | false]
- **sql-dialect**: [mysql | postgresql | sqlite | sqlserver | 未确定]（needs-database=true 时必填）

> 注：project-mode 和 needs-database 由 code-architect agent 在 code-designer 阶段自动识别和设置。
> sql-dialect 由 spec-creation 通过 AskUserQuestion 确认（needs-database=true 时），或由 code-designer 自动检测。

---

## 概述

[需求的简要说明]

---

## 核心功能

- [功能1]
- [功能2]
- [功能N]

---

## 场景列表

| 场景 | 描述 | Case数 | 优先级 |
|------|------|--------|--------|
| [场景1] | [描述] | [N] | [优先级] |
| [场景2] | [描述] | [N] | [优先级] |

详见：[场景文档链接](./scenarios/)

---

## 数据模型

[主要数据模型的简要说明]

详见：[data-models.md](./data-models.md)

---

## 业务规则

[主要业务规则的简要说明]

详见：[business-rules.md](./business-rules.md)

---

## 术语表

详见：[glossary.md](./glossary.md)

---

## 确认记录

### 确认轮次1
日期：[YYYY-MM-DD]
- [ ] 需求理解确认
- [ ] 场景列表确认
- [ ] 核心功能确认

### 确认轮次2
日期：[YYYY-MM-DD]
- [ ] 场景细节确认
- [ ] 异常情况确认
- [ ] 数据模型确认

### 确认轮次3
日期：[YYYY-MM-DD]
- [ ] 最终规范确认
- [ ] 无遗漏问题确认

---

## 多项目协作 ⭐🆕

<!--
  当需求涉及多个项目/仓库时由 spec-creation 填充
  单项目场景下标注"不涉及"
-->
- 协作模式：[single | monorepo | multi-repo | same-repo]（默认 single）
- 涉及项目：[不涉及 / project-a, project-b, project-c]
- 协作链路类型：[不涉及 / 上下游服务 | 前后端分离仓库 | 微服务集群 | Monorepo多Package]

> 注：协作模式=single 时，以下字段留空。非 single 时由 code-designer 阶段生成详细协作计划。
