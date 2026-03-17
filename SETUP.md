# Setup Guide

How to adopt the agent-templates kit in your project.

## Prerequisites

- A git repository
- Claude Code (or another AI coding agent that reads markdown context files)
- 30 minutes for initial setup

## Step 1: Copy the Core Files

```bash
# From the agent-templates directory:
PROJECT=/path/to/your/project

# The Cold Start Quad
cp QUICKCONTEXT.template.md  "$PROJECT/QUICKCONTEXT.md"
cp KNOWN_ISSUES.template.md  "$PROJECT/KNOWN_ISSUES.md"
cp TODO.template.md          "$PROJECT/TODO.md"

# Core config (only if these don't already exist)
cp AGENTS.template.md        "$PROJECT/AGENTS.md"
cp CLAUDE.template.md        "$PROJECT/CLAUDE.md"

# Agent orchestration (optional but recommended)
cp -r agents/                "$PROJECT/agents/"
```

If you already have AGENTS.md or CLAUDE.md, diff the templates against yours
and merge the sections you're missing.

## Step 2: Customize Each File

Work through each file in this order. Every file has `<!-- comment -->`
blocks explaining what to customize. Remove the comments when done.

### CLAUDE.md (15 min)

This is the highest-leverage file — Claude Code reads it every session.

1. **Project name & description** — Replace `<PROJECT_NAME>` with yours
2. **Core Tenets** — Replace the example tenets with your actual principles
3. **Commands** — Fill in your build/test/lint commands
4. **Structure** — Map your project's directory layout
5. **Coding Style** — Describe your conventions
6. **Testing** — Describe your test approach and fill in the cascade tiers
7. **Allowed Commands** — Adjust for your package manager and tools
8. **Environment Variables** — List your project's env vars

### AGENTS.md (15 min)

1. **Core Tenets** — Mirror from CLAUDE.md (keep in sync)
2. **Agent Autonomy** — Adjust the "Requires Discussion" list for your
   architecture's irreversible decisions
3. **Project Structure** — Describe your layout and module ownership
4. **Active Workstreams** — List current priorities
5. **Testing Cascade** — Customize commands for your test runner
6. **Doc Ownership** — Map workstreams to their docs

### QUICKCONTEXT.md (5 min)

1. **Project** — One-liner description
2. **Current Branch & State** — Fill in current branch, test counts
3. **What's In Progress** — What you're actively working on
4. **What's Next** — Immediate priorities (verify against code first!)
5. **Update the freshness date**

### TODO.md (5 min)

1. **P0 items** — Your most urgent tasks
2. **Audit existing TODOs** — Run `grep -rn "TODO:" src/` and either fix them
   or add entries here + convert to `TRACKED-TASK:` in code
3. **Update freshness and last-synced dates**

### KNOWN_ISSUES.md (5 min)

1. **Gotchas** — Things that have tripped you up
2. **Common errors** — Recurring error messages with solutions
3. **Workarounds** — Temporary fixes currently in the codebase

### agents/ directory (optional, 10 min)

1. **subagent-guidelines.md** — Customize the architectural change detection
   section for your project's contracts and sensitive areas
2. **subagent-prompts/** — Review the templates, remove ones you don't need,
   customize ones you'll use (especially ux-review.md and code-review.md —
   their criteria should reflect your standards)
3. **Add .gitignore** for `agents/results/` if you don't want ephemeral
   results committed

## Step 3: Verify

```bash
# Confirm all Cold Start Quad files exist
ls QUICKCONTEXT.md KNOWN_ISSUES.md TODO.md AGENTS.md CLAUDE.md

# Confirm no leftover template placeholders
grep -rn "<PROJECT_NAME>\|<pkg-manager>\|YYYY-MM-DD" \
  QUICKCONTEXT.md KNOWN_ISSUES.md TODO.md AGENTS.md CLAUDE.md

# Confirm no untracked TODOs (the two-tag system starts now)
grep -rn "TODO:" --include="*.ts" --include="*.go" --include="*.py" src/ || echo "Clean"

# Test the agent experience: start a new Claude Code session and see
# if it reads the files in order and understands the project
```

## Step 4: Commit

```bash
git add QUICKCONTEXT.md KNOWN_ISSUES.md TODO.md AGENTS.md CLAUDE.md
git add agents/  # if using subagent orchestration
git commit -m "docs: adopt agent-templates information environment

Add Cold Start Quad (QUICKCONTEXT, KNOWN_ISSUES, TODO, AGENTS),
Claude Code config (CLAUDE.md), and agent orchestration templates.
See: https://github.com/willackerly/agent-templates"
```

## Ongoing Maintenance

The templates are a starting point. The real value comes from maintaining them:

| Cadence | Action |
|---------|--------|
| **Every session start** | Agent reads Cold Start Quad, verifies freshness |
| **Every session end** | Update QUICKCONTEXT.md "In Progress" / "Recently Complete" |
| **Every commit** | Run TODO two-tag check, update docs if needed |
| **Weekly** | Scrub TRACKED-TASK comments, review KNOWN_ISSUES.md staleness |
| **Monthly** | Full doc-drift audit (or use the doc-drift-detector template) |

## Troubleshooting

**Agent ignores the templates:**
Claude Code reads `CLAUDE.md` automatically. Make sure `AGENTS.md` is
referenced from `CLAUDE.md`'s Cold Start section. The other files are read
because the Cold Start section tells the agent to read them.

**Templates feel too heavy for a small project:**
Start with just CLAUDE.md and QUICKCONTEXT.md. Add the others as your
project grows or as you start using parallel agents.

**Agent still makes mistakes I've corrected before:**
The correction belongs in a template or guideline, not just in conversation
history. Add it to the relevant subagent template's Anti-Patterns section
or to AGENTS.md.
