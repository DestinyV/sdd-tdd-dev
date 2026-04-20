# code-task 详细工作流

本文档包含 code-task SKILL.md 中省略的详细工作流步骤、示例和最佳实践。

## 完整工作流步骤

### 步骤1: 分析设计方案

从 `design.md` 提取以下关键信息：

```
# [功能名称] 设计方案

## 2. 设计方案
### 2.1 整体架构
[架构图]

### 2.2 核心模块设计
#### 主组件：[组件1]
- Props：[接口定义]
- 关键特性：[列表]

#### 主组件：[组件2]
...

### 2.3 状态管理
- 本地状态：[说明]
- 共享状态：[说明]

### 2.4 数据流
[数据流程图]

### 2.5 样式方案
[样式框架和响应式设计]

### 2.6 关键交互
[交互说明]

## 4. 技术方案
### 4.1 依赖库
[库列表和版本]

### 4.2 API 设计
[API 端点]
```

**提取要点**：
- 有多少个主要组件/模块？
- 各组件之间有什么依赖关系？
- 需要什么新的库和工具？
- 有什么技术风险或复杂点？

### 步骤2: 分解为编码任务

#### 2.1 按功能模块分解

**策略**：从上到下，大到小

```
整体功能
├── 模块A（前端）
│   ├── 子模块A1（组件）
│   │   ├── Task A1.1：组件实现
│   │   └── Task A1.2：样式
│   └── 子模块A2（API集成）
│       └── Task A2.1：API集成
├── 模块B（后端）
│   ├── Task B1：API端点
│   └── Task B2：业务逻辑
└── 模块C（数据库）
    └── Task C1：表结构和迁移
```

#### 2.2 评估 Task 粒度

**检查清单**：

| 检查项 | 标准 | 调整方向 |
|--------|------|---------|
| 预计耗时 | 2-4 小时最佳 | 过大（>8h）→ 拆分；过小（<1h）→ 合并 |
| 依赖清晰 | 只依赖前序 Task | 复杂依赖 → 拆分或调整顺序 |
| 交付物明确 | 能列出具体文件 | 模糊 → 细化定义 |
| 测试可验证 | 有明确的验收标准 | 无法验证 → 重新定义 |
| 技能要求 | 单一技能领域 | 跨域（UI+API+DB）→ 拆分 |

#### 2.3 建立 Task 树

使用 ASCII 图表展示任务依赖：

```
T1: 项目初始化
  ├── T2: UserAuth
  │   └── T4: API 集成（Auth）
  │       └── T6: 集成测试
  ├── T3: FormValidation
  │   └── T5: API 集成（Form）
  │       └── T6: 集成测试
  └── T7: 数据库设计
      └── T8: 数据迁移
```

### 步骤3: 定义任务详情

#### 3.1 Task 模板

```markdown
## Task [N]: [任务名称]

**优先级**: [高/中/低]
**预计耗时**: [Xh]
**并行性**: [独立/依赖T1...]

### 目标
[这个任务实现什么功能？用户价值是什么？]

### 交付物
- [ ] src/components/UserAuth.tsx (250 行)
- [ ] src/hooks/useAuth.ts (100 行)
- [ ] src/styles/auth.module.css (50 行)
- [ ] src/__tests__/UserAuth.test.ts (200 行)
- [ ] 单元测试覆盖率 ≥ 85%

### 依赖
- [如果无依赖，写 "无"]
- 依赖 T1 完成（项目初始化）
- 依赖 T2 完成（认证 Hook）

### 详细说明

#### Props 接口
```typescript
interface UserAuthProps {
  onSuccess?: (user: User) => void;
  onError?: (error: Error) => void;
  redirectTo?: string;
}
```

#### 关键方法
- `handleLogin(email, password)` - 登录处理
- `handleLogout()` - 登出处理
- `validateEmail(email)` - 邮箱验证

#### 实现步骤
1. 创建组件骨架和 Props 接口
2. 实现登录表单 UI
3. 集成认证 API
4. 处理错误情况和边界
5. 编写单元测试
6. 样式调整和响应式设计

#### 关键点
- 使用 React Hooks（useState, useContext）
- 密码不应该记录在日志中
- 需要处理网络超时
- 需要处理已过期的 token

#### TODO/未明确的部分
- [ ] 确认密码重置流程是否需要在此 Task 中实现
- [ ] 确认是否需要支持社交登录（Google、GitHub）

### 验收标准
- [ ] 能正确处理有效的邮箱和密码
- [ ] 能正确拒绝无效的邮箱格式
- [ ] 能处理网络错误并显示错误消息
- [ ] 能正确保存 token 到 localStorage
- [ ] 能正确重定向到目标页面
- [ ] 单元测试覆盖率 ≥ 85%

### 技术栈
- React 18+（Hooks）
- TypeScript（strict 模式）
- Jest + React Testing Library（单元测试）
- CSS Modules（样式）
- Axios（HTTP）

### 估时
- 实现：2h
- 测试：1h
- 调试和优化：0.5h
- **总计：3.5h**
```

#### 3.2 高质量 Task 定义的检查清单

```
□ 目标明确且可达到
□ 交付物清单完整（代码文件、测试、文档等）
□ 依赖关系准确
□ 验收标准具体且可测试
□ 没有模糊的"改进"或"优化"
□ 估时在 2-4 小时范围内
□ 技术栈明确
□ 没有未澄清的 TODO 项
```

### 步骤4: 关联测试验收标准

#### 4.1 理解 TEST-VERIFY

从 spec 的场景中提取 TEST-VERIFY：

```markdown
# 场景: UserAuth

## Case 1: 正常登录流程
**WHEN** 用户输入正确的邮箱和密码点击登录
**THEN**
```
- 登录成功，返回 { status: 'success', token: '...' }
- 保存 token 到 localStorage
- 重定向到首页
```

**TEST-VERIFY**:
- TV-Auth-1: 验证登录成功时返回正确的 token
- TV-Auth-2: 验证 token 正确保存到 localStorage
- TV-Auth-3: 验证重定向到正确的页面
```

#### 4.2 关联 Task 到 TEST-VERIFY

```markdown
## Test Case Mapping（测试用例映射）

| Task ID | Task 名称 | TEST-VERIFY | Test Case ID | Mock Data 位置 |
|---------|----------|------------|-------------|-------------|
| T1 | UserAuth 组件 | TV-Auth-1, TV-Auth-2, TV-Auth-3 | TC-1.1, TC-1.2, TC-1.3 | spec/data-models.md#user-001 |
| T2 | 邮箱验证 Hook | TV-Auth-4 | TC-2.1 | spec/data-models.md#invalid-email |
| T3 | API 集成 | TV-Auth-1, TV-Auth-2 | TC-3.1, TC-3.2 | spec/data-models.md#api-response |
```

**映射说明**：
- 每个 Task 都应该有对应的 TEST-VERIFY
- 一个 Task 可以验证多个 TEST-VERIFY
- 一个 TEST-VERIFY 可以跨越多个 Task（需要标记为跨 Task）
- Mock Data 指向 spec 中定义的测试数据

#### 4.3 提取 Mock Data

从 spec 的 scenarios 中提取 Mock 数据：

```markdown
# 规范中的 Mock 数据

**文件位置**: spec-dev/[req_id]/spec/scenarios/user-auth.md

## Case 1: 正常登录

**数据**:
```json
{
  "input": {
    "email": "user@example.com",
    "password": "securePassword123"
  },
  "expected": {
    "status": "success",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid-1",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```
```

### 步骤5: 规划并行执行

#### 5.1 分析依赖关系

```
Task ID | Task 名称 | 依赖 | 优先级
--------|---------|------|------
T1 | 项目初始化 | 无 | 1
T2 | UserAuth | T1 | 1
T3 | FormValidation | T1 | 1
T4 | AuthAPI 集成 | T2 | 2
T5 | FormAPI 集成 | T3 | 2
T6 | 数据库设计 | 无 | 1
T7 | 数据迁移 | T6 | 2
T8 | 集成测试 | T4, T5, T7 | 3
```

#### 5.2 确定关键路径

**关键路径算法**：找最长的依赖链

```
T1 (1h) → T2 (3h) → T4 (2h) → T8 (2h)  = 8h（关键路径）
T1 (1h) → T3 (3h) → T5 (2h) → T8 (2h)  = 8h（关键路径）
T1 (1h) → T6 (2h) → T7 (1h) → T8 (2h)  = 6h
```

关键路径：**8h**（T1→T2→T4→T8 或 T1→T3→T5→T8）

#### 5.3 分批并行执行

```
Batch 1 (开始)：
├── T1 (1h)  - 项目初始化
├── T6 (2h)  - 数据库设计
│   这两个可以并行，因为没有依赖关系

Batch 2 (T1, T6 完成后)：
├── T2 (3h)  - UserAuth
├── T3 (3h)  - FormValidation
└── T7 (1h)  - 数据迁移（依赖 T6）
   这三个可以并行

Batch 3 (T2, T3, T7 完成后)：
├── T4 (2h)  - AuthAPI 集成
└── T5 (2h)  - FormAPI 集成
   这两个可以并行

Batch 4 (T4, T5 完成后)：
└── T8 (2h)  - 集成测试

总耗时：1h + 3h + 2h + 2h = 8h（关键路径）
不是 1h + 3h + 3h + 1h + 2h + 2h + 2h = 14h（顺序执行）
节省了 6 小时！
```

### 步骤6: 生成任务列表文档

生成完整的 `tasks.md` 文件：

```markdown
# 任务列表

**项目**: [项目名]
**对应设计**: [design/design.md](../design/design.md)
**生成日期**: 2026-03-26

---

## 📊 任务总览

- **总 Task 数**: 8
- **并行批次**: 4
- **关键路径**: T1 → T2 → T4 → T8（8h）
- **预计总耗时**: 8h（并行执行）vs 14h（顺序执行）
- **优化节省**: 43%

---

## 📋 快速导航

1. [任务并行执行计划](#任务并行执行计划)
2. [Test Case Mapping](#test-case-mapping映射表)
3. [任务详情](#任务详情)
4. [质量标准](#质量标准)

---

## 任务并行执行计划

### Batch 1（并行性：2）
启动时间：0h

**T1: 项目初始化**
- 耗时：1h
- 并行性：独立
- 依赖：无

**T6: 数据库设计**
- 耗时：2h
- 并行性：独立
- 依赖：无

### Batch 2（并行性：3）
启动时间：2h（等待 T6 和 T1 都完成）

**T2: UserAuth**
**T3: FormValidation**
**T7: 数据迁移**

### Batch 3（并行性：2）
...

---

## Test Case Mapping 映射表

[完整的映射表]

---

## 任务详情

[所有 Task 的完整定义]

---

## 质量标准

[代码质量、功能测试、设计一致性要求]
```

---

## 任务定义示例

### 示例1：前端组件 Task

```markdown
## Task 1: 实现 UserAuth 组件

**优先级**: 高（关键路径）
**预计耗时**: 3h
**并行性**: 依赖 T1（项目初始化）

### 目标
实现用户认证表单组件，支持邮箱/密码登录、表单验证、错误处理。

### 交付物
- [ ] src/components/UserAuth.tsx (组件实现)
- [ ] src/hooks/useAuth.ts (认证逻辑提取)
- [ ] src/types/auth.ts (类型定义)
- [ ] src/__tests__/UserAuth.test.tsx (单元测试)
- [ ] 单元测试覆盖率 ≥ 85%
- [ ] 代码审查通过

### 依赖
- T1（项目初始化）

### 验收标准
- [ ] 能正确验证邮箱格式
- [ ] 能正确验证密码强度
- [ ] 能处理登录成功（保存 token，重定向）
- [ ] 能处理登录失败（显示错误消息）
- [ ] 能处理网络超时
- [ ] 支持回车键登录
- [ ] 响应式设计（mobile 兼容）
- [ ] 单元测试覆盖率 ≥ 85%

### 技术栈
- React 18+（Hooks）
- TypeScript（strict）
- CSS Modules
- Axios（HTTP）
- Jest + React Testing Library

### 估时
- 实现：2h
- 测试：0.5h
- 调试：0.5h
- **总计：3h**
```

### 示例2：后端 API Task

```markdown
## Task 4: 实现用户认证 API

**优先级**: 高（关键路径）
**预计耗时**: 2h
**并行性**: 依赖 T1（项目初始化）

### 目标
实现用户登录和登出 API 端点，包括请求验证、数据库查询、token 生成。

### 交付物
- [ ] src/routes/auth.ts (路由定义)
- [ ] src/controllers/authController.ts (业务逻辑)
- [ ] src/services/authService.ts (数据库操作)
- [ ] src/__tests__/auth.test.ts (单元测试)
- [ ] 单元测试覆盖率 ≥ 85%

### API 端点
- POST /api/auth/login (邮箱、密码)
- POST /api/auth/logout (无参数)
- POST /api/auth/refresh (refresh token)

### 验收标准
- [ ] 登录端点能返回有效的 JWT token
- [ ] 能正确验证用户凭证
- [ ] 能处理用户不存在的情况
- [ ] 能处理密码错误的情况
- [ ] 能记录登录日志（不含敏感信息）
- [ ] 单元测试覆盖率 ≥ 85%

### 技术栈
- Node.js/Express
- TypeScript（strict）
- PostgreSQL
- JWT（认证）
- Jest（测试）

### 估时
- 实现：1.5h
- 测试：0.5h
- **总计：2h**
```

### 示例3：数据库 Task

```markdown
## Task 6: 数据库设计和迁移

**优先级**: 高（关键路径）
**预计耗时**: 2h
**并行性**: 独立

### 目标
设计用户、订单、产品相关的数据库表结构，创建迁移脚本。

### 交付物
- [ ] migrations/001_create_users_table.sql
- [ ] migrations/002_create_orders_table.sql
- [ ] migrations/003_create_products_table.sql
- [ ] schemas/schema.sql (完整 schema)
- [ ] docs/database-design.md (设计文档)

### 表结构
- users (id, email, password_hash, name, created_at)
- orders (id, user_id, total, status, created_at)
- order_items (id, order_id, product_id, quantity, price)
- products (id, name, price, description, stock)

### 验收标准
- [ ] 所有表都有正确的主键和外键
- [ ] 所有必要的索引都已创建
- [ ] 能成功运行迁移脚本
- [ ] 数据完整性约束正确
- [ ] 设计文档完整

### 技术栈
- PostgreSQL 14+
- Flyway 或 Knex（迁移工具）

### 估时
- 设计：1h
- 迁移脚本：0.5h
- 文档：0.5h
- **总计：2h**
```

---

## 最佳实践

### 1. Task 粒度控制

**过大的 Task（>8h）**：
```
❌ "实现整个订单系统"
✅ "实现订单列表展示"、"实现订单详情页"、"实现订单状态更新"
```

**过小的 Task（<1h）**：
```
❌ "添加一个导入语句"
✅ "实现 OrderList 组件（包含列表展示、排序、筛选、分页）"
```

### 2. 依赖关系管理

**清晰的依赖**：
```
T1 → T2 → T3（线性，容易并行）
```

**复杂依赖**：
```
T1 → T2 → T4
T1 → T3 → T4
（钻石依赖，需要特别注意）
```

**环形依赖**：
```
T1 → T2 → T3 → T1（❌ 错误，需要重新设计）
```

### 3. Test Case Mapping 完整性

**100% 覆盖**：
- 每个 TEST-VERIFY 都有对应的 Test Case
- 每个 Task 都能验证至少一个 TEST-VERIFY
- Mock Data 来源清晰

**不完整的 Mapping**：
```
❌ TV-1 在 spec 中定义，但没有对应的 Test Case
❌ Task T1 没有关联任何 TEST-VERIFY
❌ Mock Data 位置不清
```

### 4. 估时准确性

**估时技巧**：
- 参考历史 Task 的实际耗时
- 考虑技术风险和复杂度
- 为不确定性预留 20% 的 buffer
- 将大 Task 拆分多个小 Task 分别估时

**常见错误**：
```
❌ "3-5 小时"（模糊）
✅ "3 小时"（具体）

❌ 估时过乐观（忽视复杂度）
✅ 基于经验和 buffer（留有余地）
```

---

**关键理念**：清晰、完整、可并行的任务定义，是高效开发的基础。
