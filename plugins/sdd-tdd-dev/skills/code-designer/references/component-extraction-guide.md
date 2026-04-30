# 组件提取指南

> code-designer 阶段组件拆分决策和 code-execute 阶段实现参考

## 拆分触发器判定表

当满足以下任一条件时，应考虑拆分组件：

| 触发条件 | 阈值 | 说明 |
|---------|------|------|
| 代码行数过多 | Template > 150 行 或 Script > 200 行 | 复杂度超标 |
| 多职责 | 组件处理 2+ 不相关的业务逻辑 | 违反 SRP |
| Props 过多 | Props > 5 个 | 接口过大，考虑分组或拆分 |
| 多处复用 | 相同/相似 UI 在 2+ 处使用 | 应提取共享组件 |
| 可独立测试 | 组件逻辑需要单独测试 | 测试隔离需求 |
| 复杂条件分支 | 嵌套条件 > 3 层 | 提取为独立判断逻辑 |

## 拆分决策流程图

```
需要新 UI 模块？
  ↓
是否已在项目中存在？─── 是 ──→ 复用现有组件
  ↓否
是否会在多处使用？─── 是 ──→ 提取为共享组件
  ↓否
是否职责单一？─── 否 ──→ 按职责拆分
  ↓是
是否超过复杂度阈值？─── 是 ──→ 按功能区块拆分
  ↓否
保持为单一组件 ✅
```

## 前端组件拆分

### 容器/展示组件分离

```vue
<!-- 容器组件（Container）：负责数据获取和业务逻辑 -->
<!-- UserListContainer.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue';
import UserList from './UserList.vue';
import { fetchUsers } from '@/api/user';

const users = ref([]);
const loading = ref(false);

onMounted(async () => {
  loading.value = true;
  users.value = await fetchUsers();
  loading.value = false;
});
</script>
<template>
  <UserList :users="users" :loading="loading" @delete="handleDelete" />
</template>

<!-- 展示组件（Presentational）：只负责渲染 -->
<!-- UserList.vue -->
<script setup lang="ts">
defineProps<{ users: User[]; loading: boolean }>();
defineEmits<{ (e: 'delete', id: number): void }>();
</script>
<template>
  <div v-if="loading">加载中...</div>
  <div v-else>
    <UserCard v-for="user in users" :key="user.id" :user="user" @delete="$emit('delete', user.id)" />
  </div>
</template>
```

### Hooks 提取

```typescript
// useOrder.ts
export function useOrder() {
  const orders = ref<Order[]>([]);
  const loading = ref(false);

  async function fetchOrders() {
    loading.value = true;
    orders.value = await orderApi.list();
    loading.value = false;
  }

  async function createOrder(data: CreateOrderInput) {
    const order = await orderApi.create(data);
    orders.value.push(order);
    return order;
  }

  return { orders, loading, fetchOrders, createOrder };
}
```

### 组合优于继承

```vue
<!-- ❌ 错误：深层继承 -->
<!-- BaseForm -> AuthForm -> LoginForm -->

<!-- ✅ 正确：组合 -->
<BaseForm>
  <FormField name="username" />
  <FormField name="password" type="password" />
  <FormActions>
    <BaseButton type="primary" @click="login">登录</BaseButton>
  </FormActions>
</BaseForm>
```

## 后端代码拆分

### 服务提取

```typescript
// ❌ 错误：一个 God Service
class UserService {
  register() { /* 注册 */ }
  login() { /* 登录 */ }
  sendEmail() { /* 发邮件 */ }
  generateToken() { /* 生成 Token */ }
  sendSMS() { /* 发短信 */ }
  validatePassword() { /* 验证密码 */ }
  // ... 20+ 方法
}

// ✅ 正确：按职责拆分
class UserAuthService { register() { } login() { } validatePassword() { } }
class TokenService { generateToken() { } verifyToken() { } }
class NotificationService { sendEmail() { } sendSMS() { } }
```

### 中间件提取

```typescript
// ❌ 错误：每个路由都重复验证
router.post('/orders', async (req, res) => {
  if (!req.user) return res.status(401).json({ code: 40101 });
  const errors = validateOrder(req.body);
  if (errors.length) return res.status(400).json({ code: 40001, errors });
  // 业务逻辑
});

// ✅ 正确：提取为中间件
router.use(authMiddleware);
router.post('/orders', validateOrderMiddleware, async (req, res) => {
  // 业务逻辑
});
```

## 反模式

### 过度拆分

```vue
<!-- ❌ 错误：每个小元素都拆组件 -->
<UserList>
  <UserListItem>
    <UserName>
      <UserAvatar>
        <img />
      </UserAvatar>
    </UserName>
  </UserListItem>
</UserList>

<!-- ✅ 正确：合理粒度 -->
<UserList :users="users" @delete="handleDelete" />
<!-- UserList 内部包含 UserItem -->
```

### 过早抽象

```typescript
// ❌ 错误：第一次出现就提取抽象
abstract class BaseRepository { /* 复杂抽象逻辑 */ }
class UserRepo extends BaseRepository {}
class OrderRepo extends BaseRepository {}

// ✅ 正确：第二次出现时再提取
class UserRepo { /* 直接实现 */ }
class OrderRepo { /* 直接实现 */ }
// 当发现两者有大量相同代码时，再考虑提取基类
```

### 拆分后高耦合

```typescript
// ❌ 错误：拆分后仍然紧密耦合
class UserList {
  orderService: OrderService;  // 不应该知道的依赖
  paymentGateway: PaymentGateway;  // 不应该知道的依赖
}

// ✅ 正确：通过接口或事件通信
class UserList {
  constructor(
    private userService: UserService,
    private eventBus: EventBus,
  ) {}
}
```
