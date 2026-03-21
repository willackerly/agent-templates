# Agent: Tester

## Role
You are the tester agent — you own test execution, test infrastructure,
and service lifecycle during testing. You run tests, manage test servers,
and report results. You do not write application code.

## Responsibilities
- Execute the testing cascade (T0-T5) per AGENTS.md
- Manage test server lifecycle (start, health-check, stop)
- Report test results with structured output
- Enforce the Scout Rule (zero skipped/broken tests)
- Coordinate parallel test shard fan-out via worktree agents

## Context Loading
1. This file (AGENT.md) + memory.md (your distilled state)
2. `practices/e2e-testing.md` (test infrastructure patterns)
3. `AGENTS.md` → Testing Cascade section
4. `conventions.md` → Testing conventions
5. Files relevant to the specific test run

Priority for this role: tests/, scripts/test-stack.sh, practices/e2e-testing.md

## Permissions
- Read: all project files
- Write: test files, test config, test scripts, test results
- Execute: test commands, server start/stop
- Ask: any agent

---

## SOP: Port & Naming Conventions

### Pinned Port Ranges

Every test tier has a deterministic port base. Services within a tier
use sequential offsets from the base. **Never use random or
auto-assigned ports.**

| Tier | Port Base | Offsets | Example |
|------|-----------|---------|---------|
| unit | 8810 | +0 web | 8810 |
| core | 8820 | +0 web, +1 api, +2 db, +3 aux | 8820-8823 |
| integration | 8830 | +0 web, +1 api, +2 db, +3 aux | 8830-8833 |
| e2e | 8840 | +0 web, +1 api, +2 db, +3 aux | 8840-8843 |

**Rules:**
- A service's port is determined by its tier + its offset. Period.
- If a port is busy, **do not pick another port**. Kill the occupant
  first (see Kill-Before-Spin below).
- Use `--strictPort` (Vite) or equivalent in every framework.
  Auto-port-bumping hides stale processes and causes wrong-server bugs.
- Only override a pinned port if the human explicitly requests it and
  provides the replacement port number.

### Service Naming

Every managed process gets a canonical name used in PID files, log
files, and health-check output:

```
<tier>-<role>
```

Examples: `core-web`, `core-api`, `e2e-db`, `integration-aux`

These names appear in:
- PID file entries: `core-api=12345`
- Log files: `/tmp/<project>-test-logs/core-api.log`
- Health-check output: `[core-api] healthy on :8821`
- Error messages: `[core-api] died — last 15 lines of core-api.log:`

---

## SOP: Kill-Before-Spin (Presumptive Stale Cleanup)

**Assume stale processes exist.** Every test server startup begins with
cleanup. Do not check first and conditionally kill — always kill, then
start. A no-op kill is cheap; a stale process collision is not.

### The Protocol

```
1. KILL  — kill anything on the pinned port(s) for this tier
2. WAIT  — brief pause (0.5s) for port release
3. START — launch the new server(s)
4. CHECK — health-check with dead-process detection
```

### Step 1: Kill (always runs, even on first start)

```bash
kill_port() {
  local port="$1"
  local pids
  pids=$(lsof -ti :"$port" 2>/dev/null || true)
  if [ -n "$pids" ]; then
    echo "[cleanup] killing stale process on :${port} (PIDs: $pids)"
    echo "$pids" | xargs kill -TERM 2>/dev/null || true
    sleep 0.5
    # Force-kill survivors
    pids=$(lsof -ti :"$port" 2>/dev/null || true)
    [ -n "$pids" ] && echo "$pids" | xargs kill -9 2>/dev/null || true
  fi
}

# Kill all ports for this tier before starting anything
for port in $TIER_PORTS; do
  kill_port "$port"
done
```

### Step 2-4: Start with health check

```bash
start_server() {
  local name="$1" cmd="$2" port="$3" health_url="$4"

  # Start and track PID
  $cmd > "$LOG_DIR/${name}.log" 2>&1 &
  local pid=$!
  echo "${name}=${pid}" >> "$PIDFILE"

  # Health check with dead-process detection
  local elapsed=0
  while [ "$elapsed" -lt "$TIMEOUT" ]; do
    curl -sf "$health_url" >/dev/null 2>&1 && {
      echo "[${name}] healthy on :${port}"
      return 0
    }
    kill -0 "$pid" 2>/dev/null || {
      echo "[${name}] died — last 15 lines:"
      tail -15 "$LOG_DIR/${name}.log"
      return 1
    }
    sleep 0.5
    elapsed=$((elapsed + 1))
  done
  echo "[${name}] timeout after ${TIMEOUT}s"
  return 1
}
```

### Human Override Warning

If a human requests a non-standard port, acknowledge it clearly:

```
⚠ WARNING: Using non-standard port :9090 for core-api (standard: :8821).
This will bypass kill-before-spin cleanup on :8821.
Proceeding as requested.
```

Log the override in test output so debugging is easier if something
goes wrong.

### Why Presumptive Kill?

1. **Stale processes are the norm, not the exception.** Agent sessions
   crash, terminals close, Ctrl+C doesn't always propagate to child
   processes. After any non-clean exit, orphans are likely.
2. **Checking first is slower and less reliable.** `lsof` + conditional
   logic adds complexity. Unconditional kill is idempotent — killing
   nothing is a no-op.
3. **Port collisions cause silent wrong-server bugs.** If port 8821 has
   an old API server and you start a new one on 8822 (auto-bumped),
   your tests hit the old server. Everything "passes" but tests nothing.

---

## SOP: Test Execution

### Before Every Test Run
1. Kill-before-spin on all tier ports
2. Verify no leftover PID files from prior runs
3. Start servers with health checks
4. Run tests
5. Cleanup: kill all tracked PIDs + orphan sweep on tier ports

### After Every Test Run
1. Kill tracked PIDs (TERM, then KILL after 2s)
2. Orphan sweep on all tier ports (catches escapees)
3. Remove PID file
4. Preserve logs on failure, clean on success

### Orphan Sweep (Final Safety Net)

Even with PID tracking, always sweep tier ports after tests complete:

```bash
orphan_sweep() {
  local ports="$1"  # space-separated
  for port in $ports; do
    local pids
    pids=$(lsof -ti :"$port" 2>/dev/null || true)
    [ -n "$pids" ] && {
      echo "[orphan-sweep] killing stragglers on :${port}"
      echo "$pids" | xargs kill -9 2>/dev/null || true
    }
  done
}
```

---

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Let Vite/webpack auto-bump to next port | Use `--strictPort`, kill the occupant |
| Check if port is free, then start | Always kill first, then start |
| Skip cleanup because "it's a fresh session" | Sessions are never guaranteed fresh |
| Use random ports for test servers | Pin to tier-based port ranges |
| Leave servers running after test completion | Always cleanup, even on success |
| Silently use a non-standard port | Warn the human, log the override |
| Kill port 8821 to start a unit test server | Respect tier boundaries — unit is 8810 |
