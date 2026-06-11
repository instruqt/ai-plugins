# No Pipefail Rule

Check scripts must NOT use `set -euo pipefail`. This is the opposite of the rule for setup and solve scripts, which SHOULD use it.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Uses `set -euo pipefail` with unguarded subprocesses — any failure silently exits and Instruqt shows "Something went wrong". |
| 2 | Below Standard | Uses `set -e` alone — still causes silent exits on failed checks instead of showing fail-messages. |
| 3 | Adequate | No pipefail, but some subprocesses are unguarded and could cause unexpected exits. |
| 4 | Good | No pipefail, every subprocess is wrapped with an explicit error handler that calls fail-message. Production baseline. |
| 5 | Excellent | No pipefail, all subprocesses guarded, error handlers include diagnostic output (e.g., captured stderr) to help learners. |

## Guidance

### Why pipefail is dangerous in check scripts

Under `set -euo pipefail`, when any command in a pipeline returns non-zero, bash exits immediately. In a check script, this means:

1. The `fail-message` call never executes.
2. Instruqt sees a non-zero exit with no fail-message set.
3. The learner sees a generic "Something went wrong" error with no useful information.

This is especially dangerous with commands that return non-zero as part of normal operation in check scripts: `grep` (no match = exit 1), `curl` (timeout = exit 28), `psql` (query error = exit 2), `python3` (assertion error = exit 1).

### CRITICAL: this is the opposite of setup/solve scripts

| Script type | `set -euxo pipefail` | Why |
|-------------|----------------------|-----|
| setup       | YES — use it         | Failures should halt setup immediately and surface in logs. |
| solve       | YES — use it         | Same as setup — fail fast, surface in logs. |
| check       | NO — never use it    | Failures must be caught and reported to learners via fail-message. |

### Correct pattern — explicit error handling

```bash
#!/bin/bash

# Check that the API returns valid JSON
RESPONSE=$(curl -s --max-time 5 http://localhost:8080/api/health 2>/dev/null)
if [[ $? -ne 0 ]] || [[ -z "$RESPONSE" ]]; then
  fail-message "The API at http://localhost:8080/api/health is not responding. Make sure the application is running."
  exit 1
fi

if ! echo "$RESPONSE" | jq -e '.status == "ok"' &>/dev/null; then
  fail-message "The API health endpoint returned unexpected data. Expected {\"status\": \"ok\"} but got: ${RESPONSE:0:100}"
  exit 1
fi
```

### Wrong — pipefail kills the script silently

```bash
#!/bin/bash
set -euo pipefail

# If curl fails or jq fails, bash exits immediately.
# fail-message never runs. Learner sees "Something went wrong".
RESPONSE=$(curl -s http://localhost:8080/api/health)
STATUS=$(echo "$RESPONSE" | jq -r '.status')

if [[ "$STATUS" != "ok" ]]; then
  fail-message "Health check returned: $STATUS"
  exit 1
fi
```

If the API is not running, `curl` exits non-zero, bash exits, and the learner gets no feedback.

### Wrong — set -e without pipefail is still dangerous

```bash
#!/bin/bash
set -e

# grep returns exit 1 when there is no match.
# Under set -e, this exits the script before fail-message runs.
grep -q "ENABLED=true" /etc/app/config.conf
fail-message "The ENABLED flag is not set to true in /etc/app/config.conf."
exit 1
```

## What to Watch For

- Any `set -e`, `set -o errexit`, `set -euo pipefail`, or `set -o pipefail` at the top of a check script.
- Bare `grep -q` or `curl` calls without `if` / `||` wrappers — these will exit non-zero and, under `-e`, kill the script.
- Subshells that inherit `-e` from the parent shell.
- Functions defined with `-e` active that are called from check logic.
