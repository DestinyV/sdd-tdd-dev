# TDD集成实施计划 - 详细任务分解

## 项目概览

**目标**：将TDD（测试驱动开发）完整融合到sdd-dev-plugin工作流

**总时间**：7周（5-7周基础工作 + 1周培训）

**交付物**：
- ✅ 修改2个现有Skill（spec-creation、code-task、code-execute、code-test）
- ✅ 创建1个新Skill（test-design）
- ✅ 创建1个新Agent（test-designer）
- ✅ 更新文档和示例

---

## 阶段1：轻量级改进（第1周）

### 任务1.1：修改spec-creation - 添加TEST-VERIFY格式

**目标**：在spec阶段定义测试场景和验收标准

**工作清单**：
- [ ] 1.1.1 修改 `plugins/sdd-dev-plugin/skills/spec-creation/SKILL.md`
  - 添加"TEST-VERIFY"格式说明
  - 定义验收标准格式
  - 定义mock数据示例格式
  - 添加示例场景

- [ ] 1.1.2 创建 `plugins/sdd-dev-plugin/skills/spec-creation/test-verify-template.md`
  - TEST-VERIFY格式模板
  - Mock数据模板
  - 验收标准模板

- [ ] 1.1.3 修改 `plugins/sdd-dev-plugin/commands/sdd-dev.md`
  - 更新spec阶段说明
  - 添加TEST-VERIFY的说明

**输出**：
- `spec-dev/{name}/spec/scenarios/` (新增TEST-VERIFY)
- 更新的文档和模板

---

### 任务1.2：修改code-task - 添加test-case-mapping

**目标**：在task阶段关联test case和验收标准

**工作清单**：
- [ ] 1.2.1 修改 `plugins/sdd-dev-plugin/skills/code-task/SKILL.md`
  - 添加"Test Case Mapping"说明
  - 定义test-case-mapping表格格式
  - 添加验收标准的测试覆盖要求

- [ ] 1.2.2 创建 `plugins/sdd-dev-plugin/skills/code-task/test-case-mapping-template.md`
  - 映射表格模板
  - 如何从TEST-VERIFY生成test case
  - 示例

- [ ] 1.2.3 修改 `plugins/sdd-dev-plugin/commands/sdd-dev.md`
  - 更新task阶段说明

**输出**：
- `spec-dev/{name}/tasks/tasks.md` (新增test-case-mapping)
- 更新的文档和模板

---

### 任务1.3：阶段1文档和验证

**工作清单**：
- [ ] 1.3.1 创建 `docs/TDD_PHASE1_EXAMPLE.md`
  - 完整的阶段1示例
  - 展示TEST-VERIFY的定义
  - 展示test-case-mapping

- [ ] 1.3.2 验证兼容性
  - 确保不破坏现有工作流
  - 新格式是可选的（向后兼容）

**输出**：
- 阶段1示例文档
- 兼容性验证报告

---

## 阶段2：核心TDD（第2-4周）

### 任务2.1：创建test-design Skill

**目标**：设计和生成测试规范及test case

**工作清单**：
- [ ] 2.1.1 创建 `plugins/sdd-dev-plugin/skills/test-design/` 目录

- [ ] 2.1.2 创建 `plugins/sdd-dev-plugin/skills/test-design/SKILL.md`
  - 职责说明
  - 工作流程（7个步骤）
  - 输入输出定义
  - 关键原则

- [ ] 2.1.3 创建 `plugins/sdd-dev-plugin/skills/test-design/test-case-designer-prompt.md`
  - test-designer Agent的提示词
  - 如何生成test case
  - mock数据定义
  - fixture生成规则

- [ ] 2.1.4 创建 `plugins/sdd-dev-plugin/skills/test-design/test-spec-template.md`
  - test-spec.md的模板
  - Test Case表格格式
  - Mock数据格式
  - 测试框架代码模板

- [ ] 2.1.5 创建 `plugins/sdd-dev-plugin/skills/test-design/README.md`
  - test-design的使用指南
  - 工作流说明
  - 常见问题

**输出**：
- `spec-dev/{name}/tests/test-spec.md` (测试规范)
- `spec-dev/{name}/tests/fixtures.json` (mock数据)
- `spec-dev/{name}/tests/test-*.template` (测试框架)

---

### 任务2.2：创建test-designer Agent

**目标**：专业的测试设计Agent

**工作清单**：
- [ ] 2.2.1 创建 `plugins/sdd-dev-plugin/agents/test-designer.md`
  - 角色定义
  - 核心能力
  - 与其他Agent的关系
  - 决策规则
  - 关键指标

**内容**：
```
# test-designer Agent

## 角色
测试规范设计专家，负责将业务需求转换为详细的test case和测试规范

## 核心能力
1. Test Case设计 - 从TEST-VERIFY提取test case
2. Mock策略定义 - 定义依赖的mock
3. Fixture生成 - 生成测试数据
4. 框架代码 - 生成测试框架代码
5. 覆盖率分析 - 检查test case覆盖是否完整

## 与其他Agent的关系
- spec-creation 提供 TEST-VERIFY
- code-task 提供 Task定义
- code-executor 消费输出（运行这些测试）

## 关键指标
- Test Case完整性：所有TEST-VERIFY都有对应test case
- Mock清晰性：每个test的依赖都明确定义
- 可执行性：生成的框架代码可直接运行
```

---

### 任务2.3：修改code-execute - 集成TDD流程

**目标**：实现TDD的红-绿-重构循环

**工作清单**：
- [ ] 2.3.1 修改 `plugins/sdd-dev-plugin/skills/code-execute/SKILL.md`
  - 添加"TDD流程"章节
  - 修改执行流程（先生成测试，再生成代码）
  - 添加红-绿-重构说明
  - 更新输入输出

- [ ] 2.3.2 创建 `plugins/sdd-dev-plugin/skills/code-execute/tdd-flow.md`
  - 详细的TDD流程说明
  - Phase 1: Test Implementation (红)
  - Phase 2: Implementation (绿)
  - Phase 3: Refactor (重构)
  - Phase 4: Review (审查)

- [ ] 2.3.3 创建/修改 `plugins/sdd-dev-plugin/skills/code-execute/tdd-implementer-prompt.md`
  - TDD方式的实现提示词
  - 如何运行测试
  - 如何生成最小化实现
  - 如何重构

- [ ] 2.3.4 更新 `plugins/sdd-dev-plugin/commands/sdd-dev.md`
  - 更新execute阶段说明
  - 添加TDD流程说明

**流程**：
```
For each Task:
  Step 1: 读取 test-design 的 test-spec.md
  Step 2: 生成测试代码 (test-*.test.ts)
  Step 3: 运行测试 → 预期失败 (红)
  Step 4: 生成最小化实现代码
  Step 5: 运行测试 → 预期通过 (绿)
  Step 6: 重构优化 (保持通过)
  Step 7: 规范审查
  Step 8: 质量审查
```

**输出**：
- 测试代码 + 实现代码（同步）
- 测试通过率100%
- 覆盖率≥80%

---

### 任务2.4：更新code-executor Agent（或创建tdd-executor）

**目标**：支持TDD的代码执行

**工作清单**：
- [ ] 2.4.1 修改 `plugins/sdd-dev-plugin/agents/code-executor.md`
  或创建 `plugins/sdd-dev-plugin/agents/tdd-executor.md`
  - 添加TDD能力说明
  - 红-绿-重构循环
  - 与test-designer的协作
  - 新的决策规则

---

### 任务2.5：阶段2示例和文档

**工作清单**：
- [ ] 2.5.1 创建 `docs/TDD_PHASE2_EXAMPLE.md`
  - 完整的TDD实施示例
  - test-design的输出
  - code-execute的TDD流程
  - 最终的代码和测试

- [ ] 2.5.2 创建 `docs/TDD_TROUBLESHOOTING.md`
  - 常见问题和解决方案
  - 红-绿-重构的常见错误
  - 调试技巧

**输出**：
- test-design Skill完整文档
- test-designer Agent定义
- 修改后的code-execute SKILL
- 更新的code-executor Agent
- 示例和常见问题

---

## 阶段3：优化code-test（第5-6周）

### 任务3.1：修改code-test - 聚焦集成和E2E

**目标**：清化code-test职责，移除单元测试生成

**工作清单**：
- [ ] 3.1.1 修改 `plugins/sdd-dev-plugin/skills/code-test/SKILL.md`
  - 移除"单元测试设计和生成"章节
  - 添加"集成测试设计"章节（增强）
  - 添加"E2E测试设计"章节（增强）
  - 添加"性能测试"章节（新增）
  - 更新"覆盖率验证"为"覆盖率检查"
  - 重新组织职责结构

- [ ] 3.1.2 创建 `plugins/sdd-dev-plugin/skills/code-test/integration-test-prompt.md`
  - 集成测试设计提示词
  - 多个Task的协作测试
  - 如何定义集成test case

- [ ] 3.1.3 创建 `plugins/sdd-dev-plugin/skills/code-test/e2e-test-prompt.md`
  - E2E测试设计提示词
  - 完整业务流程测试
  - 如何定义E2E test case

- [ ] 3.1.4 创建 `plugins/sdd-dev-plugin/skills/code-test/performance-test-prompt.md`
  - 性能测试提示词
  - 性能指标定义
  - 基准和阈值

- [ ] 3.1.5 更新 `plugins/sdd-dev-plugin/commands/sdd-dev.md`
  - 更新test阶段说明
  - 强调集成、E2E、性能的重点

**新流程**：
```
Step 1: 验证单元测试通过 ✅
  npm test --unit → 100% pass

Step 2: 设计集成测试
  多个Task的协作验证
  生成integration tests

Step 3: 设计E2E测试
  完整业务流程
  生成e2e tests

Step 4: 执行性能测试
  性能基准测试
  生成performance tests

Step 5: 覆盖率验证
  检查总覆盖率 ≥ 80%
  闭环验证矩阵
```

---

### 任务3.2：更新code-reviewer Agent

**目标**：调整审查策略，与新的职责对齐

**工作清单**：
- [ ] 3.2.1 修改 `plugins/sdd-dev-plugin/agents/code-reviewer.md`
  - 更新职责说明
  - 不再审查单元测试（由execute阶段完成）
  - 重点审查集成、E2E、性能测试
  - 闭环验证的新定义

---

### 任务3.3：更新说明文档

**工作清单**：
- [ ] 3.3.1 修改 `plugins/sdd-dev-plugin/README.md`
  - 更新工作流程说明
  - 添加阶段2的test-design
  - 更新code-test的说明

- [ ] 3.3.2 修改 `plugins/sdd-dev-plugin/skills/README.md`
  - 添加test-design到技能表
  - 更新整体工作流
  - 更新阶段说明

- [ ] 3.3.3 创建 `docs/TDD_PHASE3_EXAMPLE.md`
  - 完整的code-test示例
  - 集成测试示例
  - E2E测试示例
  - 闭环验证示例

---

### 任务3.4：整体文档更新

**工作清单**：
- [ ] 3.4.1 修改 `docs/BEST_PRACTICES.md`
  - 添加TDD最佳实践
  - 红-绿-重构的最佳实践
  - 测试分层的最佳实践

- [ ] 3.4.2 创建 `docs/TDD_COMPLETE_WORKFLOW.md`
  - 完整的TDD+SDD工作流说明
  - 从需求到代码再到测试
  - 完整示例

**输出**：
- 修改后的code-test SKILL
- 更新的code-reviewer Agent
- 更新的文档
- 示例和最佳实践

---

## 阶段4：培训和优化（第7周）

### 任务4.1：团队培训

**工作清单**：
- [ ] 4.1.1 创建 `docs/TDD_TRAINING_GUIDE.md`
  - 面向不同角色的培训指南
  - Spec作者如何定义TEST-VERIFY
  - Task定义者如何映射test case
  - Test设计者如何使用test-design
  - 开发者如何进行TDD实现
  - QA如何进行集成和E2E测试

- [ ] 4.1.2 准备培训材料
  - 幻灯片/演示文稿
  - 视频演示（可选）
  - 常见问题FAQ

---

### 任务4.2：试点项目

**工作清单**：
- [ ] 4.2.1 选择试点项目
  - 小规模项目（1个需求）
  - 完整流程体验

- [ ] 4.2.2 执行试点
  - 走通所有阶段
  - 记录问题和反馈

- [ ] 4.2.3 反馈收集和优化
  - 修复发现的问题
  - 优化工作流

---

### 任务4.3：文档和工具优化

**工作清单**：
- [ ] 4.3.1 创建快速参考指南
  - 各个阶段的快速检查清单
  - 常用命令

- [ ] 4.3.2 优化提示词和模板
  - 基于试点反馈优化
  - 增加示例和说明

---

## 任务统计

| 阶段 | 任务数 | 预计工作量 | 关键任务 |
|------|--------|----------|--------|
| 1 | 3 | 1周 | - |
| 2 | 5 | 2-3周 | 2.1, 2.3 |
| 3 | 4 | 1-2周 | 3.1 |
| 4 | 3 | 1周 | - |
| **总计** | **15** | **5-7周** | - |

---

## 关键路径

```
阶段1 → 阶段2 (关键：2.1 + 2.3)
   ↓
   → 验证2.1和2.3的可用性
   ↓
阶段3 (取决于阶段2)
   ↓
   → 验证整体流程
   ↓
阶段4 (可选但推荐)
```

---

## 资源需求

| 角色 | 需求 | 时间 |
|------|------|------|
| Skill开发者 | 2人 | 全程 |
| Agent设计者 | 1人 | 2-3周 |
| 文档编写 | 1人 | 全程 |
| 测试/QA | 1人 | 后期 |

---

## 风险和缓解

| 风险 | 概率 | 影响 | 缓解 |
|------|------|------|------|
| 阶段2 TDD流程复杂 | 中 | 高 | 充分的文档和示例 |
| code-executor需要大改 | 中 | 中 | 分阶段改进，保持兼容 |
| 团队学习曲线陡 | 高 | 中 | 充分的培训和试点 |
| 向后兼容性问题 | 低 | 高 | 谨慎的改动，充分验证 |

---

## 成功标准

✅ 完成所有15个任务
✅ 通过试点项目验证
✅ 团队培训完成
✅ 文档完整详细
✅ 0个breaking changes
✅ 现有项目仍可用

---

## 下一步

1. **确认任务计划** - 获得团队同意
2. **分配资源** - 安排人员和时间
3. **开始阶段1** - 立即执行轻量级改进
4. **每周同步** - 追踪进度，解决问题

