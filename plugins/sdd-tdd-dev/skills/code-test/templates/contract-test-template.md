# 接口契约测试模板

> 基于 api-contract.md 自动生成，用于验证实现与接口契约的一致性。

## 后端契约测试

### {METHOD} {path}

**测试文件**: `tests/contract/{normalized-path}.test.{ext}`

#### 成功响应测试

```typescript
// 验证响应字段、类型、结构与 api-contract.md 一致
describe('{METHOD} {path} - 成功响应', () => {
  it('应返回正确的字段和类型', async () => {
    const response = await request(app)
      .{method}('{path}')
      .expect(200);

    expect(response.body).toHaveProperty('code', 0);
    expect(response.body).toHaveProperty('data');
    expect(response.body).toHaveProperty('message');
    // 按 api-contract.md 定义逐项验证字段
  });
});
```

#### 错误响应测试

// 按 api-contract.md 中定义的错误码逐一测试
// 每个错误码对应一个测试用例

#### 边界条件测试

// 空值、超长字符串、特殊字符等

---

（按上述格式重复每个接口）
