📍 **You are here:** [Try It](QUICKSTART.md) → **Love It** → [Master It](CASE-STUDIES.md)
**Prerequisites:** Basic rebar setup
**Related:** [Feature development workflow](FEATURE-DEVELOPMENT.md)

# Agent Coordination Quickstart

**The complete guide to rebar's agent ecosystem**

Rebar provides two types of agents for different coordination needs:
1. **Role-based agents** — Persistent experts who answer questions and accumulate knowledge
2. **Subagent templates** — Task-specific workers for delegating focused work

This guide helps you choose the right approach for any situation.

---

## Quick Decision Tree

### "I have a question" → **Role-Based Agents**
Use `ask <role> "<question>"` for persistent experts who know your codebase:

```bash
ask architect "Should we add caching to the user service?"
ask product "Does this contract match our user stories?"
ask englead "Are we ready to merge this feature?"
ask steward "What contracts need attention?"
```

### "I need work done" → **Subagent Templates**
Use subagent prompt templates for focused, task-specific work:

```bash
# Review code quality
claude --prompt agents/subagent-prompts/code-review.md

# Security audit
claude --prompt agents/subagent-prompts/security-surface-scan.md

# Test a specific subset
claude --prompt agents/subagent-prompts/test-shard-runner.md
```

---

# Role-Based Agents (Persistent Experts)

**What they are:** Persistent agents that accumulate knowledge about your project and answer questions in their domain.

**How they work:** Each agent has memory and focuses on specific responsibilities. They read your project files contextually when answering questions.

## The 6 Core Roles

### 🏗️ **Architect**
```bash
ask architect "Should the auth service handle encryption or delegate?"
ask architect "Review CONTRACT:S3-AUTH.1.0 for integration issues"
ask architect "What's the dependency graph for the user service?"
```

**Focus:** System design, contracts, technical architecture, integration patterns
**Reads:** architecture/, DESIGN.md, contracts, integration points
**Memory:** Design decisions, architectural patterns, dependency relationships

### 📋 **Product**
```bash
ask product "Does the auth contract meet our UX requirements?"
ask product "What user scenarios are missing from this feature?"
ask product "Priority order for these three contract changes?"
```

**Focus:** User requirements, BDD scenarios, feature prioritization, user experience
**Reads:** product/, features/, user stories, BDD scenarios
**Memory:** User needs, product decisions, feature evolution

### 👥 **Engineering Lead**
```bash
ask englead "QA checklist for this multi-agent development?"
ask englead "Are we ready to ship this contract implementation?"
ask englead "Coordinate these three parallel feature branches"
```

**Focus:** Delivery coordination, quality assurance, team management, release planning
**Reads:** QUICKCONTEXT.md, TODO.md, testing results, CI status
**Memory:** Team capacity, delivery patterns, quality standards

### 🔍 **Steward**
```bash
ask steward "Health report for all contracts"
ask steward "Which contracts are blocking the release?"
ask steward "Summary of testing gaps across the codebase"
```

**Focus:** Automated quality scanning, contract health, testing gaps, compliance
**Reads:** All contracts, test files, compliance scripts
**Memory:** Quality trends, common issues, system health patterns

### 🧪 **Tester**
```bash
ask tester "Test strategy for the new auth service"
ask tester "Coverage gaps in the payment flow"
ask tester "E2E test plan for the mobile API"
```

**Focus:** Test strategy, coverage analysis, quality assurance, testing automation
**Reads:** Test files, test results, coverage reports
**Memory:** Testing patterns, common failure modes, quality strategies

### 🔄 **Merger**
```bash
ask merger "Integration plan for these 3 feature branches"
ask merger "Merge conflict resolution strategy"
ask merger "Post-worktree coordination checklist"
```

**Focus:** Branch integration, merge conflicts, post-development coordination
**Reads:** Git status, branch differences, integration tests
**Memory:** Merge strategies, conflict patterns, integration workflows

---

# Subagent Templates (Task-Specific Workers)

**What they are:** Focused prompt templates for delegating specific work to ephemeral agents.

**How they work:** Copy a template, customize it for your situation, run it with Claude Code. The agent does the work and exits.

## By Category

### 📝 **Reviews** (Quality Assessment)

#### **Code Review** — Multi-dimensional analysis
```bash
claude --prompt agents/subagent-prompts/code-review.md
```
**Use when:** Need comprehensive code quality assessment
**Covers:** Correctness, performance, security, maintainability, style
**Output:** Structured findings with priority levels and recommendations

#### **UX Review** — User experience audit
```bash
claude --prompt agents/subagent-prompts/ux-review.md
```
**Use when:** Evaluating interface, interaction, or user experience
**Covers:** Accessibility, responsive design, visual consistency, error states
**Output:** UX recommendations with impact assessment

#### **Security Surface Scan** — Security audit
```bash
claude --prompt agents/subagent-prompts/security-surface-scan.md
```
**Use when:** Need security assessment before shipping
**Covers:** Input validation, authentication, crypto usage, data exposure
**Output:** Security findings ranked by severity

### 🔍 **Analysis** (Deep Investigation)

#### **Contract Audit** — Interface conformance check
```bash
claude --prompt agents/subagent-prompts/contract-audit.md
```
**Use when:** Verify implementation matches contract specification
**Covers:** Method signatures, behavior contracts, error handling, test coverage
**Output:** Conformance report with specific violations

#### **Feature Inventory** — Behavioral mapping
```bash
claude --prompt agents/subagent-prompts/feature-inventory.md
```
**Use when:** Need exhaustive understanding before major changes
**Covers:** All behaviors, edge cases, dependencies, test coverage
**Output:** Complete behavioral inventory for safe refactoring

#### **Doc Drift Detector** — Documentation accuracy
```bash
claude --prompt agents/subagent-prompts/doc-drift-detector.md
```
**Use when:** Suspect documentation is out of sync with code
**Covers:** Stale claims, broken references, missing documentation
**Output:** Documentation gaps and inconsistencies

### 🧪 **Testing** (Quality Validation)

#### **Test Shard Runner** — Parallel test execution
```bash
claude --prompt agents/subagent-prompts/test-shard-runner.md
```
**Use when:** Need to run tests in parallel across worktrees
**Covers:** Isolated test execution, pass/fail/flaky detection
**Output:** Per-test results with timing and failure analysis

### 🔄 **Coordination** (Integration Management)

#### **Merge Coordinator** — Post-worktree integration
```bash
claude --prompt agents/subagent-prompts/merge-coordinator.md
```
**Use when:** Integrating multiple agent worktrees
**Covers:** Cherry-pick strategy, conflict resolution, integration testing
**Output:** Step-by-step merge plan with verification steps

---

## Usage Patterns

### **Single Agent Session**
```bash
# Get architectural guidance
ask architect "Review this service design"

# Then implement with context
claude --prompt agents/subagent-prompts/code-review.md
```

### **Multi-Agent Coordination**
```bash
# Get multiple perspectives
ask architect "Technical feasibility of user sync"
ask product "User scenarios for sync conflicts"
ask englead "Implementation timeline estimate"

# Then coordinate implementation
claude --prompt agents/subagent-prompts/feature-inventory.md
claude --prompt agents/subagent-prompts/test-shard-runner.md
```

### **Quality Gates**
```bash
# Pre-merge quality assessment
claude --prompt agents/subagent-prompts/contract-audit.md
claude --prompt agents/subagent-prompts/security-surface-scan.md
ask steward "Final health check before shipping"
```

---

## When to Use Which

### **Questions & Guidance** → Role-Based Agents
- "Should I implement X or Y?"
- "What's the architectural impact of this change?"
- "Are we ready to ship this feature?"
- "What user scenarios am I missing?"

### **Focused Work** → Subagent Templates
- "Review this code for security issues"
- "Generate comprehensive test coverage"
- "Audit contract compliance"
- "Coordinate multi-branch merge"

### **Knowledge Building** → Role-Based Agents
- Building understanding over multiple sessions
- Project-specific context and history
- Cross-cutting concerns and relationships
- Strategic decision-making

### **Task Execution** → Subagent Templates
- One-time focused analysis
- Parallel work distribution
- Quality gate enforcement
- Specialized technical tasks

---

## Advanced Patterns

### **Agent Handoffs**
```bash
# Architect designs → Product validates → Englead coordinates
ask architect "Design auth service integration"
ask product "Validate this design meets user needs"
ask englead "Implementation plan for 3-agent parallel development"
```

### **Quality Cascades**
```bash
# Progressive quality assessment
claude --prompt agents/subagent-prompts/code-review.md
claude --prompt agents/subagent-prompts/contract-audit.md
claude --prompt agents/subagent-prompts/security-surface-scan.md
ask steward "Final compliance check"
```

### **Fan-Out Coordination**
```bash
# Distribute work across multiple worktrees
ask merger "Plan 4-agent feature implementation"
claude --prompt agents/subagent-prompts/test-shard-runner.md  # Agent 1
claude --prompt agents/subagent-prompts/code-review.md      # Agent 2
claude --prompt agents/subagent-prompts/ux-review.md        # Agent 3
claude --prompt agents/subagent-prompts/merge-coordinator.md # Merge back
```

---

## Quick Reference

### Role-Based Agent Commands
```bash
ask architect "<question>"    # System design, contracts, technical architecture
ask product "<question>"      # User requirements, BDD scenarios, prioritization
ask englead "<question>"      # Delivery coordination, QA, team management
ask steward "<question>"      # Quality scanning, contract health, compliance
ask tester "<question>"       # Test strategy, coverage analysis, quality assurance
ask merger "<question>"       # Branch integration, merge conflicts, coordination
```

### Most Common Subagent Templates
```bash
claude --prompt agents/subagent-prompts/code-review.md           # Code quality
claude --prompt agents/subagent-prompts/contract-audit.md        # Contract compliance
claude --prompt agents/subagent-prompts/security-surface-scan.md # Security audit
claude --prompt agents/subagent-prompts/test-shard-runner.md     # Parallel testing
claude --prompt agents/subagent-prompts/merge-coordinator.md     # Integration
```

### Template Customization
```bash
# Copy and customize templates
cp agents/subagent-prompts/code-review.md my-custom-review.md
# Edit to focus on specific concerns, then:
claude --prompt my-custom-review.md
```

---

## Next Steps

### **New to agent coordination?**
- **[FEATURE-DEVELOPMENT.md](FEATURE-DEVELOPMENT.md)** — See agents working together in a complete workflow

### **Need persistent agent sessions?**
- **[bin/README.md](bin/README.md)** — ASK CLI setup for 10x context efficiency

### **Advanced coordination patterns?**
- **[practices/multi-agent-orchestration.md](practices/multi-agent-orchestration.md)** — Fan-out strategies, sharding, pre-launch audits
- **[practices/worktree-collaboration.md](practices/worktree-collaboration.md)** — Parallel development without merge conflicts

### **Creating custom templates?**
- **[agents/subagent-prompts/_example-template.md](agents/subagent-prompts/_example-template.md)** — Template for creating new templates

**Remember:** Role-based agents accumulate knowledge and answer questions. Subagent templates execute focused tasks. Use both together for coordinated development that scales from solo work to multi-agent swarms.