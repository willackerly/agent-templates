# Feedback: Day-One Adoption Learnings (Full Contract System Migration)

**Source project:** Dapple SafeSign (pdf-signer-web)
**What happened:** Full contract system adoption in a single session — scaffolding, 17 contracts, 169 headers, 4 role definitions, ASK CLI integration
**Date:** 2026-03-18

---

## The Session in Numbers

| Metric | Count |
|--------|-------|
| Contracts created | 17 |
| CONTRACT: headers (Tier 1) | 47 |
| Architecture: headers (Tier 2) | 121 |
| Total headers | 169 (across 168 files) |
| Worktree agents used | 18 (across 3 parallel phases) |
| Commits | 32 |
| Wall clock time | ~3 hours (including doc consistency work before contracts) |
| Files created | ~60 |
| Files modified | ~170 |

---

## What Worked

### 1. Fanout by Directory = Zero Merge Conflicts

Sharding worktree agents by directory (not by contract) prevented all merge conflicts. 18 worktree agents, 3 parallel phases, 0 conflicts. The cherry-picks were all clean except for 4 auto-merges on files that had been edited earlier in the session (JSDoc additions) — git resolved these automatically.

**Key principle:** Shard by filesystem boundary, not by logical boundary. Two agents working on different contracts that touch the same directory WILL conflict. Two agents working on different directories that implement the same contract WON'T.

### 2. The Registry Check Script Needs CONTRACT-GAPS.md Awareness

`check-registry.sh` iterates over `architecture/CONTRACT-*.md` files and needs to skip `CONTRACT-GAPS.md` (it's a tracking file, not a contract). The template script's exclusion list has `CONTRACT-TEMPLATE.md` and `CONTRACT-REGISTRY.md` but not `CONTRACT-GAPS.md`.

**Fix needed in template:** Add `CONTRACT-GAPS.md` to the skip list in both loops of `check-registry.sh`.

### 3. The Registry Format vs check-registry.sh Expectations

The registry check script greps for exact filenames (e.g., `CONTRACT-S1-ENVELOPE-SERVICE.1.0.md`) in the registry file. But the natural registry format uses table columns: `| S1 | ENVELOPE-SERVICE | 1.0 |`. The grep doesn't match.

**Two solutions (we chose #1):**
1. Add a "Contract Files" section to the registry with a plain list of filenames
2. Make the script smarter about parsing table rows

Option 1 is simpler and more resilient. Recommend adding this to the registry template.

### 4. Orphan Detection Finds Real Gaps

The `check-registry.sh` orphan detection (contracts with no implementing code) immediately found two real issues:

- **C3-DAPPLE-HELPER-DATA**: implementing files are JavaScript in `reference-implementations/biometric3/`, outside `packages/`. The script only searches `.ts/.tsx/.go/.py` files. JS files in non-standard directories are invisible.
- **P2-SIGNING-LINK-TOKEN**: `signing-link.ts` was tagged `S1-ENVELOPE-SERVICE` (its parent service) instead of `P2-SIGNING-LINK-TOKEN` (its own protocol contract). This is a legitimate classification question — a file can belong to a service AND implement a protocol.

**Recommendation:** The contract system should support dual-tagging. The template's `conventions.md` shows a "Multiple Contracts" example but the enforcement scripts don't account for it in orphan detection.

### 5. ASK CLI Needs `claude` Binary on PATH

The ASK CLI (`bin/ask`) shells out to `claude` for agent reasoning. This requires the Claude Code CLI binary to be on PATH. In our case it was at `/Users/will/.local/bin/claude`, which wasn't in the default PATH for the env settings.

**The trap:** The `ask` command works from the user's terminal (which sources `.zshrc`) but not from inside Claude Code's Bash tool (which uses the `env.PATH` from settings). You need BOTH:
1. `~/.local/bin` in the shell profile (for terminal use)
2. `~/.local/bin` in `.claude/settings.local.json` env.PATH (for Claude Code use)

**Recommendation:** `ask init` should check for `claude` on PATH and warn if not found, suggesting the PATH fix.

### 6. The Tier System Works in Practice

The three-tier classification (Tier 1: contract-owning, Tier 2: architecture-belonging, Tier 3: no header) made the stamping phases tractable:

- **Tier 1 was obvious:** API routes, shared types, crypto modules, SDK entry points. No ambiguity.
- **Tier 2 was obvious for ~90% of files:** DB operations belong to their route's service, React components belong to the envelope service, hooks belong to whatever domain they serve.
- **Tier 2 had ~10% judgment calls:** Does `pdf-form-fill.ts` belong to S1-ENVELOPE-SERVICE or should it have its own contract? Does `share-presenter.ts` belong to S3-SYNC-SERVICE or S1? We defaulted to the parent service and tracked the judgment calls in CONTRACT-GAPS.md.

**Recommendation:** Add "when in doubt, Tier 2 under the parent service" as explicit guidance.

---

## What Surprised Us

### 1. 336 Contract References from 169 Headers

Each header produces ~2 grep-matchable references (`@contract CONTRACT:X` and `@see architecture/CONTRACT-X.md`). With 169 files, that's 336 searchable reference points. The contract system is immediately queryable — `grep -rn "CONTRACT:S1-ENVELOPE-SERVICE" packages/` returns 30+ files instantly.

This is the value proposition working exactly as designed.

### 2. Contract Documents Were Fast to Generate

The 6 parallel worktree agents created 17 contracts in ~25 minutes. The key: each contract had rich source material already (our architecture docs and API specs). The agents were essentially reformatting existing knowledge into the contract template format, not inventing contracts from scratch.

**Insight for other projects:** If you have good architecture docs already, contract creation is a reformatting exercise, not a design exercise. If you don't have architecture docs, you have a bigger problem than contracts.

### 3. The ASK CLI Made Role Definitions Immediately Useful

Within minutes of creating the role AGENT.md files, we could `ask product "Should we add Google Sign-In?"` and get a response that correctly applied the persona decision framework and said "no" with reasoning. The role definitions aren't just documentation — they're immediately executable context.

**This validates the hierarchy as a practical tool, not just an organizational concept.**

---

## Recommendations for the Templates

### 1. `check-registry.sh` Fixes

- Add `CONTRACT-GAPS.md` to both exclusion loops
- Support a "Contract Files" section format (list of filenames) for reliable matching
- Consider dual-tag awareness in orphan detection

### 2. `conventions.md` Additions

- "When in doubt, Tier 2 under the parent service" guidance
- Note that JS/JSX files outside standard directories may not be found by enforcement scripts
- Dual-tagging example with enforcement implications

### 3. Registry Template Update

Add a "Contract Files" section to `CONTRACT-REGISTRY.template.md`:

```markdown
## Contract Files

<!-- Used by check-registry.sh to verify all contract files are tracked.
     Add the exact filename of each contract here. -->
- CONTRACT-{ID}-{NAME}.{VERSION}.md
```

### 4. `ask init` Improvements

- Check for `claude` binary on PATH, warn if missing
- Suggest PATH fix for `.claude/settings.local.json` env
- Detect existing role directories and skip creation (it already does this — good)

### 5. Adoption Timing Guidance in SETUP.md

Add this insight: "If you already have architecture docs and API specs, contract adoption is a 2-3 hour reformatting exercise with 6-way worktree fanout. If you don't have architecture docs, write those first — you can't formalize contracts for systems you haven't documented."

### 6. The "Shining Example" Pattern

Projects that want to be showcase adoptions should follow this sequence:
1. Doc consistency (fix all numeric claims, stale refs)
2. API spec parity (one spec per route module)
3. Contract scaffolding (methodology, conventions, architecture/, agents/)
4. Contract creation (worktree fanout from existing docs)
5. Header stamping (Tier 1 then Tier 2, both via worktree fanout)
6. Role definitions (4 AGENT.md files with project-specific context)
7. ASK CLI integration (immediate payoff from role definitions)
8. BDD Gherkin scenarios (capstone — last, not first)

This is the reverse of the greenfield sequence (BDD → contracts → code) but it's the only practical order for existing projects with mature codebases.
