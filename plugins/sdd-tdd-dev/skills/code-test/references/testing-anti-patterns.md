# 测试反模式

> code-test 阶段审查时应检查的常见测试反模式

## 1. 测试 Mock 行为而非真实行为

**问题**：Mock 了业务逻辑，导致测试通过但实际功能不工作。

```typescript
// ❌ 错误：Mock 了整个服务
jest.mock('../UserService', () => ({
  getUser: jest.fn().mockReturnValue({ id: 1 })
}));

// ✅ 正确：只 Mock 外部依赖（数据库、HTTP）
jest.mock('../db', () => ({
  query: jest.fn().mockResolvedValue([{ id: 1 }])
}));
```

## 2. 向生产类添加 test-only 方法

**问题**：为测试便利修改生产代码。

```typescript
// ❌ 错误：生产代码被测试污染
class UserService {
  // @ts-expose-for-testing
  _internalValidate() { ... }
}

// ✅ 正确：通过公共接口测试
```

## 3. 不理解依赖就 Mock

**问题**：Mock 了不应该 Mock 的内部函数。

```typescript
// ❌ 错误：Mock 了内部工具函数
jest.spyOn(utils, 'formatDate').mockReturnValue('2024-01-01');

// ✅ 正确：让内部函数真实执行，只 Mock 外部边界
```

## 4. 不完整的 Mock 数据结构

**问题**：Mock 数据缺少必要字段，测试通过但实际调用失败。

```typescript
// ❌ 错误：缺少必需字段
const mockUser = { id: 1 };

// ✅ 正确：完整的数据结构
const mockUser = {
  id: 1,
  name: 'Test',
  email: 'test@example.com',
  createdAt: new Date(),
  roles: ['user']
};
```

## 5. 集成测试被当作事后补充

**问题**：只有单元测试，集成测试是后来补的。

**正确做法**：
- 单元测试保证逻辑正确
- 集成测试保证组件间协作
- 两者同等重要，同步编写

## 6. 测试中的魔法数字和硬编码

```typescript
// ❌ 错误
expect(result.items.length).toBe(5);

// ✅ 正确
const EXPECTED_ITEM_COUNT = 5;
expect(result.items.length).toBe(EXPECTED_ITEM_COUNT);
```

## 7. 测试依赖执行顺序

```typescript
// ❌ 错误：测试间共享状态
let userId: number;

test('create user', () => { userId = create(); });
test('update user', () => { update(userId); }); // 依赖上一个测试

// ✅ 正确：每个测试独立
test('create and update user', () => {
  const userId = create();
  update(userId);
});
```

## 审查检查清单

- [ ] 是否只 Mock 外部依赖（DB、HTTP、FS）？
- [ ] Mock 数据结构是否完整？
- [ ] 测试是否通过公共接口执行？
- [ ] 测试间是否相互独立？
- [ ] 是否有 always-true 的断言？
- [ ] 是否遗漏了错误路径测试？
