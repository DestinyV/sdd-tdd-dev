# spec-creation README

快速导航和常用操作指南。

---

## 📌 快速开始（2分钟）

### 这个 Skill 做什么？

通过交互式问卷，收集需求信息，生成 BDD 格式的规范文档。

**核心流程**：
```
收集需求 → 交互问卷 → 规范生成 → BDD格式 → 规范目录
```

### 基本输出

生成 `spec-dev/{requirement_desc_abstract}/spec/` 目录，包含：
- `requirement.md` - 规范总览
- `scenarios/` - BDD 格式场景（WHEN-THEN）
- `data-models.md` - 数据模型定义
- `business-rules.md` - 业务规则
- `glossary.md` - 术语表
- `test-verify.md` - TEST-VERIFY 验收标准

---

## 📚 文档导航

### 核心概念（5分钟）
👉 **从这里开始**：阅读 [SKILL.md](./SKILL.md)
- Skill的职责和工作流程
- 何时使用
- 关键约束

### TEST-VERIFY 模板
👉 **了解验收标准格式**：查看 [references/test-verify-template.md](./references/test-verify-template.md)

---

## 🔑 关键概念

### 1. 规范的五个部分

| 部分 | 内容 | 作用 |
|------|------|------|
| **requirement.md** | 需求总览、目标、范围 | 快速理解需求 |
| **scenarios/** | BDD格式场景（WHEN-THEN） | 清晰的功能场景 |
| **data-models.md** | 数据模型、字段、关系 | 数据结构定义 |
| **business-rules.md** | 业务规则和约束 | 规则明确化 |
| **glossary.md** | 术语定义 | 消除歧义 |
| **test-verify.md** | 验收标准（TEST-VERIFY） | 定义成功标准 |

### 2. BDD 场景格式

```markdown
## 场景：用户成功登录

### WHEN（给定）
用户在登录页面
AND 输入有效的邮箱和密码
AND 邮箱已验证
AND 账户未被锁定

### THEN（那么）
系统应该返回登录token
AND 显示"登录成功"提示
AND 自动跳转到首页
AND 设置session有效期为24小时
```

### 3. TEST-VERIFY 验收标准

```markdown
## TV-Auth-1: 用户能成功登录

WHEN 用户输入有效的邮箱和密码
THEN 系统应该
  1. 返回有效的access_token
  2. 返回有效的refresh_token
  3. 设置session有效期为24小时

Mock数据：
- 有效邮箱：user@example.com
- 有效密码：ValidPassword123
- 预期token格式：JWT

边界值测试：
- 邮箱格式验证
- 密码长度验证（最小8字符）
- Session超时处理
```

### 4. 数据模型定义

```markdown
## User 模型

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| id | String | ✅ | 用户ID（UUID） |
| email | String | ✅ | 邮箱（唯一） |
| password_hash | String | ✅ | 密码哈希 |
| created_at | DateTime | ✅ | 创建时间 |
| updated_at | DateTime | ✅ | 更新时间 |
| is_active | Boolean | ✅ | 是否激活 |

关系：
- User 1:N Session
- User 1:N LoginAttempt
```

### 5. 业务规则示例

```markdown
## 规则：邮箱唯一性

类型：约束
应用场景：用户注册、修改邮箱
说明：每个邮箱只能关联一个用户账户
实现：
- 数据库unique约束
- 注册时检查邮箱是否已存在

## 规则：登录失败锁定

类型：安全规则
应用场景：用户登录
说明：连续登录失败5次后锁定账户2小时
实现：
- 记录每次失败的登录尝试
- 5次失败后设置is_locked=true
- 2小时后自动解锁
```

---

## ⚠️ 常见陷阱

### 陷阱1: 场景不够BDD

```markdown
# ❌ 不够BDD
场景：用户登录
说明：用户输入邮箱和密码，点击登录按钮，系统处理登录请求...

# ✅ 标准BDD格式
## 场景：用户成功登录

WHEN 用户在登录页面输入有效的邮箱和密码
AND 点击登录按钮

THEN 系统应该返回access_token和refresh_token
AND 显示"登录成功"提示
AND 自动跳转到首页
```

### 陷阱2: TEST-VERIFY不够具体

```markdown
# ❌ 太模糊
TV-Auth-1: 用户能登录
WHEN 用户输入邮箱和密码
THEN 系统处理登录

# ✅ 具体且可测试
TV-Auth-1: 用户能成功登录并获取token

WHEN 用户输入有效邮箱：user@example.com
AND 输入有效密码：ValidPass123
AND 邮箱已验证

THEN 系统应该返回：
  - status_code: 200
  - access_token: (JWT格式)
  - token_expires_in: 3600秒
  - user: { id, email, name }
```

### 陷阱3: 遗漏错误场景

```markdown
# ❌ 不完整（只有成功场景）
场景1：用户成功登录
场景2：用户成功登出

# ✅ 完整（包含错误场景）
场景1：用户成功登录
场景2：用户输入错误的密码
场景3：用户邮箱未验证
场景4：用户账户被锁定
场景5：用户成功登出
```

---

## 📋 工作流检查清单

在生成规范时：

```
□ 步骤1: 收集需求信息
  □ 需求名称和描述
  □ 业务背景
  □ 关键功能
  □ 非功能需求（性能、安全等）

□ 步骤2: 交互问卷
  □ 确认功能清单
  □ 确认业务场景
  □ 确认数据模型
  □ 确认边界条件

□ 步骤3: 生成BDD场景
  □ 每个主要功能至少1个成功场景
  □ 每个功能至少1-2个错误场景
  □ 使用标准WHEN-THEN格式
  □ 明确所有假设条件

□ 步骤4: 定义数据模型
  □ 列出所有核心数据对象
  □ 定义字段和类型
  □ 标记关键字段（ID、唯一键等）
  □ 清晰的字段说明

□ 步骤5: 列出业务规则
  □ 验证规则（邮箱格式、密码强度等）
  □ 唯一性规则
  □ 业务约束
  □ 安全规则

□ 步骤6: 定义术语
  □ 所有专业术语都定义了吗？
  □ 术语定义清晰且一致吗？
  □ 避免歧义

□ 步骤7: 提取TEST-VERIFY
  □ 每个场景都有验收标准
  □ 验收标准可测试
  □ Mock数据明确
  □ 边界值定义清楚
```

---

## 💡 最佳实践

### 1. 场景应该真实可用

```markdown
# ❌ 太抽象
场景：处理订单
WHEN 用户进行某个操作
THEN 系统做某些处理

# ✅ 真实具体
场景：用户创建订单
WHEN 用户在订单页面选择商品"iPhone 14"
AND 输入数量"2"
AND 选择地址"北京市朝阳区"
AND 点击"创建订单"按钮
THEN 系统应该
  1. 验证商品是否存在
  2. 验证库存是否充足
  3. 创建订单记录
  4. 返回订单ID
  5. 发送确认邮件
```

### 2. TEST-VERIFY应该是可自动化测试的

```markdown
# ❌ 无法测试
TV-Order-1: 创建订单功能好用

# ✅ 可自动化测试
TV-Order-1: 系统能创建订单并返回正确的订单信息

WHEN 调用POST /orders
WITH productId="PROD-001" quantity=2 address="北京"
THEN
  - 返回状态码200
  - 响应包含orderId（非空）
  - 响应的status="pending"
  - 响应包含createdAt（当前时间）
```

### 3. Mock数据应该具体

```markdown
# ❌ 不够具体
Mock数据：用户信息

# ✅ 具体可用
Mock数据：
- 有效邮箱：john.doe@example.com
- 有效密码：SecurePass123!
- 无效邮箱：invalid-email-format
- 非存在邮箱：notexist@example.com
- 短密码：Pass12（<8字符）

Mock数据文件：
- fixtures/user-valid.json
- fixtures/user-invalid.json
- fixtures/user-boundary.json
```

### 4. 场景覆盖要完整

```markdown
# 覆盖清单

## 成功场景（Happy Path）
- [ ] 用户成功登录
- [ ] 用户成功注册

## 错误场景（Error Cases）
- [ ] 邮箱不存在
- [ ] 密码错误
- [ ] 邮箱未验证
- [ ] 账户被锁定

## 边界场景（Boundary Cases）
- [ ] 邮箱为空
- [ ] 密码为空
- [ ] 密码过长
- [ ] 并发登录
```

---

## 🔗 相关Skills

- → **code-designer** 接收规范进行设计
- → **code-task** 接收规范中的TEST-VERIFY进行任务映射
- → **test-design** 接收TEST-VERIFY进行测试设计

---

## 📖 推荐阅读顺序

**快速上手（15分钟）**：
1. SKILL.md（职责和流程）
2. SKILL.md 中的"工作流程"部分

**深入学习（20分钟）**：
1. SKILL.md 的完整内容
2. references/test-verify-template.md（验收标准模板）

**需要时查阅**：
- SKILL.md（遗忘时查询）
- references/test-verify-template.md（验收标准模板）

---

## ❓ 常见问题

**Q: 场景应该有多详细？**

A: 足以清晰地定义业务流程，但不需要编写具体的UI步骤。通常可以描述到"用户进行某操作"的层级。

**Q: TEST-VERIFY和场景有什么区别？**

A:
- **场景**：描述一个完整的业务流程（从用户角度）
- **TEST-VERIFY**：定义验收标准（从测试角度）

一个场景可能对应多个TEST-VERIFY。

**Q: 如果规范还不完整怎么办？**

A: 没关系！规范可以迭代：
1. 先生成初版规范（覆盖主要功能）
2. 后续需求补充时（新增场景、规则等）再更新规范
3. 使用 spec-archive 来管理规范库的演进

**Q: Mock数据应该放在哪里？**

A: 创建 fixtures/ 目录：
```
spec-dev/project-name/fixtures/
├── user-valid.json
├── user-invalid.json
├── order-valid.json
└── ...
```

然后在TEST-VERIFY中引用这些文件路径。

---

**更新时间**：2026-03-26
**对应版本**：sdd-tdd-dev v2.1.0
