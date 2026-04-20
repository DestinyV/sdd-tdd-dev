# Git-Worktrees指南（code-execute改进）

## 概述

本指南说明如何在code-execute阶段使用git-worktrees创建隔离的Task工作环境，确保修复循环的安全性、可追踪性和可恢复性。

**版本**：v2.2（2026-03-23）
**更新内容**：为code-execute执行环节添加git-worktrees能力

---

## 为什么需要git-worktrees？

### 问题场景

在多阶段审查的修复循环中，可能出现以下问题：

1. **并行Task冲突** - 多个Task同时修改相同文件导致git冲突
2. **版本混乱** - 多轮修复后，难以追踪"哪个commit是什么修复"
3. **修复失败回滚困难** - 修复出现问题，难以恢复到修复前状态
4. **审计追踪困难** - 看不清"问题→修复→验证"的完整链条

### 解决方案

使用git-worktrees为每个Task创建独立分支：

```
main分支（稳定）
    │
    ├─ worktree: T1-UserAuth
    │  ├─ 初始实现commit
    │  ├─ 修复commit 1（规范审查发现的问题）
    │  ├─ 修复commit 2（质量审查发现的问题）
    │  └─ cherry-pick到main ✅
    │
    ├─ worktree: T2-FormValidation
    │  ├─ 初始实现commit
    │  └─ cherry-pick到main ✅
    │
    └─ worktree: T3-ErrorHandling
       ├─ 初始实现commit
       ├─ 修复commit 1（失败，需要重做）
       └─ 删除worktree，从main重新开始 ↻
```

---

## Worktree工作流

### 1. 创建Worktree

#### 📌 执行前必须确认：选择基础分支

**极其重要**：在创建任何worktree之前，必须和用户明确确认从哪个分支生成worktree。

**使用工具**：`AskUserQuestion` 工具（统一的用户交互方式）

##### 确认流程（必须执行）

使用 AskUserQuestion 工具向用户显示以下选项：

```
question: "请选择Worktree创建时的基础分支？"
header: "Worktree基础分支"
multiSelect: false
options:
  - label: "当前分支（推荐）"
    description: "feature/fix_sdd_dev_plugin_0320 - 包含当前分支的所有改动"
  - label: "Master分支"
    description: "master - 基于生产稳定版本"
  - label: "Develop分支"
    description: "develop - 基于最新开发版本"
  - label: "其他分支"
    description: "输入自定义分支名称"
```

##### 选择方式说明

| 选择 | 分支 | 优点 | 使用场景 |
|------|------|------|---------|
| **[A] 当前分支** | feature/fix... | 包含当前改动、无需硬编码 | ✅ 修复当前特性分支 |
| **[B] Master** | master | 生产稳定版本 | 独立特性、bug修复 |
| **[C] Develop** | develop | 最新开发版本 | 基于最新进展开发 |
| **[D] 其他分支** | 自定义 | 灵活指定 | 指定特定分支 |

#### 命令

```bash
# 方案A：基于当前分支（推荐 ✅）
# 确认后执行（自动适配任何分支名）
git worktree add .claude/worktrees/{task-id}-{task-name} HEAD
cd .claude/worktrees/{task-id}-{task-name}

# 方案B：基于指定分支（需明确确认）
# 用户确认后执行
git worktree add .claude/worktrees/{task-id}-{task-name} master
cd .claude/worktrees/{task-id}-{task-name}

# 方案C：基于其他分支（需明确确认）
git worktree add .claude/worktrees/{task-id}-{task-name} {branch-name}
cd .claude/worktrees/{task-id}-{task-name}
```

#### 规范

- **路径**：`.claude/worktrees/{task-id}-{task-name}`
- **task-id**：来自tasks.md（如T1、T2、T3）
- **task-name**：Task的短名称（如UserAuth、FormValidation）
- **基础分支**：✅ **执行前必须向用户确认**（不要假设分支名）
  - 推荐：当前分支（HEAD）- 自动适配任何分支名
  - 备选：明确指定 master/main/develop 等
  - **禁止**：直接假设 master，不询问用户
- **⚠️ 重要**：
  - 使用 HEAD 而不是硬编码分支名，以避免分支不存在导致 fatal: Not a valid object name 的问题
  - 从当前所在的分支生成 worktree，不要默认从 master 生成
  - **兜底方案**：如果用户未明确选择，从当前分支（HEAD）生成

#### 示例

```bash
# ✅ 正确：执行前询问用户

# 场景1：用户选择 [A] 当前分支（feature/xxx）
当前分支：feature/fix_sdd_dev_plugin_0320
$ git worktree add .claude/worktrees/T1-UserAuth HEAD
$ cd .claude/worktrees/T1-UserAuth

# 场景2：用户选择 [B] Master分支
当前分支：feature/fix_sdd_dev_plugin_0320
$ git worktree add .claude/worktrees/T1-UserAuth master
$ cd .claude/worktrees/T1-UserAuth

# 场景3：用户未明确选择（兜底）→ 使用当前分支
当前分支：feature/fix_sdd_dev_plugin_0320
$ git worktree add .claude/worktrees/T1-UserAuth HEAD  # 兜底使用HEAD
$ cd .claude/worktrees/T1-UserAuth

# ❌ 错误：不询问用户，直接从master生成
$ git worktree add .claude/worktrees/T1-UserAuth master  # 假设用户想要master
```

### 2. 编码和提交

#### 初始实现
```bash
cd .claude/worktrees/T1-UserAuth

# 编码实现...
# 编写测试...
# 自审查...

# 提交初始实现
git add .
git commit -m "feat(T1): UserAuth - 初始实现

- 用户注册/登录
- token管理
- 单元测试（覆盖率>80%）"
```

#### 修复提交（规范审查）
```bash
# 规范审查发现问题：Props接口不一致

# 修复问题
# 编辑代码...
# 重新测试...

# 提交修复
git commit -m "fix(T1): 修复Props接口定义不一致

- 调整Props.onSuccess返回类型为Promise<void>
- 更新组件使用方式以匹配新接口
- 重新测试确保正常工作"
```

#### 修复提交（质量审查）
```bash
# 质量审查发现问题：代码复杂度过高

# 修复问题
# 提取公共逻辑...

# 提交修复
git commit -m "refactor(T1): 降低代码复杂度

- 提取validateToken为独立函数
- 提取refreshToken逻辑为hook
- 减少validateUser中的嵌套层级"
```

### 3. 提交合并

当Task的所有审查都通过后，将worktree中的修复合并到main分支。

#### 方案A：保留修复历史（推荐）

**适用场景**：复杂Task（多次修复）、需要审计追踪

```bash
# 查看修复历史
cd .claude/worktrees/T1-UserAuth
git log --oneline

# 输出示例：
# abc1234 fix(T1): 修复Props接口定义不一致
# def5678 refactor(T1): 降低代码复杂度
# ghi9012 feat(T1): UserAuth - 初始实现
```

```bash
# 回到main分支
cd [项目根目录]
git checkout main

# cherry-pick所有修复（从oldest到newest）
git cherry-pick ghi9012  # 初始实现
git cherry-pick def5678  # 修复1
git cherry-pick abc1234  # 修复2

# 验证提交
git log --oneline | head -5
```

**优点**：
- ✅ 保留完整的修复历史
- ✅ 便于code review和问题追踪
- ✅ 清晰的commit message说明修复内容

#### 方案B：Squash为单个clean commit（简洁）

**适用场景**：简单Task（无修复或修复简单）、代码库希望保持简洁

```bash
# 进入worktree
cd .claude/worktrees/T1-UserAuth

# 查看修复历史（了解修复过程）
git log --oneline

# 回到main分支
cd [项目根目录]
git checkout main

# Squash merge（所有worktree commit合并为1个）
git merge --squash .claude/worktrees/T1-UserAuth

# 创建consolidated commit
git commit -m "feat(T1): UserAuth完整实现

## 实现内容
- 用户注册/登录功能
- Token管理和刷新机制
- 单元测试（覆盖率>80%）

## 修复过程
- 修复1：Props接口定义不一致 ✅
- 修复2：降低validateUser代码复杂度 ✅

## 审查结果
- 规范审查：✅ 通过
- 质量审查：✅ 通过"
```

**优点**：
- ✅ Git历史简洁
- ✅ 一个commit代表一个Task的完整工作
- ✅ commit message中仍记录修复过程

### 4. 清理Worktree

Task完成后，删除worktree释放资源：

```bash
# 删除worktree
git worktree remove .claude/worktrees/T1-UserAuth

# 验证清理成功
git worktree list

# 输出示例：
# /path/to/repo/.claude/worktrees/T2-FormValidation  abc123
# /path/to/repo/.claude/worktrees/T3-ErrorHandling   def456
# /path/to/repo                                        (bare)
```

---

## 修复失败处理

### 场景：修复失败，需要重新开始

```bash
# 当前worktree中的修复有问题
cd .claude/worktrees/T1-UserAuth

# 查看当前状态
git status
git log --oneline

# 决定：需要重新开始（而不是继续修复）
cd [项目根目录]

# 删除worktree（包含所有失败的commit）
git worktree remove .claude/worktrees/T1-UserAuth --force

# 验证删除
git worktree list

# 从当前分支重新创建worktree
git worktree add .claude/worktrees/T1-UserAuth HEAD

# 重新开始实现
cd .claude/worktrees/T1-UserAuth
# 重新编码...
```

**优点**：
- ✅ 失败的修改不会污染main分支
- ✅ 可以安全地从头开始
- ✅ 不影响其他Task的worktree

---

## 并行Task管理

### 场景：多Task同时执行

```
main分支
  │
  ├─ worktree T1（在执行中...）
  │  └─ 修复循环中...
  │
  ├─ worktree T2（等待前置Task）
  │  └─ 准备工作中...
  │
  └─ worktree T3（规范审查中...）
     └─ 等待审查结果...
```

### 关键原则

1. **隔离性** - 每个worktree独立，互不影响
2. **不交叉** - 不在不同worktree间移动代码
3. **顺序合并** - Task完成后依次cherry-pick到main
4. **避免冲突** - 不同Task修改不同文件，worktree避免git冲突

### 最佳实践

```bash
# ✅ 正确：为每个Task创建独立worktree（使用HEAD）
git worktree add .claude/worktrees/T1-UserAuth HEAD
git worktree add .claude/worktrees/T2-FormValidation HEAD
git worktree add .claude/worktrees/T3-ErrorHandling HEAD

# ✅ 正确：在各自的worktree中独立工作
cd .claude/worktrees/T1-UserAuth
# 编码、测试、修复...

# ✅ 正确：完成后逐次合并到main
git checkout main
git cherry-pick .claude/worktrees/T1-UserAuth

# ❌ 错误：从另一个worktree创建worktree
git worktree add .claude/worktrees/T2 T1  # 不要这样做！

# ❌ 错误：多个worktree间相互复制代码
cp .claude/worktrees/T1/src/auth.ts .claude/worktrees/T2/
```

---

## 执行报告中的Worktree信息

### 记录模板

每个Task在执行报告中应记录以下worktree信息：

```markdown
### Task 1: UserAuth

- 状态：✅ 完成
- **Worktree**：.claude/worktrees/T1-UserAuth
- 迭代：2次（规范+质量审查各发现1个问题，已修复）
- 审查结果：
  - 规范审查：❌ 问题 → ✅ 修复后通过
    - 问题：Props接口定义不一致
    - 修复commit：abc1234 "fix(T1): 修复Props接口定义..."
  - 质量审查：❌ 问题 → ✅ 修复后通过
    - 问题：validateUser代码复杂度过高
    - 修复commit：def5678 "refactor(T1): 降低代码复杂度..."
- **提交信息**：
  - 初始实现：ghi9012 "feat(T1): UserAuth - 初始实现"
  - 修复commit列表：
    1. abc1234 - fix(T1): 修复Props接口定义不一致
    2. def5678 - refactor(T1): 降低代码复杂度
  - 合并方案：cherry-pick（保留修复历史便于审计）
  - 清理状态：✅ 已清理
```

### 修复历史说明

在执行报告中清晰说明修复过程：

```markdown
## 修复追踪

| Task | 审查阶段 | 发现问题 | 修复commit | 验证结果 |
|------|---------|---------|-----------|---------|
| T1 | 规范 | Props接口不一致 | abc1234 | ✅ 通过 |
| T1 | 质量 | 代码复杂度高 | def5678 | ✅ 通过 |
| T2 | 规范 | 无 | - | ✅ 通过 |
| T3 | 规范 | 缺少错误处理 | xyz7890 | ✅ 通过 |
| T3 | 质量 | TypeScript any类型 | mno3456 | ✅ 通过 |

**总结**：
- 总修复次数：4次
- 规范审查修复：2次
- 质量审查修复：2次
- 修复成功率：100%（无需重做的修复）
```

---

## 故障排查

### 常见问题

#### Q1：worktree中的修改如何同步到main？

```bash
# ❌ 错误：直接在worktree中push
cd .claude/worktrees/T1-UserAuth
git push  # 这会推送到远程worktree分支

# ✅ 正确：cherry-pick到main后再push
cd [项目根目录]
git checkout main
git cherry-pick .claude/worktrees/T1-UserAuth
git push
```

#### Q2：worktree中的代码与其他Task冲突怎么办？

```bash
# 冲突通常来自不同worktree修改同一文件
# 解决方案：确保Task分解清晰，各Task修改不同文件

# 如果真的有冲突，可以在cherry-pick时解决
git cherry-pick .claude/worktrees/T1-UserAuth
# 解决冲突...
git add .
git cherry-pick --continue
```

#### Q3：误删了worktree怎么办？

```bash
# worktree删除后，分支仍在reflog中
# 如果worktree尚未提交到main，可以恢复

# 列出所有reflog
git reflog

# 找到worktree最后的commit
# 然后cherry-pick或merge

# 警告：只有在worktree删除前的commit可以恢复
# 因此重要的是定期cherry-pick到main
```

#### Q4：如何在worktree间共享代码？

```bash
# ❌ 错误：不应该有Task间的代码依赖
# 这说明Task分解有问题

# ✅ 正确：通过当前分支共享
# T1先完成 → cherry-pick到分支
# T2开始 → 拉取最新分支，包含T1的代码

git worktree add .claude/worktrees/T2-FormValidation HEAD  # 基于当前分支
```

---

## 检查清单

### 创建Worktree时
- [ ] 从main创建（不从其他worktree）
- [ ] 命名格式正确：.claude/worktrees/{task-id}-{name}
- [ ] cd到worktree目录准备工作

### 编码时
- [ ] 所有修改都在该worktree中进行
- [ ] 不跨worktree复制代码
- [ ] 每次修复作为独立commit

### 提交合并时
- [ ] 查看修复历史（git log --oneline）
- [ ] 决定使用cherry-pick还是squash merge
- [ ] cherry-pick：从oldest到newest依次合并
- [ ] squash：使用merge --squash并创建consolidated commit
- [ ] 验证合并结果（git log查看main中的新commit）

### 清理时
- [ ] 确认commit已合并到main
- [ ] 删除worktree：git worktree remove
- [ ] 验证清理成功：git worktree list

---

## 总结

git-worktrees在code-execute中的核心优势：

| 特性 | 优势 |
|------|------|
| **隔离性** | 每个Task独立工作，互不污染 |
| **可恢复性** | 修复失败可删除worktree重新开始 |
| **可追踪性** | worktree commit历史清晰记录修复过程 |
| **审计性** | 便于问题溯源和代码审查 |
| **并行安全** | 多Task同时执行不产生git冲突 |

通过规范使用git-worktrees，code-execute的修复循环变得更加安全、可控、可追踪。

---

**最后更新**：2026-03-23
**版本**：v1.0（初版）
