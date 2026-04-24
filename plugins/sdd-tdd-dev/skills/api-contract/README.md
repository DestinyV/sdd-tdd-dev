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

1. **完整性** - 每个前端交互都有对应后端接口
2. **命名规范** - RESTful 风格一致性
3. **类型一致性** - 前后端类型匹配
4. **错误处理** - 错误码完整
5. **项目约定遵循** - 符合现有项目风格

## 版本管理

api-contract.md 带版本号（初始 v1），后续接口变更必须更新版本号并记录变更日志。

## 快速模式

当 `mode: quick` 时：
- 简化为从 spec 场景中提取接口清单，生成精简版 api-contract.md
- 审查范围缩小为完整性和错误处理两项
