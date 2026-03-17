# agent-templates

A complete starter kit for contract-driven, agent-powered software development.

**Start here:** [methodology.md](methodology.md) — the full philosophy.
Contracts are the operating system. BDD first. Max autonomy within contracts.
Trust but verify. Everything else implements this.

---

## What's Included

### The Methodology

| File | Purpose |
|------|---------|
| [methodology.md](methodology.md) | **The philosophy.** Contract-driven development, BDD-first, agent autonomy model, anti-drift mechanisms. Read this to understand *why* everything else is structured the way it is. |

### The Cold Start Quad

Four files every agent reads on session start, in order:

| Order | Template | Purpose |
|-------|----------|---------|
| 1 | [README.template.md](README.template.md) | Universal first-read — project orientation, architecture overview, cold start instructions |
| 2 | [QUICKCONTEXT.template.md](QUICKCONTEXT.template.md) | What's true right now — branch, test counts, active work, blockers |
| 3 | [TODO.template.md](TODO.template.md) | Tasks + known issues + blockers (two-tag tracking system) |
| 4 | [AGENTS.template.md](AGENTS.template.md) | How we work — norms, testing cascade, contracts, agent collaboration |

Plus: [CLAUDE.template.md](CLAUDE.template.md) — Claude Code-specific configuration.

### The Contract System

| File | Purpose |
|------|---------|
| [architecture/README.md](architecture/README.md) | How the contract system works — naming, linking, versioning |
| [architecture/CONTRACT-TEMPLATE.md](architecture/CONTRACT-TEMPLATE.md) | Annotated template for new contracts |
| [architecture/CONTRACT-REGISTRY.template.md](architecture/CONTRACT-REGISTRY.template.md) | Index of all contracts |

### Agent Orchestration

| Directory | Purpose |
|-----------|---------|
| [agents/](agents/) | Subagent guidelines, prompt index, and templates |
| [agents/subagent-prompts/](agents/subagent-prompts/) | UX review, security scan, code review, contract audit, doc drift, feature inventory, test sharding |

### Project Profiles

Different projects need different subsets. Pick your profile:

| Profile | Best For |
|---------|----------|
| [web-app](profiles/web-app.md) | SPA, SSR, web frontend + API |
| [api-service](profiles/api-service.md) | Backend API, microservice |
| [crypto-library](profiles/crypto-library.md) | Security-critical library |
| [cli-tool](profiles/cli-tool.md) | Command-line tool |

### Supporting

| Directory | Purpose |
|-----------|---------|
| [feedback/](feedback/) | Lightweight feedback mechanism — agents drop suggestions here |
| [profiles/](profiles/) | Project-type adoption guides |
| [learnings-from-opendockit.md](learnings-from-opendockit.md) | Battle-tested patterns and failure analysis from 5,800+ tests and 9 simultaneous agents |

## Quick Start

See [SETUP.md](SETUP.md) for the full adoption guide. The short version:

```bash
PROJECT=/path/to/your/project

# Core docs
cp README.template.md       "$PROJECT/README.md"
cp QUICKCONTEXT.template.md "$PROJECT/QUICKCONTEXT.md"
cp TODO.template.md         "$PROJECT/TODO.md"
cp AGENTS.template.md       "$PROJECT/AGENTS.md"
cp CLAUDE.template.md       "$PROJECT/CLAUDE.md"
cp methodology.md           "$PROJECT/methodology.md"

# Contract system
cp -r architecture/         "$PROJECT/architecture/"

# Agent orchestration (optional)
cp -r agents/               "$PROJECT/agents/"

# Customize everything (follow the <!-- comments --> in each template)
```

## Design Philosophy

See [methodology.md](methodology.md) for the complete philosophy. In brief:

1. **Contracts are the operating system** — don't implement without a contract
2. **BDD first** — encode who and why before writing contracts
3. **Max autonomy within contracts** — agents are unrestricted inside contract boundaries
4. **Trust but verify** — freshness markers, pre-launch audits, filesystem as truth
5. **Encode corrections as infrastructure** — if you've corrected an agent, put it in a template
6. **Fast inner loops** — Testing Cascade T0-T5, iterate at the speed of a single test
