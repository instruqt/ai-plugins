# Set Flags

Solve scripts use `set -euxo pipefail`. Every failure should surface immediately, and xtrace provides debugging output in Instruqt logs. This is the opposite of check scripts, which must NOT use these flags.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No set flags — failures are silently swallowed, solve appears to succeed but leaves the environment in a broken state. |
| 2 | Below Standard | Uses `set -e` alone — failures exit but no trace output for debugging, and pipelines can hide errors. |
| 3 | Adequate | Uses `set -euo pipefail` — catches failures and unset variables, but no xtrace for debugging. |
| 4 | Good | Uses `set -euxo pipefail` — all failures surface, xtrace provides full debugging output in logs. Production baseline. |
| 5 | Excellent | Uses `set -euxo pipefail` and handles known-acceptable non-zero exits explicitly (e.g., `|| true` for idempotent creates) without disabling the flags globally. |

## Guidance

### The correct shebang and flags

Every solve script should start with:

```bash
#!/bin/bash
set -euxo pipefail
```

**On non-bash hosts.** Alpine's `/bin/sh` (BusyBox `ash`) and other minimal images may ship no bash. Prefer installing bash early in setup (`apk add --no-cache bash` on Alpine) so solve scripts keep the full `set -euxo pipefail`. When that isn't possible, write the solve script as `#!/bin/sh` + `set -eu` — the portable subset; `-o pipefail` and `-x` are not guaranteed in POSIX sh. On a `/bin/sh` host, `set -eu` is the baseline (treat it as the score-4 equivalent); the `-x`/pipefail levels in the table above apply to bash.

### What each flag does

| Flag | Effect | Why it matters for solve |
|------|--------|------------------------|
| `-e` (errexit) | Exit immediately when a command returns non-zero. | A failed solve step should stop execution, not continue and create a half-built state. |
| `-u` (nounset) | Treat unset variables as an error. | Catches typos and missing variable assignments before they cause silent data corruption. |
| `-x` (xtrace) | Print each command to stderr before executing it. | The trace output appears in Instruqt logs, making it possible to debug solve failures without reproducing them locally. |
| `-o pipefail` | A pipeline returns the exit code of the last command that failed, not just the last command. | Without this, `curl ... \| jq ...` succeeds even if `curl` fails, as long as `jq` exits 0. |

### Why solve scripts NEED these flags (and check scripts do NOT)

| Script type | Flags | Reason |
|-------------|-------|--------|
| solve | `set -euxo pipefail` | Fail fast. A broken solve must surface immediately in logs so the author can debug it. There is no learner-facing feedback — only logs. |
| check | None | Check scripts must catch every error and produce a `fail-message`. Under `-e`, the script exits before `fail-message` runs, and the learner sees a generic error. |

### Handling expected non-zero exits

Some commands legitimately return non-zero in idempotent patterns. Use `|| true` to suppress these without disabling flags globally:

```bash
#!/bin/bash
set -euxo pipefail

# createdb returns non-zero if database already exists — this is fine
psql -U postgres -c "CREATE DATABASE lab;" 2>/dev/null || true

# docker rm returns non-zero if container does not exist — this is fine
docker rm -f app 2>/dev/null || true
docker run -d --name app nginx:1.25

# kubectl create namespace returns non-zero if it exists — this is fine
kubectl create namespace app 2>/dev/null || true
kubectl apply -f /opt/manifests/deployment.yaml
```

### Bad — no flags

```bash
#!/bin/bash

# No flags. If psql fails, the script continues.
# The database does not exist, but the script exits 0.
# Check then fails with a confusing error.
psql -U postgres -d lab -c "CREATE TABLE IF NOT EXISTS users (id SERIAL, name TEXT);"
psql -U postgres -d lab -c "INSERT INTO users (name) VALUES ('alice');"
```

### Bad — disabling flags mid-script

```bash
#!/bin/bash
set -euxo pipefail

# Turning off errexit to "handle" an error
set +e
some-flaky-command
set -e
# This defeats the purpose. If some-flaky-command is expected to fail,
# use || true. If it's not expected to fail, let it fail.
```

### The xtrace benefit

With `-x`, every command is logged before execution:

```
+ psql -U postgres -c 'CREATE DATABASE lab;'
+ psql -U postgres -d lab -c 'CREATE TABLE IF NOT EXISTS users ...'
+ psql -U postgres -d lab -c 'INSERT INTO users ...'
```

When a solve fails during `instruqt track test`, this trace in the logs shows exactly which command failed and with what arguments. Without `-x`, you only see the error message with no context about which step produced it.

## What to Watch For

- Missing `set` flags entirely — the most common problem.
- `set -e` without `-o pipefail` — pipeline failures are hidden.
- `set -euo pipefail` without `-x` — debugging solve failures requires guesswork.
- `set +e` blocks that re-enable errexit later — these hide real errors.
- Commented-out set flags (`# set -euxo pipefail`) — usually means the author hit an error and disabled flags instead of fixing the root cause.
- `|| true` on commands that should NOT be expected to fail — this suppresses real errors.
