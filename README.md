# DestinyV Marketplace

[中文](./README_ZH.md) | English

Enterprise-grade Claude Code plugin marketplace,with high-quality development workflow plugins that help teams establish standards, improve efficiency, and ensure quality.

## 🎯 Overview

DestinyV Marketplace is an open-source Claude Code plugin marketplace that provides verified, production-grade plugins and skills for enterprise development teams. Our goals:

- 📦 Centralized management of high-quality Claude Code plugins
- 🚀 Accelerate enterprise AI-assisted development workflows
- 🔧 Provide plug-and-play development standards and best practices
- 🤝 Build an AI-driven development tool ecosystem

---

## 🚀 Quick Start

### 1️⃣ Add Marketplace in Claude Code

#### Option A: Automatic Installation (Recommended)

```bash
# Run in your project
/plugin install https://github.com/DestinyV/sdd-tdd-dev.git
```

#### Option B: Manual Configuration

1. Open your project's `.claude/config.json` (create if it doesn't exist)

2. Add the marketplace source:
   ```json
   {
     "marketplaces": [
       {
         "name": "DestinyV Marketplace",
         "url": "https://github.com/DestinyV/sdd-tdd-dev.git",
         "type": "git"
       }
     ]
   }
   ```

3. Execute in Claude Code:
   ```bash
   /marketplace list    # View available plugins
   /marketplace install sdd-tdd-dev  # Install plugin
   ```

---

### 2️⃣ Create a New Plugin

#### Standard Plugin Structure

All plugins must follow the following directory structure, placed in the `/plugins` directory of this marketplace:

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json                  # Plugin metadata (required)
├── skills/                          # Skills directory (optional)
│   ├── skill-1/
│   │   └── SKILL.md                # Skill definition file
│   ├── skill-2/
│   │   └── SKILL.md
│   └── package.json                # Dependencies (if any)
├── agents/                          # Agents directory (optional)
│   ├── agent-1.md
│   └── agent-2.md
├── docs/                           # Documentation directory
│   ├── USAGE.md                    # Usage guide
│   ├── INSTALLATION.md             # Installation instructions
│   ├── BEST_PRACTICES.md           # Best practices
│   ├── FAQ.md                      # FAQ
│   └── examples/                   # Usage examples
├── README.md                       # Plugin overview (required)
├── CHANGELOG.md                    # Changelog
├── CLAUDE.md                       # Development guide
├── LICENSE                         # License (required)
└── package.json                    # npm package config (if needed)
```

#### Minimal Plugin Example

**plugin.json** - Plugin Metadata
```json
{
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": "Your Name",
  "license": "MIT",
  "skills": [
    {
      "name": "skill-name",
      "path": "./skills/skill-name/SKILL.md"
    }
  ],
  "commands": [],
  "hooks": [],
  "agents": []
}
```

**README.md** - Plugin Introduction
```markdown
# Your Plugin Name

Brief description of plugin functionality.

## Features

- Feature 1
- Feature 2

## Installation

```bash
/plugin install your-plugin@DestinyV-marketplace
```

## Usage

```bash
/your-command
```
```

**skills/your-skill/SKILL.md** - Skill Definition
```markdown
---
name: your-skill
description: What this skill does
---

Skill content and implementation instructions.
```

#### Steps to Create a Plugin

1. **Create plugin directory in marketplace**
   ```bash
   mkdir plugins/your-plugin-name
   cd plugins/your-plugin-name
   ```

2. **Initialize required files**
   ```bash
   mkdir -p .claude-plugin skills/your-skill docs agents
   touch .claude-plugin/plugin.json README.md LICENSE CLAUDE.md
   touch skills/your-skill/SKILL.md
   ```

3. **Write plugin code and documentation**
   - Complete `plugin.json` configuration
   - Implement skill functionality
   - Write comprehensive README and documentation

4. **Test plugin**
   ```bash
   # Test locally in your project
   npx skills add ./plugins/your-plugin-name
   ```

5. **Submit to marketplace (see Contribution Guide below)**

---

## 📦 Available Plugins

### 🏆 sdd-tdd-dev - Enterprise Full-Stack SDD+TDD Development Workflow

**Latest Version**: v2.3.1 (2026-03-23)

An enterprise-grade Claude Code plugin providing complete Spec-Driven Development (SDD) and Test-Driven Development (TDD) workflows. Supports AI-assisted development for frontend, backend, database, microservices, and other project types.

#### Core Features

- ✅ **Spec-Driven Development (SDD)** - Define specs, AI generates code based on specs
- ✅ **Complete Workflow** - Spec → Design → Task → Execute → Test → Archive (6 stages)
- ✅ **TDD Implementation** - RED-GREEN-REFACTOR-REVIEW four-stage TDD process
- ✅ **Smart Sub-Agents** - 5 specialized roles (Architect, Explorer, Executor, Reviewer, Archive Specialist) working in parallel
- ✅ **Quality Assurance** - Spec review, code quality review, unit tests ≥80%, integration/E2E/performance testing
- ✅ **Isolated Work Environment** - Git-Worktrees isolation, safe redo for failed fixes
- ✅ **Spec Accumulation** - Automatic archiving of verified specs to enterprise spec library
- ✅ **Plug and Play** - Interactive questionnaire, automated workflows, comprehensive documentation

#### Workflow

```
Requirements → /spec-creation (Spec) 
            → /code-designer (Design) 
            → /code-task (Task) 
            → /code-execute (Coding+TDD) 
            → /code-test (Testing) 
            → /spec-archive (Archive) 
            → Complete ✅
```

#### 6 Core Skills

| Skill | Stage | Function | Output |
|-------|-------|----------|--------|
| **spec-creation** | Spec | Requirements analysis and spec generation (BDD format) | spec-dev/{req}/spec/ |
| **code-designer** | Design | Architecture and technical design planning | spec-dev/{req}/design/design.md |
| **code-task** | Task | Task breakdown and checklist definition | spec-dev/{req}/tasks/tasks.md |
| **code-execute** | Execute | Code generation + TDD workflow + Git-Worktrees isolation | src/ + execution report |
| **code-test** | Test | Integration/E2E/Performance testing + closed-loop verification | tests/ + test report |
| **spec-archive** | Archive | Spec archiving and accumulation to enterprise spec library | spec-dev/spec/ |

#### 5 Core Agents

- **code-architect** - Architecture design expert, analyzes project patterns and plans architecture
- **code-explorer** - Code exploration expert, deep analysis of similar feature implementations
- **code-executor** - Code execution expert, writes high-quality code task by task
- **code-reviewer** - Code review expert, checks spec compliance and quality issues
- **spec-archiver** - Spec archiving expert, benchmark analysis, conflict detection, smart merging

#### Quick Start

```bash
# 0. Enter plugin
/sdd-tdd-dev:sdd-dev
# Input requirements

# 1. Generate spec (interactive)
/spec-creation

# 2-6. Automatic subsequent steps
# - /code-designer Design stage (automatic)
# - /code-task Task breakdown (automatic)
# - /code-execute Code execution+TDD (automatic, with Git-Worktrees isolation)
# - /code-test Test verification (automatic)
# - /spec-archive Spec archiving (automatic)
```

#### 📖 Complete Documentation

- 📌 [Full README](./plugins/sdd-tdd-dev/README.md) - Detailed plugin introduction and workflow
- 📌 [中文 README](./plugins/sdd-tdd-dev/README_ZH.md) - Chinese version of plugin documentation
- 🚀 [Quick Start](./plugins/sdd-tdd-dev/docs/USAGE_EN.md) - 5-minute getting started guide
- 📋 [Installation](./plugins/sdd-tdd-dev/docs/INSTALLATION_EN.md) - Detailed installation and troubleshooting
- 💡 [Best Practices](./plugins/sdd-tdd-dev/docs/BEST_PRACTICES_EN.md) - Full-stack SDD best practices
- 📚 [Skills Documentation](./plugins/sdd-tdd-dev/skills/README_EN.md) - 6 Skills detailed documentation
