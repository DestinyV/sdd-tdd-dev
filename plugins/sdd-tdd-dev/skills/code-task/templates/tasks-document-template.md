# 任务列表文档模板

```markdown
# 任务列表

## 执行模式
- 模式：[standard | quick]（继承自 requirement.md）
- TDD要求：[必须执行 | 可跳过]
- 子代理要求：[必须使用 | 可选]
- 测试覆盖要求：[≥85% | ≥60% | 跳过]

## 任务总览
- 总 Task 数：[N]
- 并行批次：[N]
- 关键路径：[T1→T4→T6]（[X]h）
- 基础设施任务：[T-DB, T-CONF, T-MON]（如适用）

## 快速导航
- [Batch 1](#batch-1--t1-t2-t3)
- [Batch 2](#batch-2--t4-t5)
- [Batch 3](#batch-3--t6)

## 并行执行计划
- Batch 1（0h）：T1, T2, T3 → 3 parallelism
- Batch 2（4h）：T4, T5 → 2 parallelism
- Batch 3（8h）：T6 → 1 parallelism

## Test Case Mapping
| Task ID | Task 名称 | TEST-VERIFY | Test Case ID | Browser Test ID | Mock Data |
|---------|---------|-------------|-------------|----------------|-----------|
| T1 | [名称] | TV-1.1 | TC-1.1.1 | BT-1.1.1 | mock_xxx.json |

## TDD 追踪表（code-execute 阶段填写）⭐

| Task | RED | GREEN | REFACTOR | REVIEW | 覆盖率 | 状态 |
|------|-----|-------|----------|--------|--------|------|
| T1 | ⏳ | ⏳ | ⏳ | ⏳ | - | 待开始 |
| T2 | ⏳ | ⏳ | ⏳ | ⏳ | - | 待开始 |

> 此表由 code-execute 阶段填写，code-test 阶段验收。

## Task 详情

### Task 1: [名称]
[使用 task-template.md 定义]
```

详见：
- [单个Task定义模板](./task-template.md)
- [前端基础设施任务指南](./frontend-tasks-guide.md)（前端/全栈场景）
