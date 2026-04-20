# 性能测试提示词

用于指导实现子代理进行性能测试的设计和实现。

---

## 什么是性能测试

性能测试验证应用在特定负载和环境条件下的表现，包括响应时间、吞吐量、资源使用等。

**关键指标**：
- **响应时间** - 处理请求需要多长时间 (ms)
- **吞吐量** - 单位时间内处理的请求数 (requests/sec)
- **错误率** - 失败请求的比例 (%)
- **资源使用** - CPU、内存、磁盘使用情况

**四种性能测试**：
```
负载测试    → 正常负载下的性能基线
压力测试    → 系统极限在哪里
耐久性测试  → 长时间运行是否稳定
峰值测试    → 突发流量的表现
```

---

## 何时需要性能测试

### ✅ 应该做性能测试

1. **关键业务功能**
   - 用户登录
   - 商品搜索
   - 支付处理
   - 订单查询

2. **高并发场景**
   - 高访问量的接口
   - 实时推送
   - 秒杀活动
   - 黑五大促

3. **大数据处理**
   - 批量导入导出
   - 报表生成
   - 数据分析查询
   - 日志处理

4. **资源密集操作**
   - 图片/视频处理
   - PDF生成
   - 复杂计算
   - 数据库聚合

### ❌ 不需要性能测试

- 低频操作（管理后台的操作）
- 原型开发阶段
- 性能要求低的内部工具
- 单用户操作

---

## 性能测试设计指南

### 步骤1: 定义性能指标

明确要达到的性能目标。

#### 1.1 响应时间指标

```
P50 (Median) - 50%的请求在这个时间以内
P95         - 95%的请求在这个时间以内（关键）
P99         - 99%的请求在这个时间以内（SLA）
Max         - 最慢的请求

建议值：
- 简单查询  → P95 < 100ms
- 复杂查询  → P95 < 500ms
- 报表生成  → P95 < 2000ms
- 大文件上传 → P95 < 5000ms
```

#### 1.2 吞吐量指标

```
RPS (Requests Per Second) - 每秒请求数

建议值：
- 正常负载  → 1000 RPS
- 峰值负载  → 5000 RPS
- 压力测试  → > 10000 RPS

计算方式：
RPS = 并发用户数 × 用户请求速率
```

#### 1.3 错误率指标

```
失败请求 / 总请求 × 100%

建议值：
- 正常负载  → < 0.1%
- 峰值负载  → < 0.5%
- 压力测试  → < 2%

失败原因分类：
- 4xx 错误 - 客户端错误（应排除）
- 5xx 错误 - 服务器错误（需要追查）
- 超时     - 网络或服务器问题
- 连接失败 - 网络问题
```

#### 1.4 资源使用指标

```
CPU利用率
├─ 正常 → < 50%
├─ 高   → 50-80%
└─ 危险 → > 80%

内存使用
├─ 正常 → < 60% 可用内存
├─ 高   → 60-80%
└─ 危险 → > 80%（会导致OOM）

磁盘I/O
├─ 正常 → < 50%
└─ 高   → > 50%（会成为瓶颈）

网络带宽
├─ 正常 → < 50% 可用带宽
└─ 高   → > 50%
```

### 步骤2: 定义测试场景

#### 2.1 负载测试场景

```
目的：建立性能基线

场景设置：
时间段    用户数    说明
─────────────────────────
0-1m      0        准备阶段
1-5m      10       逐步增加用户
5-10m     100      正常负载
10-15m    100      保持稳定
15-20m    0        关闭用户

验证指标：
✅ P95响应时间 < 目标
✅ 错误率 < 0.1%
✅ CPU < 50%
✅ 内存稳定（无持续增长）
```

#### 2.2 压力测试场景

```
目的：找到系统的极限

场景设置：
时间段    用户数    说明
─────────────────────────
0-2m      100      基线负载
2-4m      200      增加
4-6m      500      增加
6-8m      1000     继续增加
8-10m     1000     保持
10-15m    ⬆️       继续增加直到失败

预期结果：
- 系统应在某个点失败
- 应识别瓶颈（CPU、内存、I/O）
- 记录最大吞吐量

通常结果：
< 100 RPS  → 单机无法应对
< 500 RPS  → 需要优化
< 5000 RPS → 可能需要扩展
> 5000 RPS → 表现良好
```

#### 2.3 耐久性测试场景

```
目的：验证长时间运行的稳定性

场景设置：
时间段    用户数    说明
─────────────────────────
持续24小时 正常负载  模拟真实使用

验证指标：
✅ 内存是否持续泄漏
✅ 响应时间是否逐渐降低
✅ 错误率是否增加
✅ 连接是否正常释放

常见问题：
- 内存泄漏 → Java/Go的垃圾回收问题
- 连接泄漏 → 数据库连接未关闭
- 缓存溢出  → 缓存策略不当
```

#### 2.4 峰值测试场景

```
目的：测试突发流量的处理能力

场景设置：
时间段    用户数    说明
─────────────────────────
0-2m      10        基线
2-2.5m    1000      突然增加（黑五、秒杀）
2.5-3m    1000      保持峰值
3-5m      10        恢复正常

验证指标：
✅ 峰值期间错误率是否可接受
✅ 队列/缓冲是否溢出
✅ 恢复时间（从峰值回到正常）
```

### 步骤3: 建立性能基准

记录初始的性能表现。

```typescript
// 性能基准记录
const performanceBaseline = {
  endpoint: '/api/search',

  // 负载测试基准（100个并发用户）
  loadTest: {
    p50: 45,    // ms
    p95: 120,   // ms
    p99: 250,   // ms
    rps: 800,   // requests/sec
    errorRate: 0.05, // %
  },

  // 资源使用基准
  resources: {
    cpu: 35,    // %
    memory: 512, // MB
    connections: 50
  },

  // 记录时间
  recordedAt: '2026-03-23',
  environment: 'staging',
  commitHash: 'abc123def456'
};
```

---

## 性能测试实现

### 工具选择

| 工具 | 推荐场景 | 特点 |
|------|---------|------|
| **k6** | 云原生、开源 | 用Go写的，性能好，脚本简洁 |
| **JMeter** | 全面测试 | 功能完整，学习曲线高 |
| **Locust** | Python项目 | Python写的，易于扩展 |
| **Gatling** | Scala/Java | 高吞吐量，轻量级 |
| **wrk** | 简单HTTP | 超轻量级，C写的 |
| **ab** | 快速测试 | Apache自带，最简单 |

**推荐**：k6（现代，易用，支持云执行）

### 实现模板

#### k6 脚本示例

```javascript
// tests/performance/search.load.js

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend, Counter } from 'k6/metrics';

// 自定义指标
const errorRate = new Rate('errors');
const searchDuration = new Trend('search_duration');
const successfulSearches = new Counter('successful_searches');

// 配置
export const options = {
  stages: [
    { duration: '1m', target: 10 },    // 1分钟内增加到10个用户
    { duration: '5m', target: 100 },   // 5分钟内增加到100个用户
    { duration: '10m', target: 100 },  // 保持100个用户10分钟
    { duration: '5m', target: 0 },     // 5分钟内关闭所有用户
  ],

  // 性能阈值
  thresholds: {
    'http_req_duration': ['p(95)<500'],  // 95%请求<500ms
    'http_req_failed': ['rate<0.1'],     // 错误率<0.1%
    'errors': ['rate<0.1'],
  },

  // 其他选项
  ext: {
    loadimpact: {
      projectID: 12345,
      name: 'Search API Performance Test'
    }
  }
};

// VU函数（虚拟用户）
export default function (data) {
  // 1. 登录
  let loginRes = http.post('http://localhost:3000/api/login', {
    email: 'test@example.com',
    password: 'password123'
  });

  let isLoginSuccessful = check(loginRes, {
    'login successful': (r) => r.status === 200,
  });

  if (!isLoginSuccessful) {
    errorRate.add(1);
    return;
  }

  let token = loginRes.json('token');

  // 2. 搜索（关键操作）
  const searchQueries = [
    'laptop',
    'iphone',
    'gaming pc',
    'headphones'
  ];

  const query = searchQueries[Math.floor(Math.random() * searchQueries.length)];

  const startTime = new Date();
  let searchRes = http.get(
    `http://localhost:3000/api/search?q=${query}`,
    {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    }
  );
  const duration = new Date() - startTime;

  // 记录指标
  searchDuration.add(duration);

  let isSearchSuccessful = check(searchRes, {
    'search status 200': (r) => r.status === 200,
    'search response time < 500ms': (r) => r.timings.duration < 500,
    'response has items': (r) => r.json('items').length > 0,
  });

  if (isSearchSuccessful) {
    successfulSearches.add(1);
  } else {
    errorRate.add(1);
  }

  // 3. 随机操作其他接口（模拟真实用户）
  if (Math.random() < 0.5) {
    const productId = searchRes.json('items.0.id');
    http.get(
      `http://localhost:3000/api/products/${productId}`,
      {
        headers: { 'Authorization': `Bearer ${token}` }
      }
    );
  }

  // 等待模拟思考时间
  sleep(Math.random() * 5 + 1); // 1-6秒
}

// 汇总统计
export function handleSummary(data) {
  return {
    'stdout': textSummary(data, { indent: ' ', enableColors: true }),
    'performance-test-result.json': JSON.stringify(data),
  };
}
```

#### JMeter 脚本示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2">
  <TestPlan guiclass="TestPlanGui" testclass="TestPlan">
    <elementProp name="TestPlan.user_defined_variables" elementType="Arguments"/>
  </TestPlan>

  <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup">
    <!-- 线程配置 -->
    <stringProp name="ThreadGroup.num_threads">100</stringProp>
    <stringProp name="ThreadGroup.ramp_time">300</stringProp>
    <stringProp name="ThreadGroup.duration">600</stringProp>
    <elementProp name="ThreadGroup.main_controller"
                 elementType="LoopController">
      <boolProp name="LoopController.continue_forever">false</boolProp>
      <stringProp name="LoopController.loops">-1</stringProp>
    </elementProp>
  </ThreadGroup>

  <!-- HTTP请求 -->
  <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy">
    <stringProp name="HTTPSampler.domain">localhost</stringProp>
    <stringProp name="HTTPSampler.port">3000</stringProp>
    <stringProp name="HTTPSampler.protocol">http</stringProp>
    <stringProp name="HTTPSampler.path">/api/search</stringProp>
    <stringProp name="HTTPSampler.method">GET</stringProp>
    <Arguments>
      <HTTPArgument name="q" value="laptop"/>
    </Arguments>
  </HTTPSamplerProxy>

  <!-- 声明 HTTP Header -->
  <HeaderManager guiclass="HeaderPanel" testclass="HeaderManager">
    <Arguments>
      <Header>
        <stringProp name="Header.name">Content-Type</stringProp>
        <stringProp name="Header.value">application/json</stringProp>
      </Header>
    </Arguments>
  </HeaderManager>

  <!-- 监听器 - 聚合报告 -->
  <ResultCollector guiclass="AggregateReport" testclass="ResultCollector">
    <stringProp name="filename">result.jtl</stringProp>
  </ResultCollector>
</jmeterTestPlan>
```

### 运行和分析

#### k6 运行命令

```bash
# 本地运行
k6 run tests/performance/search.load.js

# 输出结果到JSON
k6 run tests/performance/search.load.js \
  --out json=result.json

# 分析结果
k6 run tests/performance/search.load.js \
  --vus 100 \
  --duration 5m

# 云执行（k6 Cloud）
k6 cloud tests/performance/search.load.js
```

#### 结果分析

```
性能测试结果分析

HTTP请求统计
─────────────────────────────────────
请求数：50,000
成功：49,950 (99.9%)
失败：50 (0.1%)

响应时间统计
─────────────────────────────────────
平均 (Mean)：245ms
中位数 (P50)：180ms
95分位 (P95)：520ms
99分位 (P99)：890ms
最大值 (Max)：2,500ms

吞吐量
─────────────────────────────────────
平均RPS：167 req/s
峰值RPS：200 req/s

资源使用
─────────────────────────────────────
服务器CPU：45%
服务器内存：1.2GB / 8GB (15%)
数据库CPU：35%
数据库内存：512MB / 8GB (6%)

性能评分
─────────────────────────────────────
✅ P95 520ms < 目标 500ms ⚠️ 超过1ms
✅ 错误率 0.1% = 目标 0.1% ✅ 符合
✅ CPU 45% < 60% ✅ 安全
✅ 内存 15% < 70% ✅ 安全

建议
─────────────────────────────────────
1. P95响应时间略高，考虑：
   - 数据库查询优化（添加索引）
   - 结果缓存（Redis）
   - 查询分页限制

2. 可以扩展到1000 RPS
   - 当前瓶颈：数据库查询
   - 解决方案：读写分离、分片
```

---

## 常见问题和解决方案

### Q1: 如何识别性能瓶颈？

**A**: 排查步骤：

```
1. 检查CPU利用率
   ├─ < 50% → 不是CPU问题，可能是网络或I/O
   └─ > 80% → CPU瓶颈，需要优化算法或扩展

2. 检查内存使用
   ├─ 稳定 → 没有内存泄漏
   └─ 持续增长 → 内存泄漏，需要排查

3. 检查I/O（磁盘）
   ├─ iowait < 10% → I/O不是瓶颈
   └─ iowait > 20% → 磁盘I/O瓶颈

4. 检查网络
   ├─ 带宽使用 < 50% → 网络充足
   └─ 丢包率 > 0.1% → 网络问题

5. 检查应用日志
   ├─ 慢查询日志 → 数据库查询慢
   ├─ GC日志 → 垃圾回收导致停顿
   └─ 错误日志 → 重试增加负担

6. APM工具（New Relic, DataDog）
   ├─ 追踪每个请求
   ├─ 识别慢操作
   └─ 分布式追踪
```

### Q2: 性能下降后如何追踪原因？

**A**: 比较方法：

```
1. 对比基准
   当前性能基准 vs 历史基准
   → 如果下降 > 10% → 需要调查

2. 对比版本
   运行当前代码 vs 上一个稳定版本
   → 如果新版本慢 → 最近的改动有问题

3. 对比配置
   不同的JVM参数、数据库配置等
   → 确定最佳配置

4. 火焰图分析
   CPU使用时间分布
   → 找到最耗时的函数
```

### Q3: 性能测试中如何处理第三方API？

**A**: 三种方法：

```
1. Mock外部API（推荐）
   - 可靠性高（不依赖第三方）
   - 可测试极端情况
   - 快速反馈

2. 使用测试环境API
   - 更接近真实
   - 但可能有速率限制
   - 可能产生真实数据

3. 使用VCR（录制和回放）
   - 第一次真实调用，录制响应
   - 后续使用录制的响应
   - 平衡真实性和可靠性
```

### Q4: 如何确保性能测试的准确性？

**A**: 最佳实践：

```
1. 隔离测试环境
   - 独立的硬件
   - 没有其他负载
   - 网络质量稳定

2. 预热测试
   - 先运行几分钟让系统稳定
   - 再开始测量
   - 排除冷启动效应

3. 多次重复
   - 至少运行3次
   - 取平均值和标准差
   - 避免偶然结果

4. 记录环境信息
   - 硬件配置
   - 操作系统
   - 依赖软件版本
   - JVM参数等

5. 排除干扰因素
   - 关闭其他应用
   - 停止定时任务
   - 禁用监控收集
```

### Q5: 性能测试的结果如何保存和对比？

**A**: 实践方案：

```javascript
// 性能测试结果保存
const result = {
  timestamp: new Date().toISOString(),
  version: getAppVersion(),
  commit: getGitCommit(),

  metrics: {
    p95: 520,
    p99: 890,
    rps: 167,
    errorRate: 0.1
  },

  resources: {
    cpu: 45,
    memory: 1.2
  },

  environment: {
    hardware: 'AWS c5.2xlarge',
    os: 'Ubuntu 20.04',
    nodeVersion: 'v14.17.0'
  }
};

// 保存到数据库或文件
await savePerformanceResult(result);

// 对比
const previous = await getLatestResult();
const regression = compareResults(result, previous);

if (regression.p95 > 10) {
  // P95响应时间增加> 10% → 告警
  alertSlack(`Performance regression: P95 +${regression.p95}%`);
}
```

---

## 性能测试检查清单

在进行性能测试前，确保：

- [ ] 定义了清晰的性能指标（P95、RPS、错误率等）
- [ ] 建立了性能基准
- [ ] 准备了测试环境（隔离、稳定）
- [ ] 准备了测试数据（足够真实）
- [ ] Mock了外部依赖
- [ ] 设置了性能阈值
- [ ] 编写了性能测试脚本
- [ ] 进行了预热测试
- [ ] 运行了至少3次并取平均值
- [ ] 分析了结果并识别瓶颈
- [ ] 记录了环境配置
- [ ] 对比了历史基准

---

## 总结

性能测试是确保应用在真实负载下表现良好的关键：

✅ **早期测试**：在开发阶段而非上线后
✅ **持续监控**：建立性能基准，定期对比
✅ **聚焦关键**：优先测试高频操作
✅ **完整分析**：不仅看数字，要找原因

**从Phase 2继承的Context**：
- code-execute已通过TDD确保功能正确
- 现在code-test在Phase 3负责性能验证

**开始性能测试设计！** 🚀

