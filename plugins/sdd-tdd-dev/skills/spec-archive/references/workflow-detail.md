# spec-archive 详细工作流

本文档包含 spec-archive SKILL.md 中省略的详细工作流步骤、示例和最佳实践。

---

## 完整工作流步骤

### 步骤1: 分析需求Spec和主规范

#### 1.1 读取需求Spec结构

```bash
# 检查需求spec目录结构
ls -la spec-dev/order-form/spec/

# 输出示例：
# README.md          (需求总览)
# scenarios/         (BDD场景)
# data-models.md    (数据模型)
# business-rules.md (业务规则)
# glossary.md       (术语表)
```

分析每个文件的内容：

**README.md（需求总览）**
- 需求名称：UserAuth（用户认证）
- 场景数量：12个
- 核心功能：用户登录、注册、修改密码、忘记密码

**scenarios/ 目录**
```
scenarios/
├── user-login.md              # 用户登录
├── user-signup.md             # 用户注册
├── user-forgot-password.md    # 忘记密码
├── user-password-change.md    # 修改密码
└── user-mfa.md               # 多因素认证
```

**data-models.md（数据模型）**
- User 模型：id, email, password_hash, created_at
- Session 模型：id, user_id, token, expires_at
- MFADevice 模型：id, user_id, secret, verified

**business-rules.md（业务规则）**
- 密码最小长度8个字符
- 邮箱必须唯一
- 登录失败5次后锁定账户
- Session有效期24小时

**glossary.md（术语表）**
- MFA：多因素认证
- Session：用户会话
- Token：访问令牌

#### 1.2 检查主规范是否存在

```bash
# 检查主规范目录
ls -la spec-dev/spec/

# 如果存在
echo "主规范已存在，进行步骤2（合并）"

# 如果不存在
echo "主规范不存在，进行步骤3（初始化）"
```

### 步骤2: 场景对标和差异分析

#### 2.1 场景对标矩阵

创建对标表格，逐个比对新场景和已有场景：

```markdown
## 场景对标分析

### 认证相关（Authentication）

| 新需求场景 | 主规范中的对应场景 | 分类 | 处理方式 |
|----------|-----------------|------|---------|
| user-login | User Login | 完全相同 | 去重：保留主规范版本 |
| user-signup | User Registration | 完全相同 | 去重：保留主规范版本 |
| user-logout | User Logout | 完全相同 | 去重：保留主规范版本 |
| user-forgot-password | Password Reset | 完全相同 | 去重：保留主规范版本 |
| user-password-change | Change Password | 相关但不同 | 合并：两个是不同的流程 |
| user-mfa | Multi-Factor Auth | 全新 | 新增：添加到主规范 |
| user-password-strength | - | 特殊情况 | 新增：作为validation规则 |

### 会话管理（Session Management）

| 新需求场景 | 主规范中的对应场景 | 分类 | 处理方式 |
|----------|-----------------|------|---------|
| user-session-timeout | Session Timeout | 相关但不同 | 合并：扩展现有规则 |
| user-concurrent-login | - | 全新 | 新增：处理并发登录 |

## 汇总

- 完全相同：4个（去重）
- 相关但不同：2个（合并）
- 全新场景：2个（新增）
- **总计：4个去重 + 2个合并 + 2个新增 = 8个最终场景**
```

#### 2.2 对标的三个结果

**结果1：完全相同的场景**
```
user-login.md 在新需求中描述为：
"用户输入邮箱和密码，系统验证后返回token"

主规范中的描述为：
"用户输入邮箱和密码，系统验证后返回token"

→ 完全相同，去重，保留主规范版本
→ 删除新需求中的user-login.md
```

**结果2：相关但不同的场景**
```
新需求的 user-password-change.md：
"认证用户修改自己的密码"

主规范的 user-password-reset.md：
"未认证用户通过邮件重置密码"

→ 相关但逻辑不同，都需要保留
→ 在主规范中分别描述为：
  - Change Password: 已认证用户修改
  - Reset Password: 未认证用户通过邮件重置
```

**结果3：全新的场景**
```
新需求的 user-mfa.md：
"用户启用多因素认证，支持TOTP和SMS"

主规范中没有相关场景

→ 全新，直接添加到主规范的scenarios/
```

### 步骤3: 规范合并

#### 3.1 场景库合并

```bash
# 1. 创建合并工作区
cp -r spec-dev/spec spec-dev/spec-backup-$(date +%Y%m%d)

# 2. 对比两个scenarios目录
diff -r spec-dev/{requirement_id}/spec/scenarios/ spec-dev/spec/scenarios/

# 3. 执行合并
# 对于完全相同的场景 → 删除新的，保留旧的
# 对于相关但不同的场景 → 两个都保留，添加注释说明差异
# 对于全新的场景 → 复制新的到主规范
```

**实际合并操作示例**：

```bash
# 去重操作
# user-login.md 在两个地方都有，保留主规范版本
rm spec-dev/order-form/spec/scenarios/user-login.md

# 合并操作
# user-password-change.md 是新的，添加到主规范
cp spec-dev/order-form/spec/scenarios/user-password-change.md \
   spec-dev/spec/scenarios/user-password-change.md

# 新增操作
# user-mfa.md 是全新的，直接复制
cp spec-dev/order-form/spec/scenarios/user-mfa.md \
   spec-dev/spec/scenarios/user-mfa.md
```

**合并后的结果**：
```
spec-dev/spec/scenarios/
├── user-login.md                    # 保留（完全相同）
├── user-signup.md                   # 保留（完全相同）
├── user-forgot-password.md          # 保留（完全相同）
├── user-password-change.md          # 新增（相关但不同）
├── user-password-reset.md           # 原有
├── user-mfa.md                      # 新增（全新）
├── user-logout.md                   # 保留（完全相同）
└── ...
```

#### 3.2 数据模型合并

**分析两个data-models.md的差异**：

```markdown
## 数据模型对标

### User 模型

**新需求定义**：
```
User:
  - id (String): 用户ID
  - email (String): 邮箱
  - password_hash (String): 密码哈希
  - created_at (DateTime): 创建时间
  - mfa_enabled (Boolean): 是否启用MFA
  - mfa_secret (String): MFA密钥
```

**主规范定义**：
```
User:
  - id (String): 用户ID
  - email (String): 邮箱
  - password_hash (String): 密码哈希
  - created_at (DateTime): 创建时间
```

**差异分析**：
- id, email, password_hash, created_at → 完全相同，去重
- mfa_enabled, mfa_secret → 新增字段，添加到主规范

**合并后**：
```
User:
  - id (String): 用户ID
  - email (String): 邮箱
  - password_hash (String): 密码哈希
  - created_at (DateTime): 创建时间
  - mfa_enabled (Boolean): 是否启用MFA [新增于 order-form]
  - mfa_secret (String): MFA密钥 [新增于 order-form]
```
```

**完整的合并过程**：

```bash
# 比对所有模型
diff -u spec-dev/spec/data-models.md \
       spec-dev/order-form/spec/data-models.md

# 手动编辑主规范的data-models.md
# 1. 保留原有的所有模型
# 2. 添加新增的字段到相应模型
# 3. 添加新增的模型
# 4. 添加注释说明来源
```

#### 3.3 业务规则合并

**分析两个business-rules.md的差异**：

```markdown
## 业务规则对标

| 新需求规则 | 主规范中的对应规则 | 分类 | 处理方式 |
|----------|-----------------|------|---------|
| 密码最小长度8字符 | 密码最小长度8字符 | 完全相同 | 去重 |
| 邮箱必须唯一 | 邮箱必须唯一 | 完全相同 | 去重 |
| 登录失败5次后锁定 | - | 全新 | 新增 |
| Session有效期24小时 | Token有效期24小时 | 相同 | 去重（更新描述） |
| MFA必须验证 | - | 全新 | 新增 |
```

**实际操作**：

```bash
# 编辑 spec-dev/spec/business-rules.md
# 1. 保留原有的所有规则
# 2. 新增的规则添加到相应section
# 3. 冲突的规则标记为需要解决
# 4. 相关的规则添加cross-reference
```

#### 3.4 术语表合并

```markdown
## 术语表合并

### 新增术语

新需求引入的术语：
- **MFA (Multi-Factor Authentication)**: 多因素认证，额外的安全验证方法
- **TOTP (Time-based One-Time Password)**: 基于时间的一次性密码
- **SMS OTP**: 通过短信发送的一次性密码

### 已有术语

主规范中已有：
- **Session**: 用户会话，用于维持登录状态
- **Token**: 访问令牌，用于认证请求
- **Password Hash**: 密码哈希，密码的加密存储形式

### 合并结果

新增术语数：3个
已有术语数：3个
总计术语数：6个
```

### 步骤4: 一致性检查

#### 4.1 检查清单

```markdown
## 一致性检查清单

### 场景完整性
- [ ] 所有新场景都有明确的WHEN-THEN格式
- [ ] 所有场景都有对应的验收标准（TEST-VERIFY）
- [ ] 场景之间没有重复或冲突
- [ ] 场景覆盖了所有业务规则

### 数据模型完整性
- [ ] 所有模型都有清晰的字段定义
- [ ] 所有字段都有类型标注
- [ ] 模型之间的关系清晰（外键等）
- [ ] 没有重复定义的模型

### 业务规则完整性
- [ ] 所有规则都有清晰的描述
- [ ] 所有规则都在某个场景中被应用
- [ ] 规则之间没有冲突
- [ ] 特殊情况都有对应的规则

### 术语表完整性
- [ ] 所有新术语都在glossary.md中定义
- [ ] 所有术语在scenarios中都有使用
- [ ] 术语定义清晰且一致
- [ ] 没有遗漏的术语

### 跨文件一致性
- [ ] scenarios中使用的术语在glossary中都有定义
- [ ] scenarios中引用的模型在data-models中都有定义
- [ ] scenarios中应用的规则在business-rules中都有定义
- [ ] data-models中的每个模型至少在一个场景中被使用
```

#### 4.2 运行一致性检查

```bash
# 1. 检查是否存在孤立的模型（未在任何场景中使用）
grep -r "Order\|User\|Payment" spec-dev/spec/scenarios/
grep "Order\|User\|Payment" spec-dev/spec/data-models.md

# 2. 检查是否存在未定义的术语
grep -o "[A-Z][A-Z_]*" spec-dev/spec/scenarios/*.md | \
  while read term; do
    grep -q "$term" spec-dev/spec/glossary.md || \
    echo "未定义的术语: $term"
  done

# 3. 检查是否存在未应用的规则
while read rule; do
  grep -q "$rule" spec-dev/spec/scenarios/*.md || \
  echo "未应用的规则: $rule"
done < spec-dev/spec/business-rules.md
```

#### 4.3 一致性报告

```markdown
## 一致性检查报告

### 检查结果
- 总场景数：18个 ✅
- 总模型数：8个 ✅
- 总规则数：12个 ✅
- 总术语数：22个 ✅

### 完整性检查
- 孤立模型：0个 ✅
- 未定义术语：0个 ✅
- 未应用规则：0个 ✅
- 场景完整性：100% ✅

### 冲突检查
- 模型冲突：0个 ✅
- 规则冲突：0个 ✅
- 术语冲突：0个 ✅

### 最终结论
✅ 所有检查都通过，规范库一致性良好
```

### 步骤5: 生成归档报告

#### 5.1 报告结构

在 `spec-dev/spec/archive-log.md` 中记录所有归档历史：

```markdown
# 规范库归档日志

## 最近的归档

### 2026-03-26 order-form 需求

**基本信息**
- 需求ID：order-form
- 归档时间：2026-03-26 14:30:00
- 操作者：Claude
- 状态：✅ 完成

**归档内容统计**
- 新场景数：18个
- 新模型数：4个
- 新规则数：8个
- 新术语数：15个

**合并结果**
- 去重场景：4个
- 合并场景：2个
- 新增场景：12个
- 去重模型：2个
- 新增字段：6个
- 新增模型：2个
- 去重规则：6个
- 新增规则：2个
- 去重术语：10个
- 新增术语：5个

**一致性检查**
✅ 所有场景都有明确的WHEN-THEN格式
✅ 所有模型都有清晰的定义
✅ 所有规则都在至少一个场景中应用
✅ 所有术语都有定义和使用
✅ 没有冲突或遗漏

**规范库当前状态**
- 总场景数：42个
- 总模型数：12个
- 总规则数：18个
- 总术语数：35个

---

## 历史记录

### 2026-03-20 user-auth 需求
- 新增：8个场景，3个模型，5个规则
- 结果：✅ 完成

### 2026-03-15 payment 需求
- 新增：6个场景，2个模型，3个规则
- 结果：✅ 完成

...
```

#### 5.2 追踪信息

为每个合并的部分添加来源追踪：

```markdown
## scenarios/user-mfa.md

来源：order-form需求（2026-03-26）
状态：新增
描述：多因素认证场景

---
WHEN 用户登录后
AND 用户已启用MFA
THEN 系统要求选择MFA方式
AND 用户选择TOTP验证
...
```

### 步骤6: 可选的清理操作

#### 6.1 备份原需求目录

```bash
# 备份原需求的spec-dev目录
cp -r spec-dev/order-form spec-dev/archive/order-form-backup-$(date +%Y%m%d)

# 验证备份完整性
diff -r spec-dev/order-form spec-dev/archive/order-form-backup-*/
```

#### 6.2 删除原需求目录（谨慎操作）

```bash
# 仅在以下条件下执行：
# 1. 已获得用户的明确确认
# 2. 已完成备份
# 3. 已在archive-log中记录

# 删除原需求的spec-dev目录
rm -rf spec-dev/order-form

# 验证删除
ls spec-dev/  # 确认order-form已删除
```

---

## 常见问题

### Q: 如何处理两个版本的数据模型冲突？

A: 场景示例

```
新需求的User模型中 age 是 Integer
主规范中 age 没有定义

→ 简单，直接添加age字段

---

但如果：
新需求的User模型中 age 是 Integer
主规范中 age 是 String

→ 冲突！需要：
1. 标记冲突在archive-log中
2. 标记在文档中 [CONFLICT]
3. 等待用户或team review确认
4. 不要直接覆盖
```

### Q: 如何验证规范库的质量？

A: 使用以下几个维度：

```
1. 完整性：所有模型都被场景使用吗？
2. 一致性：同一术语的定义都相同吗？
3. 追踪性：每个内容都能追踪到来源需求吗？
4. 可维护性：新增需求能否轻松找到重复？
5. 演进性：规范库在逐步完善吗？
```

### Q: 规范库的长期维护策略是什么？

A: 建议：

```
1. 定期评审（每个季度）
  - 检查冗余和重复
  - 识别可以整合的相似规则

2. 版本管理
  - 每月生成快照
  - 记录演进历史

3. 团队协作
  - 重大合并需要技术review
  - 记录设计决策

4. 持续优化
  - 基于新需求不断改进结构
  - 提炼通用的最佳实践
```

---

## 最佳实践

### 1. 保留完整的审计日志

```markdown
在 archive-log.md 中记录：
- 什么时候归档的
- 从哪个需求归档的
- 什么被添加、修改、删除了
- 是否有冲突需要解决
- 团队的批准
```

### 2. 建立追踪机制

```markdown
每个归档的规范都应该包含来源信息：

# scenarios/user-mfa.md

## 来源
- 需求：order-form
- 归档日期：2026-03-26
- 原始版本：/archive/order-form-backup-20260326/spec/scenarios/user-mfa.md
```

### 3. 定期清理和优化

```
操作：
- 每个新需求归档后，检查是否有冗余规则可以合并
- 每个季度进行一次major review
- 记录优化历史
```

### 4. 版本控制最佳实践

```bash
# 每次归档前创建分支
git checkout -b archive/order-form-2026-03-26

# 完成所有合并和检查后
git commit -m "archive: merge order-form spec into main spec library"

# 合并回主分支
git checkout master
git merge archive/order-form-2026-03-26
```

---

**关键理念**：规范库是企业的知识资产，通过规范的归档流程和完整的审计日志，确保知识的积累和可追踪性。每个需求都为规范库贡献一块拼图。
