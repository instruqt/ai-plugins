# Script Errors

Common script failures and debugging patterns for Instruqt track setup, check, and solve scripts.

## Setup freezes during apt-get install

**Symptom:** Setup appears to freeze partway through apt-get.

**Cause:** Missing `export DEBIAN_FRONTEND=noninteractive` (or `export DEBIAN_FRONTEND noninteractive` without `=`).

**Fix:** Add `export DEBIAN_FRONTEND=noninteractive` at top of setup script. Verify with `declare -p DEBIAN_FRONTEND`.

---

## Silent check script failure ("Something went wrong" modal)

**Symptom:** Generic "Something went wrong" modal with no useful info.

**Cause:** Unguarded subprocess under `set -euo pipefail` exits non-zero, bash exits silently.

**Fix:** Wrap each subprocess with an explicit handler (`VAR=$(...) || fail-message "reason"; exit 1`) or check the Instruqt debug log for the actual error.

---

## fail-message not found (exit 127)

**Symptom:** Check script fails with "command not found".

**Cause:** Raw Docker Hub images (`ubuntu:22.04`, `python:3.11-slim`) don't include Instruqt helpers.

**Fix:** Guard with `|| true` (`fail-message "..." || true`) or use `echo` + `exit 1` as fallback.

---

## pipefail + tee swallows exit code

**Symptom:** Setup proceeds on partial data even though the inner command failed.

**Cause:** `tee` exits 0 even when the heredoc body fails, masking failure under pipefail.

**Fix:** Capture `${PIPESTATUS[1]}` immediately after the pipe.

---

## set -eux pipefail doesn't enable pipefail

**Symptom:** Pipeline failures not caught.

**Cause:** Missing `-o` flag; `pipefail` is interpreted as a positional parameter.

**Fix:** Use `set -euxo pipefail`.

---

## #set -euxo pipefail is a comment

**Symptom:** Setup completes even when commands fail, no xtrace output.

**Cause:** `#s` not `#!/` -- the line is treated as a comment.

**Fix:** Remove the `#`.

---

## nohup jobs die between challenges

**Symptom:** Background process from a prior challenge is gone.

**Cause:** The Instruqt runner doesn't preserve nohup across challenge boundaries.

**Fix:** Warm caches synchronously, or restart the process in the per-challenge setup script.

---

## Bootstrap sentinel not found

**Symptom:** Setup commands fail because the host isn't ready.

**Cause:** Not waiting for `/opt/instruqt/bootstrap/host-bootstrap-completed`.

**Fix:** Add at the top of setup:
```bash
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done
```
Use `gcp-bootstrap-completed` for cloud-client hosts.

---

## dpkg lock contention on Ubuntu 22+/24+

**Symptom:** `apt-get install` blocks or fails silently.

**Cause:** `unattended-upgrades` or `apt-daily.service` holds the dpkg lock at first boot.

**Fix:** Add a `wait_for_dpkg()` function before any apt calls:
```bash
wait_for_dpkg() {
  while fuser /var/lib/dpkg/lock-frontend /var/lib/apt/lists/lock >/dev/null 2>&1; do
    sleep 2
  done
}
```

---

## NEEDRESTART_MODE prompt on Ubuntu 22+

**Symptom:** `apt` hangs asking about service restarts.

**Cause:** `needrestart` is enabled by default on Ubuntu 22+.

**Fix:** Export `NEEDRESTART_MODE=a` alongside `DEBIAN_FRONTEND=noninteractive`:
```bash
export DEBIAN_FRONTEND=noninteractive
export NEEDRESTART_MODE=a
```
