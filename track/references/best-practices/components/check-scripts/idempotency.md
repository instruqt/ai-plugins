# Idempotency

Check scripts must be side-effect-free on the resource being validated. Clicking Check multiple times must not change the state of the lab environment.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Check script mutates lab state — re-clicking Check changes the outcome or breaks the environment. |
| 2 | Below Standard | Check script creates artifacts (temp files, log entries) that accumulate on repeated runs and could interfere with later checks. |
| 3 | Adequate | No obvious mutations, but check uses commands with side effects that could subtly alter state (e.g., `docker run` instead of `docker ps`). |
| 4 | Good | No state mutations. All checks are pure reads. Re-clicking Check any number of times has no effect on the environment. Production baseline. |
| 5 | Excellent | Pure reads, and any per-check temporary state (temp files, subprocesses) is cleaned up via trap-based cleanup, leaving zero footprint. |

## Guidance

### Why idempotency matters

Learners click Check frequently — often multiple times before they have finished the task. If a check script has side effects, each click could:

- Insert duplicate rows into a database.
- Create additional files or containers.
- Consume limited resources (API quotas, disk space).
- Change the state that the check itself validates, causing it to pass or fail incorrectly.

### Read-only operations only

A check script should only observe state, never modify it.

```bash
# Good — read-only checks
pgrep -x nginx                           # checks if process exists
kubectl get pods -n app                    # reads pod state
psql -tAc "SELECT count(*) FROM users;"   # reads a count
stat /etc/app/config.conf                  # checks file existence
curl -s http://localhost:8080/health       # reads endpoint
grep -q "ENABLED=true" /etc/app/config    # reads file content
```

```bash
# Bad — mutating operations in a check script
docker run --rm alpine echo "test"         # creates and destroys a container
echo "checked" >> /tmp/check.log           # appends to a file on every check
psql -c "INSERT INTO audit(event) ..."     # inserts a row
kubectl apply -f /tmp/test-probe.yaml      # creates a resource
touch /tmp/.check_ran                      # creates a sentinel file
```

### Temporary state with trap cleanup

If a check genuinely needs temporary state (e.g., a temp file to hold intermediate output), clean it up with a trap:

```bash
#!/bin/bash

TMPFILE=$(mktemp)
trap 'rm -f "$TMPFILE"' EXIT

kubectl get pods -n app -o json > "$TMPFILE" 2>/dev/null

READY_COUNT=$(jq '[.items[] | select(.status.phase == "Running")] | length' "$TMPFILE")
if [[ "$READY_COUNT" -lt 3 ]]; then
  fail-message "Expected at least 3 running pods in the app namespace but found $READY_COUNT."
  exit 1
fi
```

The trap ensures the temp file is removed whether the check passes or fails.

### Subtle side effects to avoid

- `docker exec ... sh -c "echo test > /tmp/probe"` — writes inside a container.
- `systemctl restart nginx` — a check should never restart a service the learner configured.
- `apt-get update` — modifies package cache, burns time, and can fail.
- `git clone` or `wget` — downloads files into the filesystem.
- SQL writes of any kind, even to "audit" or "temp" tables.

## What to Watch For

- Any `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, or `ALTER` SQL statements.
- Any `docker run`, `docker exec` with write operations, or `kubectl apply/create/delete`.
- File writes (`>`, `>>`, `tee`, `touch`, `mkdir`, `cp`, `mv`) outside of a trap-cleaned temp directory.
- Service restarts (`systemctl restart`, `service ... restart`).
- Commands that trigger side effects as a byproduct (e.g., `ssh` with host key acceptance, `git clone`).
