# agent-templates

A complete starter kit for agent-driven software development. Drop these
templates into any project to get a fully wired information environment
that makes AI coding agents (Claude Code, etc.) dramatically more effective.

## The Problem

Agent output quality is bounded by the quality of the context agents receive,
and context degrades over time unless actively maintained. Most projects give
agents a README and hope for the best. This repo gives them a structured
information environment: orientation, norms, task tracking, hazard awareness,
behavioral contracts, and reusable prompt templates — all designed to prevent
the failure modes we've observed across thousands of agent-hours of real
development.

See [learnings-from-opendockit.md](learnings-from-opendockit.md) for the
full story behind these patterns.

## What's Included

### The Cold Start Quad

Four files every agent reads on session start, in order:

| File | Purpose | Template |
|------|---------|----------|
| `QUICKCONTEXT.md` | 30-second orientation — what's true right now | [QUICKCONTEXT.template.md](QUICKCONTEXT.template.md) |
| `KNOWN_ISSUES.md` | What will bite you — blockers, gotchas, workarounds | [KNOWN_ISSUES.template.md](KNOWN_ISSUES.template.md) |
| `TODO.md` | What needs doing — tracked tasks with the two-tag system | [TODO.template.md](TODO.template.md) |
| `AGENTS.md` | How we work — norms, testing cascade, doc maintenance | [AGENTS.template.md](AGENTS.template.md) |

### Core Configuration

| File | Purpose | Template |
|------|---------|----------|
| `CLAUDE.md` | Claude Code project instructions — commands, style, autonomy | [CLAUDE.template.md](CLAUDE.template.md) |

### Agent Orchestration

Reusable prompt templates for subagent delegation — both single-invocation
specialized tasks and parallel fan-out:

| Directory | Contents |
|-----------|----------|
| [agents/](agents/) | README, guidelines, index, and prompt templates |
| [agents/subagent-prompts/](agents/subagent-prompts/) | UX review, security scan, code review, contract audit, doc drift detection, feature inventory, test shard runner |

### Learnings

| File | Contents |
|------|----------|
| [learnings-from-opendockit.md](learnings-from-opendockit.md) | Battle-tested patterns from 5,800+ tests, 9 simultaneous agents, and hard-won failure analysis |

## Quick Start

See [SETUP.md](SETUP.md) for the full adoption guide. The short version:

```bash
# 1. Copy templates into your project
cp QUICKCONTEXT.template.md  /your/project/QUICKCONTEXT.md
cp KNOWN_ISSUES.template.md  /your/project/KNOWN_ISSUES.md
cp TODO.template.md          /your/project/TODO.md
cp AGENTS.template.md        /your/project/AGENTS.md
cp CLAUDE.template.md        /your/project/CLAUDE.md

# 2. Copy the agents/ directory for subagent orchestration
cp -r agents/                /your/project/agents/

# 3. Customize each file (follow the <!-- comments --> in each template)

# 4. Remove the <!-- comment --> blocks when done
```

## Design Philosophy

1. **Layered context** — Agents read orientation first, then norms, then tasks,
   then hazards. Forcing a reading order prevents agents from diving into code
   before they understand the project.

2. **Trust but verify** — Every status-bearing document has a freshness marker.
   Agents cross-reference doc claims against git log and the filesystem before
   trusting them.

3. **Encode corrections as infrastructure** — If you've corrected an agent
   ("no, not like that"), that correction belongs in a template or guideline,
   not in your memory. Templates make agents learn across sessions.

4. **Separation of concerns** — QUICKCONTEXT is "what's true now."
   AGENTS.md is "how we work." TODO.md is "what needs doing."
   KNOWN_ISSUES.md is "what will bite you." Each has one job.

5. **Anti-drift by design** — Freshness timestamps, two-tag TODO tracking,
   pre-commit checks, and doc-drift detection templates all fight the #1
   enemy of agent-driven development: information decay.
