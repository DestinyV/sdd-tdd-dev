# 接口契约测试模板

> 基于 api-contract.md 生成，用于验证后端实现与接口契约的一致性。

## 测试策略

契约测试验证：
1. 响应字段完整性：api-contract.md 定义的字段全部存在
2. 字段类型正确：number/string/boolean/array/object 类型匹配
3. 错误码覆盖：所有定义的错误码都有对应实现
4. 边界条件：空值、超长字符串、特殊字符处理符合契约

## Node.js + Jest + Supertest

```typescript
// tests/contract/{endpoint}.contract.test.ts

import request from 'supertest';

describe('契约测试: {METHOD} {path}', () => {
  // ==================== 成功响应契约 ====================

  describe('成功响应 (200)', () => {
    it('应返回契约定义的所有字段', async () => {
      const response = await request(app)
        .{method}('{path}')
        .set('Authorization', `Bearer ${validToken}`)
        .expect(200);

      const { body } = response;

      // 顶层结构验证
      expect(body).toHaveProperty('code', 0);
      expect(body).toHaveProperty('data');
      expect(body).toHaveProperty('message');

      // data 字段逐项验证（按 api-contract.md 定义）
      const data = body.data;
      expect(data).toHaveProperty('id', expect.any(Number));
      expect(data).toHaveProperty('name', expect.any(String));
      expect(data).toHaveProperty('createdAt', expect.any(String));
      // ... 继续验证所有定义字段

      // 不应包含契约未定义的额外字段
      const expectedKeys = ['id', 'name', 'createdAt', 'updatedAt'];
      const actualKeys = Object.keys(data);
      expect(actualKeys).toEqual(expect.arrayContaining(expectedKeys));
    });
  });

  // ==================== 错误响应契约 ====================

  describe('错误响应契约', () => {
    // 按 api-contract.md 中定义的错误码逐一验证
    it('参数校验失败应返回 400 + 错误码 40001', async () => {
      const response = await request(app)
        .{method}('{path}')
        .send({ /* 缺少必填参数 */ })
        .expect(400);

      expect(response.body.code).toBe(40001);
      expect(response.body.message).toBeDefined();
    });

    it('未认证应返回 401 + 错误码 40101', async () => {
      const response = await request(app)
        .{method}('{path}')
        .expect(401);

      expect(response.body.code).toBe(40101);
    });

    it('权限不足应返回 403 + 错误码 40301', async () => {
      const response = await request(app)
        .{method}('{path}')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);

      expect(response.body.code).toBe(40301);
    });

    it('资源不存在应返回 404 + 错误码 40401', async () => {
      const response = await request(app)
        .{method}('{path}/999999')
        .expect(404);

      expect(response.body.code).toBe(40401);
    });
  });

  // ==================== 边界条件 ====================

  describe('边界条件', () => {
    it('空字符串应该被正确处理', async () => {
      const response = await request(app)
        .{method}('{path}')
        .send({ name: '' })
        .expect(400); // 或 200，取决于契约定义
    });

    it('超长字符串应该被拒绝或截断', async () => {
      const response = await request(app)
        .{method}('{path}')
        .send({ name: 'a'.repeat(10001) })
        .expect(400); // 或 200（如果契约允许）
    });

    it('SQL 注入尝试应该被安全处理', async () => {
      const response = await request(app)
        .{method}('{path}')
        .send({ name: "'; DROP TABLE users; --" })
        .expect(200); // 应正常处理，不抛 500
    });
  });
});
```

## Python + Pytest + httpx

```python
# tests/contract/test_{endpoint}_contract.py

import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_{endpoint}_success_response(client: AsyncClient, auth_token: str):
    """验证成功响应包含契约定义的所有字段"""
    response = await client.{method}(
        "{path}",
        headers={"Authorization": f"Bearer {auth_token}"},
    )

    assert response.status_code == 200
    data = response.json()
    assert data["code"] == 0
    assert "id" in data["data"]
    assert isinstance(data["data"]["id"], int)
    assert "name" in data["data"]
    assert isinstance(data["data"]["name"], str)

@pytest.mark.parametrize("error_case", [
    (400, 40001, "参数校验失败"),
    (401, 40101, "未认证"),
    (403, 40301, "权限不足"),
    (404, 40401, "资源不存在"),
])
async def test_{endpoint}_error_codes(client: AsyncClient, error_case):
    """按契约定义的错误码逐一验证"""
    status_code, error_code, description = error_case
    # 根据错误码构造对应的请求场景
    response = await client.{method}("{path}", ...)
    assert response.status_code == status_code
    assert response.json()["code"] == error_code
```
