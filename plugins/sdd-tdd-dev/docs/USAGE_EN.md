# Usage Guide

## Quick Start (5 Minutes)

The Spec-Driven Development workflow consists of 6 stages:

### Step 1: Requirements Specification

```bash
/spec-creation [Requirements Description]
```

The AI performs requirements analysis and generates specification documents through interactive confirmation.

**Input**:
- Requirements title
- Detailed requirements description
- Related background information

**Output**: `spec-dev/{requirement_desc_abstract}/spec/`
- `README.md` - Specification overview
- `scenarios/*.md` - BDD format scenarios (WHEN-THEN)
- `data-models.md` - Data models
- `business-rules.md` - Business rules
- `glossary.md` - Glossary

### Step 2: Code Design Planning

```bash
/code-designer
```

The AI performs design analysis based on specifications and generates a detailed design plan.

**Output**: `spec-dev/{requirement_desc_abstract}/design/design.md`
- Requirements analysis
- Design solution (architecture, components, state management, data flow, styles, interactions)
- Comparison with reference designs
- Technical solution and API design
- Design decision records

### Step 3: Task List Generation

```bash
/code-task
```

The AI converts the design solution into a detailed code-level task list.

**Output**: `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`
- Task dependencies
- Detailed Task definitions (goals, deliverables, acceptance criteria)
- Quality standards

### Step 4: Code Execution (TDD Workflow)

```bash
/code-execute
```

The AI assigns sub-agents for each Task implementation with multi-stage review.

**Output**:
- `src/` - Generated source code
- `spec-dev/{requirement_desc_abstract}/execution/execution-report.md` - Execution report

**Execution Process**:
- Implementer sub-agent: Code implementation
- Spec Review: Verify code conforms to design
- Quality Review: Check code quality
- Fix Cycle: Fix issues and re-review

### Step 5: Testing and Verification

```bash
/code-test
```

The AI performs comprehensive testing and closed-loop verification.

**Output**:
- `tests/` - Generated test files
- `spec-dev/{requirement_desc_abstract}/testing/testing-report.md` - Test report

**Includes**:
- Code quality review (Lint, TypeScript check)
- TDD unit testing (RED-GREEN-REFACTOR-REVIEW): Completed by code-execute through TDD workflow
- High-level testing (Integration, E2E, Performance): Executed by code-test
- Closed-loop verification (TEST-VERIFY→Test→Code→Result)
- Coverage analysis (Unit tests ≥80%, complete Integration/E2E/Performance verification)

### Step 6: Spec Archiving

```bash
/spec-archive
```

(Automatically triggered after all tests pass in code-test)

The AI archives verified specifications to the enterprise spec library.

**Output**:
- `spec-dev/spec/` - Updated main spec library
- `spec-dev/spec/archive-report.md` - Archive report

---

## Detailed Usage Workflow

### Scenario 1: React Project - Order Management Form

#### 1.1 Requirements Specification
```bash
/spec-creation Need to implement order management form supporting search, sort, pagination, batch operations

# AI performs interactive analysis, generates specs through multi-round confirmation
# - Decomposes requirements into business scenarios
# - Refines and confirms each scenario
# - Generates BDD format specification documents
```

**Output**: `spec-dev/REQ-001/spec/`
- README.md - Specification overview
- scenarios/scenario-1.md etc. - Scenario details (WHEN-THEN format)
- data-models.md - Data models
- business-rules.md - Business rules
- glossary.md - Glossary

#### 1.2 Code Design Planning
```bash
/code-designer

# AI will:
# 1. Read specs from spec-dev/REQ-001/spec/
# 2. Analyze requirements, identify design patterns
# 3. Perform requirement-reference difference analysis
# 4. Generate detailed design solution
```

**Output**: `spec-dev/REQ-001/design/design.md`
- Requirements analysis (core features, user scenarios)
- Design solution (architecture, component design, state management, data flow, styles, interactions)
- Comparison with reference designs
- Technical solution and API design
- Design decision records

#### 1.3 Task List Generation
```bash
/code-task

# AI will:
# 1. Analyze design solution in design.md
# 2. Decompose into specific coding tasks
# 3. Define goals, deliverables, acceptance criteria for each Task
```

**Output**: `spec-dev/REQ-001/tasks/tasks.md`
- Task dependency graph
- Task 1: Implement OrderTable component
- Task 2: Implement search and sort functionality
- Task 3: Implement pagination
- Task 4: Implement batch operations
- Task 5: Write tests

#### 1.4 Code Execution (TDD Workflow)
```bash
# User reviews design and tasks, approves execution

/code-execute

# AI will:
# 1. Assign independent sub-agents for each Task
# 2. Execute TDD workflow:
#    🔴 RED Stage: Write failing unit tests (test framework from test-spec.md)
#    🟢 GREEN Stage: Implement minimum code to pass tests
#    🔵 REFACTOR Stage: Optimize code while tests pass
#    ✅ REVIEW Stage: Quality and spec review
# 3. Perform spec review (ensure code matches design.md)
# 4. Perform quality review (ensure code quality)
# 5. Fix issues and re-review if needed

# Result: Unit test coverage ≥85%, code verified through TDD
```

**Output**:
- `src/components/OrderTable.tsx` - Order table component
- `src/components/OrderForm.tsx` - Order form component
- `src/hooks/useOrderManagement.ts` - Business logic Hook
- `src/types/order.ts` - Type definitions
- `spec-dev/REQ-001/execution/execution-report.md` - Execution report

#### 1.5 High-Level Testing and Verification (Phase 3)
```bash
# Compilation verification passed

/code-test

# AI will:
# 1. Perform comprehensive code review (Lint, TypeScript check)
# 2. Verify unit tests (ensured by code-execute TDD workflow)
# 3. Execute high-level tests:
#    - Integration tests: Verify multi-Task collaboration
#    - E2E tests: Verify complete business workflows
#    - Performance tests: Establish performance benchmarks
# 4. Closed-loop verification: TEST-VERIFY → Test → Code → Result
# 5. Generate detailed test report and verification matrix
```

**Output**:
- `tests/components/OrderTable.test.tsx` - Table tests
- `tests/hooks/useOrderManagement.test.ts` - Hook tests
- `spec-dev/REQ-001/testing/testing-report.md` - Test report

---

### Scenario 2: Vue Project - Data Dashboard

```bash
# 1. Requirements Specification
/spec-creation Need to create data dashboard supporting real-time data, multi-chart display, custom panels

# 2. Code Design
/code-designer
# AI analyzes specs and generates design solution

# 3. Task Decomposition
/code-task
# AI converts design to specific tasks

# 4. Code Execution
/code-execute
# AI executes all Tasks, generates code

# 5. Test Verification
/code-test
# AI performs testing and closed-loop verification

# 6. Spec Archiving (automatic after tests pass)
/spec-archive
# AI archives verified specs to main spec library
```

---

### Scenario 3: Complete Workflow Example

```bash
# Step 1: Requirements Specification
/spec-creation Need to implement user authentication module supporting email registration, password reset, social login

# [AI performs interactive analysis, confirms requirements...]
# Output: spec-dev/REQ-002/spec/

# Step 2: Design Planning
/code-designer
# [AI reads specs, performs design analysis...]
# Output: spec-dev/REQ-002/design/design.md

# Step 3: Task Decomposition
/code-task
# [AI decomposes tasks...]
# Output: spec-dev/REQ-002/tasks/tasks.md

# Step 4: Code Execution
/code-execute
# [AI executes all Tasks, multi-stage review...]
# Output: src/ + spec-dev/REQ-002/execution/execution-report.md

# Step 5: Test Verification
/code-test
# [AI performs testing and closed-loop verification...]
# Output: tests/ + spec-dev/REQ-002/testing/testing-report.md

# Step 6: Spec Archiving (automatic)
# AI archives verified specs
# Output: spec-dev/spec/ + archive-report.md

# Complete! Code quality meets standards, ready for release
```

---

## Common Questions and Tips

### Q1: Which command should I run first?

**A:** Start with `/spec-creation` to generate project specifications. This is the first step in the spec-driven development workflow, transforming requirements into documented specifications.

### Q2: What does spec-creation do?

**A:** `/spec-creation` takes requirements description, performs interactive confirmation to break them into specific business scenarios, and generates BDD format specification documents (WHEN-THEN format). Specs are the foundation for all subsequent design and development.

### Q3: How to use code-designer?

**A:** `/code-designer` should be run after spec-creation completes. It reads specification documents, performs code design analysis, and generates detailed design solutions (design.md), including architecture, component design, state management, technical solutions, etc.

### Q4: What's the difference between code-task and code-designer outputs?

**A:**
- **code-designer outputs design.md** - Architecture-level design solution, describing "how it should be designed"
- **code-task outputs tasks.md** - Code-level task checklist, describing "what specific coding tasks need to be done"

### Q5: What are the two reviews in code-execute?

**A:**
1. **Spec Review** - Ensures generated code conforms to design.md design solution
2. **Quality Review** - Ensures code quality meets standards (no Lint errors, TypeScript check passes, etc.)

Both reviews must pass. Issues are automatically fixed and re-reviewed.

### Q6: Can I skip any stage?

**A:** Not recommended. Each stage serves a specific purpose:
- Skipping specification leads to design deviations from requirements
- Skipping design leads to frequent modifications later
- Skipping task decomposition leads to execution confusion
- Skipping review reduces code quality
- Skipping testing cannot verify feature completeness

### Q7: What if test coverage is below 80%?

**A:** Continue writing test cases until coverage reaches 80%. This is the minimum quality guarantee. code-test will provide specific uncovered areas.

### Q8: How to handle issues found during testing?

**A:** Return to source code to fix issues, then re-run tests. Do not modify test cases to accommodate code.

### Q9: How to handle requirement changes?

**A:**
1. Update specification documents in spec-dev/{requirement_desc_abstract}/spec/
2. Confirm impact scope in code-designer stage
3. Update design.md and tasks.md
4. Re-execute affected Tasks
5. Re-run tests

### Q10: What if the workflow encounters errors?

**A:**
1. Check error records in execution report
2. Identify error cause (usually unclear design or requirement misunderstanding)
3. Fix specification or design documents
4. Re-execute the relevant stage

---

## Best Practices

### 1. Thorough Requirements Specification
In the spec-creation stage, perform thorough interactive analysis to ensure specs are complete.

```bash
# ✅ Good practice
Through multi-round confirmation, decompose all business scenarios, confirm WHEN-THEN conditions are clear

# ❌ Bad practice
Rush through, vague spec content, frequent corrections needed later
```

### 2. Design Should Be Detailed Enough
In the code-designer stage, the design solution should be detailed enough to directly guide coding.

```bash
# ✅ Good practice
- Props interfaces clearly defined
- State management approach specified
- Data flow clearly documented
- Interaction workflows detailed

# ❌ Bad practice
Vague design, constantly confirming details during coding
```

### 3. Clear Task Decomposition
In the code-task stage, tasks should be clearly decomposed, each Task with clear deliverables and acceptance criteria.

```bash
# ✅ Good practice
- Each Task completable within 4 hours
- Dependencies clearly documented
- Acceptance criteria measurable

# ❌ Bad practice
Tasks too large or small, vague acceptance criteria
```

### 4. Strict Review Process
In the code-execute stage, both spec review and quality review must pass.

```bash
# ✅ Spec review should check:
# - Props design matches design.md
# - Feature implementation aligns with design.md
# - Interaction workflows match design

# ✅ Quality review should check:
# - Consistent code style
# - Complete types, no any
# - Test coverage ≥80%
```

### 5. Closed-Loop Verification
In the code-test stage, ensure complete Task→Code→Test correspondence.

```bash
# Verification items:
# 1. Feature completeness - Are all designed features implemented and tested?
# 2. Interface consistency - Do code interfaces match design?
# 3. Data integrity - Do data models match design?
```

---

## 🌐 Multi-Platform Usage

The commands above use Claude Code syntax (`/spec-creation`, `/code-designer`, etc.). The workflow is **platform-agnostic** and can be adapted to other AI coding tools:

| Platform | How to Trigger Skills |
|----------|---------------------|
| **Claude Code** | Use `/skill-name` commands directly |
| **OpenAI Codex** | Load skill content as system prompt, follow workflow steps |
| **GitHub Copilot** | Paste skill content as initial context, execute manually |
| **Cursor** | Load skill definitions in chat, follow steps |
| **Gemini CLI** | Load skill content, execute workflow |
| **OpenCode** | Load skill content, follow workflow steps |

The 6-stage workflow (Spec → Design → Task → Execute → Test → Archive) is a **process framework** that can be executed on any AI coding platform. The skill definitions in `SKILL.md` files are standard Markdown and can be loaded as context.

For detailed tool mappings, see `skills/using-superpowers/references/`.

---

## Workflow Summary

```
┌──────────────────────────────────────────────┐
│ 1. /spec-creation [Requirements Description]  │
│    Output: spec-dev/{name}/spec/              │
│    Specify requirements, decompose scenarios  │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 2. /code-designer                             │
│    Output: spec-dev/{name}/design/design.md   │
│    Design analysis, architecture planning     │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 3. /code-task                                 │
│    Output: spec-dev/{name}/tasks/tasks.md     │
│    Task decomposition, deliverable definition │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 4. /code-execute                              │
│    Output: src/ + execution-report.md         │
│    Code implementation + spec + quality review│
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 5. /code-test                                 │
│    Output: tests/ + testing-report.md         │
│    Code review + unit/integration/E2E tests   │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 6. /spec-archive (automatic)                  │
│    Output: spec-dev/spec/ + archive-report.md │
│    Spec archiving and accumulation            │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
                 ✅ Feature complete, code quality meets standards
```

---

For more details, see [README.md](../README.md) and [Best Practices](./BEST_PRACTICES_EN.md)
