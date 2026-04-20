# TDD执行阶段、Skills和Agent调用清单

## 📋 概览

本文档明确说明在sdd-dev-plugin中，TDD（测试驱动开发）执行的5个完整阶段，以及每个阶段对应的Skill、Agent和关键活动。

```
需求规范化      设计规划        任务分解        TDD实现        高层测试
    ↓              ↓              ↓              ↓              ↓
/spec-creation  /code-designer  /code-task   /code-execute   /code-test
    ↓              ↓              ↓              ↓              ↓
Phase 0        Phase 1         Phase 2        Phase 3         Phase 4
（规范）       （设计）         （任务）       （编码测试）     （验证测试）
```

---

## Phase 0: 需求规范化（Specification Phase）

### 职责
将自然语言需求转化为**结构化的BDD格式规范**，为后续设计和开发奠定基础。

### 对应Skill
- **Skill**: `/spec-creation`
- **位置**: `plugins/sdd-dev-plugin/skills/spec-creation/SKILL.md`

### 工作流程

#### 步骤0.1: 需求收集和澄清
1. 用户提供需求描述（可以是自然语言或初步规范）
2. AI进行交互式问卷，澄清以下信息：
   - 项目名称、描述、技术栈
   - 业务场景和核心功能
   - 设计模式和参考组件
   - 特殊需求和约束条件

#### 步骤0.2: 规范文档生成
AI根据澄清结果，生成以下结构化规范文档：

**输出目录**: `spec-dev/{requirement_desc_abstract}/spec/`

```
spec-dev/{requirement_desc_abstract}/spec/
├── README.md                    # 规范总览
├── scenarios/
│   ├── scenario-1.md           # 业务场景 1（WHEN-THEN-TEST-VERIFY格式）
│   ├── scenario-2.md           # 业务场景 2
│   └── ...
├── data-models.md              # 数据模型定义
├── business-rules.md           # 业务规则和约束
└── glossary.md                 # 术语表
```

#### 步骤0.3: 规范内容详解

每个场景文件（scenarios/*.md）包含：

```markdown
## [业务场景名称]

### WHEN（场景触发条件）
描述什么情况下触发该业务场景...

### THEN（预期结果）
描述该业务场景的预期结果...

### TEST-VERIFY（可测试的验收标准）⭐ TDD关键
- [ ] 应该[动作]，[期望结果]
- [ ] 应该验证[字段/属性]为[值]
- [ ] 应该[行为]不应该[禁止行为]
- [ ] 边界情况：[边界条件]

### Mock Data（测试数据和边界值）
有效输入示例、边界值范围等，供后续单元测试使用
```

### 关键约束
- ✅ **规范优先** - 规范是整个TDD工作流的源头
- ✅ **可测试** - TEST-VERIFY部分明确定义了可验证的验收标准
- ✅ **完整** - 包含所有业务场景和数据模型定义
- ❌ **不进行设计** - Phase 0只负责规范定义，不涉及代码架构

### 输出验收标准
- 规范文档完整（README、scenarios、data-models、business-rules、glossary）
- 所有场景都有WHEN-THEN-TEST-VERIFY-MockData
- 没有待明确的TODO项

---

## Phase 1: 设计规划（Design Phase）

### 职责
根据Phase 0的规范，进行**详细的代码设计规划**，包括架构设计、技术方案、组件设计等。

### 对应Skill
- **Skill**: `/code-designer`
- **位置**: `plugins/sdd-dev-plugin/skills/code-designer/SKILL.md`

### 工作流程

#### 步骤1.1: 规范分析和确认
1. 读取 `spec-dev/{requirement_desc_abstract}/spec/` 中的规范文档
2. 确认理解需求的所有关键点
3. 与用户确认设计思路

#### 步骤1.2: 架构设计
设计以下内容：
- 整体架构（单体 vs 微服务 vs 前后端分离等）
- 关键模块划分
- 数据流设计
- 组件或类的设计

#### 步骤1.3: 技术方案
定义以下方面：
- 技术栈选择及原因
- 框架和库的使用
- API设计（REST/GraphQL）
- 数据库架构
- 性能优化方案
- 安全性考虑

#### 步骤1.4: 设计文档生成

**输出文件**: `spec-dev/{requirement_desc_abstract}/design/design.md`

```markdown
# 设计方案文档

## 需求分析
- 核心功能概述
- 关键业务流程
- 用户场景分析

## 架构设计
- 整体架构说明
- 模块划分
- 数据流图

## 关键组件设计
- 组件1的设计
  - Props和接口
  - 内部状态管理
  - 生命周期
- 组件2的设计
- ...

## 技术方案
- 技术栈
- 框架选择
- API设计
- 数据库设计
- 性能方案

## 设计决策记录（Design Decision Record）
| 决策 | 选项 | 选择 | 原因 |
|-----|------|------|------|
| 状态管理 | Redux/Context/Zustand | ... | ... |
| 测试框架 | Jest/Vitest | ... | ... |
| ... | ... | ... | ... |
```

### 关键约束
- ✅ **基于规范** - 设计必须严格基于Phase 0的规范
- ✅ **详细完整** - 设计要足够详细，能指导后续任务分解
- ✅ **可验证** - 设计方案中的关键点要在后续Task中能验证
- ❌ **不生成代码** - Phase 1是设计，不涉及实际代码实现

### 输出验收标准
- 设计文档完整（架构、技术方案、组件设计、设计决策）
- 设计与规范一致
- 所有设计决策都有明确的原因说明
- 用户已审查并批准设计方案

---

## Phase 2: 任务分解（Task Phase）

### 职责
将Phase 1的设计方案**转化为可执行的代码级任务列表**，每个Task都有明确的目标、交付物、验收标准。

### 对应Skill
- **Skill**: `/code-task`
- **位置**: `plugins/sdd-dev-plugin/skills/code-task/SKILL.md`

### 工作流程

#### 步骤2.1: 设计分析
1. 读取 `spec-dev/{requirement_desc_abstract}/design/design.md`
2. 理解整个架构和设计方案
3. 确认关键的技术点和实现策略

#### 步骤2.2: 任务分解
将设计分解为具体的编码任务，遵循以下原则：
- 每个Task是一个**独立可完成的工作单元**（4小时内可完成）
- Task之间有清晰的**依赖关系**
- 每个Task都关联规范中的**TEST-VERIFY验收标准**

#### 步骤2.3: 任务定义

**输出文件**: `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`

每个Task包含以下信息：

```markdown
## Task T1: [Task名称]

### 目标
[具体的目标描述]

### 交付物清单
- [ ] [文件1或功能1]
- [ ] [文件2或功能2]

### 依赖关系
- 前置Task: T0（如果有）
- 后续Task: T2, T3（如果有）

### 验收标准
[列出该Task的验收标准]

### 技术栈
[涉及的框架、库、工具]

### Test Case Mapping（TDD关键 ⭐）
将规范中的TEST-VERIFY映射到该Task的测试用例：

| # | 验收标准 | 来源 | 对应实现 | 测试ID |
|----|---------|------|--------|--------|
| 1 | 应该支持XXX | Scenario1#1 | src/api.ts:fetchData() | TC-T1.1.1 |
| 2 | 应该验证参数 | Scenario1#2 | src/api.ts:validate() | TC-T1.1.2 |
| 3 | 边界：最小值0 | Scenario2 | src/api.ts:fetchData() | TC-T1.2.1 |

### Mock数据关联
来自规范的测试数据：
- 有效输入：{ ... }
- 边界值：{ ... }
- 错误情况：{ ... }
```

#### 步骤2.4: 并行执行计划

在tasks.md中定义并行执行计划：

```markdown
## 并行执行计划

### Batch 1（第1批，可并行）
- T1: UserAuth（无依赖）
- T2: FormValidation（无依赖）
- T3: DataFetching（无依赖）

### Batch 2（第2批，依赖Batch 1）
- T4: AuthenticatedFormSubmit（依赖 T1+T2）
- T5: DataDisplay（依赖 T3）

### Batch 3（第3批，依赖Batch 2）
- T6: ErrorHandling（依赖 T4+T5）
- T7: Performance（依赖 T6）
```

### 关键约束
- ✅ **关联规范** - 每个Task都要关联规范中的TEST-VERIFY
- ✅ **清晰验收** - 验收标准和交付物要明确
- ✅ **合理粒度** - 每个Task在4小时内可完成
- ✅ **依赖清晰** - 任务之间的依赖关系明确

### 输出验收标准
- 任务列表完整（所有设计点都被覆盖）
- 每个Task都有TEST-VERIFY映射
- 任务依赖关系清晰
- 并行执行计划可行
- 用户已确认任务列表

---

## Phase 3: 代码执行（Execute Phase）

### 职责
**通过TDD流程执行Phase 2的任务列表**，进行**多阶段审查**，完成**单元测试**。这是整个SDD工作流中最核心的Phase，包含4个子阶段（RED-GREEN-REFACTOR-REVIEW）。

### 对应Skill
- **Skill**: `/code-execute`
- **位置**: `plugins/sdd-dev-plugin/skills/code-execute/SKILL.md`

### 核心特性
- **TDD驱动** - 严格遵循RED-GREEN-REFACTOR-REVIEW流程
- **多阶段审查** - 规范审查 + 代码质量审查
- **Git-Worktrees隔离** - 每个Task在独立的worktree中工作
- **单元测试覆盖** - ≥85%的代码覆盖率
- **可修复循环** - 发现问题可在worktree中修复和重新审查

### 工作流程

#### 步骤3.0: 准备执行环境

1. 读取 `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`
2. 提取所有Task信息和并行执行计划
3. 创建TodoList追踪Task完成状态
4. 准备代码上下文和参考实现

#### 步骤3.1: 并行批处理

根据Task之间的依赖关系，分批并行执行：

```
Batch 1 (T1, T2, T3) → 并行执行 3 个 Task
    ↓ (等待 Batch 1 完成)
Batch 2 (T4, T5)    → 并行执行 2 个 Task
    ↓ (等待 Batch 2 完成)
Batch 3 (T6, T7)    → 并行执行 2 个 Task
```

**对于每个Batch**：
- 为Batch中的每个Task分配独立的 **code-executor Agent实例**
- 这些Agent实例**并行执行**，互不影响
- 等待Batch中所有Task完成后，再进入下一个Batch

#### 步骤3.2: 单个Task的TDD执行流程

对于分配到的每个Task，code-executor Agent按以下流程执行：

##### 3.2.1: 准备Task上下文

```
1. 读取Task完整信息（目标、交付物、验收标准、Test Case Mapping）
2. 读取关联的规范中的TEST-VERIFY
3. 读取Test Case Mapping表
4. 准备Mock数据
5. 如果有前置Task，等待前置Task完成
```

##### 3.2.1b: 创建隔离工作环境（Git-Worktree）

为该Task创建独立的git-worktree分支，确保隔离性和可恢复性：

```bash
# 创建worktree
git worktree add .claude/worktrees/T1-UserAuth main
cd .claude/worktrees/T1-UserAuth

# 后续所有编码、修复都在该worktree中进行
```

**Worktree的优势**：
- ✅ **完全隔离** - Task在独立的git树中工作，互不污染
- ✅ **安全修复** - 修复失败可直接删除worktree重新开始
- ✅ **清晰审计** - 每个worktree的commit历史清晰记录Task的所有修复
- ✅ **并行安全** - 多Task并行执行时，worktree避免git冲突

详见: [Git-Worktrees指南](./code-execute/git-worktrees-guide.md)

##### 3.2.2: RED阶段 - 编写失败的单元测试

分配 **Implementer Agent** 执行以下操作：

```
1. 读取Task中的Test Case Mapping表
2. 对每个测试用例ID（TC-T1.1.1等），生成失败的单元测试
3. 基于规范中的TEST-VERIFY和Mock数据编写测试
4. 运行测试 → 确保所有测试都失败（RED状态）
5. 在worktree中提交：git commit -m "test(T1): 编写T1的单元测试"
```

**测试框架选择**：
- 前端（React/Vue）: Jest或Vitest
- 后端（Node.js）: Jest或其他框架
- 其他语言：根据技术栈选择

**TEST代码示例**：

```typescript
// 基于规范的TEST-VERIFY编写测试
describe('T1: UserAuth', () => {
  describe('当用户使用有效的用户名密码登录', () => {
    it('应该验证参数并返回成功', () => {
      // 测试对应：TC-T1.1.1 (来自Scenario1#1)
      const result = authenticate({ username: 'john', password: 'pass123' });
      expect(result).toEqual(expect.objectContaining({
        success: true,
        token: expect.any(String)
      }));
    });

    it('应该验证用户名不为空', () => {
      // 测试对应：TC-T1.1.2 (来自Scenario1#2)
      expect(() => authenticate({ username: '', password: 'pass' }))
        .toThrow('Username is required');
    });
  });

  describe('当输入边界值', () => {
    it('应该处理最短的有效用户名', () => {
      // 测试对应：TC-T1.2.1 (来自Scenario2的边界值)
      const result = authenticate({ username: 'a', password: 'pass' });
      expect(result.success).toBe(true);
    });
  });
});
```

##### 3.2.3: GREEN阶段 - 实现最少代码

分配 **Implementer Agent** 执行以下操作：

```
1. 编写最少量的代码让测试通过
2. 遵循以下原则：
   - 优先让测试通过，而不是完美设计
   - 使用最简单的实现方式
   - 如果代码不清晰，会在REFACTOR阶段改进
3. 运行测试 → 确保所有测试都通过（GREEN状态）
4. 在worktree中提交：git commit -m "feat(T1): 实现T1的基本功能"
```

**IMPL代码示例**：

```typescript
// 最少代码实现，让测试通过
export function authenticate(credentials: { username: string; password: string }) {
  if (!credentials.username) {
    throw new Error('Username is required');
  }
  if (!credentials.password) {
    throw new Error('Password is required');
  }

  // 生成简单的token（实际应该调用后端）
  const token = Buffer.from(`${credentials.username}:${credentials.password}`).toString('base64');

  return {
    success: true,
    token
  };
}
```

##### 3.2.4: REFACTOR阶段 - 改进代码质量

分配 **Implementer Agent** 执行以下操作：

```
1. 在测试通过的前提下重构代码
2. 关注点：
   - 代码可读性：变量名、函数名、逻辑清晰
   - 类型完整：所有参数和返回值都有TypeScript类型
   - 错误处理：完善错误处理和验证逻辑
   - 代码复用：提取公共逻辑、消除重复代码
   - 遵循最佳实践：SOLID原则等
3. 重新运行测试 → 确保所有测试仍然通过
4. 在worktree中提交：git commit -m "refactor(T1): 改进T1的代码质量"
```

**REFACTOR后的代码示例**：

```typescript
// 改进后的代码：清晰的类型、错误处理、验证逻辑
interface Credentials {
  username: string;
  password: string;
}

interface AuthResult {
  success: boolean;
  token: string;
  expiresIn?: number;
}

class AuthenticationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'AuthenticationError';
  }
}

export async function authenticate(credentials: Credentials): Promise<AuthResult> {
  // 验证输入
  validateCredentials(credentials);

  // 调用认证服务
  const result = await authService.verify(credentials);

  if (!result.valid) {
    throw new AuthenticationError('Invalid credentials');
  }

  // 生成token
  const token = await generateToken(credentials.username);

  return {
    success: true,
    token,
    expiresIn: 3600
  };
}

function validateCredentials(credentials: Credentials): void {
  if (!credentials.username?.trim()) {
    throw new AuthenticationError('Username is required');
  }
  if (!credentials.password) {
    throw new AuthenticationError('Password is required');
  }
  if (credentials.username.length < 3) {
    throw new AuthenticationError('Username must be at least 3 characters');
  }
}
```

##### 3.2.5: REVIEW阶段 - 多阶段审查

完成编码后，进行两阶段审查，确保代码质量：

**第1次审查: 规范审查（Specification Review）**

分配 **Spec-Reviewer Agent** 执行以下检查：

```
检查点：
1. 代码是否符合 design.md 的架构规范？
   - 模块结构是否符合设计中的模块划分？
   - 接口定义是否与设计中的API设计一致？
   - 数据流是否符合设计中的数据流规范？

2. 代码是否遵循规范中的技术方案？
   - 使用的框架和库是否与设计中的技术选择一致？
   - 实现方式是否符合设计中的技术方案？
   - 命名规范是否与设计中的命名约定一致？

3. Props/接口是否与任务定义一致？
   - 函数参数是否与Task中的交付物定义一致？
   - 返回值是否符合预期？
   - 错误处理方式是否符合设计？

4. 功能实现是否完整？
   - 所有Test Case都有对应的实现吗？
   - 所有验收标准都被满足了吗？
   - 是否有遗漏的功能点？

输出：规范审查报告
```

**规范审查报告示例**：

```markdown
## 规范审查报告 - T1: UserAuth

### 整体评价
✅ PASS - 代码符合design.md中的规范

### 详细检查

#### 架构一致性
✅ PASS - 模块结构符合design.md中的模块划分
✅ PASS - authenticate() 接口与设计中的API定义一致
✅ PASS - 数据流（credentials → validation → token generation）符合设计

#### 技术方案一致性
✅ PASS - 使用TypeScript strict模式，符合设计中的类型安全要求
✅ PASS - 实现了错误处理，符合设计中的错误处理方案
✅ PASS - 命名规范（camelCase）符合项目规范

#### Props/接口一致性
✅ PASS - 参数类型完整：Credentials { username: string; password: string }
✅ PASS - 返回值类型完整：AuthResult { success: boolean; token: string; ... }

#### 功能完整性
✅ PASS - 所有Test Case (TC-T1.1.1 ~ TC-T1.2.1) 都有实现
✅ PASS - 所有TEST-VERIFY验收标准都被满足

### 审查建议
无。代码符合规范。

### 结论
✅ PASS - 规范审查通过，可进入质量审查
```

如果规范审查不通过：

```markdown
## 规范审查报告 - T1: UserAuth

### 整体评价
❌ FAIL - 代码不符合design.md中的规范，需要修改

### 问题清单

1. **CRITICAL**: authenticateUser() 函数名与 design.md 中的 authenticate() 不一致
   - 需要改为 authenticate()
   - 影响：接口不一致

2. **MAJOR**: 没有实现错误处理
   - design.md 中明确要求实现 AuthenticationError
   - 目前直接抛出通用Error
   - 需要创建自定义错误类

3. **MAJOR**: 参数验证不完整
   - TC-T1.1.2要求验证密码长度，但代码未实现
   - 需要添加password长度验证

### 修复方案
1. 修改函数名为 authenticate()
2. 创建 AuthenticationError 自定义错误类
3. 完善参数验证逻辑（密码长度）
4. 重新提交并运行测试验证

### 结论
❌ FAIL - 规范审查失败，请按上述方案修复后重新审查
```

**如果规范审查失败，执行修复循环**：

```
1. Implementer Agent 根据反馈修复代码
2. 在worktree中提交修复：git commit -m "fix(T1): 修复规范审查问题"
3. 重新进行规范审查
4. 直到规范审查通过
```

---

**第2次审查: 质量审查（Code Quality Review）**

分配 **Code-Quality-Reviewer Agent** 执行以下检查：

```
检查点：

1. 代码是否符合项目编码规范？
   - ESLint/Prettier检查是否通过？
   - 代码风格是否一致？

2. TypeScript strict模式是否通过？
   - 是否有any类型（禁止）？
   - 是否有隐式any？
   - 所有参数和返回值都有类型注解吗？

3. 是否有Lint错误或警告？
   - 检查ESLint输出
   - 处理所有CRITICAL和MAJOR问题

4. 代码复杂度是否过高？
   - 函数长度是否超过50行？
   - 循环嵌套层数是否过深？
   - 圈复杂度（CC）是否过高（>15）？

5. 是否有性能问题？
   - 是否有不必要的循环？
   - 是否有不必要的重新计算？
   - 是否有内存泄漏（如未清理的事件监听）？

6. 测试覆盖率是否达到 ≥ 85%？
   - 运行 npm run test:coverage
   - 检查覆盖率报告

输出：质量审查报告
```

**质量审查报告示例**：

```markdown
## 代码质量审查报告 - T1: UserAuth

### 整体评价
✅ PASS - 代码质量达到要求

### 详细检查

#### ESLint和Prettier
✅ PASS - 0个Lint错误
✅ PASS - 0个格式化问题

#### TypeScript strict模式
✅ PASS - 0个TypeScript错误
✅ PASS - 0个any类型
✅ PASS - 所有参数和返回值都有完整的类型注解

#### 代码复杂度
✅ PASS - 所有函数长度 < 50行
✅ PASS - 最高圈复杂度（CC）= 8，< 15

#### 代码覆盖率
✅ PASS - 单元测试覆盖率 = 92% (> 85%)
   - authenticate() 函数：100%
   - validateCredentials() 函数：85%
   - 所有分支都被覆盖

### 审查建议
无。代码质量达到要求。

### 结论
✅ PASS - 质量审查通过，Task完成！
```

如果质量审查不通过：

```markdown
## 代码质量审查报告 - T1: UserAuth

### 整体评价
❌ FAIL - 代码质量未达到要求，需要改进

### 问题清单

1. **CRITICAL**: 存在TypeScript错误（2个）
   - Line 15: password: any （禁止使用any）
   - Line 22: 返回值类型未标注
   - 需要完善类型注解

2. **MAJOR**: 代码覆盖率不足（78% < 85%）
   - generateToken() 函数未被测试覆盖
   - 需要添加generateToken()的单元测试

3. **MAJOR**: 验证逻辑复杂度过高（CC=18）
   - validateCredentials() 函数有过多的if嵌套
   - 需要重构为多个较小的验证函数

### 修复方案
1. 完善TypeScript类型注解（移除any）
2. 添加缺失的单元测试（generateToken）
3. 重构validateCredentials()函数，拆分为多个验证函数

### 结论
❌ FAIL - 质量审查失败，请按上述方案改进后重新审查
```

**如果质量审查失败，执行修复循环**：

```
1. Implementer Agent 根据反馈改进代码
2. 运行 npm run lint 和 npm run test:coverage 验证
3. 在worktree中提交修复：git commit -m "refactor(T1): 改进代码质量"
4. 重新进行质量审查
5. 直到质量审查通过
```

---

##### 3.2.6: 将Task提交合并回main分支

Task的RED-GREEN-REFACTOR-REVIEW全部通过后：

```bash
# 在worktree中查看提交历史
git log --oneline

# 切换回main分支
cd <project-root>

# 将worktree的修改合并到main
# 方案1: Cherry-pick所有commits
git cherry-pick <commit1> <commit2> <commit3>

# 方案2: Squash merge（推荐）
git merge --squash .claude/worktrees/T1-UserAuth
git commit -m "feat(T1): 实现用户认证功能"

# 方案3: 直接merge
git merge --ff-only .claude/worktrees/T1-UserAuth
```

详见: [Git-Worktrees指南](./code-execute/git-worktrees-guide.md)

---

#### 步骤3.3: 执行报告生成

所有Task完成后，生成执行报告：

**输出文件**: `spec-dev/{requirement_desc_abstract}/execution/execution-report.md`

```markdown
# 执行报告

## 执行摘要
- 总Task数: 7
- 完成Task数: 7
- 平均迭代次数: 1.3
- 总代码行数: 1200
- 平均单元测试覆盖率: 87%

## 时间统计
- 总耗时: 8小时
- 规范审查平均耗时: 20分钟/Task
- 质量审查平均耗时: 15分钟/Task

## 每个Task的执行情况

### T1: UserAuth
- **状态**: ✅ 完成
- **代码行数**: 150
- **测试覆盖率**: 92%
- **迭代次数**: 1（一次通过）
- **TDD流程**:
  - RED: 编写了8个单元测试，全部失败 ✅
  - GREEN: 实现基本认证逻辑，所有测试通过 ✅
  - REFACTOR: 完善错误处理和类型注解，测试仍通过 ✅
  - REVIEW: 规范审查通过，质量审查通过 ✅
- **Git提交**:
  - test(T1): 编写T1的单元测试
  - feat(T1): 实现T1的基本功能
  - refactor(T1): 改进T1的代码质量
- **输出文件**:
  - src/auth.ts (150 LOC)
  - src/auth.test.ts (120 LOC)

### T2: FormValidation
- **状态**: ✅ 完成
- ... (类似T1的详细信息)

## 问题和修复记录

### T3中发现的问题（规范审查失败）
**问题**: 使用了不符合设计的数据结构
- 规范审查反馈：validateForm()返回值与design.md定义不一致
- 修复：更改返回值类型，重新命名变量
- 修复后：规范审查通过

### T5中发现的问题（质量审查失败）
**问题**: 代码覆盖率不足（78% < 85%）
- 质量审查反馈：缺少3个关键函数的单元测试
- 修复：补充单元测试，覆盖边界情况
- 修复后：覆盖率提升到91%，质量审查通过

## 最终结论

✅ **执行完成**
- 所有Task都通过了RED-GREEN-REFACTOR-REVIEW的TDD流程
- 所有Task都通过了规范审查和质量审查
- 平均单元测试覆盖率达到87%（> 85%要求）
- 所有代码都已合并回main分支

⚠️ **建议**
1. T5的asyncValidateForm()可以进一步性能优化
2. 建议建立关于错误处理的编码规范文档
```

### 关键约束
- ✅ **严格TDD** - 必须遵循RED-GREEN-REFACTOR-REVIEW流程
- ✅ **两道审查** - 规范审查和质量审查都要通过
- ✅ **单元测试** - 覆盖率≥85%
- ✅ **修复循环** - 审查失败必须修复后重新审查
- ✅ **Git隔离** - 使用git-worktree为每个Task隔离

### 输出验收标准
- 所有Task都通过RED-GREEN-REFACTOR-REVIEW流程
- 所有Task都通过规范审查
- 所有Task都通过质量审查
- 单元测试覆盖率≥85%
- 代码已合并回main分支
- 执行报告完整

---

## Phase 4: 高层测试和闭环验证（Test Phase）

### 职责
对Phase 3执行的代码进行**高层测试**（集成、E2E、性能）和**完整的闭环验证**。

### 对应Skill
- **Skill**: `/code-test`
- **位置**: `plugins/sdd-dev-plugin/skills/code-test/SKILL.md`

### 核心特性
- **高层测试** - 集成、E2E、性能测试（不重复单元测试）
- **闭环验证** - 验证规范TEST-VERIFY → 代码实现 → 测试用例 → 测试结果的完整对应
- **完整追踪** - 从规范到代码到测试的完整链条

### 工作流程

#### 步骤4.1: 代码质量审查

进行最终的代码质量检查：

```
1. 运行ESLint检查: npm run lint
2. 运行TypeScript strict检查: npm run type-check
3. 对比design.md中的设计规范
   - 架构是否符合设计？
   - Props/接口是否一致？
   - 样式是否一致？
```

#### 步骤4.2: 单元测试验证

验证Phase 3中code-execute生成的单元测试：

```
1. 运行单元测试: npm test
2. 检查覆盖率报告: npm run test:coverage
3. 验证覆盖率≥85%
4. 检查测试是否通过
```

**注意**: code-test不重复执行单元测试，只是验证覆盖率和测试质量

#### 步骤4.3: 集成测试

测试多个Task/模块间的协作：

分配 **Integration-Test Designer** 创建集成测试：

```markdown
## 集成测试设计

### IntegrationTest 1: UserAuth + Form Validation
**场景**: 用户进行表单提交时的认证和验证流程

**测试步骤**:
1. 初始化 UserAuth 和 FormValidation 模块
2. 调用 validateForm()（T2）
3. 当验证通过时，调用 authenticate()（T1）
4. 验证返回值是否正确

**Mock策略**:
- Mock后端API调用
- 真实运行验证逻辑

**代码实例**:
```typescript
describe('Integration: UserAuth + FormValidation', () => {
  it('should authenticate after form validation passes', async () => {
    // 初始化模块
    const formValidator = new FormValidator();
    const authenticator = new Authenticator();

    // 验证表单
    const validationResult = formValidator.validate({ ... });
    expect(validationResult.valid).toBe(true);

    // 认证
    const authResult = await authenticator.authenticate({ ... });
    expect(authResult.success).toBe(true);
  });
});
```

### IntegrationTest 2: DataFetching + DataDisplay
...
```

#### 步骤4.4: E2E测试

测试完整的业务流程（从UI到API到数据库）：

分配 **E2E Test Designer** 创建E2E测试：

```markdown
## E2E测试设计

### E2ETest 1: 用户登录和仪表板加载
**场景**: 用户登录并查看个人仪表板

**测试步骤**:
1. 打开登录页面
2. 输入有效的用户名和密码
3. 点击登录按钮
4. 等待跳转到仪表板
5. 验证仪表板页面已加载和数据正确

**使用工具**: Cypress或Playwright

**代码示例**:
```
describe('E2E: User Login and Dashboard', () => {
  it('should login and load dashboard', () => {
    cy.visit('/login');
    cy.get('[data-testid="username"]').type('john');
    cy.get('[data-testid="password"]').type('pass123');
    cy.get('[data-testid="login-btn"]').click();

    cy.url().should('contain', '/dashboard');
    cy.get('[data-testid="dashboard"]').should('be.visible');
    cy.get('[data-testid="user-name"]').should('contain', 'John');
  });
});
```

### E2ETest 2: 表单提交和数据保存
...
```

#### 步骤4.5: 性能测试

建立性能基准和监控：

分配 **Performance Test Designer** 创建性能测试：

```markdown
## 性能测试设计

### 性能基准 1: 认证响应时间
**指标**: 认证端点的响应时间
**基准**: P95 < 500ms
**测试工具**: K6 或 Artillery

**测试脚本**:
```
export default function() {
  const authResponse = http.post('http://localhost:3000/auth/login', {
    username: 'test',
    password: 'pass123'
  });

  check(authResponse, {
    'status is 200': (r) => r.status === 200,
    'response time P95 < 500ms': (r) => r.timings.duration < 500,
  });
}
```

### 性能基准 2: 列表加载时间
**指标**: 加载1000条数据的时间
**基准**: < 2s
...
```

#### 步骤4.6: 闭环验证（关键！）

创建**验证矩阵**，确保规范TEST-VERIFY→代码→测试的完整对应：

**输出**: 闭环验证矩阵表格

```markdown
## 闭环验证矩阵

| 规范验收标准 | 来源 | Task | 代码实现 | 单元测试 | 集成测试 | E2E测试 | 性能测试 | 验证结果 |
|-----------|------|------|--------|---------|---------|---------|---------|---------|
| 应该支持用户名和密码认证 | Scenario1#1 | T1 | src/auth.ts:authenticate() | TC-1.1.1 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ 通过 |
| 应该验证用户名不为空 | Scenario1#2 | T1 | src/auth.ts:validateCredentials() | TC-1.1.2 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ 通过 |
| 应该验证密码长度 | Scenario2 | T1 | src/auth.ts:validateCredentials() | TC-1.2.1 ✅ | IntegrationTest-1 ✅ | E2ETest-1 ✅ | - | ✅ 通过 |
| 应该支持表单数据验证 | Scenario3 | T2 | src/form.ts:validateForm() | TC-2.1.1 ✅ | IntegrationTest-2 ✅ | E2ETest-2 ✅ | - | ✅ 通过 |
| 应该显示错误信息 | Scenario4 | T3 | src/components/ErrorDisplay.tsx | TC-3.1.1 ✅ | IntegrationTest-2 ✅ | E2ETest-2 ✅ | - | ✅ 通过 |
| 应该以< 500ms响应认证请求 | Scenario1#1 | T1 | src/auth.ts | - | - | - | PerfTest-1 ✅ | ✅ 通过 |
| 应该以< 2s加载数据列表 | Scenario5 | T4 | src/api.ts | - | - | E2ETest-3 ✅ | PerfTest-2 ✅ | ✅ 通过 |

## 验证覆盖率
- ✅ 规范TEST-VERIFY数量: 18
- ✅ 已验证的TEST-VERIFY: 18
- ✅ 验证覆盖率: 100%
- ✅ 所有规范都有对应的代码实现和测试验证
```

#### 步骤4.7: 测试报告生成

**输出文件**: `spec-dev/{requirement_desc_abstract}/testing/testing-report.md`

```markdown
# 测试报告

## 测试摘要
- 单元测试: 47个，全部通过 ✅
- 覆盖率: 87% (> 85%要求)
- 集成测试: 8个，全部通过 ✅
- E2E测试: 5个，全部通过 ✅
- 性能测试: 3个基准，全部达标 ✅
- 整体: **✅ 所有测试通过**

## 单元测试结果
... (由code-execute的TDD流程生成，code-test不重复)

## 集成测试结果
- IntegrationTest-1: 用户认证 + 表单验证 ✅ PASS
- IntegrationTest-2: 数据获取 + 数据显示 ✅ PASS
- ... (其他集成测试)

## E2E测试结果
- E2ETest-1: 用户登录和仪表板 ✅ PASS
- E2ETest-2: 表单提交和数据保存 ✅ PASS
- ... (其他E2E测试)

## 性能测试结果
- 认证响应时间: P95 = 380ms (< 500ms) ✅ PASS
- 数据加载时间: 1.8s (< 2s) ✅ PASS
- ...

## 闭环验证
**验证覆盖率**: 100% (18/18 规范TEST-VERIFY都有验证)

所有规范中的TEST-VERIFY都有对应的：
1. ✅ 代码实现（在code-execute中完成）
2. ✅ 单元测试（在code-execute中完成）
3. ✅ 集成/E2E/性能测试（在code-test中完成）
4. ✅ 测试结果验证（全部通过）

## 结论
✅ **所有测试通过**，代码质量达标，可以上线！

## 建议
1. 建立性能监控，定期检查P95响应时间
2. 添加更多边界情况的E2E测试
3. 考虑添加压力测试和耐久性测试
```

### 关键约束
- ✅ **不重复单元测试** - code-test只验证，不重复执行code-execute的单元测试
- ✅ **高层测试完整** - 集成、E2E、性能都要完成
- ✅ **闭环验证** - 确保TEST-VERIFY→代码→测试→结果的完全对应
- ✅ **完整追踪** - 验证矩阵要清晰展示追踪链

### 输出验收标准
- 所有代码质量检查通过（ESLint、TypeScript）
- 单元测试覆盖率≥85%
- 所有集成测试通过
- 所有E2E测试通过
- 所有性能基准达标
- 闭环验证覆盖率100%
- 测试报告完整

---

## Agent调用总结表

| Phase | Skill | Agent实例 | 职责 | 输入 | 输出 |
|-------|-------|----------|------|------|------|
| **Phase 0** | `/spec-creation` | 主Agent | 需求规范化，生成BDD规范 | 需求描述 | `spec-dev/{req}/spec/` |
| **Phase 1** | `/code-designer` | 主Agent | 设计规划，分析规范和设计架构 | spec/ | `spec-dev/{req}/design/design.md` |
| **Phase 2** | `/code-task` | 主Agent | 任务分解，将设计转为Task清单 | design.md | `spec-dev/{req}/tasks/tasks.md` |
| **Phase 3** | `/code-execute` | 主Agent | 任务管理、并行调度、审查协调 | tasks.md | 调度subagent |
| | | Implementer Agent（多个） | 编码实现、TDD流程（RED-GREEN-REFACTOR） | 单个Task | 代码+测试 |
| | | Spec-Reviewer Agent | 规范审查，验证设计一致性 | 代码 | 规范审查报告 |
| | | Code-Quality-Reviewer Agent | 质量审查，验证代码质量 | 代码 | 质量审查报告 |
| **Phase 4** | `/code-test` | 主Agent | 测试管理、报告生成 | src/ + tasks.md | 测试报告 |
| | | Integration-Test Designer | 集成测试设计和执行 | src/ | 集成测试代码+结果 |
| | | E2E-Test Designer | E2E测试设计和执行 | src/ | E2E测试代码+结果 |
| | | Performance-Test Designer | 性能测试设计和执行 | src/ | 性能测试代码+基准 |

---

## 关键概念说明

### TDD流程（RED-GREEN-REFACTOR-REVIEW）

```
┌─────────────────────────────────────────────────┐
│         RED 阶段：编写失败的单元测试             │
│  写一个失败的测试 → 运行测试 → 确保测试失败    │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│      GREEN 阶段：实现最少代码让测试通过         │
│   实现代码 → 运行测试 → 确保所有测试通过      │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│      REFACTOR 阶段：改进代码质量（测试通过前提）│
│   改进代码 → 运行测试 → 确保测试仍然通过      │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│           REVIEW 阶段：多阶段审查               │
│   1. 规范审查 → 2. 质量审查 → 修复循环         │
│   直到两次审查都通过 → Task完成                │
└─────────────────────────────────────────────────┘
```

### 闭环验证链

```
规范定义（Phase 0）
    ↓
TEST-VERIFY（规范中定义的可验证验收标准）
    ↓
Task定义（Phase 2）
    ↓
Test Case Mapping（Task到测试用例的映射）
    ↓
单元测试实现（Phase 3 RED阶段）
    ↓
代码实现（Phase 3 GREEN-REFACTOR阶段）
    ↓
单元测试验证（Phase 3 REVIEW阶段）
    ↓
集成/E2E/性能测试（Phase 4）
    ↓
闭环验证矩阵（验证所有TEST-VERIFY都通过）
    ↓
✅ 功能完成，代码质量达标，可以上线
```

---

## 总结

sdd-dev-plugin中的TDD完整工作流包括5个Phase：

| Phase | 主要目标 | Skill | 关键交付物 |
|-------|---------|-------|-----------|
| **Phase 0** | 规范定义 | /spec-creation | 规范文档 + TEST-VERIFY |
| **Phase 1** | 架构设计 | /code-designer | 设计方案文档 |
| **Phase 2** | 任务分解 | /code-task | 任务清单 + Test Case Mapping |
| **Phase 3** | TDD编码 | /code-execute | 代码 + 单元测试（覆盖≥85%） |
| **Phase 4** | 高层测试 | /code-test | 集成/E2E/性能测试 + 闭环验证 |

每个Phase都有明确的职责、工作流程、关键约束和输出验收标准。通过严格遵循这个完整的TDD工作流，确保代码质量和交付物的一致性。
