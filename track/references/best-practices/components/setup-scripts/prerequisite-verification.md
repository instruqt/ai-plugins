# Prerequisite Verification

A setup script is not done when it has *run* the install commands — it is done when the capabilities the challenge depends on are actually **functional**. The most common silent track failure is a setup script that installs something which is then unusable: a CLI on `PATH` but not authenticated, a runtime present under a different name than the assignment calls, a package installed for the wrong interpreter, a service started but not yet accepting connections. The script exits 0, the learner hits a wall, and nothing flagged it.

Every setup script must therefore do two things for each capability it provides: **install/configure it**, then **verify it works** — and fail loudly if verification fails. The list of capabilities to verify comes from the Prerequisites manifest in the challenge plan (and the track plan, for track-level setup).

## The principle: install ≠ functional

"Present" and "working" are different claims, and the assignment depends on *working*. Pick a verify command that proves the functional outcome, not just existence:

| Capability | Weak check (existence only) | Strong check (functional) |
|------------|------------------------------|----------------------------|
| Cloud CLI authenticated | `command -v gcloud` | `gcloud auth list --filter=status:ACTIVE --format='value(account)'` returns an account |
| Cluster reachable | `command -v kubectl` | `kubectl cluster-info` succeeds (and context is the expected one) |
| Runtime + package | `command -v python3` | `python3 -c "import anthropic"` succeeds |
| Service up | process is running | `curl -fsS localhost:8080/health` returns 200 (see `readiness-patterns.md`) |
| Tool configured | binary exists | the tool runs its first real subcommand without error |

## The verification tail

End every setup script with a block that asserts each provided capability. A small helper keeps it readable and makes the failure message point at the exact capability:

```bash
#!/bin/bash
set -euxo pipefail

# ... bootstrap sentinel wait, installs, configuration ...

# --- Verification tail: every capability must be functional before the challenge starts ---
verify() {
  # $1 = human description, $2... = command that must succeed
  local desc="$1"; shift
  if ! "$@" >/dev/null 2>&1; then
    echo "SETUP VERIFICATION FAILED: ${desc}" >&2
    echo "  command: $*" >&2
    exit 1
  fi
}

verify "gcloud authenticated"            gcloud auth list --filter=status:ACTIVE --format='value(account)'
verify "kubectl reaches cluster"         kubectl cluster-info
verify "python3 can import the SDK"      python3 -c "import anthropic"
verify "order API answering"             curl -fsS http://localhost:8080/health
```

Because setup scripts run under `set -euxo pipefail`, an unverified install failure may already abort — but only if that specific command returns non-zero. The tail catches the larger class of failures where the install *succeeded* but the result is unusable. It runs on every sandbox start, needs no test harness, and turns a silent runtime wall into a loud, located setup failure.

For checks that legitimately take time to become true (a service still starting), wrap the verify in the retry/backoff helper from `readiness-patterns.md` rather than a bare call — do not just `sleep`.

## Patterns by capability class

### CLI on PATH

Verify the binary resolves *and* runs. `command -v` alone misses a broken or partial install:

```bash
verify "terraform usable" terraform version
```

### CLI that needs auth or a backend

Existence says nothing about whether it can do anything. Verify against the actual backend:

```bash
verify "gcloud authenticated"  gcloud auth list --filter=status:ACTIVE --format='value(account)'
verify "kubectl context"       bash -c '[ "$(kubectl config current-context)" = "workshop" ]'
verify "helm can reach repo"   helm search repo bitnami/nginx --version 15.0.0
```

### Language runtime + packages

Two traps live here, and both are tool-class-general (Python is just the most frequent example):

1. **Interpreter naming.** The binary the assignment tells the learner to run must be the one that exists. `python` vs `python3`, `pip` vs `pip3`, `node` vs `nodejs` differ across base images. Decide the canonical name, ensure it resolves (symlink or alias if needed), and verify *that* name:

   ```bash
   verify "python3 resolves" command -v python3
   ```

2. **Packages belong to an interpreter, not the machine.** A package installed for one interpreter (or inside a venv) is invisible to another. Verify the import using the exact interpreter the learner will invoke — and if a virtual environment is involved, remember that activation in setup does **not** persist into the learner's shell. Either install into the system/user environment the learner's shell will use, or make activation automatic (e.g. via `~/.bashrc` or a wrapper), then verify through that same path:

   ```bash
   verify "SDK importable by the learner's interpreter" python3 -c "import anthropic, dotenv"
   ```

The same shape applies to any runtime: `node -e "require('@some/sdk')"`, `ruby -e "require 'somegem'"`, `go run` a tiny import probe, etc.

### Service reachable

Defer to `readiness-patterns.md` — poll the actual readiness signal (HTTP status, port accept, sentinel file) with bounded retries, then `verify` the final state.

### Environment variable present

If the assignment or later scripts depend on a variable, verify it is set and non-empty (and account for hot-start, where `INSTRUQT_*` vars may be empty during pre-provisioning — see `hot-start-awareness.md`):

```bash
verify "API key present" bash -c '[ -n "${WORKSHOP_API_KEY:-}" ]'
```

## What to Watch For

- Verifying existence (`command -v`) when the assignment needs function (auth, a reachable backend, a working import). Existence checks are necessary but rarely sufficient.
- Packages installed for a different interpreter or inside a venv that the learner's shell never activates — verify through the exact interpreter/path the learner uses.
- Interpreter/CLI name mismatches between the image and the assignment (`python` vs `python3`). Pin the canonical name and verify it resolves.
- Track-level capabilities re-verified (or worse, re-installed) in every challenge. Verify track-provided capabilities in track setup; challenge setup verifies only what it adds.
- `sleep`-based waits instead of bounded readiness polling for services — use `readiness-patterns.md`.
- A verification tail that swallows output so thoroughly the failure is unlocatable. Always echo which capability failed and the command that was run.
