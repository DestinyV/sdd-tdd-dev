# R&D SDD Workflow - Skills Collection (Full-Stack Development)

This is the **sdd-tdd-dev-plugin** project's 6 Claude Code Skills, implementing the **Spec-Design-Task-Execute-Test-Archive** workflow, supporting full-stack development (frontend, backend, mobile, full-stack applications).

## 📋 Workflow Overview

```
User Requirements
    ↓
[1] /spec-creation → Generate design specs and reference component list
    ↓
[2] /code-designer → Code design planning based on specs
    ↓
[3] /code-task → Convert design to detailed task list
    ↓
[4] /code-execute → Execute tasks, generate code implementation
    ↓
[5] /code-test → Code review, testing, closed-loop verification
    ↓
[6] /spec-archive → Spec archiving, accumulate to enterprise spec library ✨New
    ↓
✅ Feature complete, code deployed, specs accumulated
```

---

## 🎯 Skills Details

| Skill | Input | Output | Description |
|-------|-------|--------|-------------|
| **spec-creation** | Requirements description | spec-dev/{requirement_desc_abstract}/spec/ | BDD format spec generation, including scenarios, data models, business rules |
| **code-designer** | spec/ | spec-dev/{requirement_desc_abstract}/design/design.md | Code design planning based on specs, generates architecture and technical solutions |
| **code-task** | design.md | spec-dev/{requirement_desc_abstract}/tasks/tasks.md | Decompose design into code-level task lists |
| **code-execute** | tasks.md | src/ + spec-dev/{requirement_desc_abstract}/execution/execution-report.md | Execute tasks, two-stage review (spec + quality). **v2.3.1+**: Creates git-worktree for each Task, isolating work environments, supporting safe fix cycles and parallel execution |
| **code-test** | src/ + tasks.md | tests/ + spec-dev/{requirement_desc_abstract}/testing/testing-report.md | Integration/E2E/Performance testing, closed-loop verification (unit tests completed by code-execute TDD workflow) |
| **spec-archive** | spec/ | spec-dev/spec/ | Spec benchmark analysis, conflict detection, smart merging, accumulate to main spec library |

---

## 🚀 Quick Usage

### Step 1: Requirements Specification
```bash
/spec-creation [Requirements Description]
```
Generates `spec-dev/{requirement_desc_abstract}/spec/`
- README.md - Spec overview
- scenarios/*.md - BDD format scenarios
- data-models.md - Data models
- business-rules.md - Business rules
- glossary.md - Glossary

### Step 2: Code Design
```bash
/code-designer
```
Generates `spec-dev/{requirement_desc_abstract}/design/design.md`

### Step 3: Generate Task List
```bash
/code-task
```
Generates `spec-dev/{requirement_desc_abstract}/tasks/tasks.md`

### Step 4: Execute Code
```bash
/code-execute
```
Generates `src/` + `spec-dev/{requirement_desc_abstract}/execution/execution-report.md`

**v2.3.1+ Git-Worktrees Support**:
- Creates independent git worktree for each Task (`.claude/worktrees/{task-id}-{name}`)
- Coding, fixes all performed in worktree, each fix as independent commit
- After fix completion, cherry-pick or squash merge to main branch
- If fix fails, delete worktree and restart, no pollution to main branch
- See [Git-Worktrees Guide](./code-execute/git-worktrees-guide.md) and [Quick Reference](./code-execute/QUICK_REFERENCE.md)

### Step 5: Testing and Verification
```bash
/code-test
```
Generates `tests/` + `spec-dev/{requirement_desc_abstract}/testing/testing-report.md`

**New Phase 3 High-Level Testing** (code-test now focuses on integration, E2E, performance testing):
- Integration testing: See `code-test/integration-test-prompt.md`
- E2E testing: See `code-test/e2e-test-prompt.md`
- Performance testing: See `code-test/performance-test-prompt.md`

**Important Note**: Unit testing is completed by `/code-execute` TDD workflow (RED-GREEN-REFACTOR-REVIEW), `code-test` does not repeat unit testing review

### Step 6: Spec Archiving ✨New
```bash
/spec-archive
```
(Automatically triggered after all code-test tests pass)

Generates `spec-dev/spec/` + `archive-report.md`

---

## 📁 Skills Directory Structure

```
skills/
├── spec-creation/
│   ├── SKILL.md                     # Spec generation workflow
│   └── README.md
├── code-designer/
│   ├── SKILL.md                     # Design planning workflow
│   └── README.md
├── code-task/
│   ├── SKILL.md                     # Task decomposition workflow
│   └── README.md
├── code-execute/
│   ├── SKILL.md                     # Code execution workflow (with git-worktrees support)
│   ├── git-worktrees-guide.md       # Git-Worktrees isolated work environment detailed guide ✨v2.3.1 new
│   ├── QUICK_REFERENCE.md           # Git-Worktrees quick reference card ✨v2.3.1 new
│   ├── implementer-prompt.md        # Implementer sub-agent prompt
│   ├── spec-reviewer-prompt.md      # Spec review prompt
│   ├── code-quality-reviewer-prompt.md  # Quality review prompt
│   └── README.md
├── code-test/
│   ├── SKILL.md                     # Testing workflow (Integration/E2E/Performance)
│   ├── integration-test-prompt.md   # Integration test design guide ✨Phase 3 new
│   ├── e2e-test-prompt.md           # E2E test design guide ✨Phase 3 new
│   ├── performance-test-prompt.md   # Performance test design guide ✨Phase 3 new
│   └── README.md
├── spec-archive/                    # ✨New
│   ├── SKILL.md                     # Spec archiving workflow
│   └── README.md
├── package.json                     # Skills declaration
└── README.md                        # This file
```

---

## 🔑 Core Design Principles

### 1. **Spec-First**
- Project first generates `design-spec/README.md` through `/spec-creation`
- All subsequent design and development based on this spec
- Spec defines reference components and design patterns

### 2. **Design-Driven**
- Deep design analysis through `/code-designer`
- Generate detailed design solutions including architecture, technical solutions, design decisions
- Design must be approved before entering Task stage

### 3. **Task-Clear**
- Convert design to executable task list through `/code-task`
- Each Task has clear goals, deliverables, acceptance criteria
- Task list provides clear guidance for subsequent execution

### 4. **Execute-Rigorous**
- Multi-stage review through `/code-execute`
  - Spec review: Ensure code conforms to design.md
  - Quality review: Ensure code quality meets standards
  - Fix cycle: Issues must be fixed and re-reviewed
- Each Task passes two review checkpoints

### 5. **Test-Complete**
- High-level testing and closed-loop verification through `/code-test`
- Full coverage of integration testing, E2E testing, performance testing
- **Note**: Unit testing guaranteed by `/code-execute` TDD workflow (not repeated)
- Closed-loop verification: Ensure Task-code-test complete correspondence

### 6. **Spec-Archive** ✨New
- Archive verified specs to main spec library through `/spec-archive`
- Benchmark analysis and conflict detection ensure spec library consistency and completeness
- Build enterprise spec library for future requirement reference

---

## ✅ Workflow Features

### Flexibility
- **Custom Patterns** - Users define business patterns in spec-creation, not preset
- **Design-Driven** - Each requirement goes through detailed code-design before execution
- **Clear Tasks** - code-task ensures complete, clear task definitions, avoiding ambiguity
- **Full-Stack Support** - Supports frontend, backend, mobile, full-stack application requirements and design

### Quality Assurance
- **Multi-Stage Review** - Spec review + quality review in code-execute
- **Fix Cycle** - Issues must be fixed and re-reviewed before proceeding
- **Complete Testing** - Unit, integration, E2E full coverage in code-test
- **Tech Stack Adaptation** - Adjust design and implementation norms based on project tech stack (React/Vue/Node.js/Java/Python/Go, etc.)

### Traceability
- **Complete Documentation** - Each stage generates detailed design/task/report documents
- **Decision Records** - Design stage records all design decisions and reasons
- **Issue Records** - Execution and testing stages record all discovered issues and fixes

---

## 📖 Detailed Documentation

- **Project Main README** → [`../README_EN.md`](../README_EN.md)
- **Installation Guide** → [`../docs/INSTALLATION_EN.md`](../docs/INSTALLATION_EN.md)
- **Usage Guide** → [`../docs/USAGE_EN.md`](../docs/USAGE_EN.md)
- **Best Practices** → [`../docs/BEST_PRACTICES_EN.md`](../docs/BEST_PRACTICES_EN.md)

---

## 🔧 Skill Detailed Documentation

Each Skill has complete `SKILL.md` documentation in its own directory, including:
- Responsibility description
- Workflow (phased explanation)
- Key constraints
- Danger signals

Examples:
- `spec-creation/SKILL.md` - Spec generation workflow
- `code-designer/SKILL.md` - Design planning workflow
- `code-task/SKILL.md` - Task decomposition workflow
- `code-execute/SKILL.md` - Code execution workflow
- `code-test/SKILL.md` - Testing workflow (Integration/E2E/Performance, no repeat of unit testing) ✨Phase 3 optimized
- `spec-archive/SKILL.md` - Spec archiving workflow ✨New

---

## 🎓 Usage Suggestions

### First Time Use
1. Read `../README_EN.md` to understand the entire workflow
2. Execute `/spec-creation` to generate project specs
3. Choose a requirement to trial run the complete Spec → Design → Task → Execute → Test → Archive workflow

### Daily Development
1. New requirement → `/code-designer` design
2. Design approved → `/code-task` task list
3. `/code-execute` code execution
4. `/code-test` test verification
5. `/spec-archive` spec archiving ✨New
6. Complete!

### Customize Workflow
- Modify processes and principles in each `SKILL.md` to adapt to project characteristics
- Adjust execute/review prompts to fit specific needs
- See `../docs/CUSTOMIZATION.md` to learn how to customize

---

## 🔑 Core Constraints

### ✅ Must Do
- Run `/spec-creation` to generate specs before first use
- All requirements go through `/code-designer` for design
- Only enter `/code-task` after design approval
- All Tasks pass two reviews + TDD workflow in `/code-execute` (ensuring unit test coverage)
- All code passes Integration/E2E/Performance testing and closed-loop verification in `/code-test`
- Execute `/spec-archive` after tests pass to accumulate specs to main spec library ✨New

### ❌ Must Not Do
- Skip design stage and directly code
- Skip spec or quality reviews in code-execute
- Continue to next Task without fixing discovered issues
- Modify source code logic to make tests pass
- Ignore closed-loop verification (Task-code-test correspondence)
- Skip spec archiving process, preventing spec library accumulation and improvement ✨New

---

## 🌐 Multi-Platform Support

The SDD+TDD workflow skills are **platform-agnostic** and designed to work across multiple AI coding platforms:

| Platform | Skill Loading | Notes |
|----------|--------------|-------|
| **Claude Code** | Native `/skill` commands | Full workflow support |
| **OpenAI Codex** | Context injection | See [Codex Tools Mapping](../using-superpowers/references/codex-tools.md) |
| **GitHub Copilot** | Context injection | See [Copilot Tools Mapping](../using-superpowers/references/copilot-tools.md) |
| **Cursor** | Chat context | Paste skill content as context |
| **Gemini CLI** | Context injection | See [Gemini Tools Mapping](../using-superpowers/references/gemini-tools.md) |
| **OpenCode** | Context injection | See [OpenCode Tools Mapping](../using-superpowers/references/opencode-tools.md) |

### How to Use on Other Platforms

1. Read the `SKILL.md` file content
2. Load as system prompt or initial context
3. Execute the workflow steps manually
4. Reference tool mappings for platform-specific commands

---

## 🚀 Architecture Features

### Separation of Concerns
- **spec-creation**: Define specs
- **code-designer**: Design solutions
- **code-task**: Decompose tasks
- **code-execute**: Execute coding
- **code-test**: Verify quality

### Quality Gates
```
Spec ✅ → Design ✅ → Task ✅ → Execute Review ✅ → Test Review ✅ → Deploy
```

Each stage has clear outputs and acceptance criteria.

---

## 💡 Best Practices

1. **Design First** - Sufficient design prevents later modifications
2. **Clear Tasks** - Clear Task definitions improve execution efficiency
3. **Strict Review** - Multi-stage review catches issues early
4. **Complete Documentation** - Detailed documentation facilitates tracking and future maintenance
5. **Complete Testing** - High coverage testing is quality assurance

---

## 📝 Changelog

### v2.2.0 (2026-03-20) ✨New
- ✅ **Spec Archiving** - New spec-archive skill and spec-archiver Agent
- ✅ **Spec Accumulation** - Automatically archive verified requirement specs to enterprise main spec library
- ✅ **Scenario Splitting** - Support integration of new specs through scenario splitting and smart merging
- ✅ **Conflict Detection** - Automatic detection and handling of spec conflicts with decision suggestions
- ✅ **Spec Benchmark Analysis** - New benchmark analysis for scenarios, data models, business rules, terminology
- ✅ **Version Management** - Support spec version tracking and evolution history

### v2.1.0 (2026-03-10)
- ✅ Full-stack development support: frontend, backend, mobile, full-stack applications
- ✅ Tech stack expansion: Node.js, Python, Go, Java and other backend frameworks
- ✅ Database design support: SQL, NoSQL data model design
- ✅ API design specification: REST/GraphQL API design and validation
- ✅ Deployment and configuration: containerization, environment configuration, CI/CD pipeline design
- ✅ Microservices architecture: coordinated design and task decomposition for multiple services

### v2.0.0 (2026-03-09)
- ✅ Refactored workflow to Spec-Design-Task-Execute-Test-Archive 6 stages
- ✅ spec-creation: R&D spec generation (replaces original spec-generator)
- ✅ code-designer: Code design planning (replaces original ai-planning)
- ✅ code-task: Task list generation (new)
- ✅ code-execute: Code execution (replaces original ai-code-execution)
- ✅ code-test: Code review and testing (replaces original ai-test-creation)
- ✅ spec-archive: Spec archiving and accumulation (new)
- ✅ Full-stack development support, currently focusing on frontend (React/Vue/Angular/Svelte)

---

**Make AI-assisted R&D development standardized, efficient, and reliable!** 🚀
