# TEST-VERIFY 模板指南

用于在规范阶段定义可测试的验收标准。

---

## 模板1：基础TEST-VERIFY格式

```markdown
# 场景: [场景名称]

## 场景描述
[场景的业务描述]

## 前置条件
- [条件1]
- [条件2]

## Case 1: [case名称]

**WHEN** (触发条件)
- 条件1：[具体条件]
- 条件2：[具体条件]

**THEN** (预期结果)
- 结果1：[具体的预期结果]
- 结果2：[具体的预期结果]

### TEST-VERIFY (可测试的验收标准)
- [ ] 应该[动作]，[期望结果]
- [ ] 应该验证[字段/属性]为[值]
- [ ] 应该[行为]不应该[禁止行为]
- [ ] 应该返回状态码[code]
- [ ] 应该在[时间约束]内完成

### Mock Data (测试数据和边界值)

**有效输入示例**：
```json
{
  "field1": "valid_value",
  "field2": 100,
  "field3": "2024-03-20"
}
```

**边界值**：
- `field2` 最小值：0（应接受）
- `field2` 最大值：10000（应接受）
- `field2` 边界外：-1（应拒绝）、10001（应拒绝）

**特殊值**：
- `field1` 为空字符串：应拒绝
- `field1` 包含特殊字符：应处理
- `field3` 日期格式不对：应验证失败

## Case 2: [异常情况案例]

[按同样格式重复]
```

---

## 模板2：前端表单场景

```markdown
# 场景: [表单功能名称]

## 场景描述
用户通过[表单]完成[操作]

## Case 1: 表单字段验证

**WHEN**
- 用户在[字段名]中输入[值]
- 用户点击[动作]

**THEN**
- 显示验证提示：[提示内容]
- [字段]变为[状态]

### TEST-VERIFY
- [ ] 表单包含所有必填字段
- [ ] 每个字段都有对应的验证规则
- [ ] 输入有效值时验证通过
- [ ] 输入无效值时显示错误提示
- [ ] 错误提示内容正确

### Mock Data

**有效数据**：
```json
{
  "username": "user@example.com",
  "password": "SecurePass123!",
  "age": 25
}
```

**无效数据**：
- username: "" (空值)
- username: "invalid-email" (格式错误)
- password: "123" (过短)
- age: -5 (无效范围)
- age: 200 (超出范围)

## Case 2: 表单提交

**WHEN**
- 表单所有字段都已正确填写
- 用户点击提交按钮

**THEN**
- 显示加载状态
- 发送请求到[API端点]
- 提交成功后显示成功提示

### TEST-VERIFY
- [ ] 点击提交后按钮变为禁用状态
- [ ] 显示加载动画/加载提示
- [ ] 成功后跳转到[页面]
- [ ] 成功提示自动关闭或有关闭按钮

### Mock Data

**提交数据**：
```json
{
  "username": "user@example.com",
  "password": "SecurePass123!",
  "age": 25
}
```

**期望响应**：
```json
{
  "status": 201,
  "message": "User created successfully",
  "data": {
    "id": "uuid-12345",
    "username": "user@example.com",
    "createdAt": "2024-03-20T10:00:00Z"
  }
}
```

## Case 3: 错误处理

**WHEN**
- 提交请求时服务器返回500错误

**THEN**
- 显示错误提示：服务器出错
- 提交按钮恢复可用状态
- 用户可以重新提交

### TEST-VERIFY
- [ ] API错误时显示用户友好的错误提示
- [ ] 不显示技术细节信息
- [ ] 提交按钮恢复可点击状态
- [ ] 用户输入数据被保留

### Mock Data

**错误响应**：
```json
{
  "status": 500,
  "message": "Internal Server Error"
}
```
```

---

## 模板3：API场景

```markdown
# 场景: [API功能名称]

## 场景描述
调用[API端点]完成[操作]

## Case 1: 成功请求

**WHEN**
- 发送[HTTP方法]请求到[端点]
- 请求体包含有效数据

**THEN**
- 返回200 OK状态码
- 返回预期的数据结构

### TEST-VERIFY
- [ ] HTTP状态码为200
- [ ] 响应包含所有必要字段
- [ ] 返回数据类型正确
- [ ] 数据库中创建了新记录
- [ ] 响应时间 < 500ms

### Mock Data

**请求**：
```json
POST /api/users
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "age": 30
}
```

**响应**：
```json
{
  "status": 200,
  "data": {
    "id": "user-123",
    "username": "john_doe",
    "email": "john@example.com",
    "age": 30,
    "createdAt": "2024-03-20T10:00:00Z"
  }
}
```

## Case 2: 验证失败

**WHEN**
- 请求数据缺少必填字段
- 或字段值不符合验证规则

**THEN**
- 返回400 Bad Request
- 返回详细的验证错误信息

### TEST-VERIFY
- [ ] HTTP状态码为400
- [ ] 错误响应包含具体的字段错误
- [ ] 错误消息易于理解

### Mock Data

**无效请求**（缺少email）：
```json
{
  "username": "john_doe",
  "age": 30
}
```

**错误响应**：
```json
{
  "status": 400,
  "errors": [
    {
      "field": "email",
      "message": "Email is required"
    }
  ]
}
```

## Case 3: 未授权请求

**WHEN**
- 请求不包含有效的身份验证令牌
- 或令牌已过期

**THEN**
- 返回401 Unauthorized
- 不处理请求

### TEST-VERIFY
- [ ] HTTP状态码为401
- [ ] 不创建/修改任何数据
- [ ] 返回清晰的错误消息

### Mock Data

**缺少认证的请求**：
```json
GET /api/protected-resource
(没有Authorization header)
```

**响应**：
```json
{
  "status": 401,
  "message": "Unauthorized: Missing or invalid authentication token"
}
```
```

---

## 模板4：数据库/业务逻辑场景

```markdown
# 场景: [业务操作名称]

## 场景描述
执行[业务操作]时的数据和状态变化

## Case 1: 正常操作

**WHEN**
- [触发条件1]
- [触发条件2]

**THEN**
- [数据库操作1]
- [状态变化1]

### TEST-VERIFY
- [ ] 数据库中创建/更新了正确的记录
- [ ] 所有相关的字段都被正确设置
- [ ] 没有创建多余的记录
- [ ] 相关的关联数据被正确更新
- [ ] 数据库约束没有被违反

### Mock Data

**操作前的数据状态**：
```json
{
  "orders": [
    { "id": "order-1", "userId": "user-1", "status": "draft" }
  ],
  "users": [
    { "id": "user-1", "balance": 1000 }
  ]
}
```

**操作内容**：
```
提交订单（order-1）
```

**操作后的期望数据状态**：
```json
{
  "orders": [
    { "id": "order-1", "userId": "user-1", "status": "submitted", "submittedAt": "2024-03-20T10:00:00Z" }
  ],
  "users": [
    { "id": "user-1", "balance": 900 }
  ],
  "audit_logs": [
    { "orderId": "order-1", "action": "submit", "timestamp": "2024-03-20T10:00:00Z" }
  ]
}
```

## Case 2: 边界情况

**WHEN**
- [边界条件1]

**THEN**
- [期望的边界处理]

### TEST-VERIFY
- [ ] 边界值被正确处理
- [ ] 没有数据精度丢失
- [ ] 没有溢出或下溢

### Mock Data

**边界值**：
- 金额：0（应接受）、-1（应拒绝）、99999999.99（应接受）
```

---

## 最佳实践

1. **具体化TEST-VERIFY**
   - 使用明确的测试语言（"应该..."）
   - 避免模糊的表述

2. **完整的Mock数据**
   - 提供有效的示例数据
   - 提供边界值
   - 提供错误数据

3. **可测试性**
   - TEST-VERIFY应该能自动化验证
   - 避免"通常"、"大约"这样的词

4. **覆盖完整**
   - 包括正常流程
   - 包括异常情况
   - 包括边界值

5. **与设计层次对应**
   - 前端：UI交互、表单验证
   - API：请求/响应、状态码
   - 数据库：数据一致性、约束

---

