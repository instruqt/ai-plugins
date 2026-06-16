# Write Scripts

Encode lifecycle script conventions for setup, check, solve, and cleanup scripts.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Before Writing

Load relevant guidance using `discover-best-practices`:
- `references/best-practices/components/setup-scripts/` — bootstrap sentinels, DEBIAN_FRONTEND, tool installation, prerequisite verification
- `references/best-practices/components/check-scripts/` — assertion patterns, fail-message, no-pipefail rule
- `references/best-practices/components/solve-scripts/` — idempotent operations, full-chain defensive
- `references/best-practices/components/cleanup-scripts/` — resource teardown, state reset

## Script Conventions by Type

### Setup Scripts

```bash
#!/bin/bash
set -euxo pipefail
```

- Bootstrap sentinel wait at the top (poll for `/opt/instruqt/bootstrap/host-bootstrap-completed`)
- `export DEBIAN_FRONTEND=noninteractive` before any `apt-get`
- Use `apt-get` not `apt`
- **Track-level vs per-challenge placement:** `track_scripts/setup-<host>` runs *once* for the whole track — put shared, challenge-independent capabilities there (base CLIs, runtimes, authenticated cloud access, shared services, starter projects). A challenge's own `<NN-slug>/setup-<host>` runs before that challenge and holds only challenge-specific state. Never repeat a shared install in every challenge; declare it in the track plan's Track-Level Prerequisites and install it in `track_scripts/`. See `references/best-practices/components/setup-scripts/track-vs-challenge-setup.md`.
- Hot-start awareness: `INSTRUQT_USER_EMAIL` and similar vars may be empty during pre-provisioning
- **Verification tail (required):** end the script by asserting every capability it provides from the challenge plan's Prerequisites manifest is actually *functional* — not just installed. Use a `verify` helper that runs the capability's Verify command and `exit 1`s with a located message on failure. Install ≠ functional: a CLI on `PATH` may be unauthenticated, a package may belong to the wrong interpreter, a service may not yet accept connections. See `prerequisite-verification.md` for the helper and per-tool-class patterns; use `readiness-patterns.md` (not `sleep`) for capabilities that take time to become true.

### Check Scripts

```bash
#!/bin/bash
```

- **NO `set -euo pipefail`** — check scripts must run all assertions, not bail on the first failure
- Use `fail-message` helper (available on Instruqt agent images) for user-facing error messages
- Each assertion: test a condition, if it fails call `fail-message "..."` and `exit 1`
- Hard timeout: 60 seconds — no polling loops, no long-running commands
- Must be idempotent and side-effect-free (read-only operations)
- On non-Instruqt images (raw cloud images), `fail-message` may not exist — fall back to `echo "reason"; exit 1`

### Solve Scripts

```bash
#!/bin/bash
set -euxo pipefail
```

- Must produce the exact state that passes the check script
- Idempotent: safe to run multiple times (use `create-if-not-exists` patterns)
- Full-chain defensive: re-create all prerequisites for `skipping_enabled: true` support
- Each step should correspond to what the assignment asks the learner to do

### Cleanup Scripts

```bash
#!/bin/bash
set -euxo pipefail
```

- Tear down resources created during the challenge
- Use `|| true` for deletions that may fail if the resource doesn't exist
- Guard against empty variables before deletion commands
- Place cost-sensitive teardowns in track-level cleanup for reliability

## Naming Convention

Scripts are named by type and target host:
- `setup-<hostname>` — runs on `<hostname>` before the challenge
- `check-<hostname>` — validates learner work on `<hostname>`
- `solve-<hostname>` — auto-solves the challenge on `<hostname>`
- `cleanup-<hostname>` — resets state on `<hostname>` after the challenge

## Shebang and Set Flags

The code blocks above show the **bash** form, which is the default. Pick the shebang and flags by what the host image provides:

1. **bash available (default).** Debian/Ubuntu images, or any host with `bash` on `PATH`.
   - `#!/bin/bash`; setup/solve/cleanup use `set -euxo pipefail`; check scripts use no `set -e`/`-o pipefail`.

2. **Alpine container without bash.** Alpine's `/bin/sh` is BusyBox `ash` and ships no `bash`. Install it once at the top of the track/challenge setup script, then use the bash form everywhere after:
   ```sh
   #!/bin/sh
   set -eu
   apk add --no-cache bash
   ```
   The bootstrapping setup script itself is `#!/bin/sh` + `set -eu` (it runs before bash exists). Every script after the install — solve, cleanup, check — uses the bash form. This is the preferred Alpine path: it keeps the `-x` xtrace and `-o pipefail` guarantees, which surface in the Instruqt logs and are where authors debug `instruqt track test` failures.

3. **POSIX sh only (fallback).** A host with no bash and no way to add it (minimal/distroless images). Use POSIX sh:
   - `#!/bin/sh`; setup/solve/cleanup use `set -eu` (POSIX sh has no portable `-o pipefail` or `-x`); check scripts use no `set -e`.

When unsure which case applies, check the host image in the challenge plan / `config.yml`.
