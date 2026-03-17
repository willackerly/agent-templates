# Known Issues

<!-- FRESHNESS: Update this date every time you modify this file -->
<!-- freshness: YYYY-MM-DD -->

**What will bite you.** Read this before starting work. Organized by severity.

---

## Active Blockers

<!-- Things that PREVENT work from proceeding. These need resolution before
     the blocked work can continue. Include who/what is blocked and any
     workarounds. Remove promptly when resolved (move to Recently Resolved). -->

<!-- Example:
### OCIS auth token expiry
**Blocks:** all integration tests
**Symptoms:** 401 errors after ~1 hour of testing
**Workaround:** Restart the dev stack (`docker compose down && docker compose up`)
**Tracking:** TODO.md "Infrastructure" section
-->

_None currently._

## Gotchas

<!-- Things that CAUSE CONFUSION but don't block work. These are the
     "I wish someone had told me" items — counter-intuitive behaviors,
     misleading names, surprising defaults, undocumented dependencies.
     Keep indefinitely until the underlying cause is fixed. -->

<!-- Example:
### Playwright HTML reporter blocks in non-interactive mode
If you run Playwright without `open: 'never'` in the reporter config,
it starts an HTTP server after test failure and hangs waiting for Ctrl+C.
**Fix:** Always use `reporter: [['html', { open: 'never' }], ['list']]`
-->

_None documented yet._

## Common Errors & Solutions

<!-- Errors agents (or humans) hit repeatedly, with the known fix.
     Format: error message → cause → solution.
     These accumulate over time and save enormous repeated debugging. -->

<!-- Example:
### `Error: EADDRINUSE :::8821`
**Cause:** Orphaned test server from a previous run
**Fix:** `lsof -ti :8821 | xargs kill` or `./scripts/test-stack.sh stop`
-->

_None documented yet._

## Workarounds In Place

<!-- Temporary fixes that are currently active in the codebase.
     These MUST link to a tracking item (TODO.md or issue tracker).
     Include: what the workaround does, why it's needed, and when/how
     it should be removed. -->

<!-- Example:
### Blob URL workaround for web workers
**Where:** `packages/render/src/worker-loader.ts:15`
**Why:** CDN serves `.mjs` files with wrong MIME type
**Tracking:** TODO.md "Infrastructure" → "Fix CDN MIME types"
**Remove when:** CDN configuration is updated to serve .mjs as application/javascript
-->

_None documented yet._

## Recently Resolved

<!-- Issues fixed in the last 2-4 weeks. Keep temporarily so agents don't
     waste time investigating problems that have already been solved.
     Archive or delete after a month. -->

<!-- Example:
### ~~Font metrics mismatch~~ (resolved 2026-03-15)
Was causing 14 test failures across 3 packages. Fixed by regenerating
the metrics bundle with Inter/Aptos entries. See commit `abc1234`.
-->

_None yet._

---

<!-- MAINTENANCE NOTES:
1. Add issues AS SOON AS you discover them — don't wait for a "good time"
2. Include the FIX, not just the problem. An issue without a solution is
   just a complaint.
3. Move resolved issues to "Recently Resolved" — don't just delete them.
   Other agents may be investigating the same problem.
4. Workarounds MUST link to tracking. Untracked workarounds become permanent.
5. Review this file at the start of each session — remove stale entries,
   add new discoveries.
-->
