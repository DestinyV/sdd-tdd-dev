# 更新日志

所有重要的项目更新将在此文档中记录。

## [2.5.0] - 2026-04-21

### 新增 - 六大强制约束增强

#### 1. HARD-GATE 拦截点
- 在 code-execute 关键节点增加硬拦截机制
- 测试框架可用性确认（无测试框架时 AskUserQuestion 暂停）
- 测试文件存在性检查（standard 模式下无测试文件不允许编码）
- 工作模式最终确认（AskUserQuestion 三选一）

#### 2. 子代理派遣强制化
- standard 模式下必须使用 `Agent(subagent_type="sdd-tdd-dev:code-executor")` 工具
- 提供具体的工具调用模板
- 快速模式可选子代理或直接主上下文执行

#### 3. TDD 状态追踪机制
- 每个 Task 必须输出 TDD 四阶段日志（RED→GREEN→REFACTOR→REVIEW）
- 执行报告包含 TDD 总览表
- tasks.md 模板新增 TDD 追踪表

#### 4. 测试基础设施检查前置
- code-execute 步骤1.5 新增测试环境验证
- 无测试框架时自动暂停并询问用户
- 支持 Jest/Vitest/Pytest 多种框架选择

#### 5. 执行流程自检清单
- code-execute 结尾强制 12 项自检
- 自检不通过 → 回到对应 Task 重新执行
- 不通过不允许生成执行报告、不允许进入 code-test

#### 6. 快速模式/标准模式明确切换
- spec-creation 输出 requirement.md 新增 6 字段模式标签
  - 模式、TDD要求、子代理要求、测试覆盖要求、触发原因、影响范围
- code-task 在 tasks.md 头部继承模式标签
- code-execute/code-test 读取并严格遵循模式标签

### 修改文件
- `skills/code-execute/SKILL.md` - 新增步骤0/1.5/3.7/3.8，强化约束
- `skills/spec-creation/SKILL.md` - 增强模式标签输出
- `skills/code-task/SKILL.md` - 新增模式标签读取和传递
- `skills/code-test/SKILL.md` - 新增模式标签读取
- `skills/code-task/templates/tasks-document-template.md` - 新增 TDD 追踪表
- `skills/package.json` - 版本升级，key_features 更新
- `CLAUDE.md` - 版本升级，工作流特色更新

---

## [1.0.0] - 2026-02-09

### 新增
- ✅ **spec-generator skill** - 帮助团队快速生成ai-doc规范
  - 交互式问卷引导
  - 支持用户自定义业务模式
  - 自动生成README.md和使用指南

- ✅ **ai-planning skill** - 需求分析和方案设计
  - 需求分析
  - 技术设计
  - 问题确认
  - 输出ExecutionGuide

- ✅ **ai-code-execution skill** - 代码生成和实现
  - 加载ExecutionGuide
  - 参考Class驱动
  - 代码生成
  - 微调治理
  - 输出执行报告

- ✅ **ai-test-creation skill** - 测试生成和验证
  - 功能点提取
  - 测试用例设计
  - 测试代码生成
  - 闭环验证
  - 输出测试报告

### 文档
- ✅ README.md - 项目总体介绍
- ✅ 安装说明 - 详细的安装步骤
- ✅ 使用指南 - 详细的使用方式和场景
- ✅ CHANGELOG.md - 更新日志

### 特性
- ✅ 完全通用的规范框架（无任何硬编码的业务逻辑）
- ✅ 参考Class驱动的开发方式
- ✅ 完整的Plan-Exe-Test三阶段工作流
- ✅ 支持任何技术栈和行业

---

## 版本计划

### v1.1.0（计划中）
- 支持更多的技术栈示例（Go、Rust等）
- 增强的错误处理和提示
- 更完善的文档

### v1.2.0（计划中）
- 支持多team的规范共享
- 规范版本管理
- 开发流程的可视化

### v2.0.0（计划中）
- 完全的AI驱动的设计系统
- 自动化的代码review
- 生产环境的完全自动化

---

## 贡献指南

欢迎提交Issue和Pull Request！

### 报告Bug
- 描述问题的详细步骤
- 提供错误日志和截图
- 说明你的环境信息

### 建议功能
- 描述用例
- 说明期望的行为
- 提供参考链接（如有）

### 提交Pull Request
1. Fork项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. Commit更改 (`git commit -m 'Add some AmazingFeature'`)
4. Push到分支 (`git push origin feature/AmazingFeature`)
5. 打开Pull Request

---
