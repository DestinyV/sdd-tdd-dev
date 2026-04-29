---
name: code-test
description: |
  第5步：测试验证和闭环（Test阶段）

  输入：src/ + spec-dev/{requirement_desc_abstract}/ (来自code-execute的TDD实现)
  输出：tests/ + spec-dev/{requirement_desc_abstract}/testing/testing-report.md

  功能：对code-execute生成的代码进行全面审查和高层测试（集成、E2E、性能），执行闭环验证。
  ✅ 单元测试由code-execute的TDD流程保证，此处不重复

  核心职责：
  - 代码质量静态分析（Lint、TypeScript、复杂度）
  - 高层测试：集成测试、E2E测试、性能测试
  - 闭环验证：Task → 代码 → 测试 → 结果 完全对应
  - 输出详细的测试报告

  前置Skill：code-execute ✓ + code-task ✓
  下一步：/spec-archive (规范归档) 或 完成！✅
---

# code-test

## 职责

对code-execute生成的代码进行全面验证和测试。

**核心流程**：
1. 代码质量静态分析（Lint、TypeScript、复杂度）
2. 高层测试设计和执行（集成、E2E、性能）
3. 闭环验证（验收标准 → 测试 → 代码 → 结果）
4. 输出测试报告和验证矩阵

**输出路径**：`spec-dev/{requirement_desc_abstract}/testing/testing-report.md`

## 何时使用

**前置条件**：
- ✅ code-execute 已完成，所有Task都通过了规范和质量审查
- ✅ code-task 已完成，任务列表已明确（标准模式）
- ✅ 所有源代码已生成且编译通过
- ✅ **模式标签读取**：从 `spec-dev/{requirement_desc_abstract}/spec/requirement.md` 读取工作模式标签
- ✅ **快速模式**：code-execute 已通过质量审查（跳过规范审查和 task）

### 快速模式执行（当 spec 标注为快速模式时）

**前提**：requirement.md 中 `模式: quick` 标签已确认。

在快速模式下，code-test 执行以下精简流程：

1. **步骤1**：代码质量静态分析（Lint + TypeScript）—— 保持不变
2. **步骤2**：代码审查 —— 只做质量审查（跳过规范审查）
3. **步骤3**：高层测试 —— 精简范围：
   - ✅ 执行：集成测试（3.1）
   - ✅ 执行：浏览器 E2E 测试（3.2）
   - ⚡ 跳过：视觉回归测试（3.3）
   - ⚡ 跳过：组件 UI 测试（3.4）
   - ⚡ 跳过：MCP Browser 测试（3.5）
   - ⚡ 跳过：性能测试（3.6）
4. **步骤4**：闭环验证 —— 保持不变
5. **步骤5**：生成精简测试报告（只包含实际执行的测试类型）
6. **覆盖率要求**：根据模式标签中的 `测试覆盖要求` 判定（≥60% 为 quick 模式标准）

---

## 工作流程

<HARD-GATE>
在前端/全栈场景中，未实际执行浏览器 E2E 测试（或确认项目无 Playwright 并给出安装指引）
之前，不声明测试通过，不生成测试报告，不进行闭环验证。
无论任务看起来多么简单，此规则都适用。
</HARD-GATE>

### 步骤1: 代码质量静态分析

运行项目的Lint和TypeScript检查：

```bash
npm run lint        # ESLint 检查
npm run type-check  # TypeScript strict 检查
npm run test:unit   # 单元测试（应 ≥ 85% 覆盖率）
```

**检查标准**：
- ✅ 0个Lint错误
- ✅ 0个TypeScript错误
- ✅ 无any类型（strict模式）
- ✅ 单元测试覆盖率 ≥ 85%（语句、分支、函数都要达标）

**单元测试真实性验证**（额外检查）：
- ✅ 测试覆盖了正常路径、边界情况、错误处理
- ✅ 每个测试只验证一个行为
- ✅ 断言明确具体（不使用 toBeTruthy、toBeFalsy 等模糊断言）
- ✅ 验收标准（TEST-VERIFY）都有对应的单元测试
- ❌ 排除虚假测试（always-true、未测试业务逻辑、跳过错误处理）

如有问题，返回code-execute修复。

### 步骤2: 代码审查

对代码进行系统审查，检查：

| 维度 | 检查点 |
|------|--------|
| **功能完整性** | Task定义的所有功能都实现了？所有边界、错误情况都处理了？ |
| **代码质量** | 易读易维护？职责单一？有适当抽象和复用？ |
| **类型安全** | 所有参数和返回值都有类型？避免了any？ |
| **性能** | 有不必要的重新渲染/计算？正确使用了优化API（memo、useMemo等）？ |
| **一致性** | 与design.md一致？与项目规范一致？ |

详见：`references/code-reviewer.md`

### 步骤3: 高层测试设计和执行

#### 3.0 后端测试基础设施检测 ⭐🆕（后端/全栈场景）

> 在执行任何高层测试之前，先检测项目是否具备后端测试能力。

**检测项目**：

| 检测项 | 检测方式 | 未检测到时的处理 |
|--------|---------|----------------|
| 后端测试框架 | 检查 package.json/pom.xml/go.mod 中是否有 jest/vitest/pytest/junit/go test | 使用 AskUserQuestion 询问是否添加 |
| HTTP 测试工具 | 检查是否有 supertest/httpx/rest-assured/httptest | 建议使用对应技术栈的工具 |
| 数据库测试 | 检查是否有内存数据库/测试数据库配置 | 建议使用内存数据库（SQLite/H2） |
| 测试配置文件 | 检查 jest.config.js / pytest.ini / 等是否存在 | 生成基础配置文件 |

**AskUserQuestion 模板**：

```
question: "项目当前缺少后端测试基础设施，是否要添加？"
header: "后端测试能力"
multiSelect: true
options:
  - { label: "添加 API 测试框架", description: "Node.js: jest+supertest / Python: pytest+httpx / Java: junit+rest-assured" }
  - { label: "添加集成测试配置", description: "内存数据库、测试环境变量、数据库隔离策略" }
  - { label: "添加性能测试工具", description: "k6（推荐）或 JMeter，用于 API 压测" }
  - { label: "跳过", description: "暂不添加后端测试基础设施（⚠️ 不推荐）" }
```

**添加测试基础设施**：

根据项目技术栈生成对应的基础配置：

| 技术栈 | 需添加的文件 |
|--------|------------|
| Node.js | `package.json` 中添加 jest/supertertest 依赖，生成 `jest.config.js` |
| Python | 生成 `pytest.ini`、`conftest.py`、`requirements-test.txt` |
| Java | 在 `pom.xml`/`build.gradle` 中添加测试依赖 |
| Go | 生成 `go test` 所需的 `*_test.go` 骨架文件 |

**检测通过后**，继续执行后续测试步骤。

#### 3.1 后端集成测试 ⭐🆕（后端/全栈场景）

测试服务端各组件/模块的协作：

```bash
npm run test:integration  # Node.js
pytest tests/integration/  # Python
mvn test -Dtest=IntegrationTest  # Java
go test ./tests/integration/...  # Go
```

**覆盖范围**：
- Repository/DAO 层与数据库的交互
- Service 层业务逻辑（使用真实 Repository，Mock 外部依赖）
- API 层与 Service 层的协作
- 事务边界和一致性验证
- 缓存策略正确性（Redis 读/写/过期）

**数据库测试策略**：
| 策略 | 适用场景 | 说明 |
|------|---------|------|
| 内存数据库 | 首选 | SQLite（Node.js/Python）、H2（Java）—— 每次测试重建 |
| 事务回滚 | 需要真实 DB 时 | 每个测试在事务中执行，测试结束回滚 |
| 测试数据库 | 复杂集成场景 | 独立测试数据库，测试前清空数据 |

**集成测试模板**：`templates/backend-api-test.template.ts`

#### 3.1.5 接口契约测试 ⭐🆕（fullstack 模式强制）

<HARD-GATE>当 project-mode 为 fullstack 时，必须执行接口契约测试，验证后端实际返回的字段、类型、结构与 api-contract.md 定义一致。契约测试失败意味着实现与接口契约不一致，必须修复后重新运行测试，不允许跳过。</HARD-GATE>

**测试内容**：

| 测试类型 | 验证内容 |
|---------|---------|
| 契约测试（后端） | 后端实际返回的字段、类型、结构与 api-contract.md 定义一致 |
| 契约测试（前端） | 前端发送的请求格式与 api-contract.md 定义一致 |
| 错误码覆盖测试 | api-contract.md 中定义的所有错误码在后端都有对应实现 |
| 边界条件测试 | 空值、超长字符串、特殊字符等边界情况的接口行为符合契约定义 |

**测试生成**：

基于 `api-contract.md` 和 `templates/contract-test-template.md` 自动生成：
- 后端契约测试用例：针对每个接口，验证响应字段、类型、错误码
- 前端集成测试用例：针对每个接口调用，验证请求格式、响应解析

**失败处理**：
- 契约测试失败 → 输出详细差异报告（期望值 vs 实际值）
- 必须修复后重新运行测试
- 不允许跳过契约测试

#### 3.2 浏览器 E2E 测试 ⭐（前端/全栈场景）

通过 Playwright 无头浏览器执行端到端测试，验证完整的用户操作流程：

```bash
npx playwright test --grep "@e2e"
```

**验证内容**：
- 完整用户操作流程（登录、表单提交、导航等）
- 页面跳转和 URL 变化
- DOM 元素状态和文本内容（使用 data-testid）
- 数据持久化验证
- 错误处理和边界情况

**环境检查**：
```bash
npx playwright --version  # 检查 Playwright 是否安装
```

如未安装，参考安装指引：`references/frontend-browser-testing.md`

详见：`references/frontend-browser-testing.md`（环境配置、测试编写、工具函数）

#### 3.3 视觉回归测试 ⭐（前端/全栈场景）

通过 Playwright 截图对比检测 UI 视觉变化：

```bash
npx playwright test --grep "@visual"
```

**验证内容**：
- 关键页面布局无异常变化（桌面/移动/平板视口）
- 组件渲染一致性
- 响应式布局正确性
- 暗色/亮色主题切换

**基准截图管理**：
- 首次运行生成基准：`npx playwright test --grep "@visual" --update-snapshots`
- 更新基准（有意变更）：`npx playwright test --update-snapshots`

详见：`references/frontend-browser-testing.md`

#### 3.4 组件 UI 测试 ⭐（前端/全栈场景）

通过 Playwright 或 Vitest + Testing Library 测试组件级渲染和交互：

```bash
npx playwright test --grep "@component"
# 或
npm run test:component
```

**验证内容**：
- 组件渲染正确性
- Props 传递和默认值
- 事件触发和回调
- 交互行为（点击、悬停、键盘导航）
- 快照对比

详见：`references/frontend-browser-testing.md`

#### 3.5 MCP Browser 探索性测试 ⭐（前端/全栈场景）

通过 MCP Browser Server 进行 AI 驱动的交互式浏览器测试：

1. **配置 MCP Server**：参考 `templates/mcp-browser-server.md`
2. **导航验证**：操控浏览器访问各页面，截图验证渲染
3. **交互探索**：点击各交互元素，验证响应正确性
4. **状态检查**：执行 JavaScript 检查 localStorage/Cookie/页面状态

**适用场景**：
- 探索性测试和一次性验证
- 截图验证关键页面渲染
- 快速验证交互流程

详见：`templates/mcp-browser-server.md`

#### 3.6 性能测试

测试性能指标：

```bash
npm run test:performance
```

**覆盖范围**：
- 加载时间
- 交互响应时间
- 内存占用

**后端性能测试** ⭐🆕（后端/全栈场景）：

```bash
k6 run tests/performance/api-load-test.js  # k6 API 压测
```

| 指标 | 目标值 | 检测方式 |
|------|--------|---------|
| P95 响应时间 | < 500ms | k6 / JMeter |
| P99 响应时间 | < 1000ms | k6 / JMeter |
| 错误率 | < 1% | k6 / JMeter |
| RPS（每秒请求数） | > 100 | k6 / JMeter |
| CPU 使用率 | < 80% | 系统监控 |
| 内存使用率 | < 80% | 系统监控 |

**性能测试模板**：`references/performance-test-prompt.md`（k6 + JMeter 完整脚本示例）

### 步骤4: 闭环验证

生成验证矩阵，确保：
- **TEST-VERIFY（规范）** → **测试用例（实现）** → **测试结果（通过）** 完全对应
- 每个验收标准都有对应的测试
- 每个测试都通过

**验证矩阵示例**：

| 验收标准 | 测试类型 | 测试用例 | 结果 |
|---------|---------|---------|------|
| 用户能创建订单 | E2E | test_user_create_order | ✅ 通过 |
| 订单金额正确计算 | 集成 | test_order_total_calculation | ✅ 通过 |
| 系统在100ms内响应 | 性能 | test_order_api_response_time | ✅ 通过 |

### 步骤5: 生成测试报告

**在生成报告前，必须**：
1. 实际运行所有测试命令（不能引用之前的结果）
2. 捕获并记录完整输出
3. 验证每个 TEST-VERIFY 都有对应通过的测试
4. 前端项目：确认浏览器 E2E 测试已执行或给出安装指引

报告必须包含实际运行的命令和输出，而非引用历史记录。

**验证铁律**：
```
未运行测试命令 → 不能声称测试通过
```

生成报告到 `spec-dev/{requirement_desc_abstract}/testing/testing-report.md`：

```markdown
# 测试报告

## 测试总览
- 总Task数：4
- 代码质量：✅ 通过（0 Lint错误，0 TS错误）
- 单元测试：✅ 通过（覆盖率 87%）
- 集成测试：✅ 通过（12/12）
- 浏览器 E2E 测试：✅ 通过（8/8）⭐
- 视觉回归测试：✅ 通过（6/6，0 差异）⭐
- 组件 UI 测试：✅ 通过（15/15）⭐
- MCP Browser 测试：✅ 通过（探索性验证完成）⭐
- 性能测试：✅ 通过（所有指标达标）

## 质量指标
- 平均覆盖率：87%
- Lint错误：0
- TypeScript错误：0
- 代码复杂度：正常范围

## 浏览器测试结果
| 测试类型 | 通过 | 失败 | 跳过 |
|---------|------|------|------|
| E2E (Chromium) | 8 | 0 | 0 |
| E2E (Mobile) | 4 | 0 | 0 |
| 视觉回归 | 6 | 0 | 0 |
| 组件 UI | 15 | 0 | 0 |

## 闭环验证
- 总验收标准数：18
- 对应的测试数：18（100%）
- 通过的测试数：18（100%）
- BROWSER-TESTABLE 覆盖率：100%

## 总结
代码质量达标，所有测试通过，浏览器测试全覆盖，可以发布！✅
```

---

## 关键约束

✅ **必须做**：
- 所有Lint和TypeScript检查通过
- 单元测试覆盖率 ≥ 85%（来自code-execute的TDD实现）
- 验证单元测试的真实性（覆盖正常路径、边界、错误处理、业务规则）
- 高层测试（集成、E2E、性能）全覆盖
- **（后端/全栈场景）后端测试基础设施检测** ⭐🆕
- **（后端/全栈场景）集成测试覆盖 Repository/Service/API 层** ⭐🆕
- **（后端/全栈场景）API 契约测试通过（fullstack 模式强制）** ⭐🆕
- **（前端/全栈场景）浏览器 E2E 测试全覆盖** ⭐
- **（前端/全栈场景）视觉回归测试覆盖关键页面** ⭐
- **（前端/全栈场景）组件 UI 测试覆盖所有组件** ⭐
- 闭环验证完成（100%对应，TEST-VERIFY → 测试 → 代码 → 结果）
- **BROWSER-TESTABLE 验收标准 100% 覆盖** ⭐

❌ **禁止做**：
- 降低测试标准以加快进度
- 跳过代码审查
- 修改源代码以适应测试（反向修改）
- 遗漏任何验收标准的测试
- 接收虚假的单元测试（always-true断言、未测试边界、跳过错误处理等）
- 为了凑覆盖率而添加无意义的测试
- **跳过浏览器测试（前端/全栈场景）** ⭐

---

## 相关资源

| 资源 | 说明 |
|------|------|
| `references/code-reviewer.md` | 代码审查指南和检查清单 |
| `references/testing-anti-patterns.md` | 🆕 测试反模式检查清单（Mock行为、不完整数据结构等） |
| `references/frontend-browser-testing.md` | ⭐ 前端浏览器测试完整指南（E2E、视觉回归、组件UI、MCP Browser） |
| `references/e2e-test-prompt.md` | E2E测试设计指南（已更新链接到新模板） |
| `references/performance-test-prompt.md` | 性能测试设计指南 |
| `references/pressure-scenarios/` | ⭐ Skill 压力测试场景定义（验证 Skill 在压力下的有效性） |
| `references/using-superpowers/` | ⭐ 多平台适配引用文档（Copilot/Codex/Gemini/OpenCode 工具映射） |
| `templates/playwright.config.ts` | ⭐ Playwright 配置模板（E2E + 视觉回归） |
| `templates/browser-test-helpers.ts` | ⭐ 浏览器测试工具函数库 |
| `templates/frontend-e2e-test.template.ts` | ⭐ E2E 测试代码模板 |
| `templates/visual-regression.template.ts` | ⭐ 视觉回归测试代码模板 |
| `templates/frontend-component-ui.template.ts` | ⭐ 组件 UI 测试代码模板 |
| `templates/backend-api-test.template.ts` | 🆕 后端 API 测试模板（多技术栈：Node.js/Python/Go） |
| `templates/backend-e2e-api-test.template.ts` | 🆕 后端 E2E API 测试模板（完整业务流程验证） |
| `templates/backend-db-migration-test.template.ts` | 🆕 数据库迁移测试模板（正向/回滚/数据完整性） |
| `templates/contract-test-template.md` | 🆕 接口契约测试模板（字段/类型/错误码验证） |
| `templates/mcp-browser-server.md` | ⭐ MCP Browser Server 配置和使用指南 |
| `scripts/run-browser-tests.sh` | ⭐ 浏览器测试自动化执行脚本 |

---

**关键理念**：确保代码质量、测试完整、闭环验证，最后才能发布。
