# Git-Worktrees 快速参考卡

## 🚀 快速开始（3分钟）

### 为Task创建worktree
```bash
# 推荐：使用HEAD（自动适配任何分支名）
git worktree add .claude/worktrees/{task-id}-{name} HEAD
cd .claude/worktrees/{task-id}-{name}
```

### 编码和提交
```bash
# 编码、测试...
git add .
git commit -m "feat({task-id}): {Task名称} - 初始实现"
```

### 规范审查❌ → 修复 → 重新审查✅
```bash
# 修复问题
git add .
git commit -m "fix({task-id}): 修复[问题描述]"
```

### 质量审查❌ → 修复 → 重新审查✅
```bash
# 修复问题
git add .
git commit -m "refactor({task-id}): [优化描述]"
```

### 提交合并（cherry-pick推荐）
```bash
cd [项目根目录]
git checkout main
git cherry-pick .claude/worktrees/{task-id}-{name}
```

### 清理worktree
```bash
git worktree remove .claude/worktrees/{task-id}-{name}
```

---

## 📋 关键数字

| 项目 | 数值 |
|------|------|
| Worktree路径 | `.claude/worktrees/{task-id}-{name}` |
| 修复失败时重做 | 删除worktree + 从main重新创建 |
| 支持的并行Task数 | 无限制（隔离设计） |
| 单个Task的修复次数 | 不限（问题→修复→验证循环） |
| 提交方案 | cherry-pick（推荐）或 squash merge |

---

## ⚠️ 记住这5点

1. **每个Task一个worktree** - 不要多Task共享一个worktree
2. **每次修复提交一次** - fix commit / refactor commit都要提交
3. **Task完成后必清理** - git worktree remove以释放资源
4. **修复失败可重来** - 删除worktree，从main重新开始
5. **保存修复历史** - 使用cherry-pick（不是squash）便于审计

---

## 🔍 检查清单（Task完成前）

- [ ] Worktree中的所有修复都已提交
- [ ] 规范审查已通过（✅）
- [ ] 质量审查已通过（✅）
- [ ] **🆕 代码完整性检查已通过**（无框架代码、无TODO注释）
  - [ ] 没有只有注释的样式块（`<style>` 只有 `//` 注释）
  - [ ] 没有只有 `// TODO` 的函数体
  - [ ] 没有空的事件处理器或回调函数
  - [ ] 所有条件分支都有实现
  - [ ] 没有被注释掉的代码块
- [ ] 所有TODO项已澄清并实现
- [ ] 修复commit已cherry-pick到main
- [ ] Worktree已清理

---

## 🆘 遇到问题？

| 问题 | 解决方案 |
|------|---------|
| worktree中的修改如何同步到main | cherry-pick到main（不要直接push） |
| 多Task冲突 | 不会发生（独立worktree） |
| 修复失败了 | 删除worktree，从main重新开始 |
| 不小心删除worktree | 使用git reflog恢复（如未合并） |
| Task间需要共享代码 | 说明Task分解有问题，重新评估 |

---

## 🆕 代码完整性检查（快速版）

在质量审查阶段，检查是否有**框架代码被遗留**：

### 禁止的代码模式 ❌

```vue
<!-- ❌ 样式块只有注释 -->
<style lang="less" scoped>
  // 样式规则遵循现有审核项的风格
</style>

<!-- ❌ 函数只有TODO -->
<script>
export default {
  methods: {
    handleSubmit() {
      // TODO: 实现表单提交
    }
  }
}
</script>

<!-- ❌ 事件处理器是空的 -->
<template>
  <button @click="onClick">提交</button>
</template>

<script>
const onClick = () => {}; // 空函数
</script>
```

### 正确的做法 ✅

```vue
<!-- ✅ 完整的样式实现 -->
<style lang="less" scoped>
  .form { padding: 16px; }
  .input { border: 1px solid #ccc; }
</style>

<!-- ✅ 或根本不写样式块 -->

<!-- ✅ 函数有完整实现 -->
<script>
export default {
  methods: {
    async handleSubmit() {
      const result = await submitForm();
      // 完整的实现...
    }
  }
}
</script>

<!-- ✅ 事件处理器有完整实现 -->
<script>
const onClick = () => {
  console.log('Button clicked');
  // 具体的逻辑...
};
</script>
```

**详细检查清单**：查看 `code-completeness-checklist.md`

---
