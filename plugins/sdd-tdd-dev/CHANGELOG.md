# 更新日志

所有重要的项目更新将在此文档中记录。

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
