# Worktree 创建确认协议

**版本**：v1.1（2026-04-08）

## 概述

本文档规定了在 code-execute 阶段创建 git worktree 时必须遵循的确认协议。

**核心原则**：
- ✅ **执行前必须向用户确认** - 不要假设任何分支名
- ✅ **使用 AskUserQuestion 工具** - 统一的用户交互方式
- ✅ **避免直接从 master 生成** - 除非用户明确选择
- ✅ **提供多个分支选项** - 当前分支/master/其他分支
- ✅ **兜底逻辑完善** - 用户未选择时自动使用当前分支

---

## 用户确认流程

### 第1步：检测当前分支信息

在执行任何 worktree 创建命令前，**必须**先检测并显示当前分支信息：

```bash
# 获取当前分支名
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

# 获取所有本地分支列表
git branch -a

# 示例输出
# feature/fix_sdd_dev_plugin_0320
# master
# develop
```

### 第2步：使用 AskUserQuestion 工具向用户展示选项

**重要**：使用 Claude Code 的 `AskUserQuestion` 工具与用户交互，**不要**使用其他方式（如 shell 交互、提示文本等）。

#### 实现方式

调用 AskUserQuestion 工具，参数如下：

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
          "description": "feature/fix_sdd_dev_plugin_0320 - 包含当前分支的所有改动，适合修复当前特性"
        },
        {
          "label": "Master分支",
          "description": "master - 基于生产稳定版本，适合独立特性或bug修复"
        },
        {
          "label": "Develop分支",
          "description": "develop - 基于最新开发版本，适合基于最新开发进展"
        },
        {
          "label": "其他分支",
          "description": "输入自定义分支名称（如：release/v1.0）"
        }
      ]
    }
  ]
}
```

#### 用户界面示例

```
📋 Worktree基础分支

请选择Worktree创建时的基础分支？

[A] 当前分支（推荐）
    feature/fix_sdd_dev_plugin_0320
    包含当前分支的所有改动，适合修复当前特性

[B] Master分支
    master
    基于生产稳定版本，适合独立特性或bug修复

[C] Develop分支
    develop
    基于最新开发版本，适合基于最新开发进展

[D] 其他分支
    输入自定义分支名称（如：release/v1.0）
```

### 第3步：根据用户选择执行

#### 场景 A：用户选择"当前分支（推荐）"

```bash
SELECTED_BRANCH="HEAD"
echo "✅ 基于当前分支创建worktree：$CURRENT_BRANCH"
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}
```

**优点**：
- ✅ 自动适配当前分支名，无需硬编码
- ✅ 包含当前分支的所有改动
- ✅ 兼容所有分支命名约定

#### 场景 B：用户选择"Master分支"

```bash
# 首先验证分支存在
if ! git rev-parse master >/dev/null 2>&1; then
  echo "❌ 错误：master分支不存在"
  echo ""
  echo "可用分支："
  git branch -a
  exit 1
fi

SELECTED_BRANCH="master"
echo "✅ 基于master分支创建worktree"
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}
```

#### 场景 C：用户选择"Develop分支"

```bash
# 首先验证分支存在
if ! git rev-parse develop >/dev/null 2>&1; then
  echo "❌ 错误：develop分支不存在"
  echo ""
  echo "可用分支："
  git branch -a
  exit 1
fi

SELECTED_BRANCH="develop"
echo "✅ 基于develop分支创建worktree"
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}
```

#### 场景 D：用户选择"其他分支"

```bash
# 通过 AskUserQuestion 的 "Other" 选项获取自定义输入
# 或通过额外的交互获取分支名

USER_BRANCH={user_custom_input}

# 验证分支存在
if ! git rev-parse "$USER_BRANCH" >/dev/null 2>&1; then
  echo "❌ 错误：分支 '$USER_BRANCH' 不存在"
  echo ""
  echo "可用分支："
  git branch -a
  exit 1
fi

SELECTED_BRANCH="$USER_BRANCH"
echo "✅ 基于 $SELECTED_BRANCH 分支创建worktree"
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}
```

#### 兜底场景：用户未明确选择

如果用户没有响应或关闭对话框，**自动使用当前分支（HEAD）**：

```bash
# 默认使用HEAD（当前分支）
SELECTED_BRANCH="HEAD"
echo "ℹ️  使用默认选项：当前分支"
git worktree add .claude/worktrees/{task-id}-{task-name} $SELECTED_BRANCH
cd .claude/worktrees/{task-id}-{task-name}
```

---

## 禁止清单

### ❌ 不要做的事

1. **直接假设 master 分支，不询问用户**
   ```bash
   # ❌ 错误
   git worktree add .claude/worktrees/T1 master
   ```

2. **硬编码分支名而不验证**
   ```bash
   # ❌ 错误 - 如果分支不存在会失败
   git worktree add .claude/worktrees/T1 main
   ```

3. **不显示当前分支信息就创建**
   ```bash
   # ❌ 错误 - 用户不知道从哪个分支创建
   git worktree add .claude/worktrees/T1 HEAD
   ```

4. **使用 shell 交互而不是 AskUserQuestion**
   ```bash
   # ❌ 错误 - 不统一，体验差
   read -p "Choose branch [A/B/C]: " choice
   ```

5. **创建后才告诉用户分支信息**
   ```bash
   # ❌ 错误 - 太晚了，用户无法更改
   git worktree add .claude/worktrees/T1 some-branch
   echo "已从 some-branch 创建worktree"
   ```

### ✅ 应该做的事

1. **执行前检测当前分支**
2. **使用 AskUserQuestion 向用户确认**
3. **提供多个选择选项**
4. **验证选择的分支存在**
5. **创建后确认信息输出**

---

## 多Task场景处理

### 第一个Task：完整确认流程

```
创建 Task 1 时：
1. 检测当前分支：feature/fix_sdd_dev_plugin_0320
2. 使用 AskUserQuestion 向用户确认
3. 用户选择：[A] 当前分支
4. 创建 worktree：T1-UserAuth
```

### 后续Task：继续使用相同分支

```
创建 Task 2 时的选择：

[选项1] 继续使用上一次选择的分支
选择：上一次选择了"当前分支"
结果：T2 也基于当前分支（feature/fix_sdd_dev_plugin_0320）

[选项2] 询问用户是否修改
- "上次为Task 1选择了：当前分支"
- "Task 2是否使用相同分支？[Y/n]"

[推荐] 选项1 - 保持一致性
所有Task都基于同一分支创建，避免频繁确认
```

---

## 实现检查清单

### 创建 Worktree 时

- [ ] 检测当前分支名（git rev-parse --abbrev-ref HEAD）
- [ ] 调用 AskUserQuestion 工具向用户确认分支选择
- [ ] 等待用户选择（或使用默认值）
- [ ] 验证选择的分支存在（git rev-parse $branch）
- [ ] 创建 worktree（使用用户选择的分支）
- [ ] 显示创建成功的确认信息
- [ ] 记录到执行报告中

### 执行报告中

- [ ] 记录 worktree 基础分支（用户选择的分支）
- [ ] 记录分支选择理由
- [ ] 记录 worktree 路径

**报告示例**：
```markdown
### Task 1: UserAuth

- **Worktree**：.claude/worktrees/T1-UserAuth
- **基础分支**：feature/fix_sdd_dev_plugin_0320
- **分支选择理由**：用户选择 [A] 当前分支，包含特性分支的最新改动
- 迭代：2次（规范+质量审查各发现1个问题，已修复）
- 审查结果：✅ 通过
```

---

## 常见问题

### Q1：如果用户没有选择怎么办？

**A**：使用兜底逻辑，自动选择当前分支（HEAD）。在日志中提示：

```
ℹ️  未检测到用户选择，使用默认选项：当前分支
```

### Q2：如果选择的分支不存在怎么办？

**A**：进行验证并提示错误，显示可用分支列表：

```bash
if ! git rev-parse "$BRANCH_NAME" >/dev/null 2>&1; then
  echo "❌ 分支 '$BRANCH_NAME' 不存在"
  echo ""
  echo "可用的分支："
  git branch -a
  exit 1
fi
```

### Q3：为什么不直接使用 master？

**A**：因为：
1. 分支名在不同项目中可能不同（main vs master）
2. 不同的工作流可能需要不同的基础分支
3. 用户可能在特性分支上工作，希望基于当前分支生成 worktree
4. 尊重用户意愿，而不是做假设

### Q4：如果有多个 Task，每个都要询问吗？

**A**：不需要。只需在第一个 Task 时询问一次，后续 Task 继续使用相同的分支（避免频繁打扰）。

---

## 最佳实践

### 清晰的执行流程

```
检测到当前分支：feature/fix_sdd_dev_plugin_0320

🔄 Step 1: 为Task 1 (UserAuth) 创建worktree
   使用 AskUserQuestion 工具向用户确认...
   → 用户选择：[A] 当前分支

✅ Task 1: 创建 worktree
   路径：.claude/worktrees/T1-UserAuth
   基础分支：feature/fix_sdd_dev_plugin_0320
   命令：git worktree add .claude/worktrees/T1-UserAuth HEAD

🔄 Step 2: 为Task 2 (FormValidation) 创建worktree
   继续使用上次选择的分支：feature/fix_sdd_dev_plugin_0320

✅ Task 2: 创建 worktree
   路径：.claude/worktrees/T2-FormValidation
   基础分支：feature/fix_sdd_dev_plugin_0320
```

### 完整的执行记录

```markdown
## Worktree 创建记录

| Task | Worktree路径 | 基础分支 | 选择方式 | 状态 |
|------|-------------|---------|---------|------|
| T1 | .claude/worktrees/T1-UserAuth | feature/fix_sdd_dev_plugin_0320 | 用户确认 | ✅ |
| T2 | .claude/worktrees/T2-FormValidation | feature/fix_sdd_dev_plugin_0320 | 继续使用 | ✅ |
| T3 | .claude/worktrees/T3-ErrorHandling | feature/fix_sdd_dev_plugin_0320 | 继续使用 | ✅ |

**分支选择总结**：
- 初始确认：用户选择 [A] 当前分支
- 基础分支：feature/fix_sdd_dev_plugin_0320
- 所有Task都基于同一分支创建，保持一致性
- 用户交互方式：AskUserQuestion 工具
```

---

## 总结

通过严格执行本确认协议，code-execute 的并行 worktree 创建变得：

| 特性 | 改进 |
|------|------|
| **安全性** | 用户通过 AskUserQuestion 明确确认，避免错误的分支选择 |
| **一致性** | 使用统一的 AskUserQuestion 工具，体验一致 |
| **可读性** | 清晰的分支选择选项和说明 |
| **可追踪性** | 执行报告中记录分支选择和原因 |
| **灵活性** | 支持多种分支策略和工作流 |
| **鲁棒性** | 完善的兜底逻辑，处理边界情况 |

**最后更新**：2026-04-08
**版本**：v1.1（完善 AskUserQuestion 使用）
