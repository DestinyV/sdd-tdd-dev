# Frontend SDD Best Practices Guide

This guide provides best practices and checklists for frontend development using the sdd-dev-plugin workflow.

---

## 📋 Frontend SDD Workflow Checklist

### Spec Stage (spec-creation)

#### Requirements Understanding
- [ ] Requirements description is clear and specific
- [ ] Core features have been identified
- [ ] Main scenarios have been initially extracted

#### Scenario Decomposition and Confirmation
- [ ] Requirements have been decomposed into specific business scenarios
- [ ] Each scenario has clear WHEN conditions and THEN results
- [ ] All business processes are covered
- [ ] Exception cases and boundary conditions are considered

#### Specification Documents
- [ ] spec-dev/{requirement_desc_abstract}/spec/README.md generated
- [ ] spec-dev/{requirement_desc_abstract}/spec/scenarios/*.md generated (BDD format)
- [ ] spec-dev/{requirement_desc_abstract}/spec/data-models.md generated
- [ ] spec-dev/{requirement_desc_abstract}/spec/business-rules.md generated
- [ ] spec-dev/{requirement_desc_abstract}/spec/glossary.md generated
- [ ] All specification documents reviewed and confirmed by user

#### Notes
- ⚠️ Specifications are the foundation for design and development, must be complete and accurate
- ⚠️ Use multi-round confirmation to ensure no details are missed
- ⚠️ Avoid specifications that are too general or vague

---

### Design Stage (code-designer)

#### Specification Analysis
- [ ] Specification documents (spec/) have been read
- [ ] All business scenarios and requirements understood
- [ ] Corresponding design patterns identified

#### Design Solution
- [ ] Design solution covers all features
- [ ] All design decisions have clear reasons
- [ ] Architecture is clear (component structure, data flow, state management)
- [ ] Props interfaces are clearly designed, unambiguous
- [ ] Non-functional requirements considered (performance, accessibility, responsive design)

#### Comparison with Reference Design
- [ ] Reference components identified
- [ ] Complete implementation of reference components analyzed
- [ ] Similarities and differences with reference listed
- [ ] Differences have clear reasons

#### Technical Solution
- [ ] Dependency libraries determined
- [ ] API design clarified
- [ ] State management solution determined (useState/useReducer/Redux/Zustand, etc.)
- [ ] Styling solution determined (CSS Modules/Tailwind/Styled Components, etc.)
- [ ] Data fetching method determined (Fetch/Axios/React Query, etc.)

#### Design Approval
- [ ] Design solution generated (design.md)
- [ ] All key decisions documented
- [ ] User reviewed and approved design solution

---

### Task Stage (code-task)

#### Task Decomposition
- [ ] Design decomposed into specific coding tasks
- [ ] Each Task has clear goals
- [ ] Task granularity appropriate (completable in 4-8 hours)
- [ ] Task dependencies are clear

#### Deliverable Definition
- [ ] Each Task defines specific deliverables
- [ ] Deliverables include: components/hooks/utility functions/tests/styles
- [ ] No ambiguous deliverable definitions

#### Acceptance Criteria
- [ ] Each Task has clear acceptance criteria
- [ ] Acceptance criteria are measurable (not subjective)
- [ ] Acceptance criteria include: functionality, quality, testing, documentation dimensions

#### Task Priority and Dependencies
- [ ] Basic components first, complex components later
- [ ] Tasks with dependencies clearly define predecessor Tasks
- [ ] Independent Tasks can be executed in parallel

---

### Execute Stage (code-execute)

#### Code Quality
- [ ] All code passed spec review
- [ ] All code passed quality review
- [ ] No Lint errors or warnings (strict eslint configuration)
- [ ] TypeScript strict mode error-free

#### Code Review
- [ ] Props design matches design.md
- [ ] Component structure similar to reference components
- [ ] Styling approach matches design.md solution
- [ ] API calls correct, error handling complete
- [ ] No obvious performance issues

#### Test Coverage
- [ ] Unit tests written (each Task has tests)
- [ ] Test coverage ≥ 80%
- [ ] Critical business logic has tests
- [ ] Exception cases have tests

#### Code Commit
- [ ] Code compiled and basic tests passed locally
- [ ] No console.log or TODO comments
- [ ] No outdated code comments
- [ ] Git commit messages are clear

---

### Test Stage (code-execute - Unit Testing)

**Important Note**: Unit testing is completed by `/code-execute` TDD workflow (RED-GREEN-REFACTOR-REVIEW). This section describes unit testing best practices. code-test does not repeat unit testing review, but focuses on Phase 3 high-level testing (see next section).

#### Code Quality Review
- [ ] ESLint check passed (0 errors)
- [ ] TypeScript strict check passed (0 errors)
- [ ] No unused imports or variables
- [ ] No any type (strictly followed)
- [ ] No console.log or TODO comments

#### Unit Testing
- [ ] All unit tests pass (100%)
- [ ] Tests cover normal, boundary, and exception scenarios
- [ ] Key components coverage ≥ 85%
- [ ] All Tasks have unit tests
- [ ] Test code has clear documentation comments

#### Integration Testing
- [ ] Multi-component integration tests executed
- [ ] Complete user workflow tests passed
- [ ] API integration tests passed (using mocks)
- [ ] Cross-module interaction tests passed

#### E2E Testing (Optional)
- [ ] Key user scenarios have E2E tests
- [ ] E2E tests pass in real environment
- [ ] Tests cover normal and exception flows

#### Closed-Loop Verification
- [ ] All Task features implemented
- [ ] All Task features have corresponding tests
- [ ] All code has corresponding tests
- [ ] Task-code-test complete correspondence
- [ ] No features implemented beyond Task scope

#### Final Report
- [ ] Test report generated (testing-report.md)
- [ ] Coverage analysis completed
- [ ] All issues documented
- [ ] Closed-loop verification matrix generated

### Phase 3 Stage (code-test - High-Level Testing) ✨

**Important Note**: Phase 3 code-test focuses on **integration testing, E2E testing, performance testing**, **does not repeat unit testing review** (guaranteed by code-execute TDD workflow).

#### Integration Testing Best Practices

**When Needed**:
- [ ] Multiple Tasks/modules need to collaborate
- [ ] Cross-module data flow needs verification
- [ ] API and database linkage needs verification

**Design Principles**:
- [ ] Clear test boundaries (which Tasks collaborate)
- [ ] Reasonable mock strategy (external dependencies mocked, internal real)
- [ ] Independent test data (no dependency on other tests)
- [ ] Complete data cleanup (beforeEach/afterEach)

**Common Issues**:
- ❌ Avoid test dependencies (Test A result affects Test B)
- ❌ Avoid unclear mock strategy (don't know what to mock, what to keep real)
- ❌ Avoid improper database state management (initialization, cleanup, rollback)
- ✅ Use transaction rollback or appropriate data cleanup
- ✅ Clean mocks in beforeEach, ensure test isolation

**Reference**: `code-test/integration-test-prompt.md`

#### E2E Testing Best Practices

**When Needed**:
- [ ] Complete business workflows need verification
- [ ] User end-to-end scenarios need verification
- [ ] Frontend-backend linkage needs verification

**Design Principles**:
- [ ] Real user scenarios (from user perspective)
- [ ] Complete verification points (UI, database, backend API)
- [ ] Avoid flaky tests (use explicit waits, not sleep)
- [ ] Tests independent and repeatable (beforeEach resets initial state)

**Common Issues**:
- ❌ Avoid fixed delays `setTimeout(1000)` → use `waitFor()`
- ❌ Avoid fragile selectors (easily broken by UI changes)
- ❌ Avoid missing `await` (async operations not waited)
- ❌ Avoid test dependencies (previous test result affects next)
- ✅ Use `page.waitFor()`, `page.waitForSelector()` explicit waits
- ✅ Use `data-testid` instead of text or class selectors
- ✅ Correctly handle navigation and page loading
- ✅ beforeEach resets application state or cleans data

**Reference**: `code-test/e2e-test-prompt.md`

#### Performance Testing Best Practices

**When Needed**:
- [ ] Critical business workflows (search, payment, login, etc.)
- [ ] High concurrency scenarios (expected user load)
- [ ] Performance baselines needed

**Design Principles**:
- [ ] Clear performance metrics (P95, RPS, error rate)
- [ ] Complete test scenarios (normal load, stress, endurance)
- [ ] Realistic performance goals (based on business requirements)
- [ ] Traceable results (record baseline data, identify regressions)

**Common Issues**:
- ❌ Avoid metrics without baselines (don't know what is good)
- ❌ Avoid incomplete test scenarios (missing stress, endurance tests)
- ❌ Avoid insufficient warmup (first run affects results)
- ✅ Define P95, P99 response time targets
- ✅ Define RPS (throughput) and error rate thresholds
- ✅ Define CPU, memory, network usage limits
- ✅ Include Load (normal), Stress, Endurance (24 hours), Spike (burst) scenarios
- ✅ Performance goals based on business requirements (e.g., P95 < 200ms, error rate < 0.1%)

**Reference**: `code-test/performance-test-prompt.md`

#### Closed-Loop Verification

- [ ] All TEST-VERIFY (defined in specs) have corresponding tests
- [ ] Unit tests → verify Task single features
- [ ] Integration tests → verify multi-Task collaboration
- [ ] E2E tests → verify complete business workflows
- [ ] Performance tests → verify system performance baselines
- [ ] Verification matrix: TEST-VERIFY → Test Case → Test Code → Results

---

## 🎯 Frontend Development Best Practices

### 1. Component Design

#### Props Design
```typescript
// ✅ Good practice
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  /** Button visual style */
  variant?: 'primary' | 'secondary' | 'danger';
  /** Button size */
  size?: 'small' | 'medium' | 'large';
  /** Whether disabled */
  disabled?: boolean;
  /** Click callback */
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  /** Children */
  children: React.ReactNode;
}

// ❌ Avoid
interface ButtonProps {
  // Too flexible, prone to unexpected behavior
  props?: any;
  // No types, hard to maintain
  onClick?: Function;
}
```

**Principles**:
- Use TypeScript for type definitions, avoid any
- Props should have JSDoc comments
- Avoid excessive Props, use config object for complex components
- Provide reasonable defaults

#### Component Responsibilities
```typescript
// ✅ Single Responsibility Principle
const OrderList: React.FC<OrderListProps> = (props) => {
  // Only responsible for rendering and managing list state
  // API calls passed through props
  // Styles handled through CSS Modules
};

// ❌ Violates Single Responsibility
const OrderList: React.FC<OrderListProps> = (props) => {
  // Handles: API calls, business logic, UI rendering
  // Hard to test and reuse
};
```

**Principles**:
- Each component has single responsibility, easy to understand and maintain
- Separate container and presentational components
- Extract complex business logic into custom Hooks

### 2. State Management

#### Local State
```typescript
// ✅ Use useState for simple state
const [count, setCount] = useState(0);

// ✅ Use useReducer for complex state
const [state, dispatch] = useReducer(reducer, initialState);

// ❌ Avoid excessive state
const [user, setUser] = useState<any>(null);  // Should have detailed type
```

**Principles**:
- Use useState for simple state
- Use useReducer for complex or multiple related states
- Avoid storing derived data in state (should use useMemo)

#### Global State
```typescript
// Choose one solution and strictly follow
// Option 1: Redux + Redux Toolkit
// Option 2: Zustand
// Option 3: Recoil
// Option 4: Jotai
```

**Principles**:
- Determine global state management solution during spec-creation
- All projects use the same solution consistently
- Only put truly global data in global store

### 3. Performance Optimization

#### Avoid Unnecessary Re-renders
```typescript
// ✅ Use useMemo and useCallback
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

const handleClick = useCallback(() => {
  onItemClick(item);
}, [item, onItemClick]);

// ✅ Use React.memo for presentational components
const OrderItem = React.memo<OrderItemProps>(({ order }) => {
  return <div>{order.name}</div>;
});

// ❌ Avoid
// Creating new objects/functions on every render
const config = { mode: 'edit' };  // New object every time
const onClick = () => { /* ... */ };  // New function every time
```

**Principles**:
- Use useMemo and useCallback only when necessary (after profiling)
- Use React.memo for presentational components to prevent unnecessary re-renders
- Avoid creating new objects or functions in render

#### Virtual Scrolling
```typescript
// ✅ Use virtual scrolling for large lists
<VirtualList
  items={items}
  itemHeight={50}
  height={500}
  renderItem={(item) => <OrderItem order={item} />}
/>

// ❌ Avoid rendering thousands of DOM nodes
items.map(item => <OrderItem key={item.id} order={item} />)
```

**Principles**:
- Consider virtual scrolling for lists > 100 items
- Use react-window or react-virtualized libraries

### 4. Style Management

#### CSS Solution
```typescript
// Choose one solution and strictly follow:
// Option 1: CSS Modules (Recommended - simple, enforced isolation)
// Option 2: Tailwind CSS (Recommended - rapid development)
// Option 3: Styled Components (Flexible but needs performance attention)
// Option 4: BEM + SCSS (Traditional, heavier)
```

**Principles**:
- Determine styling solution during spec-creation
- All projects use the same solution consistently
- Avoid inline styles (affects performance and maintainability)

#### Colors and Spacing
```typescript
// ✅ Use design tokens
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

// ❌ Avoid hardcoded colors and spacing
const buttonStyle = {
  background: '#007bff',
  padding: '8px 16px',
  marginBottom: '16px',
};
```

**Principles**:
- Define design tokens (colors, spacing, fonts, etc.)
- All styles reference tokens
- Use CSS variables for theme switching support

### 5. TypeScript Best Practices

#### Type Definitions
```typescript
// ✅ Complete type definitions
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

type UserId = number & { readonly __brand: 'UserId' };

// ✅ Use generics to avoid repetition
const createList = <T,>(items: T[]): List<T> => {
  return new List(items);
};

// ❌ Avoid any
const handleClick = (event: any) => {
  // Loses type checking
};

// ❌ Avoid as type assertions
const data = json as any as User;  // Dangerous!
```

**Principles**:
- Enable TypeScript strict mode
- Avoid using any, use unknown with type guards when necessary
- Annotate function parameters and return types
- Define types for API responses

#### Type Guards
```typescript
// ✅ Use type guards
function isUser(obj: any): obj is User {
  return obj && typeof obj.id === 'number' && typeof obj.name === 'string';
}

if (isUser(data)) {
  console.log(data.id);  // Safe access
}

// ✅ Use type discrimination
type Result = { success: true; data: User } | { success: false; error: string };

if (result.success) {
  console.log(result.data);  // Type automatically narrowed
}
```

### 6. Error Handling

#### API Error Handling
```typescript
// ✅ Complete error handling
try {
  const response = await fetchUser(userId);
  return response.data;
} catch (error) {
  if (error instanceof NetworkError) {
    // Handle network errors
    showNotification('Network error, please retry');
  } else if (error instanceof ValidationError) {
    // Handle validation errors
    showNotification(error.message);
  } else {
    // Handle unknown errors
    logger.error('Unknown error', error);
    showNotification('Something went wrong');
  }
}

// ❌ Avoid empty catch blocks
try {
  await fetchUser(userId);
} catch (error) {
  // Ignore all errors
}
```

**Principles**:
- Provide different handling for different error types
- Display meaningful error messages to users
- Log errors for debugging

#### Error Boundaries
```typescript
// ✅ Use ErrorBoundary to catch component errors
<ErrorBoundary fallback={<ErrorPage />}>
  <OrderList />
</ErrorBoundary>

// ❌ Ignore component render errors
<OrderList />  // If render errors, entire page crashes
```

### 7. Testing Best Practices

#### Unit Testing
```typescript
// ✅ Clear test structure
describe('OrderForm', () => {
  describe('Normal Flow', () => {
    test('should render correctly', () => {
      // Arrange
      const props = { onSubmit: jest.fn() };
      // Act
      render(<OrderForm {...props} />);
      // Assert
      expect(screen.getByText(/order form/)).toBeInTheDocument();
    });
  });

  describe('Edge Cases', () => {
    test('should handle empty values', () => {
      render(<OrderForm items={[]} onSubmit={jest.fn()} />);
      expect(screen.getByText(/no items/)).toBeInTheDocument();
    });
  });
});

// ❌ Avoid vague tests
test('everything works', () => {
  // Too broad, don't know what was tested
  expect(true).toBe(true);
});
```

**Principles**:
- Each test focuses on one feature
- Use clear test names
- Follow Arrange-Act-Assert structure
- Avoid testing implementation details, test behavior

#### Test Coverage
- **Target**: ≥ 80% line coverage
- **Focus**: Business logic, API calls, error handling must be covered
- **Can skip**: Style code, UI details, unlikely branches

---

## 🚀 Workflow Optimization Suggestions

### 1. Design Stage
- ✅ Thorough design discussions prevent later modifications
- ✅ Clear design documents accelerate Task stage
- ✅ Confirmation from all stakeholders reduces changes

### 2. Task Stage
- ✅ Detailed Task definitions improve Execute efficiency
- ✅ Clear acceptance criteria accelerate Test verification
- ✅ Reasonable task granularity prevents Tasks from being too large

### 3. Execute Stage
- ✅ Strict review process catches issues early
- ✅ Fix cycles maintain quality
- ✅ Detailed execution reports facilitate tracking

### 4. Test Stage
- ✅ Complete test coverage ensures feature completeness
- ✅ Closed-loop verification ensures Task-code-test consistency
- ✅ Detailed test reports facilitate future maintenance

---

## 📊 Quality Metrics

### Code Quality Metrics

| Metric | Target | Description |
|--------|--------|-------------|
| Lint Check | 0 errors | Use ESLint strict configuration |
| TypeScript | 0 errors | Strict mode, no any |
| Code Coverage | ≥ 80% | All modules tested |
| Cyclomatic Complexity | < 10 | Single function control flow not too complex |
| Function Lines | < 50 | Single function not too long |

### Test Quality Metrics

| Metric | Target | Description |
|--------|--------|-------------|
| Unit Test Pass Rate | 100% | All tests pass |
| Integration Test Pass Rate | 100% | Module integration tests pass |
| E2E Test Pass Rate | 100% | User workflow tests pass |
| Test Coverage | ≥ 80% | Code covered by tests |
| Critical Path Coverage | 100% | Critical business workflows fully covered |

### Performance Metrics

| Metric | Target | Description |
|--------|--------|-------------|
| First Screen Load Time | < 2s | Adjust based on project |
| Response Time | < 100ms | User interaction response time |
| Memory Usage | < 100MB | Runtime memory usage |
| Bundle Size | < 500KB | Optimized bundle size |

---

## 🎓 Learning Resources

### Frontend Frameworks
- [React Official Documentation](https://react.dev)
- [Vue Official Documentation](https://vuejs.org)
- [Angular Official Documentation](https://angular.io)

### TypeScript
- [TypeScript Official Documentation](https://www.typescriptlang.org)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

### Testing
- [Jest Documentation](https://jestjs.io)
- [React Testing Library](https://testing-library.com/react)
- [Cypress Documentation](https://cypress.io)

### Performance Optimization
- [Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/reference/react/useMemo)

---

## ✅ Final Checklist

Before release, ensure:

- [ ] All code passed Lint and TypeScript checks
- [ ] All unit tests passed
- [ ] All integration tests passed
- [ ] Test coverage ≥ 80%
- [ ] Code review passed
- [ ] Task-code-test complete correspondence
- [ ] No console.log or TODO comments
- [ ] No hardcoded values (use config or constants)
- [ ] Performance metrics met
- [ ] Documentation and comments complete

---

**Follow best practices, improve code quality!** ✨
