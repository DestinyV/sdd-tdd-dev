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

## 各阶段职责

### 1. spec-creation

- 生成 BDD 格式的需求规范
- 识别项目模式（project-mode: frontend/backend/fullstack）
- 识别数据库需求（needs-database: true/false）

### 2. code-designer

- **全栈场景识别**：读取 spec 标签，判定是否需要数据库设计和接口契约
- **现有项目约定扫描**：扫描已有接口的 URL 命名、响应格式、认证方式等
- **数据库设计**（needs-database: true 时）：表结构、索引、迁移方案
- **接口清单初稿**：作为 api-contract 阶段的输入

### 2.5. api-contract（新增）

- **接口契约定义**：将接口清单初稿转化为正式的 api-contract.md
- **五维度审查**：完整性、命名规范、类型一致性、错误处理、项目约定遵循
- **版本管理**：带版本号（初始 v1），接口变更需更新版本
- **HARD-GATE**：审查不通过拒绝进入 code-task

### 3. code-task

- **接口对齐强制检查**：验证 api-contract.md 存在且审查通过
- **provides/consumes 依赖**：每个 Task 标记实现/消费的接口
- **依赖图生成**：消费方 Task 必须在提供方 Task 完成后执行

### 4. code-execute

- **接口契约验证**：Task 执行前验证 provides/consumes 接口在 api-contract.md 中存在
- **版本锁定**：Task 执行期间锁定 api-contract.md 版本号
- **接口契约遵循**：后端严格按契约定义实现，前端严格按契约定义调用

### 5. code-test

- **接口契约测试**：验证后端实际返回的字段、类型、结构与 api-contract.md 一致
- **错误码覆盖测试**：契约中定义的所有错误码都有对应实现
- **不允许跳过**：契约测试失败必须修复后重测

## 快速模式适配

当 `mode: quick` 时，api-contract 阶段简化为：
- 从 spec 场景中提取接口清单生成精简版契约
- 审查范围缩小为完整性和错误处理
- 审查通过标准：2 项全部通过

## 向后兼容

- frontend/backend 项目自动跳过 api-contract 阶段
- 已有项目流程不受影响
