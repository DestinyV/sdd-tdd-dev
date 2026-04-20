# 前端SDD最佳实践指南

本指南提供前端开发中使用sdd-dev-plugin工作流的最佳实践和检查清单。

---

## 📋 前端SDD工作流检查清单

### Spec阶段（spec-creation）

#### 需求理解
- [ ] 需求描述清晰明确
- [ ] 已识别核心功能点
- [ ] 初步提取主要场景

#### 场景拆解和确认
- [ ] 需求已拆解为具体的业务场景
- [ ] 每个场景都有明确的WHEN条件和THEN结果
- [ ] 所有业务流程都被覆盖
- [ ] 异常情况和边界情况都被考虑

#### 规范文档
- [ ] spec-dev/{requirement_desc_abstract}/spec/README.md 已生成
- [ ] spec-dev/{requirement_desc_abstract}/spec/scenarios/*.md 已生成（BDD格式）
- [ ] spec-dev/{requirement_desc_abstract}/spec/data-models.md 已生成
- [ ] spec-dev/{requirement_desc_abstract}/spec/business-rules.md 已生成
- [ ] spec-dev/{requirement_desc_abstract}/spec/glossary.md 已生成
- [ ] 所有规范文档都经过用户确认

#### 注意事项
- ⚠️ 规范是设计和开发的基础，必须完整准确
- ⚠️ 通过多轮确认确保规范细节无遗漏
- ⚠️ 避免规范过于笼统或模糊

---

### Design阶段（code-designer）

#### 规范分析
- [ ] 已读取规范文档（spec/）
- [ ] 已理解所有业务场景和需求
- [ ] 已识别对应的设计模式

#### 设计方案
- [ ] 设计方案涵盖所有功能点
- [ ] 所有设计决策都有明确原因
- [ ] 架构清晰（组件结构、数据流、状态管理）
- [ ] Props接口设计清晰，无歧义
- [ ] 考虑了性能、可访问性、响应式等非功能需求

#### 与参考设计的对比
- [ ] 已识别参考组件
- [ ] 已分析参考组件的完整实现
- [ ] 已列出与参考的相同点和差异点
- [ ] 差异点都有明确的原因说明

#### 技术方案
- [ ] 依赖库已确定
- [ ] API设计已明确
- [ ] 状态管理方案已确定（useState/useReducer/Redux/Zustand等）
- [ ] 样式方案已确定（CSS Modules/Tailwind/Styled Components等）
- [ ] 数据获取方式已确定（Fetch/Axios/React Query等）

#### 设计审批
- [ ] 设计方案已生成（design.md）
- [ ] 所有关键决策都已记录
- [ ] 用户已审查并批准设计方案

---

### Task阶段（code-task）

#### 任务分解
- [ ] 设计已分解为具体的编码任务
- [ ] 每个Task的目标明确
- [ ] 任务粒度合适（4-8小时可完成）
- [ ] 任务依赖关系清晰

#### 交付物定义
- [ ] 每个Task都定义了具体的交付物
- [ ] 交付物包括：组件/Hook/工具函数/测试/样式
- [ ] 没有歧义的交付物定义

#### 验收标准
- [ ] 每个Task都有清晰的验收标准
- [ ] 验收标准是可衡量的（不含主观描述）
- [ ] 验收标准包括：功能、质量、测试、文档等维度

#### 任务优先级和依赖
- [ ] 基础组件在前，复杂组件在后
- [ ] 有依赖的Task明确了前置Task
- [ ] 无依赖的Task可以并行执行

---

### Execute阶段（code-execute）

#### 代码质量
- [ ] 所有代码都通过了规范审查
- [ ] 所有代码都通过了质量审查
- [ ] 没有Lint错误或警告（eslint配置严格）
- [ ] TypeScript strict模式无错误

#### 代码审查
- [ ] Props设计与design.md一致
- [ ] 组件结构与参考组件相似
- [ ] 样式方案与design.md的方案一致
- [ ] API调用正确，错误处理完整
- [ ] 没有明显的性能问题

#### 测试覆盖
- [ ] 单元测试已编写（各Task都有）
- [ ] 测试覆盖率 ≥ 80%
- [ ] 关键业务逻辑有测试
- [ ] 异常情况有测试

#### 代码提交
- [ ] 代码已在本地编译和基本测试通过
- [ ] 没有console.log和TODO注释
- [ ] 没有过时的代码注释
- [ ] Git commit message清晰

---

### Test阶段（code-execute - 单元测试）

**重要提示**：单元测试由 `/code-execute` 的TDD流程（RED-GREEN-REFACTOR-REVIEW）完成。本部分描述单元测试的最佳实践。code-test不再重复审查单元测试，而是专注于Phase 3的高层测试（见下一部分）。

#### 代码质量审查
- [ ] ESLint检查通过（0 errors）
- [ ] TypeScript strict检查通过（0 errors）
- [ ] 没有unused导入或变量
- [ ] 没有any类型（严格遵循）
- [ ] 没有console.log和TODO注释

#### 单元测试
- [ ] 所有单元测试都通过（100%）
- [ ] 测试覆盖正常、边界、异常场景
- [ ] 关键组件的覆盖率 ≥ 85%
- [ ] 所有Task都有单元测试
- [ ] 测试代码有清晰的说明注释

#### 集成测试
- [ ] 多个组件的集成测试已执行
- [ ] 完整的用户流程测试通过
- [ ] API集成测试通过（使用mock）
- [ ] 跨模块的交互测试通过

#### E2E测试（可选）
- [ ] 关键用户场景有E2E测试
- [ ] E2E测试在真实环境运行通过
- [ ] 测试覆盖正常和异常流程

#### 闭环验证
- [ ] 所有Task的功能都被实现了
- [ ] 所有Task的功能都有对应测试
- [ ] 所有代码都有对应测试
- [ ] Task-代码-测试完全对应
- [ ] 没有实现超出Task范围的功能

#### 最终报告
- [ ] 测试报告已生成（testing-report.md）
- [ ] 覆盖率分析已完成
- [ ] 所有问题都已记录
- [ ] 闭环验证矩阵已生成

### Phase 3阶段（code-test - 高层测试）✨

**重要提示**：Phase 3 code-test专注于**集成测试、E2E测试、性能测试**，**不重复审查单元测试**（由code-execute的TDD流程保证）。

#### 集成测试最佳实践

**何时需要**：
- [ ] 多个Task/模块需要协作
- [ ] 需要验证跨模块数据流
- [ ] 需要验证API和数据库的联动

**设计原则**：
- [ ] 测试边界清晰（哪些Task协作）
- [ ] Mock策略合理（外部依赖Mock，内部真实）
- [ ] 测试数据独立（不依赖其他测试）
- [ ] 数据清理完整（beforeEach/afterEach）

**常见问题**：
- ❌ 避免测试间有依赖（Test A的结果影响Test B）
- ❌ 避免Mock策略不清晰（不知道什么Mock，什么真实）
- ❌ 避免数据库状态管理不当（初始化、清理、回滚）
- ✅ 使用事务回滚或适当的数据清理
- ✅ Mock在beforeEach中清理，确保测试隔离

**参考**：`code-test/integration-test-prompt.md`

#### E2E测试最佳实践

**何时需要**：
- [ ] 需要验证完整的业务流程
- [ ] 需要验证用户的端到端场景
- [ ] 需要验证前后端联动

**设计原则**：
- [ ] 用户场景真实（从用户角度出发）
- [ ] 验证点完整（UI、数据库、后端API）
- [ ] 避免Flaky（使用显式等待，不用sleep）
- [ ] 测试独立可重复（beforeEach恢复初始状态）

**常见问题**：
- ❌ 避免使用固定延迟`setTimeout(1000)` → 使用`waitFor()`
- ❌ 避免脆弱的选择器（易因UI改变而失败）
- ❌ 避免缺少`await`（异步操作未等待）
- ❌ 避免测试间有依赖（前一个测试的结果影响后一个）
- ✅ 使用`page.waitFor()`, `page.waitForSelector()`等显式等待
- ✅ 使用`data-testid`而不是文本或class选择器
- ✅ 正确处理导航和页面加载
- ✅ beforeEach重置应用状态或清理数据

**参考**：`code-test/e2e-test-prompt.md`

#### 性能测试最佳实践

**何时需要**：
- [ ] 关键业务流程（搜索、支付、登录等）
- [ ] 高并发场景（预期承载多少用户）
- [ ] 需要建立性能基准

**设计原则**：
- [ ] 性能指标明确（P95、RPS、错误率）
- [ ] 测试场景完整（正常负载、压力、耐久性）
- [ ] 性能目标现实（基于业务需求）
- [ ] 结果可追踪（记录基准数据，识别回归）

**常见问题**：
- ❌ 避免性能指标没有基准（不知道什么是好）
- ❌ 避免测试场景不全面（缺少压力、耐久性测试）
- ❌ 避免预热不足（第一次运行影响结果）
- ✅ 定义P95、P99响应时间目标
- ✅ 定义RPS（吞吐量）和错误率阈值
- ✅ 定义CPU、内存、网络使用上限
- ✅ 包含Load（正常）、Stress（压力）、Endurance（24小时）、Spike（突发）四种场景
- ✅ 性能目标基于业务需求（如：P95 < 200ms，错误率 < 0.1%）

**参考**：`code-test/performance-test-prompt.md`

#### 闭环验证

- [ ] 所有TEST-VERIFY（规范中定义）都有对应的test
- [ ] 单元测试 → 验证Task的单个功能
- [ ] 集成测试 → 验证多Task的协作
- [ ] E2E测试 → 验证完整的业务流程
- [ ] 性能测试 → 验证系统的性能基准
- [ ] 验证矩阵：TEST-VERIFY → Test Case → 测试代码 → 结果

---

## 🎯 前端开发最佳实践

### 1. 组件设计

#### Props设计
```typescript
// ✅ 好的实践
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  /** 按钮的视觉风格 */
  variant?: 'primary' | 'secondary' | 'danger';
  /** 按钮大小 */
  size?: 'small' | 'medium' | 'large';
  /** 是否禁用 */
  disabled?: boolean;
  /** 点击回调 */
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  /** 子元素 */
  children: React.ReactNode;
}

// ❌ 避免的实践
interface ButtonProps {
  // 太灵活，容易出现意外行为
  props?: any;
  // 没有类型，难以维护
  onClick?: Function;
}
```

**原则**：
- 使用TypeScript进行类型定义，避免any
- Props应该有JSDoc注释说明
- 避免过多的Props，复杂组件考虑使用config object
- 提供合理的默认值

#### 组件职责
```typescript
// ✅ 单一职责原则
const OrderList: React.FC<OrderListProps> = (props) => {
  // 只负责渲染和管理列表状态
  // API调用通过props传入
  // 样式通过CSS Modules处理
};

// ❌ 违反单一职责
const OrderList: React.FC<OrderListProps> = (props) => {
  // 同时处理：API调用、业务逻辑、UI渲染
  // 难以测试和复用
};
```

**原则**：
- 每个组件职责单一，易于理解和维护
- 分离容器组件和展示组件
- 复杂业务逻辑提取为自定义Hook

### 2. 状态管理

#### 本地状态
```typescript
// ✅ 使用useState进行简单状态
const [count, setCount] = useState(0);

// ✅ 使用useReducer进行复杂状态
const [state, dispatch] = useReducer(reducer, initialState);

// ❌ 避免过度的状态
const [user, setUser] = useState<any>(null);  // 应该有详细类型
```

**原则**：
- 简单状态使用useState
- 复杂状态或多个相关状态使用useReducer
- 避免在state中存储派生数据（应该用useMemo计算）

#### 全局状态
```typescript
// 选择一个方案并严格遵循
// 选项1: Redux + Redux Toolkit
// 选项2: Zustand
// 选项3: Recoil
// 选项4: Jotai
```

**原则**：
- 在spec-creation时确定全局状态管理方案
- 所有项目统一使用同一个方案
- 只将真正的全局数据放在全局store

### 3. 性能优化

#### 避免不必要的重新渲染
```typescript
// ✅ 使用useMemo和useCallback
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

const handleClick = useCallback(() => {
  onItemClick(item);
}, [item, onItemClick]);

// ✅ 使用React.memo包装展示组件
const OrderItem = React.memo<OrderItemProps>(({ order }) => {
  return <div>{order.name}</div>;
});

// ❌ 避免的实践
// 在每次render时创建新对象
const config = { mode: 'edit' };  // 每次都是新对象
const onClick = () => { /* ... */ };  // 每次都是新函数
```

**原则**：
- 只在必要时使用useMemo和useCallback（性能分析后）
- 展示组件使用React.memo防止不必要的重新渲染
- 避免在render中创建新对象或函数

#### 虚拟滚动
```typescript
// ✅ 大列表使用虚拟滚动
<VirtualList
  items={items}
  itemHeight={50}
  height={500}
  renderItem={(item) => <OrderItem order={item} />}
/>

// ❌ 避免渲染数千个DOM节点
items.map(item => <OrderItem key={item.id} order={item} />)
```

**原则**：
- 列表项 > 100时考虑虚拟滚动
- 使用react-window或react-virtualized库

### 4. 样式管理

#### CSS选择方案
```typescript
// 选择一个方案并严格遵循：
// 选项1: CSS Modules (推荐 - 简单，强制隔离)
// 选项2: Tailwind CSS (推荐 - 快速开发)
// 选项3: Styled Components (灵活但需要注意性能)
// 选项4: BEM + SCSS (传统，较重)
```

**原则**：
- 在spec-creation时确定样式方案
- 所有项目统一使用同一个方案
- 避免内联样式（影响性能和维护性）

#### 颜色和间距
```typescript
// ✅ 使用设计tokens
const colors = {
  primary: '#007bff',
  success: '#28a745',
  danger: '#dc3545',
};

const spacing = {
  xs: '4px',
  sm: '8px',
  md: '16px',
  lg: '24px',
};

// ❌ 避免硬编码颜色和间距
const buttonStyle = {
  background: '#007bff',
  padding: '8px 16px',
  marginBottom: '16px',
};
```

**原则**：
- 定义设计tokens（颜色、间距、字体等）
- 所有样式都从tokens引用
- 使用CSS变量支持主题切换

### 5. TypeScript最佳实践

#### 类型定义
```typescript
// ✅ 完整的类型定义
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

type UserId = number & { readonly __brand: 'UserId' };

// ✅ 使用泛型避免重复
const createList = <T,>(items: T[]): List<T> => {
  return new List(items);
};

// ❌ 避免any
const handleClick = (event: any) => {
  // 失去类型检查
};

// ❌ 避免as类型断言
const data = json as any as User;  // 危险！
```

**原则**：
- 启用TypeScript strict模式
- 避免使用any，必要时使用unknown再做类型守卫
- 为函数参数和返回值标注类型
- 对API响应进行类型定义

#### 类型守卫
```typescript
// ✅ 使用类型守卫
function isUser(obj: any): obj is User {
  return obj && typeof obj.id === 'number' && typeof obj.name === 'string';
}

if (isUser(data)) {
  console.log(data.id);  // 安全访问
}

// ✅ 使用类型判别
type Result = { success: true; data: User } | { success: false; error: string };

if (result.success) {
  console.log(result.data);  // 类型自动缩窄
}
```

### 6. 错误处理

#### API错误处理
```typescript
// ✅ 完整的错误处理
try {
  const response = await fetchUser(userId);
  return response.data;
} catch (error) {
  if (error instanceof NetworkError) {
    // 处理网络错误
    showNotification('Network error, please retry');
  } else if (error instanceof ValidationError) {
    // 处理验证错误
    showNotification(error.message);
  } else {
    // 处理未知错误
    logger.error('Unknown error', error);
    showNotification('Something went wrong');
  }
}

// ❌ 避免空的catch块
try {
  await fetchUser(userId);
} catch (error) {
  // 忽略所有错误
}
```

**原则**：
- 为不同类型的错误提供不同的处理
- 向用户显示有意义的错误信息
- 记录错误日志用于调试

#### 异常边界
```typescript
// ✅ 使用ErrorBoundary捕获组件错误
<ErrorBoundary fallback={<ErrorPage />}>
  <OrderList />
</ErrorBoundary>

// ❌ 忽视组件渲染错误
<OrderList />  // 如果渲染出错，整个页面崩溃
```

### 7. 测试最佳实践

#### 单元测试
```typescript
// ✅ 清晰的测试结构
describe('OrderForm', () => {
  describe('正常流程', () => {
    test('应该正确渲染', () => {
      // Arrange
      const props = { onSubmit: jest.fn() };
      // Act
      render(<OrderForm {...props} />);
      // Assert
      expect(screen.getByText(/order form/)).toBeInTheDocument();
    });
  });

  describe('边界情况', () => {
    test('应该处理空值', () => {
      render(<OrderForm items={[]} onSubmit={jest.fn()} />);
      expect(screen.getByText(/no items/)).toBeInTheDocument();
    });
  });
});

// ❌ 避免的测试
test('everything works', () => {
  // 太宽泛，不知道测试了什么
  expect(true).toBe(true);
});
```

**原则**：
- 每个测试只测试一个功能点
- 使用清晰的测试名称
- 遵循Arrange-Act-Assert结构
- 避免测试实现细节，测试行为

#### 测试覆盖率
- **目标**：≥ 80%的行覆盖率
- **重点**：业务逻辑、API调用、错误处理必须覆盖
- **可以不覆盖**：样式代码、UI细节、unlikely的分支

---

## 🚀 工作流优化建议

### 1. 设计阶段
- ✅ 充分的设计讨论能避免后续修改
- ✅ 清晰的设计文档能加快Task阶段
- ✅ 获得所有相关人的确认能减少变更

### 2. Task阶段
- ✅ 详细的Task定义能提高Execute的效率
- ✅ 清晰的验收标准能加快Test的验证
- ✅ 合理的任务粒度能避免Task太大

### 3. Execute阶段
- ✅ 严格的审查流程能及早发现问题
- ✅ 问题修复的修复循环能保证质量
- ✅ 详细的执行报告能便于追踪

### 4. Test阶段
- ✅ 完整的测试覆盖能确保功能完整
- ✅ 闭环验证能确保Task-代码-测试一致
- ✅ 详细的测试报告能便于后续维护

---

## 📊 质量指标

### 代码质量指标

| 指标 | 目标 | 说明 |
|------|------|------|
| Lint检查 | 0 errors | 使用ESLint strict配置 |
| TypeScript | 0 errors | strict模式无any |
| 代码覆盖率 | ≥ 80% | 所有模块都有测试 |
| 圈复杂度 | < 10 | 单个函数控制流不过复杂 |
| 函数行数 | < 50 | 单个函数不过长 |

### 测试质量指标

| 指标 | 目标 | 说明 |
|------|------|------|
| 单元测试通过率 | 100% | 所有测试都通过 |
| 集成测试通过率 | 100% | 模块间集成测试通过 |
| E2E测试通过率 | 100% | 用户流程测试通过 |
| 测试覆盖率 | ≥ 80% | 代码被测试覆盖 |
| 关键路径覆盖 | 100% | 关键业务流程完全覆盖 |

### 性能指标

| 指标 | 目标 | 说明 |
|------|------|------|
| 首屏加载时间 | < 2s | 根据项目调整 |
| 响应时间 | < 100ms | 用户交互响应时间 |
| 内存占用 | < 100MB | 运行中的内存占用 |
| Bundle大小 | < 500KB | 优化后的打包大小 |

---

## 🎓 学习资源

### 前端框架
- [React官方文档](https://react.dev)
- [Vue官方文档](https://vuejs.org)
- [Angular官方文档](https://angular.io)

### TypeScript
- [TypeScript官方文档](https://www.typescriptlang.org)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

### 测试
- [Jest文档](https://jestjs.io)
- [React Testing Library](https://testing-library.com/react)
- [Cypress文档](https://cypress.io)

### 性能优化
- [Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/reference/react/useMemo)

---

## ✅ 最终检查清单

在发布前，确保：

- [ ] 所有代码都通过了Lint和TypeScript检查
- [ ] 所有单元测试都通过了
- [ ] 所有集成测试都通过了
- [ ] 测试覆盖率达到 ≥ 80%
- [ ] 代码审查通过了
- [ ] Task-代码-测试完全对应
- [ ] 没有console.log或TODO注释
- [ ] 没有hardcode的值（使用配置或常量）
- [ ] 性能指标达标
- [ ] 文档和注释完整

---

**遵循最佳实践，提高代码质量！** ✨

