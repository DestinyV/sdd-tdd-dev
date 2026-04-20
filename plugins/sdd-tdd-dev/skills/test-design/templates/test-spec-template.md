# test-spec.md 模板指南

用于生成完整的测试规范文档。

---

## 模板1：基础测试规范模板

```markdown
# 测试规范

**项目**：[项目名称]
**版本**：1.0
**日期**：[生成日期]

---

## 文档导航

- [项目信息](#项目信息)
- [测试框架](#测试框架)
- [测试分层](#测试分层)
- [Test Case总览](#test-case总览)
- [Mock和Fixture](#mock和fixture)
- [执行指南](#执行指南)

---

## 项目信息

| 项 | 值 |
|----|-----|
| 项目名 | [name] |
| 对应规范 | [spec/requirement.md](../spec/requirement.md) |
| 对应设计 | [design/design.md](../design/design.md) |
| 对应任务 | [tasks/tasks.md](../tasks/tasks.md) |
| 创建日期 | [date] |
| 测试作者 | test-designer |

---

## 测试框架

### 前端测试栈

| 工具 | 用途 | 版本 |
|------|------|------|
| Jest | 单元和集成测试 | 29.0+ |
| React Testing Library | 组件测试 | 13.0+ |
| msw | API Mock | 1.0+ |
| @testing-library/user-event | 用户交互模拟 | 14.0+ |

### 后端测试栈（如果适用）

| 工具 | 用途 | 版本 |
|------|------|------|
| Jest | 单元测试 | 29.0+ |
| Supertest | API测试 | 6.0+ |
| 内存数据库 | 数据库测试 | - |

### 配置文件

\`\`\`javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
\`\`\`

---

## 测试分层

```
         E2E Tests (5-10%)
       /      \
      /        \  完整业务场景、用户流程
     /          \
    /────────────\
   /              \  Integration Tests (15-25%)
  /                \ 多模块协作、API调用、数据持久化
 /──────────────────\
/                    \  Unit Tests (65-75%)
                      \ 单个函数/组件、边界值、错误处理
```

### 单元测试（Unit Tests）

**目标**：
- 测试单个函数、组件、方法
- 验证输入输出的对应关系
- 测试边界值和特殊值

**范围**：
- 业务逻辑函数
- React组件（Props、State、Events）
- Hooks
- 工具函数
- 数据转换

**Mock策略**：
- Mock所有外部依赖（API、数据库、服务）
- Mock复杂的第三方库
- 隔离单个单元

**覆盖率目标**：≥ 85%

---

### 集成测试（Integration Tests）

**目标**：
- 测试多个模块的协作
- 验证数据流和状态管理
- 测试API调用和数据库操作

**范围**：
- 组件间的交互
- 父组件与子组件的协作
- 状态管理的流程
- API调用和响应处理
- 数据库操作

**Mock策略**：
- Mock外部API（使用msw）
- 使用真实或内存数据库
- 不Mock应用内部的模块

**覆盖率目标**：≥ 70%

---

### E2E测试（E2E Tests）

**目标**：
- 测试完整的用户场景
- 验证从UI到数据库的完整链路
- 测试真实的用户交互

**范围**：
- 关键业务流程
- 用户从入口到完成的全过程
- 跨页面的流程

**工具**：
- Cypress 或 Playwright

**覆盖率目标**：关键场景 100%

---

## Test Case总览

### 统计信息

| 指标 | 数值 |
|------|------|
| 总Test Case数 | [N] |
| 单元测试 | [N] |
| 集成测试 | [N] |
| E2E测试 | [N] |
| TEST-VERIFY覆盖率 | 100% |

### Test Case分布

| Task | TEST-VERIFY | TC数 | 类型 | 说明 |
|------|-------------|------|------|------|
| Task 1: [name] | 5 | 8 | Unit + Integration | [说明] |
| Task 2: [name] | 4 | 6 | Unit | [说明] |
| Task 3: [name] | 3 | 5 | Unit + Integration | [说明] |
| 集成测试 | 2 | 3 | Integration | 多Task协作 |
| E2E测试 | 1 | 2 | E2E | 完整业务流 |
| **总计** | **15** | **24** | **Mixed** | **100%** |

---

### 完整的Test Case表格

| TC-ID | Task | TEST-VERIFY | 实现文件 | 测试类型 | 优先级 | 说明 |
|-------|------|-------------|--------|--------|--------|------|
| TC-1.1.1 | Task 1 | Case1#1 | src/components/Form.tsx | Unit | P0 | 验证表单字段 |
| TC-1.1.2 | Task 1 | Case1#2 | src/hooks/useForm.ts | Unit | P0 | 验证输入验证 |
| TC-1.2.1 | Task 1 | Case1#边界值1 | src/hooks/useForm.ts | Unit | P1 | 边界值：最小值 |
| TC-1.3.1 | Task 1 | Case2#1 | src/components/Form.tsx | Integration | P0 | 表单提交 |
| TC-2.1.1 | Task 2 | Case1#1 | src/api/userApi.ts | Unit | P0 | API调用 |
| ... | ... | ... | ... | ... | ... | ... |

---

## Mock和Fixture

### fixtures.json 结构

\`\`\`json
{
  "metadata": {
    "project": "[project-name]",
    "createdAt": "[date]",
    "version": "1.0",
    "description": "测试数据和Mock定义"
  },

  "fixtures": {
    "Task1": {
      "validInputs": [
        {
          "name": "valid_form_input",
          "data": {
            "username": "john_doe",
            "email": "john@example.com",
            "password": "SecurePass123!"
          },
          "expectedOutput": {
            "status": "success",
            "message": "验证通过"
          }
        }
      ],

      "boundaryValues": [
        {
          "name": "username_min_length",
          "data": {"username": "a", "email": "valid@example.com", "password": "SecurePass123!"},
          "expectedOutput": "should accept"
        },
        {
          "name": "username_max_length",
          "data": {"username": "a".repeat(20), "email": "valid@example.com", "password": "SecurePass123!"},
          "expectedOutput": "should accept"
        }
      ],

      "specialValues": [
        {
          "name": "username_empty",
          "data": {"username": "", "email": "valid@example.com", "password": "SecurePass123!"},
          "expectedOutput": "should reject"
        },
        {
          "name": "email_special_chars",
          "data": {"username": "john", "email": "john+test@example.com", "password": "SecurePass123!"},
          "expectedOutput": "should accept"
        }
      ]
    }
  },

  "mocks": {
    "API": {
      "baseURL": "http://api.example.com",
      "endpoints": {
        "POST /users": {
          "success": {
            "status": 201,
            "data": {"id": "user-123", "username": "john_doe", "email": "john@example.com"}
          },
          "conflict": {
            "status": 409,
            "message": "Email already exists"
          },
          "serverError": {
            "status": 500,
            "message": "Internal Server Error"
          }
        }
      }
    },

    "Database": {
      "type": "sqlite",
      "connection": "sqlite::memory:",
      "seeds": [
        {
          "table": "users",
          "data": [
            {"id": 1, "username": "admin", "email": "admin@example.com"}
          ]
        }
      ]
    }
  }
}
\`\`\`

### Mock策略说明

1. **API Mock**
   - 使用 msw (Mock Service Worker)
   - 定义于 `fixtures.json` 的 `mocks.API`
   - 包括成功响应、错误响应、超时等

2. **数据库Mock**
   - 使用内存SQLite或H2
   - 在 beforeEach 中初始化
   - 在 afterEach 中清理

3. **时间Mock**
   - 使用 jest.useFakeTimers()
   - 用于测试超时、延迟等

4. **第三方服务Mock**
   - 邮件服务、支付服务等
   - 使用 jest.mock()

---

## 执行指南

### 运行所有测试

\`\`\`bash
npm test
\`\`\`

### 运行特定Test Case

\`\`\`bash
npm test -- --testNamePattern="TC-1.1.1"
\`\`\`

### 查看测试覆盖率

\`\`\`bash
npm test -- --coverage
\`\`\`

### Watch模式

\`\`\`bash
npm test -- --watch
\`\`\`

---

## 覆盖率分析

### TEST-VERIFY覆盖矩阵

| TEST-VERIFY | 来源 | TC-ID | 框架文件 | 状态 |
|-------------|------|-------|--------|------|
| 应该支持username输入 | Case1#1 | TC-1.1.1 | test-Task1.ts | ✅ |
| 应该验证username非空 | Case1#2 | TC-1.1.2 | test-Task1.ts | ✅ |
| ... | ... | ... | ... | ... |

**总计**：15个TEST-VERIFY，15个test case，覆盖率 **100%** ✅

### 代码覆盖率目标

| 指标 | 目标 | 当前 |
|------|------|------|
| 语句覆盖率 (Statements) | ≥ 80% | TBD |
| 分支覆盖率 (Branches) | ≥ 75% | TBD |
| 函数覆盖率 (Functions) | ≥ 80% | TBD |
| 行覆盖率 (Lines) | ≥ 80% | TBD |

---

## 测试执行检查清单

- [ ] 所有fixtures.json中的数据都被使用
- [ ] 所有Mock都被正确初始化
- [ ] 所有test都能独立运行
- [ ] 没有flaky test（不稳定的测试）
- [ ] 测试执行时间 < 5秒（单元测试）
- [ ] 代码覆盖率 ≥ 80%
- [ ] TEST-VERIFY覆盖率 = 100%
- [ ] 所有test都通过

---

## 常见问题

**Q: 如何运行单个test file？**

A: \`npm test -- test-Task1.test.ts\`

**Q: 如何调试failing test？**

A: \`node --inspect-brk node_modules/.bin/jest --runInBand\`

**Q: Mock不work怎么办？**

A: 检查Mock的path是否正确，确保Mock在import前定义

**Q: 如何跳过某个test？**

A: 使用 \`test.skip('description', () => {...})\`

---
```

---

## 模板2：前端项目测试规范示例

见 TDD_PHASE2_EXAMPLE.md（后续创建）

---

## 模板3：后端项目测试规范示例

见 TDD_PHASE2_EXAMPLE.md（后续创建）

---

## 最佳实践

### 1. 清晰的组织结构

```
tests/
├── test-spec.md           # 测试规范（本文件）
├── fixtures.json          # Mock数据
├── unit/
│   ├── test-Task1.test.ts
│   ├── test-Task2.test.ts
│   └── ...
├── integration/
│   ├── test-integration.test.ts
│   └── ...
└── e2e/
    ├── test-flow.cy.ts
    └── ...
```

### 2. 清晰的命名规范

- Test file：`test-[Task].test.ts`
- Test case ID：`TC-[TaskID].[CaseID].[Seq]`
- Describe块：Task或功能名称
- Test名称：清晰描述期望行为

### 3. Mock和Fixture的管理

- 所有Mock数据定义在 `fixtures.json`
- Test文件中import fixtures
- 避免hardcode测试数据
- 定期更新Mock数据以反映API变化

### 4. 完整的TEST-VERIFY映射

- 每个TEST-VERIFY都要有对应test case
- 使用TC-ID建立追踪关系
- 在test-spec.md中维护映射矩阵

---
