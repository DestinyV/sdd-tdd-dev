# 代码完整性检查清单

## 概述

在质量审查阶段，**完整性检查**是一个关键的检查项。这个检查确保所有代码都是完整的生产级代码，而不是包含框架、示例或待实现的代码。

### 为什么这很重要？

**问题示例**：
```vue
<!-- ❌ 不完整：样式块只有注释 -->
<template>
  <div class="form">
    <input type="text" placeholder="用户名" />
  </div>
</template>

<style lang="less" scoped>
  // 样式规则遵循现有审核项的风格
</style>
```

这样的代码会导致：
- ❌ 样式没有应用，组件外观破碎
- ❌ 代码看起来像"半成品"
- ❌ 在代码审查时容易被reject
- ❌ 用户体验差（组件无样式或样式不正确）

---

## 检查清单

### 1️⃣ Vue/React 组件的结构检查

#### 模板/JSX 部分
- [ ] **不是空的或只有占位符**
  - ❌ 错误：`<template><!-- TODO: 添加表单内容 --></template>`
  - ✅ 正确：完整的组件模板

- [ ] **所有条件渲染都有完整实现**
  - ❌ 错误：
    ```vue
    <div v-if="isLoading">加载中...</div>
    <!-- 没有对应的 v-else -->
    ```
  - ✅ 正确：
    ```vue
    <div v-if="isLoading">加载中...</div>
    <div v-else>内容已加载</div>
    ```

#### 脚本部分
- [ ] **所有声明的方法都有完整实现**
  - ❌ 错误：
    ```typescript
    methods: {
      handleSubmit() {
        // TODO: 实现提交逻辑
      }
    }
    ```
  - ✅ 正确：方法有完整的实现体

- [ ] **所有props都有默认值或类型定义**
  - ❌ 错误：
    ```typescript
    const props = defineProps({
      title: String, // 没有default
    });
    ```
  - ✅ 正确：
    ```typescript
    const props = defineProps({
      title: { type: String, required: true },
      description: { type: String, default: '' }
    });
    ```

#### 样式部分
- [ ] **不能只有注释或示例代码**
  - ❌ 错误：
    ```less
    <style lang="less" scoped>
      // 样式规则遵循现有审核项的风格
    </style>
    ```
  - ✅ 正确：
    ```less
    <style lang="less" scoped>
      .form {
        padding: 16px;
        border-radius: 4px;
      }

      .form-input {
        width: 100%;
        border: 1px solid #ddd;
      }
    </style>
    ```
  - ✅ 也正确（如果不需要样式）：根本不写 `<style>` 块

### 2️⃣ 函数和方法检查

#### 函数签名
- [ ] **所有函数都有返回值**
  - ❌ 错误：
    ```typescript
    function validateEmail(email) {
      // TODO: 实现邮箱验证
    }
    ```
  - ✅ 正确：
    ```typescript
    function validateEmail(email: string): boolean {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }
    ```

- [ ] **所有参数都有类型注解**
  - ❌ 错误：`function process(data) { ... }`
  - ✅ 正确：`function process(data: UserData[]): ProcessResult { ... }`

#### 函数体
- [ ] **不能只有 throw 或 return**
  - ❌ 错误：
    ```typescript
    function createUser(userData) {
      throw new Error('Not implemented');
    }
    ```
  - ✅ 正确：完整的实现，包括错误处理

- [ ] **所有分支都有实现**
  - ❌ 错误：
    ```typescript
    if (isValid) {
      return processData(data);
    } else {
      // TODO: 处理无效数据
    }
    ```
  - ✅ 正确：所有分支都有实现

### 3️⃣ 对象和配置检查

#### 对象属性
- [ ] **所有属性都有实际值**
  - ❌ 错误：
    ```typescript
    const config = {
      title: 'User Form',
      fields: [], // 空数组，应该填充
      onSubmit: () => {}, // 空函数
    };
    ```
  - ✅ 正确：
    ```typescript
    const config = {
      title: 'User Form',
      fields: [
        { name: 'email', type: 'email', required: true },
        { name: 'password', type: 'password', required: true }
      ],
      onSubmit: handleFormSubmit,
    };
    ```

#### 回调函数
- [ ] **不能是空函数或只有注释**
  - ❌ 错误：
    ```typescript
    const handleClick = () => {
      // TODO: 处理点击事件
    };
    ```
  - ✅ 正确：
    ```typescript
    const handleClick = () => {
      const result = validateForm();
      if (result.valid) {
        submitForm();
      }
    };
    ```

### 4️⃣ 导入和导出检查

- [ ] **所有导入都被使用**
  - ❌ 错误：`import { unused } from './utils';` （但没有使用）
  - ✅ 正确：导入的只是实际使用的

- [ ] **所有导出都有意义**
  - ❌ 错误：导出了但实际没有实现的函数
  - ✅ 正确：导出生产级代码

### 5️⃣ 注释和文档检查

- [ ] **没有 TODO 注释在最终代码中**
  - ❌ 错误：
    ```typescript
    // TODO: 实现缓存机制
    function fetchData() { ... }
    ```
  - ✅ 正确：要么实现了，要么根本没有 TODO

- [ ] **示例注释应该是实际实现，而不是示例**
  - ❌ 错误：
    ```typescript
    // 样式规则遵循现有审核项的风格 (这只是示例)
    ```
  - ✅ 正确：要么删除，要么写实际的样式规则

- [ ] **没有被注释掉的代码块**
  - ❌ 错误：
    ```typescript
    // const oldImplementation = () => { ... }; // 旧代码，之后删除
    ```
  - ✅ 正确：删除不需要的代码，不要注释掉

### 6️⃣ 类型定义检查

- [ ] **没有 any 类型**
  - ❌ 错误：`const data: any = fetchData();`
  - ✅ 正确：`const data: UserData = fetchData();`

- [ ] **所有接口都有完整定义**
  - ❌ 错误：
    ```typescript
    interface User {
      name: string;
      // TODO: 添加更多字段
    }
    ```
  - ✅ 正确：
    ```typescript
    interface User {
      id: string;
      name: string;
      email: string;
      createdAt: Date;
    }
    ```

---

## 常见问题和解决方案

### 问题 1：样式块只有注释

**现象**：
```vue
<style lang="less" scoped>
  // 样式规则遵循现有审核项的风格
</style>
```

**解决方案**：
- **选项 A**：编写完整的样式
  ```less
  <style lang="less" scoped>
    .form-container {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .form-input {
      padding: 8px 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
  </style>
  ```

- **选项 B**：删除整个样式块（如果不需要自定义样式）

### 问题 2：函数只有 TODO

**现象**：
```typescript
function handleSubmit(formData) {
  // TODO: 实现表单提交逻辑
}
```

**解决方案**：
实现完整的函数体
```typescript
async function handleSubmit(formData: FormData) {
  try {
    const response = await api.submitForm(formData);
    if (response.success) {
      showSuccess('表单提交成功');
      resetForm();
    } else {
      showError(response.error);
    }
  } catch (error) {
    showError('提交失败，请重试');
    console.error(error);
  }
}
```

### 问题 3：对象属性是空的

**现象**：
```typescript
const fields = {
  username: '',
  password: '',
  // 其他字段待定义
};
```

**解决方案**：
填充所有必要的字段
```typescript
const fields = {
  username: {
    label: '用户名',
    type: 'text',
    required: true,
    maxLength: 20,
    validation: /^[a-zA-Z0-9_]+$/
  },
  password: {
    label: '密码',
    type: 'password',
    required: true,
    minLength: 8
  }
};
```

---

## 检查流程

在质量审查时，按照以下步骤进行完整性检查：

### 步骤 1: 快速扫描
逐个文件检查，寻找以下模式：
- 含有注释 `//` 或 `/**/` 但没有实际代码的块
- 含有 `TODO`、`FIXME`、`XXX` 注释的地方
- 空的函数体或只有 `throw new Error('Not implemented')`

### 步骤 2: 深入检查
对可疑的地方进行深入检查：
- 这是框架代码吗？（只有结构，没有实现）
- 这是示例代码吗？（应该被完整实现）
- 这是临时代码吗？（应该被删除）

### 步骤 3: 提出修复建议
对于每个发现的不完整代码：
- 明确指出是什么（例如："样式块只有注释"）
- 说明为什么是问题（例如："组件无样式或样式不正确"）
- 建议具体的修复方案（例如："实现完整的样式"）

---

## 质量审查时的问题模板

发现完整性问题时，使用以下模板提出修复建议：

```
### 完整性问题

**位置**: src/components/UserForm.vue (第 45-50 行)

**问题**: 样式块只有注释，没有实际的样式实现

**现象**:
```vue
<style lang="less" scoped>
  // 样式规则遵循现有审核项的风格
</style>
```

**影响**: 组件没有应用任何样式，外观破碎

**修复方案**:
- 编写完整的样式，包括：
  - 表单容器的布局（padding、margin、gap）
  - 输入框的样式（border、padding、border-radius）
  - 按钮的样式
  - 验证错误提示的样式
- 或删除整个 `<style>` 块（如果不需要自定义样式）

**优先级**: 🔴 高 - 必须修复
```

---

## 最佳实践

1. **宁删勿留** - 如果不需要某个部分（比如样式），完全删除而不是留框架
2. **完整实现** - 如果要实现某个功能，就实现完整，不留 TODO
3. **早期发现** - 在实现阶段就避免这类问题，而不是等到审查时
4. **明确设计** - 在设计阶段明确所有需要实现的部分，避免后期添加框架代码

---

**最后更新**: 2026-03-27
