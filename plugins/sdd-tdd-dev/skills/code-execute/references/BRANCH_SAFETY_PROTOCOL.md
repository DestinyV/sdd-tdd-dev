# Git 分支安全协议

**版本**：v1.0（2026-04-08）

## 概述

本文档规定了在 code-execute 阶段涉及 git 操作时的分支安全检查协议。

**核心原则**：
- ✅ **创建 worktree 前** - 向用户确认基础分支
- ✅ **代码提交前** - 确认提交到正确的分支
- ✅ **合并到主分支前** - 再次确认合并分支和目标分支
- ✅ **完全可追踪** - 所有分支操作都有明确的用户确认记录
- ⚠️ **🆕 严禁同一需求跨分支提交** - 同一Task的所有改动必须提交到同一目标分支

---

## 核心约束：同一 Task 的改动必须提交到同一分支 ⚠️

**极其重要**：在创建 worktree 时选择的基础分支决定了该 Task 所有改动的最终目标分支。

### ❌ 禁止的操作

1. **分割提交到不同分支**
   ```bash
   Task 1 创建于 feature/A 分支
       → 提交部分改动到 feature/A  ✅
       → 提交部分改动到 master      ❌ 禁止！
   ```

2. **在 Worktree 中切换分支后提交**
   ```bash
   Task 1 创建于 feature/A，Worktree 中：
       $ git checkout master         ❌ 禁止！
       $ git commit "..."
   ```

3. **同一 Task 合并到多个分支**
   ```bash
   Task 1 创建于 feature/A
       → cherry-pick 到 master      ✅
       → cherry-pick 到 develop     ❌ 禁止！
   ```

4. **Task 改动跨越多个分支**
   ```bash
   Task 1 应该：
       基础分支 → worktree 改动 → 单一目标分支 ✅

   Task 1 不应该：
       基础分支A → worktree 改动 → 目标分支B 和 分支C ❌
   ```

### ✅ 正确的操作

1. **选定目标分支后一致执行**
   ```bash
   Task 1 创建于 feature/A 分支
       → 所有改动都提交到 feature/A
       → 所有改动都合并到 master（通过cherry-pick或squash）
       ✅ 一致的目标分支
   ```

2. **明确指定基础分支**
   ```bash
   Task 1 创建于 master 分支
       → 所有改动都提交到 master
       → Task 完成后，所有改动都在 master 上
       ✅ 一致的基础分支
   ```

3. **不同 Task 可以有不同分支**
   ```bash
   Task 1 创建于 feature/A（提交到feature/A，合并到master）
   Task 2 创建于 feature/B（提交到feature/B，合并到master）
   Task 3 创建于 master（提交到master）
       ✅ 不同Task各自确定自己的分支，但同Task内部一致
   ```

### 验证机制

在 worktree 中每次提交前，检查：

```bash
# 第1步：确认当前分支
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
WORKTREE_BASE=$(git rev-parse --abbrev-ref HEAD~N)  # 初始分支

# 第2步：验证分支未变
if [ "$CURRENT_BRANCH" != "$WORKTREE_BASE" ]; then
  echo "❌ 警告：当前分支 [$CURRENT_BRANCH] 与创建worktree时的分支 [$WORKTREE_BASE] 不一致"
  echo "禁止切换分支后继续提交"
  exit 1
fi

# 第3步：确认继续操作
# 使用 AskUserQuestion 确认
```

---

## 场景1：创建 Worktree 时的分支确认

### 流程

**第1步**：检测当前分支

```bash
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
```

**第2步**：使用 AskUserQuestion 向用户确认

```json
{
  "questions": [
    {
      "question": "请选择Worktree创建时的基础分支？",
      "header": "Worktree基础分支",
      "multiSelect": false,
      "options": [
        {
          "label": "当前分支（推荐）",
          "description": "feature/fix_sdd_dev_plugin_0320 - 包含当前分支的所有改动"
        },
        {
          "label": "Master分支",
          "description": "master - 基于生产稳定版本"
        },
        {
          "label": "其他分支",
          "description": "输入自定义分支名称"
        }
      ]
    }
  ]
}
```

**第3步**：根据用户选择创建 worktree

```bash
# 用户选择的分支
SELECTED_BRANCH="$USER_CHOICE"

# 创建 worktree
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}

# 记录到执行报告
echo "✅ Worktree 创建"
echo "   基础分支：$SELECTED_BRANCH"
echo "   路径：.claude/worktrees/{task-id}-{task-name}"
```

---

## 场景2：Worktree 内的代码提交时的分支确认 ⭐ 新增

### 问题背景

在 worktree 中工作时，开发者可能：
1. ❌ 误操作到错误的分支
2. ❌ 在 worktree 中修改了 HEAD（例如 git checkout 其他分支）
3. ❌ 不清楚当前所在分支，直接提交

### 解决方案

**在每次提交前进行分支确认**

#### 提交前检查清单

```bash
# 第1步：检查当前分支
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
echo "当前分支：$CURRENT_BRANCH"

# 第2步：检查是否在 worktree 中
WORKTREE_ROOT=$(git worktree list --porcelain | grep "$(pwd)")
if [ -z "$WORKTREE_ROOT" ]; then
  echo "⚠️  警告：当前不在 worktree 中"
fi

# 第3步：查看待提交的改动
git status
git diff --cached
```

#### 提交前向用户确认

在执行 `git commit` 前，使用 AskUserQuestion 向用户确认：

```json
{
  "questions": [
    {
      "question": "确认代码提交信息？",
      "header": "代码提交确认",
      "multiSelect": false,
      "options": [
        {
          "label": "确认提交",
          "description": "提交到当前分支：feature/fix_sdd_dev_plugin_0320 (worktree)"
        },
        {
          "label": "查看详情后确认",
          "description": "先查看详细的改动后再确认"
        },
        {
          "label": "取消提交",
          "description": "暂不提交，继续修改"
        }
      ]
    }
  ]
}
```

#### 提交流程

```bash
# 第1步：显示待提交的信息
echo "========== 提交信息 =========="
echo "Worktree：.claude/worktrees/T1-UserAuth"
echo "当前分支：feature/fix_sdd_dev_plugin_0320"
echo "提交文件数：$FILE_COUNT"
echo ""
echo "提交信息："
echo "$COMMIT_MESSAGE"
echo "=============================="

# 第2步：使用 AskUserQuestion 确认

# 第3步：根据用户选择执行
case "$USER_CHOICE" in
  "确认提交")
    git add .
    git commit -m "$COMMIT_MESSAGE"
    echo "✅ 提交成功"
    ;;
  "查看详情后确认")
    git diff --cached
    echo ""
    echo "按Enter确认提交或Ctrl+C取消"
    read
    git commit -m "$COMMIT_MESSAGE"
    echo "✅ 提交成功"
    ;;
  "取消提交")
    echo "⏸️  已取消提交"
    exit 0
    ;;
esac
```

---

## 场景3：Worktree 代码合并到主分支时的确认

### 问题背景

当 Task 完成，需要将 worktree 中的代码合并到主分支时，可能出现：
1. ❌ 合并到了错误的主分支（例如合并到 develop 而非 master）
2. ❌ 合并时发生冲突，不清楚如何处理
3. ❌ 合并后才发现有问题，难以回滚

### 解决方案

#### 合并前确认清单

```bash
# 第1步：验证当前状态
echo "当前位置：$(pwd)"
echo "当前分支：$(git rev-parse --abbrev-ref HEAD)"
echo ""

# 第2步：显示 worktree 中的 commit 历史
echo "Worktree commit 历史："
git log --oneline -5

# 第3步：回到主分支
cd [项目根目录]
echo ""
echo "主分支信息："
echo "当前分支：$(git rev-parse --abbrev-ref HEAD)"
git log --oneline -5

# 第4步：验证合并目标
echo ""
echo "合并策略：cherry-pick vs merge --squash"
```

#### 合并前向用户确认

```json
{
  "questions": [
    {
      "question": "请确认代码合并的信息？",
      "header": "代码合并确认",
      "multiSelect": false,
      "options": [
        {
          "label": "Cherry-pick（保留修复历史）",
          "description": "推荐。将worktree的所有commit逐个合并到master，便于审计"
        },
        {
          "label": "Squash Merge（简洁）",
          "description": "将所有worktree commit合并为一个，保持历史简洁"
        },
        {
          "label": "取消合并",
          "description": "暂不合并，继续修改或验证"
        }
      ]
    }
  ]
}
```

#### 合并执行流程

```bash
# 用户选择 Cherry-pick
if [ "$MERGE_STRATEGY" == "cherry-pick" ]; then
  echo "使用 Cherry-pick 策略合并..."

  # 获取 worktree 中的所有 commit
  git log --oneline feature/fix_sdd_dev_plugin_0320..HEAD

  # 逐个 cherry-pick
  for commit in $(git log --reverse --oneline master..worktree-branch | awk '{print $1}'); do
    echo "Cherry-picking: $commit"
    git cherry-pick $commit
    if [ $? -ne 0 ]; then
      echo "❌ Cherry-pick 失败，手动解决冲突后继续"
      exit 1
    fi
  done

  echo "✅ Cherry-pick 完成"
fi

# 用户选择 Squash Merge
if [ "$MERGE_STRATEGY" == "squash-merge" ]; then
  echo "使用 Squash Merge 策略合并..."

  git merge --squash .claude/worktrees/T1-UserAuth
  git commit -m "$CONSOLIDATED_MESSAGE"

  echo "✅ Squash Merge 完成"
fi
```

---

## 场景4：删除 Worktree 前的确认

### 问题背景

删除 worktree 前需要确认：
1. ❌ 代码是否已合并到主分支
2. ❌ 是否有未提交的改动
3. ❌ 是否有 worktree 特有的临时文件

### 解决方案

#### 删除前检查清单

```bash
# 第1步：进入 worktree
cd .claude/worktrees/T1-UserAuth

# 第2步：检查状态
echo "========== Worktree 状态检查 =========="

# 检查未提交的改动
if [ -n "$(git status --porcelain)" ]; then
  echo "⚠️  存在未提交的改动："
  git status
  echo ""
fi

# 检查是否有 worktree 独有的 commit
MAIN_COMMITS=$(cd [主目录] && git log --oneline master)
WORKTREE_COMMITS=$(git log --oneline)

echo "Worktree commit 总数：$(echo "$WORKTREE_COMMITS" | wc -l)"
echo "Main commit 总数：$(echo "$MAIN_COMMITS" | wc -l)"

# 检查临时文件
if [ -f "temp_*" ] || [ -f ".debug" ]; then
  echo "⚠️  存在临时文件"
  ls -la temp_* .debug 2>/dev/null
  echo ""
fi

echo "===================================="
cd [项目根目录]
```

#### 删除前向用户确认

```json
{
  "questions": [
    {
      "question": "确认删除Worktree？",
      "header": "Worktree删除确认",
      "multiSelect": false,
      "options": [
        {
          "label": "确认删除",
          "description": ".claude/worktrees/T1-UserAuth - 所有改动已合并，可安全删除"
        },
        {
          "label": "查看详情后确认",
          "description": "先检查worktree状态后再删除"
        },
        {
          "label": "取消删除",
          "description": "暂不删除，继续使用worktree"
        }
      ]
    }
  ]
}
```

#### 删除执行流程

```bash
if [ "$DELETE_CHOICE" == "确认删除" ]; then
  echo "✅ 删除 worktree..."

  # 从项目根目录删除
  git worktree remove .claude/worktrees/T1-UserAuth

  # 验证删除
  if git worktree list | grep -q "T1-UserAuth"; then
    echo "❌ 删除失败"
    exit 1
  else
    echo "✅ Worktree 已删除"
  fi
fi
```

---

## 完整的提交流程示例

### 场景：Task 1 在 worktree 中完成开发

```bash
# 1. 创建 worktree（已确认分支）
git worktree add .claude/worktrees/T1-UserAuth HEAD
cd .claude/worktrees/T1-UserAuth

# 2. 开发和测试
# ...编写代码...
# ...编写测试...

# 3. 提交前确认（新增 ⭐）
echo "========== 代码提交确认 =========="
git status
echo ""
echo "分支：$(git rev-parse --abbrev-ref HEAD)"
echo "文件数：$(git status --porcelain | wc -l)"
echo "=================================="

# 4. 使用 AskUserQuestion 确认提交
# → 用户选择："确认提交"

# 5. 执行提交
git add .
git commit -m "feat(T1): UserAuth - 初始实现

- 用户注册/登录
- Token管理
- 单元测试（覆盖率>80%）"

echo "✅ 提交成功"
echo "   分支：feature/fix_sdd_dev_plugin_0320"
echo "   Commit：$(git log --oneline -1)"

# 6. 回到项目根目录
cd [项目根目录]

# 7. 合并代码到主分支（已确认分支）
# → 使用 AskUserQuestion 确认合并策略
# → 用户选择："Cherry-pick"

git checkout master
git cherry-pick .claude/worktrees/T1-UserAuth

echo "✅ 合并成功"

# 8. 删除 worktree（已确认状态）
# → 使用 AskUserQuestion 确认删除
# → 用户选择："确认删除"

git worktree remove .claude/worktrees/T1-UserAuth

echo "✅ Worktree 已删除"
```

---

## 执行报告中的记录

### 完整的分支操作记录

```markdown
## Task 1: UserAuth - 分支操作记录

### Worktree 创建
- **基础分支**：feature/fix_sdd_dev_plugin_0320（用户确认 [A]）
- **Worktree 路径**：.claude/worktrees/T1-UserAuth
- **创建时间**：2026-04-08 14:30:00

### 代码提交
| 提交操作 | 分支 | 用户确认 | 结果 |
|---------|------|---------|------|
| 提交1 | feature/fix_sdd_dev_plugin_0320 | ✅ 确认提交 | 成功 |
| 提交2 | feature/fix_sdd_dev_plugin_0320 | ✅ 查看后确认 | 成功 |
| 提交3 | feature/fix_sdd_dev_plugin_0320 | ✅ 确认提交 | 成功 |

### Commit 信息
```
abc1234 feat(T1): UserAuth - 初始实现
def5678 fix(T1): 修复Props接口定义不一致
ghi9012 refactor(T1): 降低代码复杂度
```

### 代码合并
- **合并策略**：Cherry-pick（用户确认）
- **源 Worktree**：.claude/worktrees/T1-UserAuth
- **目标分支**：master
- **合并 Commit 数**：3
- **合并结果**：✅ 成功

### Worktree 清理
- **删除前确认**：✅ 确认删除
- **删除状态**：✅ 已删除
- **删除时间**：2026-04-08 14:45:00
```

---

## 分支安全检查清单

### 创建 Worktree 时
- [ ] 检测当前分支
- [ ] 使用 AskUserQuestion 向用户确认分支选择
- [ ] 验证选择的分支存在
- [ ] 创建 worktree
- [ ] 记录分支选择到执行报告

### 代码提交时（新增 ⭐）
- [ ] 检查当前分支
- [ ] 显示待提交的改动
- [ ] 使用 AskUserQuestion 向用户确认提交
- [ ] 验证提交结果
- [ ] 记录提交信息到执行报告

### 代码合并时
- [ ] 显示 worktree 中的 commit 历史
- [ ] 显示主分支的 commit 历史
- [ ] 使用 AskUserQuestion 向用户确认合并策略
- [ ] 执行合并操作
- [ ] 验证合并结果
- [ ] 记录合并信息到执行报告

### Worktree 删除时
- [ ] 检查 worktree 状态（未提交改动、临时文件）
- [ ] 验证代码已合并
- [ ] 使用 AskUserQuestion 向用户确认删除
- [ ] 删除 worktree
- [ ] 验证删除结果
- [ ] 记录删除时间到执行报告

---

## 禁止清单

### ❌ 分支操作相关

1. **不询问用户直接提交**
   ```bash
   # ❌ 错误
   git commit -m "..." && git push
   ```

2. **不验证分支直接合并**
   ```bash
   # ❌ 错误
   git merge worktree-branch
   ```

3. **在 worktree 中修改分支后直接操作**
   ```bash
   # ❌ 错误
   cd .claude/worktrees/T1
   git checkout other-branch
   git commit -m "..."  # 现在在错误的分支上
   ```

4. **不清理就删除 worktree**
   ```bash
   # ❌ 错误
   git worktree remove --force .claude/worktrees/T1
   ```

### ✅ 应该做的事

1. 每次分支操作前确认当前分支
2. 每次重要操作前使用 AskUserQuestion 确认
3. 完整记录所有分支操作
4. 验证操作结果

---

## 总结

通过在 git 操作的关键节点添加分支确认，确保：

| 操作 | 确认方式 | 目的 |
|------|---------|------|
| **创建 Worktree** | AskUserQuestion | 确认基础分支 |
| **代码提交** | AskUserQuestion | 确认提交分支 ⭐ |
| **代码合并** | AskUserQuestion | 确认合并策略 |
| **删除 Worktree** | AskUserQuestion | 确认删除状态 |

所有操作都有明确的用户确认记录，完全可追踪，避免分支错误。

**最后更新**：2026-04-08
**版本**：v1.0（初版）
