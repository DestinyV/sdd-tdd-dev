# Task 定义模板

## Task [ID]: [Task 名称]

**类型**：[组件实现/Hooks/工具/样式/API集成/构建配置/CDN部署/前端监控/数据库/中间件/日志/监控埋点/部署配置]

**目标**：[这个任务实现什么功能]

**接口标记** ⭐🆕（fullstack 模式必填）：
- provides: [接口列表]（后端 Task，如 `[GET /api/users, POST /api/users]`）
- consumes: [接口列表]（前端 Task，如 `[GET /api/users]`）

> 注：provides/consumes 必须与 api-contract.md 中定义的接口完全一致

**交付物**：
- [ ] `src/path/to/file.ts` - [文件说明]
- [ ] `src/path/to/file.css` - [文件说明]

**依赖**：
- 依赖 Task: [T1, T2, ...] 或 无

**验收标准**：
- [ ] [验收条件1]
- [ ] [验收条件2]

**测试用例**：
- TEST-VERIFY: [TV-X.X]
- Test Case ID: [TC-X.X.X]
- Mock Data: [mock_xxx.json]

**估时**：[X]h

**详细说明**（可选）：
[实现思路、注意事项]
