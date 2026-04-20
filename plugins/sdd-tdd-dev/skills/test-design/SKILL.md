---
name: test-design
description: |
  第2.5步：测试设计和规范生成（Test Design阶段）✨新增

  输入：spec-dev/{requirement_desc_abstract}/tasks/tasks.md + spec/
  输出：spec-dev/{requirement_desc_abstract}/tests/test-spec.md

  功能：将任务中的验收标准和规范中的TEST-VERIFY转换为详细的测试规范和测试用例。
  生成可直接执行的测试框架代码。支持多种测试框架（Jest、Vitest、Pytest等）。

  新增：TDD完整支持
  - 从TEST-VERIFY提取测试用例
  - 定义Mock数据和Fixture
  - 生成测试规范文档
  - 生成测试框架代码模板
  - 支持单元、集成、E2E等多层级测试

  输出内容：
  - test-spec.md (测试规范)
  - fixtures.json (测试数据和mock定义)
  - test-*.template (测试框架代码模板)

  支持的框架：
  - 前端：Jest、Vitest、Cypress、Playwright、Testing Library
  - 后端：Jest、Pytest、JUnit、Go testing
  - 端到端：Cypress、Playwright、Selenium

  前置Skill：code-task ✓ 和 spec-creation ✓（提供TEST-VERIFY）
  下一步：/code-execute（遵循TDD红-绿-重构循环）
---

# test-design

## 职责

根据规范中的TEST-VERIFY和任务中的test-case-mapping，设计和生成详细的测试规范：

1. 读取 `spec-dev/{requirement_desc_abstract}/spec/` 中的TEST-VERIFY
2. 读取 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md` 中的test-case-mapping
3. 分析每个Test Case的需求和依赖
4. 提取和生成对应的测试用例
5. 定义Mock数据、Fixture和测试框架
6. 生成测试规范文档和框架代码
7. 验证测试覆盖率 ≥ 100%（覆盖所有TEST-VERIFY）

**输出路径**：`spec-dev/{requirement_desc_abstract}/tests/test-spec.md`

**Agent 使用**：此阶段使用 **test-designer Agent** 进行专业的测试设计

---

## 工作流程

### 阶段1: 读取和分析需求

读取以下内容：

1. **规范中的TEST-VERIFY**
   - 每个Case的验收标准
   - Mock数据（有效值、边界值、特殊值）

2. **任务中的test-case-mapping**
   - 验收标准 → 实现点的映射
   - 测试ID (TC-N.N.N)
   - Mock数据的具体位置

3. **设计方案（design.md）**
   - 技术框架选型
   - 架构设计
   - 关键组件和接口

**活动**：
- 分配 **test-designer Agent** 进行分析
- test-designer 读取spec、tasks、design中的内容
- 理解每个Task的验收标准和测试需求
- 标识关键的Test Case和边界值

**输出**：
- 测试需求分析清单
- 每个Task的test case列表
- Mock数据和Fixture定义

---

### 阶段2: 生成测试规范

为每个Task生成详细的测试规范。

**test-spec.md 结构**：

```markdown
# 测试规范

## 项目信息
- 项目名：[name]
- 对应需求：[spec/requirement.md](../spec/requirement.md)
- 对应任务：[tasks/tasks.md](../tasks/tasks.md)

## 测试框架和工具
- 框架：[Jest/Vitest/Pytest等]
- 测试库：[Testing Library/Pytest等]
- Mock库：[Jest/Unittest.mock等]
- 覆盖率工具：[Istanbul/Coverage等]

## 测试分层

### 单元测试（Unit Tests）
- Task 1 单元测试
- Task 2 单元测试
- ...

### 集成测试（Integration Tests）
- 多Task协作测试
- API集成测试
- ...

### E2E测试（E2E Tests）
- 完整业务流程测试
- ...

## 完整的Test Case表格

| TC-ID | Task | 来源验收标准 | 测试类型 | 说明 |
|-------|------|-----------|--------|------|
| TC-1.1.1 | Task 1 | Case1#1 | Unit | [说明] |
| ... | ... | ... | ... | ... |

## Mock数据和Fixture定义

[详细的mock策略、fixture生成规则等]

## 覆盖率分析

- 单元测试覆盖率：X%
- 集成测试覆盖率：X%
- 总覆盖率：≥80%
- TEST-VERIFY覆盖：100%
```

**分层策略**：

1. **单元测试**
   - 测试单个函数、组件、方法
   - 使用Mock隔离外部依赖
   - 测试边界值、特殊值

2. **集成测试**
   - 测试多个组件的协作
   - 测试API调用和数据流
   - 测试状态管理

3. **E2E测试**
   - 测试完整的用户流程
   - 真实的用户交互场景
   - 从UI到数据库的完整链路

4. **浏览器测试（前端/全栈场景）** ⭐ 新增
   - **E2E 端到端测试**：完整用户操作路径（Playwright @e2e）
   - **视觉回归测试**：页面/组件截图对比（Playwright @visual）
   - **组件 UI 测试**：组件渲染和交互（Playwright @component / Vitest）
   - 覆盖 spec 中定义的 BROWSER-TESTABLE 验收标准
   - 使用 design.md 中定义的 data-testid 选择器

---

### 阶段3: 定义Mock和Fixture

生成详细的Mock数据定义。

**fixtures.json 结构**：

```json
{
  "metadata": {
    "project": "project-name",
    "createdAt": "2024-03-20",
    "version": "1.0"
  },
  "fixtures": {
    "Task1": {
      "validInputs": [
        {
          "name": "valid_case_1",
          "data": {...},
          "expectedResult": {...}
        }
      ],
      "boundaryValues": [
        {
          "name": "min_value",
          "data": {...},
          "expectedResult": "should accept"
        }
      ],
      "specialValues": [
        {
          "name": "empty_string",
          "data": {...},
          "expectedResult": "should reject"
        }
      ]
    }
  },
  "mocks": {
    "API": {
      "baseURL": "http://api.example.com",
      "endpoints": {
        "GET /users": {...},
        "POST /users": {...}
      }
    },
    "Database": {
      "connection": "sqlite::memory:",
      "seeds": [...]
    }
  }
}
```

**Mock策略定义**：

1. **API Mock**
   - Mock哪些API端点
   - 返回哪些数据
   - 模拟哪些错误场景

2. **数据库Mock/Stub**
   - 使用内存数据库（SQLite、H2）
   - 或使用Mock库（jest.mock等）
   - Seed数据定义

3. **第三方服务Mock**
   - 邮件服务、支付服务等
   - 使用Mock库隔离

---

### 阶段4: 生成测试框架代码

为每个Task生成可直接使用的测试框架。

**test-*.template 文件结构**：

```typescript
// test-Task1.template.ts

import { /* 需要的库 */ } from 'jest';
import { fixtures } from '../fixtures.json';

// Describe Suite
describe('Task 1: [任务名]', () => {

  // Setup
  beforeEach(() => {
    // Mock初始化
    // Fixture准备
  });

  // Unit Test Cases
  describe('验证功能', () => {
    test('TC-1.1.1: 应该...', () => {
      // Arrange
      const input = fixtures.Task1.validInputs[0];

      // Act
      const result = function_to_test(input);

      // Assert
      expect(result).toBe(expected);
    });

    // 更多test cases...
  });

  // Boundary Value Tests
  describe('边界值测试', () => {
    test('TC-1.2.1: 最小值应该接受', () => {
      // ...
    });
  });

  // Error Handling Tests
  describe('错误处理', () => {
    test('TC-1.3.1: 无效输入应该拒绝', () => {
      // ...
    });
  });

  // Cleanup
  afterEach(() => {
    // Mock清理
    // 数据清理
  });
});
```

**框架支持**：

| 框架 | 类型 | 文件模板 |
|------|------|--------|
| Jest | Node.js/Frontend | test-*.test.ts |
| Vitest | Frontend | test-*.test.ts |
| Pytest | Python | test-*.py |
| Go testing | Go | test-*_test.go |
| Cypress | E2E | test-*.cy.ts |

---

### 阶段5: 验证覆盖率

确保测试覆盖所有的TEST-VERIFY。

**覆盖率检查清单**：

- [ ] **完整性** - 所有TEST-VERIFY都有对应的test case
- [ ] **映射正确** - 每个test case都关联到具体的test-case-mapping
- [ ] **Mock清晰** - 每个test都有明确的Mock和Fixture定义
- [ ] **框架完整** - 所有test case都有框架代码
- [ ] **可执行性** - 框架代码可直接运行
- [ ] **测试分层** - Unit、Integration、E2E合理分布

**覆盖率矩阵**：

```markdown
| TEST-VERIFY | TC-ID | 框架代码 | 类型 | Mock | 状态 |
|-------------|-------|--------|------|------|------|
| 应该支持查询 | TC-1.1.1 | ✅ | Unit | API | ✅ |
| 应该验证参数 | TC-1.1.2 | ✅ | Unit | No | ✅ |
| ... | ... | ... | ... | ... | ... |
```

---

## 输出要求

### test-spec.md

**必须包含**：
- [x] 项目信息和链接
- [x] 测试框架/工具选型
- [x] 测试分层（Unit/Integration/E2E/Browser/Visual/Component）
- [x] 完整的Test Case表格
- [x] Mock和Fixture定义
- [x] 覆盖率分析
- [x] 与TEST-VERIFY的映射
- [x] **与BROWSER-TESTABLE的映射（前端/全栈场景）** ⭐

**质量标准**：
- 格式清晰，易于理解
- 所有test case都有唯一的TC-ID
- 所有Mock都有明确的定义
- 所有link都正确
- **浏览器测试用例明确标注 @e2e / @visual / @component** ⭐

### fixtures.json

**必须包含**：
- [x] 每个Task的有效输入
- [x] 每个Task的边界值
- [x] 每个Task的特殊值
- [x] API Mock定义
- [x] 数据库Mock/Stub定义

**质量标准**：
- JSON格式正确
- 数据完整和可用
- 有详细的说明注释

### test-*.template

**必须包含**：
- [x] 完整的imports和setup
- [x] Describe suite组织
- [x] 每个test case的框架
- [x] Mock初始化和清理
- [x] 注释说明

**质量标准**：
- 代码可直接运行（或复制后运行）
- 变量命名清晰
- 注释完整

---

## 关键原则

### 1. TEST-VERIFY优先

所有测试都必须来自规范中的TEST-VERIFY：
- ✅ 从TEST-VERIFY提取测试需求
- ✅ 使用规范中的Mock数据
- ✅ 保持与验收标准的一致性

### 2. 完整的映射链

确保完整的追踪链：
```
TEST-VERIFY (规范)
  ↓
test-case-mapping (任务)
  ↓
TC-ID (测试规范)
  ↓
test-*.template (框架代码)
```

### 3. 分层清晰

按照Unit → Integration → E2E的分层组织：
- Unit：隔离依赖，快速反馈
- Integration：验证协作，发现接口问题
- E2E：真实场景，保证用户体验

### 4. Mock策略明确

每个Mock都要明确：
- 为什么Mock（隔离外部依赖）
- Mock什么（API/数据库/服务）
- 如何Mock（Mock库、Stub等）

### 5. 覆盖率100%

对于TEST-VERIFY的覆盖率必须是100%：
- [ ] 所有TEST-VERIFY都有对应test case
- [ ] 所有test case都有框架代码
- [ ] 没有遗漏的边界值或特殊值

---

## 与其他Skill的关系

### 读取
- ← **spec-creation** 提供 TEST-VERIFY和Mock数据
- ← **code-task** 提供 test-case-mapping和Task定义

### 输出给
- → **code-execute** 消费测试规范和框架代码
  - code-execute遵循TDD流程
  - 使用这些test case进行开发

### 反馈
- 测试执行结果 → 可能需要优化测试规范
- 覆盖率不足 → 可能需要补充test case

---

## 成功标准

✅ 完成时应满足：

1. **test-spec.md**
   - [x] 包含所有Task的测试规范
   - [x] 所有TEST-VERIFY都有对应test case
   - [x] TEST-VERIFY覆盖率 = 100%
   - [x] 测试分层清晰（Unit/Integration/E2E/Browser）
   - [x] **（前端/全栈）浏览器测试用例覆盖 BROWSER-TESTABLE 标准** ⭐

2. **fixtures.json**
   - [x] 包含所有Mock定义
   - [x] 包含所有有效值、边界值、特殊值
   - [x] 格式正确可用

3. **test-*.template**
   - [x] 每个Task都有对应框架代码
   - [x] 框架代码覆盖所有test case
   - [x] 代码可直接复制使用
   - [x] **（前端/全栈）包含 Playwright E2E/Visual/Component 测试模板** ⭐

4. **质量指标**
   - [x] 无broken links
   - [x] 格式一致
   - [x] 注释完整
   - [x] 命名规范

---

## 相关资源

| 资源 | 说明 |
|------|------|
| `references/workflow-detail.md` | 📌 **新增：完整的测试设计工作流和示例** |

---

## 常见问题

**Q: 如果Task太复杂，test case太多怎么办？**

A: 拆分成多个describe suite，按功能分组。可以有数十个test case。

**Q: 如果某个TEST-VERIFY不好测试怎么办？**

A: 在test-spec.md中说明原因，建议修改TEST-VERIFY使其更易测试。

**Q: Mock应该有多详细？**

A: 应该足以让code-executor直接使用，包括边界值和特殊值。

**Q: 需要写集成测试吗？**

A: 如果多个Task有协作，应该有集成测试验证协作。

**Q: E2E测试需要测什么？**

A: 测试完整的用户场景和业务流程（如果适用）。

---
