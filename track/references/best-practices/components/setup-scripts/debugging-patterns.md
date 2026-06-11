# Debugging Patterns

Evaluates whether setup and lifecycle scripts include diagnostic instrumentation that aids troubleshooting when scripts fail in sandboxes, where interactive debugging is not available.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No diagnostic output; script failures produce only "exit code 1" with no context; errors must be reproduced locally to diagnose |
| 2 | Below Standard | Basic `set -x` enabled but output is noisy and unstructured; no script identification in logs; error location requires manual line counting |
| 3 | Adequate | `set -x` with a banner identifying the script; but no ERR trap, no line numbers in trace output, and no structured error reporting |
| 4 | Good | Custom PS4 with script name and line numbers; ERR trap that reports the failing line; banner-echo at script entry for log navigation (production baseline) |
| 5 | Excellent | All of the above plus conditional debug mode (enable verbose output via an environment variable), structured error context (command, line, exit code), and clean separation between diagnostic and operational output |

## Guidance

Instruqt lifecycle scripts run inside sandboxes where you cannot attach a debugger or SSH in during execution. When a script fails, the only evidence is the captured stdout/stderr. Good diagnostic instrumentation makes the difference between "setup failed" and "setup failed at line 47 installing helm because the GPG key URL returned 404".

### PS4 for line-numbered xtrace

The default `set -x` output shows commands but not where they are in the script. Set `PS4` to include the script name and line number:

```bash
#!/bin/bash
export PS4='+${BASH_SOURCE[0]}[${LINENO}]: '
set -euxo pipefail
```

Output becomes:

```
+setup-workstation[12]: apt-get update
+setup-workstation[13]: apt-get install -y jq curl
+setup-workstation[15]: git clone https://github.com/example/repo.git /root/lab
```

A simpler variant for scripts that are not sourced:

```bash
export PS4='+$0[$LINENO]: '
```

### ERR trap for error context

An ERR trap fires whenever a command exits non-zero (under `set -e`). Use it to report the exact failure location:

```bash
#!/bin/bash
trap 'echo "ERROR at ${BASH_SOURCE[0]}:${LINENO} — command: ${BASH_COMMAND} — exit code: $?"' ERR
set -euxo pipefail
```

When a command fails, this produces:

```
ERROR at setup-workstation:23 — command: helm repo add bitnami https://charts.bitnami.com/bitnami — exit code: 1
```

For more structured reporting, capture additional context:

```bash
trap 'echo "=== SCRIPT FAILURE ===" >&2
      echo "Script: ${BASH_SOURCE[0]}" >&2
      echo "Line:   ${LINENO}" >&2
      echo "Command: ${BASH_COMMAND}" >&2
      echo "Exit:   $?" >&2
      echo "=== END FAILURE ===" >&2' ERR
```

### Banner-echo for log navigation

When reviewing logs from tracks with multiple hosts and challenges, finding the start of a specific script in combined output is difficult. Add a banner at the top of each script:

```bash
#!/bin/bash
echo "=== STARTING: setup-workstation (challenge 02) ==="
set -euxo pipefail

# ... setup logic ...

echo "=== COMPLETED: setup-workstation (challenge 02) ==="
```

For check scripts (where `set -x` is avoided), banners are particularly valuable because they are often the only diagnostic output:

```bash
#!/bin/bash
echo "=== CHECK: check-workstation (challenge 02) ==="

if ! systemctl is-active --quiet nginx; then
  fail-message "nginx is not running."
  exit 1
fi

echo "=== CHECK PASSED: check-workstation (challenge 02) ==="
exit 0
```

### Conditional debug mode

For scripts that produce excessive trace output, gate verbose mode on an environment variable set in `config.yml`:

```bash
#!/bin/bash
if [ "${INSTRUQT_DEBUG:-0}" = "1" ]; then
  export PS4='+${BASH_SOURCE[0]}[${LINENO}]: '
  set -x
fi
set -euo pipefail
```

Set `INSTRUQT_DEBUG=1` in `config.yml` `environment:` during development, remove it before publishing.

### Combining patterns

A production-quality setup script header combining all three patterns:

```bash
#!/bin/bash
export PS4='+${BASH_SOURCE[0]}[${LINENO}]: '
trap 'echo "ERROR at ${BASH_SOURCE[0]}:${LINENO} — command: ${BASH_COMMAND} — exit code: $?" >&2' ERR
set -euxo pipefail

echo "=== STARTING: $(basename "$0") ==="
```

This gives you: line-numbered trace output, automatic error reporting on failure, and a clear log boundary marker.

## What to Watch For

- **No `set -x` in setup scripts** — when setup fails, there is no trace of what happened; always enable xtrace in setup and solve scripts
- **Default PS4 (`+ `)** — usable but loses the script name and line number; set a custom PS4 for any script longer than ~10 lines
- **ERR trap without `set -e`** — the trap only fires under `set -e` (or `set -E` for functions); without it, the trap never triggers
- **`set -x` in check scripts** — xtrace output in check scripts pollutes the learner-visible output; use banners instead, and reserve xtrace for the `INSTRUQT_DEBUG` conditional pattern
- **Banner-echo at the end without `trap`** — the "COMPLETED" banner only prints on success; if the script fails mid-way, the last visible output is the last successful command; the ERR trap covers the failure case
- **Logging sensitive values** — `set -x` prints every command including those with passwords and tokens; either disable xtrace around sensitive commands (`set +x; ... ; set -x`) or use indirect variable expansion to avoid inlining secrets
