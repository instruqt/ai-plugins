# Timeout Awareness

Check scripts have a hard 1-minute (60-second) timeout enforced by the Instruqt platform. If a check script does not exit within 60 seconds, Instruqt kills it and shows the learner a generic timeout error.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Contains unbounded network calls, blocking waits, or polling loops that can easily exceed 60 seconds. |
| 2 | Below Standard | Most checks are fast, but one or more subprocesses lack timeouts and could hang (e.g., bare `curl` with no `--max-time`). |
| 3 | Adequate | All checks are fast in the happy path, but no defensive timeouts on subprocesses that could stall on failure. |
| 4 | Good | All checks complete well within 60 seconds. No network calls that could hang — all external commands have explicit timeouts. Production baseline. |
| 5 | Excellent | All subprocesses use `timeout --kill-after` or equivalent, with per-command budgets that sum to well under 60 seconds. Script is structured so the most common failure path exits fastest. |

## Guidance

### The 1-minute limit

Instruqt enforces a **60-second hard timeout** on check scripts. There is no configuration to change this. When the timeout fires:

- The script is killed.
- The learner sees a generic timeout error, not your fail-message.
- No partial output is preserved.

This means every code path in a check script must complete within 60 seconds, including failure paths.

### Defensive timeouts on subprocesses

Any subprocess that makes a network call, queries a database, or interacts with an external service can hang indefinitely. Always set explicit timeouts.

```bash
# Good — explicit timeout on curl
STATUS=$(curl -s -o /dev/null -w '%{http_code}' --connect-timeout 3 --max-time 5 http://localhost:8080/ 2>/dev/null)

# Good — timeout wrapper on a database query
ROW_COUNT=$(timeout --kill-after=3 10 psql -U postgres -d lab -tAc "SELECT count(*) FROM users;" 2>/dev/null)

# Good — timeout on kubectl
PODS=$(timeout --kill-after=3 10 kubectl get pods -n app -o jsonpath='{.items[*].status.phase}' 2>/dev/null)

# Bad — bare curl, can hang forever
STATUS=$(curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/)

# Bad — bare psql, can hang on connection
ROW_COUNT=$(psql -U postgres -d lab -tAc "SELECT count(*) FROM users;")
```

### Budget your time

If a check script has multiple network calls, each must have a timeout that leaves room for the others. A good rule of thumb:

- 5 seconds per network check
- 10 seconds per database query
- Reserve 10 seconds of headroom for script overhead
- Total budget: N checks x seconds_per_check + 10 < 60

### Never use polling loops in check scripts

Polling loops (`until ... do sleep N; done`) belong in setup scripts, not check scripts. A check script validates current state — it does not wait for state to become ready.

```bash
# Bad — polling loop in a check script, can easily exceed 60 seconds
until curl -s http://localhost:8080/health | grep -q '"ok"'; do
  sleep 5
done
```

```bash
# Good — single check with timeout
HEALTH=$(curl -s --max-time 5 http://localhost:8080/health 2>/dev/null)
if ! echo "$HEALTH" | grep -q '"ok"'; then
  fail-message "The health endpoint is not returning ok. Make sure the application is running and healthy."
  exit 1
fi
```

### The timeout --kill-after pattern

`timeout --kill-after=KILL_DURATION DURATION COMMAND` sends SIGTERM after DURATION, then SIGKILL after KILL_DURATION if the process has not exited. This handles processes that trap or ignore SIGTERM.

```bash
RESULT=$(timeout --kill-after=3 10 some-command 2>/dev/null)
if [[ $? -eq 124 ]]; then
  fail-message "The command timed out. This usually means the service is not running or is unresponsive."
  exit 1
fi
```

Exit code 124 from `timeout` means the command was killed due to timeout.

## What to Watch For

- Bare `curl`, `wget`, `psql`, `mysql`, `kubectl`, `docker exec`, or `ssh` calls with no timeout flags.
- Polling loops (`while`/`until` with `sleep`) — these do not belong in check scripts.
- Multiple network calls whose timeouts could sum to more than 50 seconds.
- Commands that connect to external services (not localhost) — these are especially prone to hanging.
- `timeout` without `--kill-after` — some processes ignore SIGTERM.
