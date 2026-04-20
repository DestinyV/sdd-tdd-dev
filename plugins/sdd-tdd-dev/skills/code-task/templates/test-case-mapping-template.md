# Test Case Mapping 模板指南

用于在任务阶段关联规范中的验收标准与任务中的实现及测试用例。

---

## 概述

Test Case Mapping 是TDD流程中的关键桥梁：

```
规范 (spec/)              任务 (task/)              测试 (test/)
    ↓                        ↓                        ↓
TEST-VERIFY        →   Task实现        →   测试用例
验收标准               关联映射             自动化验证
Mock数据             实现点位置            测试数据
```

---

## 模板1：基础Test Case Mapping表格

在每个Task下添加以下内容：

```markdown
## Task [N]: [任务名称]

### ... (目标、交付物等)

### Test Case Mapping

**关联规范**：`spec/scenarios/[scenario-name].md` - Case [X]

该Task实现的验收标准和测试用例映射：

| # | 验收标准 (TEST-VERIFY) | 来源位置 | 对应实现 | 测试用例ID | 说明 |
|----|----------------------|---------|--------|-----------|------|
| 1 | 应该支持[功能A] | Case1.TEST-VERIFY#1 | src/components/[File].tsx:method1() | TC-[N].1.1 | 基础功能验证 |
| 2 | 应该验证[字段B] | Case1.TEST-VERIFY#2 | src/components/[File].tsx:validate() | TC-[N].1.2 | 字段级验证 |
| 3 | 应该返回[结果C] | Case1.TEST-VERIFY#3 | src/api/[File].ts:fetchData() | TC-[N].1.3 | API响应验证 |
| 4 | 边界值：[X]最小值为0 | Case2.Mock Data | src/utils/[File].ts:validate() | TC-[N].2.1 | 边界值测试 |
| 5 | 特殊值：[Y]为空时 | Case2.Mock Data | src/components/[File].tsx:handleEmpty() | TC-[N].2.2 | 特殊情况处理 |

**说明**：
- 每一行对应一个验收标准或Mock数据边界值
- 确保规范中的所有验收标准都被映射
- TEST-VERIFY来自规范中的对应Case
- 测试用例ID为：TC-[TaskID].[CaseID].[序号]

### Mock Data 关联

该Task使用的测试数据直接来自规范：

**有效数据示例**（来自 `spec/scenarios/[scenario-name].md#Case1.Mock Data`）：
\`\`\`json
{
  "field1": "valid_value",
  "field2": 100,
  "field3": "2024-03-20"
}
\`\`\`

**边界值**（来自规范）：
- `field2` 最小值：0（应接受）→ TC-[N].2.1
- `field2` 最大值：10000（应接受）
- `field2` 边界外：-1（应拒绝）→ TC-[N].2.2

**特殊值**（来自规范）：
- `field1` 为空字符串：应拒绝 → TC-[N].2.3
- `field1` 包含特殊字符：应处理 → TC-[N].2.4
- `field3` 日期格式不对：应验证失败 → TC-[N].2.5
```

---

## 模板2：前端组件Task示例

```markdown
## Task 1: 实现 UserRegistrationForm 组件

### 目标
实现用户注册表单，支持表单验证、错误提示、提交。

### 交付物
- [ ] src/components/UserRegistrationForm.tsx
- [ ] src/hooks/useRegistrationForm.ts
- [ ] src/__tests__/UserRegistrationForm.test.tsx

### Test Case Mapping

**关联规范**：`spec/scenarios/user-registration.md`

| # | 验收标准 | 来源 | 对应实现 | 测试用例ID | 类型 |
|----|--------|------|--------|-----------|------|
| 1 | 表单包含username字段 | Case1#1 | UserRegistrationForm.tsx:render() | TC-1.1.1 | 单元测试 |
| 2 | 表单包含email字段 | Case1#2 | UserRegistrationForm.tsx:render() | TC-1.1.2 | 单元测试 |
| 3 | 表单包含password字段 | Case1#3 | UserRegistrationForm.tsx:render() | TC-1.1.3 | 单元测试 |
| 4 | username为空时验证失败 | Case1#4 | useRegistrationForm.ts:validate() | TC-1.1.4 | 单元测试 |
| 5 | email无效时验证失败 | Case1#5 | useRegistrationForm.ts:validate() | TC-1.1.5 | 单元测试 |
| 6 | password少于6字符时验证失败 | Case1#6 | useRegistrationForm.ts:validate() | TC-1.1.6 | 单元测试 |
| 7 | 所有字段有效时提交成功 | Case2#1 | UserRegistrationForm.tsx:onSubmit() | TC-1.2.1 | 集成测试 |
| 8 | 提交时显示加载状态 | Case2#2 | UserRegistrationForm.tsx:render() | TC-1.2.2 | 集成测试 |
| 9 | 成功后显示成功提示 | Case2#3 | UserRegistrationForm.tsx:handleSuccess() | TC-1.2.3 | 集成测试 |
| 10 | username为空字符串 | Case1.MockData#边界值1 | useRegistrationForm.ts:validate() | TC-1.3.1 | 单元测试 |
| 11 | username过长（100+字符） | Case1.MockData#边界值2 | useRegistrationForm.ts:validate() | TC-1.3.2 | 单元测试 |

### Mock Data 关联

**有效注册数据**（来自 `spec/scenarios/user-registration.md#Case1.Mock Data`）：
\`\`\`json
{
  "username": "user@example.com",
  "password": "SecurePass123!",
  "confirmPassword": "SecurePass123!"
}
\`\`\`

**边界值**：
| 字段 | 边界值 | 预期结果 | 测试ID |
|-----|-------|--------|--------|
| username | "" (空) | 拒绝 | TC-1.3.1 |
| username | 1-20字符 | 接受 | TC-1.3.2 |
| username | 100+字符 | 拒绝 | TC-1.3.3 |
| password | 少于6字符 | 拒绝 | TC-1.3.4 |
| password | 6-50字符 | 接受 | TC-1.3.5 |
| password | 50+字符 | 接受 | TC-1.3.6 |

**特殊值**：
- username包含特殊字符 `!@#$%` → 接受（邮箱格式允许）→ TC-1.3.7
- password包含空格 → 接受（密码允许空格）→ TC-1.3.8
- password与confirmPassword不匹配 → 拒绝 → TC-1.3.9
```

---

## 模板3：后端API Task示例

```markdown
## Task 2: 实现订单创建API

### 目标
实现POST /api/orders接口，支持订单创建、验证、持久化。

### 交付物
- [ ] src/controllers/orderController.ts
- [ ] src/services/orderService.ts
- [ ] src/__tests__/orderController.test.ts

### Test Case Mapping

**关联规范**：`spec/scenarios/order-creation.md`

| # | 验收标准 | 来源 | 对应实现 | 测试用例ID | 说明 |
|----|--------|------|--------|-----------|------|
| 1 | HTTP状态码为201 | Case1#1 | orderController.ts:create() | TC-2.1.1 | 成功响应 |
| 2 | 响应包含order_id | Case1#2 | orderService.ts:createOrder() | TC-2.1.2 | 返回ID验证 |
| 3 | 响应包含created_at | Case1#3 | orderService.ts:createOrder() | TC-2.1.3 | 时间戳验证 |
| 4 | 数据库中创建新记录 | Case1#4 | orderService.ts:createOrder() | TC-2.1.4 | 数据持久化 |
| 5 | 缺少userId时返回400 | Case2#1 | orderController.ts:validate() | TC-2.2.1 | 必填字段验证 |
| 6 | 缺少items时返回400 | Case2#2 | orderController.ts:validate() | TC-2.2.2 | 必填字段验证 |
| 7 | items为空数组时返回400 | Case2#3 | orderController.ts:validate() | TC-2.2.3 | 数组验证 |
| 8 | 无认证时返回401 | Case3#1 | orderController.ts:authenticate() | TC-2.3.1 | 认证检查 |
| 9 | 总金额为0 | Case2.MockData#1 | orderService.ts:calculateTotal() | TC-2.4.1 | 边界值 |
| 10 | 总金额为负数 | Case2.MockData#2 | orderService.ts:calculateTotal() | TC-2.4.2 | 边界值 |

### Mock Data 关联

**有效创建请求**（来自规范）：
\`\`\`json
POST /api/orders
{
  "userId": "user-123",
  "items": [
    {"productId": "prod-1", "quantity": 2, "price": 50.00},
    {"productId": "prod-2", "quantity": 1, "price": 100.00}
  ]
}
\`\`\`

**边界值**：
- `quantity` 最小值：1（应接受）→ TC-2.4.3
- `quantity` 最大值：1000（应接受）→ TC-2.4.4
- `price` 最小值：0.01（应接受）→ TC-2.4.5
- `price` 为0（应拒绝）→ TC-2.4.6
- `price` 为负数（应拒绝）→ TC-2.4.7
- `items` 数组最小长度：1（应接受）→ TC-2.4.8
- `items` 数组为空（应拒绝）→ TC-2.4.9

**错误请求示例**：
\`\`\`json
// 缺少userId
{
  "items": [{"productId": "prod-1", "quantity": 1, "price": 50.00}]
}

// 缺少items
{
  "userId": "user-123"
}

// items为空
{
  "userId": "user-123",
  "items": []
}
\`\`\`
```

---

## 最佳实践

### 1. 映射完整性检查

确保：
- ✅ 规范中的所有TEST-VERIFY都有对应的映射
- ✅ 规范中的所有Mock数据边界值都有对应的测试用例
- ✅ 每个映射都指向具体的实现代码位置
- ✅ 没有"待定"或"TBD"的映射

### 2. 测试用例ID命名规范

格式：`TC-[TaskID].[CaseID].[序号]`

示例：
- `TC-1.1.1` - Task 1, Case 1, 第1个测试
- `TC-2.3.5` - Task 2, Case 3, 第5个测试

### 3. 来源位置精确化

来源应指向规范中的具体位置：
- ✅ `spec/scenarios/user-registration.md#Case1.TEST-VERIFY#3` - 准确
- ✅ `spec/scenarios/user-registration.md#Case1.Mock Data#边界值2` - 准确
- ❌ `spec/scenarios/user-registration.md` - 太模糊
- ❌ `TEST-VERIFY` - 没有定位

### 4. 实现位置精确化

实现应指向具体的文件和方法：
- ✅ `src/components/UserForm.tsx:validateEmail()` - 准确
- ✅ `src/api/orderApi.ts:createOrder()` - 准确
- ❌ `src/` - 太模糊
- ❌ `implementation` - 未指定

### 5. 保持同步

当规范或设计变更时：
- [ ] 更新mapping表格中的来源位置
- [ ] 更新或删除不再有效的映射
- [ ] 添加新增的验收标准映射
- [ ] 更新相关的测试用例ID

---

## 与后续阶段的关系

### 在 test-design 阶段（TDD支持）

- 读取 task-case-mapping 表格
- 为每个测试用例ID生成对应的测试代码
- 输出：`spec-dev/{req_id}/tests/test-spec.md`

### 在 code-execute 阶段

- 使用 test-case-mapping 中的Mock数据进行开发
- 按映射实现对应的代码
- 运行对应的测试用例验证

### 在 code-test 阶段

- 验证所有映射中的测试用例都已通过
- 检查验收标准覆盖率 ≥ 100%
- 生成闭环验证矩阵（Task ↔ TEST-VERIFY ↔ 测试 ↔ 结果）

---

## FAQ

**Q: 如果一个TEST-VERIFY由多个Task实现怎么办？**

A: 在每个Task的映射中都列出该TEST-VERIFY，然后在说明列说明各自的责任范围。

**Q: 如果一个Test Case涉及多个规范场景怎么办？**

A: 在Task中列出所有相关的规范场景，使用"/"分隔：`spec/scenarios/scenario1.md#Case1 / scenario2.md#Case2`

**Q: TEST-VERIFY没有对应的Mock数据怎么办？**

A: 在Task的Mock Data部分补充生成测试数据，但必须基于TEST-VERIFY的意图，并注明"补充数据"。

**Q: 测试用例ID如何管理？**

A: 在Task中从`TC-[TaskID].1.1`开始递增，确保不重复。如果后续添加新测试，继续递增。

---
