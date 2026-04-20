# 代码实现质量约束 - 禁止伪代码实现

**版本**：v1.0（2026-04-08）

---

## 概述

本文档规定了在 code-execute 阶段代码实现时的关键质量约束：**禁止使用 console/print 等简单输出伪装成真正的逻辑实现**。

**核心原则**：
- ✅ **完整实现** - 所有逻辑必须有真正的实现代码
- ✅ **完全功能** - 代码必须能在生产环境真正运行和工作
- ✅ **数据处理** - 数据必须真正被处理和操作，而不仅仅被打印
- ✅ **状态管理** - 状态必须真正被管理和维护，不能只是输出状态
- ❌ **禁止伪代码** - 不能用console.log、print、echo等简单输出代替真实实现

---

## 问题场景

### ❌ 错误示例 1：使用 console.log 代替数据处理逻辑

```javascript
// ❌ 错误：伪代码实现（只是输出，没有真实逻辑）
function calculateDiscount(price, discountRate) {
  const discount = price * discountRate;
  console.log(`折扣: ${discount}`);  // ← 只是输出，不是实现
  console.log(`最终价格: ${price - discount}`);  // ← 伪代码
  // 函数没有返回值，无法被使用
}

// ✅ 正确：真实实现
function calculateDiscount(price, discountRate) {
  const discount = price * discountRate;
  const finalPrice = price - discount;
  return {
    originalPrice: price,
    discount: discount,
    finalPrice: finalPrice,
    discountPercent: discountRate * 100
  };
}

// 测试
const result = calculateDiscount(100, 0.2);
expect(result.finalPrice).toBe(80);  // ✅ 可以真正验证
```

### ❌ 错误示例 2：使用 print 代替状态管理

```python
# ❌ 错误：伪代码实现（只是输出状态，不是管理状态）
class UserService:
    def __init__(self):
        self.users = []

    def add_user(self, user):
        print(f"添加用户: {user}")  # ← 只是打印，没有真实添加
        print(f"当前用户数: {len(self.users) + 1}")  # ← 伪代码
        # 用户并没有真正被添加到列表中

# ✅ 正确：真实实现
class UserService:
    def __init__(self):
        self.users = []

    def add_user(self, user):
        if not user.get('email'):
            raise ValueError("用户email不能为空")
        self.users.append(user)
        return {
            "success": True,
            "user": user,
            "totalUsers": len(self.users)
        }

# 测试
service = UserService()
result = service.add_user({"email": "test@example.com", "name": "Test"})
assert len(service.users) == 1  # ✅ 可以真正验证状态
assert service.users[0]['email'] == "test@example.com"
```

### ❌ 错误示例 3：使用 echo 代替 API 实现

```php
// ❌ 错误：伪代码实现（只是输出，没有真实API逻辑）
function fetchUser($id) {
    echo "获取用户ID: " . $id;  // ← 只是输出，不是实现
    echo "用户名: John";  // ← 伪代码
    echo "邮箱: john@example.com";  // ← 伪代码
    // 没有真正的数据库查询或API调用
}

// ✅ 正确：真实实现
function fetchUser($id) {
    if (!is_numeric($id) || $id <= 0) {
        return [
            "success" => false,
            "error" => "Invalid user ID"
        ];
    }

    // 真实的数据库查询
    $user = database->query("SELECT * FROM users WHERE id = ?", [$id]);

    if (!$user) {
        return [
            "success" => false,
            "error" => "User not found",
            "statusCode" => 404
        ];
    }

    return [
        "success" => true,
        "data" => $user,
        "statusCode" => 200
    ];
}

// 测试
$result = fetchUser(1);
assert($result['success'] === true);  // ✅ 可以真正验证
assert($result['data']['id'] === 1);
```

### ❌ 错误示例 4：使用 console 代替 React 组件逻辑

```javascript
// ❌ 错误：伪代码实现（只是打印，没有真实的组件逻辑和状态管理）
function UserForm() {
  return (
    <form>
      <input
        onChange={(e) => console.log("输入值:", e.target.value)}  // ← 伪代码
      />
      <button onClick={() => console.log("提交表单")}>提交</button>  // ← 伪代码
      {/* 没有真正的状态管理、验证、提交逻辑 */}
    </form>
  );
}

// ✅ 正确：真实实现
function UserForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    errors: {}
  });

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
      errors: { ...prev.errors, [name]: '' }  // ← 真实的状态管理
    }));
  };

  const validateForm = () => {
    const errors = {};
    if (!formData.name.trim()) {
      errors.name = '名字不能为空';
    }
    if (!formData.email.includes('@')) {
      errors.email = '邮箱格式不正确';
    }
    return Object.keys(errors).length === 0 ? {} : errors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const errors = validateForm();
    if (Object.keys(errors).length > 0) {
      setFormData(prev => ({ ...prev, errors }));
      return;
    }

    // 真实的API调用
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });

    if (response.ok) {
      // 真实的成功处理
      setFormData({ name: '', email: '', errors: {} });
      // 可能需要重定向或刷新列表
    } else {
      // 真实的错误处理
      const error = await response.json();
      setFormData(prev => ({
        ...prev,
        errors: { submit: error.message }
      }));
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="name"
          value={formData.name}
          onChange={handleInputChange}
          placeholder="输入名字"
        />
        {formData.errors.name && <span className="error">{formData.errors.name}</span>}
      </div>

      <div>
        <input
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          placeholder="输入邮箱"
        />
        {formData.errors.email && <span className="error">{formData.errors.email}</span>}
      </div>

      <button type="submit">提交</button>
      {formData.errors.submit && <p className="error">{formData.errors.submit}</p>}
    </form>
  );
}
```

---

## 检查清单

### 代码实现时的质量检查

在写任何代码前，确保你理解：

**✅ 必须包含**：
- [ ] 真实的数据处理逻辑（不仅仅是输出）
- [ ] 真实的状态管理（不仅仅是显示状态）
- [ ] 真实的错误处理（try-catch、异常处理）
- [ ] 真实的边界情况处理（null、undefined、空值等）
- [ ] 真实的返回值或副作用（不仅仅是console.log）
- [ ] 真实的验证逻辑（输入检查、业务规则检查）
- [ ] 可被单元测试验证的实现（不是伪代码）

**❌ 禁止包含**：
- [ ] ❌ `console.log()` 作为主要逻辑
- [ ] ❌ `print()` 作为主要逻辑
- [ ] ❌ `echo` 作为主要逻辑
- [ ] ❌ `alert()` 作为主要逻辑
- [ ] ❌ 任何简单输出语句代替真实实现
- [ ] ❌ 空函数体（仅注释）
- [ ] ❌ 只有TODO的函数
- [ ] ❌ 伪装成实现的框架代码

### 质量审查时的检查

规范审查 + 质量审查 必须检查：

```javascript
// 检查清单示例（用于代码审查）

// 1. 是否有真实的业务逻辑？
❌ function process(data) {
  console.log("处理数据");  // ← 不是真实逻辑
}

✅ function process(data) {
  if (!Array.isArray(data)) {
    throw new Error("数据必须是数组");
  }
  return data
    .filter(item => item.active)
    .map(item => ({
      ...item,
      processedAt: new Date()
    }))
    .sort((a, b) => a.priority - b.priority);
}

// 2. 是否可以被单元测试真正验证？
❌ function validate(email) {
  console.log("验证邮箱");  // ← 测试无法验证
}

✅ function validate(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// 3. 是否有边界情况处理？
❌ function calculate(a, b) {
  console.log(a + b);  // ← 没有处理边界情况
}

✅ function calculate(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError("参数必须是数字");
  }
  if (!isFinite(a) || !isFinite(b)) {
    throw new RangeError("参数必须是有限数字");
  }
  return a + b;
}

// 4. 是否有真实的状态管理？
❌ function UserManager() {
  return {
    add: (user) => console.log("添加用户:", user)
  };
}

✅ function UserManager() {
  const users = [];

  return {
    add: (user) => {
      if (!user.email) throw new Error("Email required");
      users.push(user);
      return { success: true, id: users.length };
    },
    get: (id) => users[id - 1],
    getAll: () => [...users],
    delete: (id) => {
      users.splice(id - 1, 1);
      return true;
    }
  };
}
```

---

## 规范审查中的检查点

规范审查子代理 **必须** 检查：

```markdown
## 代码实现真实性检查

### 检查项：代码逻辑是否真实实现

- [ ] **函数/方法是否有真实的处理逻辑**？
  - ❌ 不能只有 console.log/print/echo
  - ✅ 必须有实际的数据处理、计算、条件判断

- [ ] **返回值是否是处理的结果**？
  - ❌ 不能返回 undefined（没有处理）
  - ❌ 不能返回 "success" 字符串（伪代码）
  - ✅ 必须返回实际处理的数据

- [ ] **状态管理是否真实**？
  - ❌ 不能只是 console.log 当前状态
  - ✅ 必须真正修改和维护状态

- [ ] **错误处理是否真实**？
  - ❌ 不能只是 console.error
  - ✅ 必须真正处理异常情况

- [ ] **代码是否可以被测试验证**？
  - ❌ 如果测试无法验证结果，说明是伪代码
  - ✅ 测试必须能真正验证逻辑的正确性

### 问题示例和修复

**问题**: 伪代码实现
```javascript
function processOrder(order) {
  console.log("订单处理中...");
  console.log("计算总价:", order.items.length);
  console.log("订单已完成");
  // 测试无法验证这个函数做了什么
}
```

**修复**: 真实实现
```javascript
function processOrder(order) {
  if (!order || !Array.isArray(order.items)) {
    throw new Error("Invalid order");
  }

  const subtotal = order.items.reduce((sum, item) => {
    return sum + (item.price * item.quantity);
  }, 0);

  const tax = subtotal * 0.1;
  const total = subtotal + tax;

  return {
    orderId: generateOrderId(),
    items: order.items,
    subtotal: subtotal,
    tax: tax,
    total: total,
    status: 'completed',
    timestamp: new Date()
  };
}
```

### 审查决定

如果发现伪代码实现：
- 状态：❌ **不通过**
- 原因：代码实现不完整，使用了伪代码
- 要求：修改代码，使用真实实现
- 重新审查：修复后必须重新提交规范审查
```

---

## 质量审查中的检查点

质量审查子代理 **可以** 通过检查单元测试来验证：

```markdown
## 代码实现真实性的单元测试验证

### 验证方法

通过单元测试验证代码的真实性：

**❌ 如果是伪代码**，测试无法真正验证：
```javascript
// 伪代码实现
function getValue() {
  console.log("获取值");
}

// 测试无法验证
test('getValue should return value', () => {
  const result = getValue();
  // getValue() 返回 undefined，无法验证逻辑
  // 这个测试就是虚假测试
});
```

**✅ 如果是真实实现**，测试可以真正验证：
```javascript
// 真实实现
function getValue() {
  return 42;
}

// 测试可以真正验证
test('getValue should return 42', () => {
  const result = getValue();
  expect(result).toBe(42);  // ✅ 真正验证了逻辑
});
```

### 单元测试的真实性标志

如果代码的单元测试具有这些特征，说明代码是真实实现：

- [ ] **测试能验证返回值** - 不是虚假的 expect
- [ ] **测试能验证状态变化** - 能检查内部状态是否真正改变
- [ ] **测试能验证错误处理** - 能检查异常是否真正被抛出
- [ ] **测试能验证边界情况** - 能检查null、undefined、空值等
- [ ] **多个测试用例相互独立** - 说明逻辑是可重复验证的
```

---

## 实施方法

### 1. 在代码提交前（实现子代理检查）

```
实现代码时，开发者需要确保：
✅ 每个函数都有真实的业务逻辑实现
✅ 每个函数都能返回可用的结果
✅ 每个函数都有错误处理
✅ 每个函数都有边界情况处理
✅ 写出的单元测试能真正验证逻辑
```

### 2. 在规范审查时（规范审查子代理检查）

```
规范审查必须检查代码逻辑是否真实：
❌ 发现console/print等伪代码 → 退回修改
✅ 代码有真实处理逻辑 → 通过审查
```

### 3. 在质量审查时（质量审查子代理检查）

```
质量审查可以通过单元测试来验证：
❌ 发现单元测试无法真正验证逻辑 → 退回修改
✅ 单元测试能真正验证逻辑 → 通过审查
```

---

## 常见问题

### Q1: 为什么禁止使用 console.log？

**A**: 因为：
1. **不是实现** - console.log 只是输出，不是逻辑实现
2. **无法测试** - 单元测试无法验证 console 的输出
3. **无法使用** - 调用者无法获取函数的结果
4. **生产不可用** - 依赖 console 的代码在生产环境中无法工作

### Q2: 什么时候可以使用 console/print？

**A**: 只在以下场景可以使用：
1. **调试阶段** - 开发者自己在开发时用来调试
2. **日志记录** - 真实实现中，为了记录日志而使用
3. **非关键逻辑** - 为了辅助，但不是核心业务逻辑

**但绝不能**：
- ❌ 用 console.log 代替函数的返回值
- ❌ 用 console.log 代替状态管理
- ❌ 用 console.log 代替错误处理
- ❌ 让单元测试依赖于 console 输出

### Q3: 伪代码实现会怎样？

**A**: 会在规范审查中被拒绝：

```
发现问题：函数 processUser 使用 console.log 代替真实实现
↓
规范审查：❌ 不通过
↓
反馈：需要使用真实实现，不能使用伪代码
↓
开发者修改：添加真实的数据处理逻辑
↓
重新提交：修改后重新审查
↓
规范审查：✅ 通过（有真实实现了）
```

### Q4: 验证码/演示代码可以使用伪代码吗？

**A**: 不可以。即使是演示或验证，也必须是真实可运行的代码。

如果需要演示，应该：
1. ✅ 编写真实的实现代码
2. ✅ 编写演示用的真实数据
3. ✅ 编写展示结果的真实逻辑
4. ❌ 不能用伪代码代替

---

## 总结

**核心约束**：
```
所有代码实现必须是真实的、完整的、可运行的、可测试的
不能使用 console/print/echo 等简单输出伪装成逻辑实现
```

**检查流程**：
```
实现 → 自审查（检查是否真实实现）
    → 规范审查（检查代码逻辑是否真实）
    → 质量审查（通过单元测试验证真实性）
    → 通过
```

**关键指标**：
```
✅ 代码能真正解决问题
✅ 单元测试能真正验证逻辑
✅ 代码在生产环境可真正运行
❌ 不能用伪代码代替实现
```

---

**最后更新**：2026-04-08
**版本**：v1.0（初版）
