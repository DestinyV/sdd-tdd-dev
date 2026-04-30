# 组件抽离指导

> code-designer 阶段组件设计决策和 code-execute 阶段实现参考

---

## 何时拆分组件

### 1. 单一职责原则 (SRP)

**规则**：一个组件只应该有一个变化的理由。

```vue
<!-- ❌ 错误：一个组件做了太多事情 -->
<script setup>
// 用户管理组件：包含搜索、表格、分页、编辑、删除、导出
</script>

<!-- ✅ 正确：拆分为职责单一的组件 -->
<UserSearch />
<UserTable :data="users" />
<Pagination :total="total" @change="fetchUsers" />
<UserEditDialog />
```

### 2. 复用性

**规则**：如果一段 UI 逻辑在多处使用，提取为独立组件。

```vue
<!-- 可复用的组件示例 -->
<BaseButton />       <!-- 在表单、列表、弹窗中多处使用 -->
<BaseInput />        <!-- 所有表单输入框 -->
<BaseTable />        <!-- 所有数据展示表格 -->
<BaseDialog />       <!-- 所有弹窗 -->
```

### 3. 复杂度阈值

**规则**：组件超过以下任一阈值时应考虑拆分：
- Template 部分超过 150 行
- Script 部分超过 200 行
- Props 超过 10 个
- Emits 超过 8 个
- 嵌套条件超过 3 层

---

## 组件粒度原则

### 原子设计原则

```
Atoms (原子)     → Button, Input, Icon
  ↓
Molecules (分子)  → SearchBar (Input + Button), FormField (Label + Input)
  ↓
Organisms (组织)  → Header (Logo + Nav + SearchBar)
  ↓
Templates (模板)  → PageLayout (Header + Sidebar + Content + Footer)
  ↓
Pages (页面)      → HomePage, DashboardPage
```

**实践指南**：
- Atoms：无业务逻辑，纯展示，可独立测试
- Molecules：组合 2-3 个 Atoms，简单交互
- Organisms：完整的功能区块，可能有数据获取
- Templates：页面布局，不包含业务数据
- Pages：包含具体业务数据的完整页面

---

## Props 设计原则

### 1. 明确类型和约束

```typescript
// ❌ 错误：类型不明确
interface Props {
  data: any;
  options?: object;
  config?: any;
}

// ✅ 正确：精确类型
interface Props {
  data: User[];
  options?: SearchOptions;
  loading: boolean;
  disabled: boolean;
}
```

### 2. 单向数据流

- Props 只读，不修改
- 需要修改时通过 Emits 通知父组件
- 避免双向绑定（除非确实需要）

### 3. Props 数量控制

- Props 超过 5 个时考虑是否应该拆分为多个组件
- 多个相关 props 可以组合为一个对象

```typescript
// 多个相关 props → 组合
interface PaginationProps {
  page: number;
  pageSize: number;
  total: number;
}
// 可以提取为 <Pagination v-bind="pagination" />
```

---

## 受控 vs 非受控组件

### 受控组件（父组件管理状态）

```vue
<!-- 受控输入框 -->
<script setup>
defineProps<{ value: string }>();
defineEmits<{ (e: 'update:value', value: string): void }>();
</script>
<template>
  <input :value="value" @input="$emit('update:value', $event.target.value)" />
</template>

<!-- 使用：状态由父组件管理 -->
<BaseInput :value="searchText" @update:value="searchText = $event" />
```

**适用场景**：表单联动、需要父组件验证、需要统一状态管理

### 非受控组件（组件内部管理状态）

```vue
<!-- 非受控输入框 -->
<script setup>
const value = ref('');
</script>
<template>
  <input v-model="value" />
</template>

<!-- 使用：组件内部管理自己的状态 -->
<SearchInput @search="handleSearch" />
```

**适用场景**：独立表单、简单交互、不需要外部状态同步

---

## 组合模式 vs 继承模式

### ✅ 优先使用组合

```vue
<!-- 组合：通过插槽组合功能 -->
<BaseDialog title="编辑用户">
  <template #body>
    <UserForm :user="user" />
  </template>
  <template #footer>
    <BaseButton type="primary" @click="save">保存</BaseButton>
  </template>
</BaseDialog>
```

### ❌ 避免深层继承

```typescript
// ❌ 深层继承链
class BaseComponent -> FormComponent -> UserFormComponent -> AdminUserFormComponent

// ✅ 使用组合
const { formState, validate, submit } = useForm();
const { userPermissions } = usePermissions();
const { auditLog } = useAudit();
```

---

## 组件拆分决策树

```
需要新组件？
  ↓
是否复用？─── 是 ──→ 提取为独立可复用组件
  ↓否
是否职责单一？─── 否 ──→ 按职责拆分
  ↓是
是否超过复杂度阈值？─── 是 ──→ 按功能区块拆分
  ↓否
保持为单一组件 ✅
```

---

## 前端组件分层

```
View 层 (纯展示)
  ↓ 通过 Props 接收数据
Container 层 (业务逻辑)
  ↓ 通过 Composition API / Hooks 获取数据和状态
Data 层 (API 调用)
  ↓ 通过 Service / API 模块获取数据
```

**实践**：
- `UserList.vue`（View）：只负责展示，通过 Props 接收数据
- `useUserList.ts`（Container）：管理状态、分页、筛选逻辑
- `userApi.ts`（Data）：负责 API 调用
