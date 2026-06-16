# Lifecycle Scripts

Bash scripts (Linux) or PowerShell scripts (Windows) that run at specific points during a track's lifecycle. They set up environments, validate learner work, provide solutions, and clean up resources.

## Structure

### File Locations

```
track/
  track_scripts/
    setup-{hostname}          # track-level setup
    cleanup-{hostname}        # track-level cleanup
  01-challenge-slug/
    setup-{hostname}          # per-challenge setup
    check-{hostname}          # per-challenge check
    solve-{hostname}          # per-challenge solve
    cleanup-{hostname}        # per-challenge cleanup
```

The filename suffix encodes the target hostname: `setup-host01`, `check-mongodb`, `solve-ubuntu`.

### Shebangs

- `#!/bin/bash` or `#!/usr/bin/env bash` — standard Linux (Debian/Ubuntu). Put flags in the body, not the shebang: setup/solve/cleanup use `set -euxo pipefail`; check scripts use no `set -e`/`set -o pipefail` (they must run every assertion).
- `#!/bin/sh` with `set -eu` — Alpine or stripped containers without bash. On Alpine, prefer `apk add --no-cache bash` in setup and then use the bash form, which keeps the `-x`/pipefail guarantees; POSIX `sh` has no portable `-o pipefail` or `-x`.
- Windows VMs: no shebang (PowerShell assumed)

See `best-practices/components/solve-scripts/set-flags.md` for the full per-shell rule.

**Bash version guard:** Some container images ship old bash (< 4.x) that lacks features like associative arrays. Force a re-exec under a known-good binary:

```bash
[ "${BASH_VERSINFO[0]:-0}" -lt 4 ] && exec /bin/bash "$0" "$@"
```

## Helper Commands

Instruqt injects helper commands into the script environment. Availability is **image-dependent** — most official Instruqt images include them, but custom or minimal images may not. Test with your target image.

### fail-message

Displays learner-facing error text when a check fails. Only meaningful in check scripts.

```bash
fail-message "The config file is missing. Create /etc/app/config.yaml as described above."
```

### set-status

Displays a success message to the learner after a check passes. The success-side counterpart to `fail-message`.

```bash
set-status "Great work! The service is configured correctly."
```

### set-workdir

Sets the default working directory for terminal tabs on a host. Call from setup scripts to land the learner in the right place.

```bash
set-workdir /root/project
```

## Script Types

### setup

Runs before the learner sees the challenge. Prepares the environment.

**Track-level** (`track_scripts/setup-{hostname}`):
- Bootstrap waits
- Image registration
- Cluster creation
- Repository cloning
- Shared certificate generation

**Per-challenge** (`{challenge-dir}/setup-{hostname}`):
- State resets between challenges
- Per-challenge data seeding
- Progressive environment changes

**Conventions:**

```bash
#!/bin/bash
set -euxo pipefail

# Wait for Instruqt bootstrap to complete
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# For cloud-client hosts, use the GCP-specific bootstrap marker:
# until [ -f /opt/instruqt/bootstrap/gcp-bootstrap-completed ]; do
#   sleep 1
# done

# Required for non-interactive apt
export DEBIAN_FRONTEND=noninteractive

apt-get update
apt-get install -y jq curl

# Verification tail: assert the capabilities this script provides are functional,
# not just installed. Fail loudly so a broken setup surfaces here, not as a wall
# the learner hits mid-challenge. See best-practices/.../setup-scripts/prerequisite-verification.md
command -v jq >/dev/null || { echo "SETUP VERIFICATION FAILED: jq not usable" >&2; exit 1; }
```

End every setup script (track-level and per-challenge) with this verification tail, asserting each capability from the plan's Prerequisites manifest. Install ≠ functional — a CLI may be unauthenticated, a package installed for the wrong interpreter, a service not yet accepting connections.

### check

Validates whether the learner completed the challenge correctly. Triggered when the learner clicks "Check".

**Conventions:**

- Exit `0` on success, exit `1` on failure.
- Use `fail-message` to display learner-facing error text.
- Use `set-status` to display success messages.
- Write granular assertions with specific remediation guidance per check.
- Be aware of the **1-minute timeout** -- checks that take longer will be killed.
- **Do NOT use `set -euo pipefail`** in check scripts. It causes silent failures when intermediate commands return non-zero, hiding the real error from the learner.

```bash
#!/bin/bash

# Verify the service is running
if ! systemctl is-active --quiet nginx; then
  fail-message "nginx is not running. Start it with: systemctl start nginx"
  exit 1
fi

# Verify the config file exists
if [ ! -f /etc/nginx/conf.d/app.conf ]; then
  fail-message "Missing /etc/nginx/conf.d/app.conf. Create the configuration file as described in the instructions."
  exit 1
fi

exit 0
```

### solve

Produces the exact state that the check script expects. Used for automated testing and the "Skip" button.

**Conventions:**

```bash
#!/bin/bash
set -euxo pipefail

# Idempotent: re-create entire prerequisite chain defensively
systemctl stop nginx 2>/dev/null || true

cat > /etc/nginx/conf.d/app.conf <<'CONF'
server {
    listen 80;
    server_name app.example.com;
    location / {
        proxy_pass http://localhost:3000;
    }
}
CONF

systemctl start nginx
systemctl enable nginx
```

Key properties:
- Must be **idempotent** -- safe to run multiple times.
- **Full-chain defensive** -- re-creates the entire prerequisite chain, not just the final step.
- Produces the exact state that `check-{hostname}` validates.

### cleanup

Runs after a challenge or track ends. Tears down resources and manages costs.

**Conventions:**

```bash
#!/bin/bash
set -euxo pipefail

# Retrieve resource IDs stored during setup
CLUSTER_ID=$(agent variable get CLUSTER_ID)

# Delete the resource
az aks delete --name "$CLUSTER_ID" --resource-group lab-rg --yes --no-wait
```

Use `agent variable get` to retrieve resource identifiers stored during setup for deletion.

## Fields

| Aspect | Value |
|--------|-------|
| Shell | Bash (Linux), PowerShell (Windows) |
| Working directory | `/root` by default on Linux VMs; `/` on containers |
| Environment | All `config.yml` `environment:` vars, all secrets, Instruqt built-in vars (see below) |
| Max script size | 5 MB — do not embed binary data or large heredocs in scripts |
| Exit codes (check) | 0 = pass, 1 = fail |

### Timeouts

Every lifecycle script has a hard timeout enforced by the platform. Scripts killed by timeout produce no learner-facing message.

| Script | Timeout |
|--------|---------|
| Track setup | 55 min |
| Track cleanup | 55 min |
| Challenge setup | 55 min |
| **Challenge check** | **1 min** |
| Challenge solve | 55 min |
| Challenge cleanup | 55 min |

The 1-minute check timeout is the critical constraint. Check scripts must be lightweight — avoid expensive operations like full cluster health checks, large file scans, or commands that wait for convergence. If a check needs to validate something slow, validate the configuration or trigger rather than the final converged state.

## Built-in Environment Variables

Instruqt injects environment variables into every lifecycle script automatically. These supplement any variables defined in `config.yml` `environment:` blocks and secrets.

### Platform Variables

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AUTH_TOKEN` | Platform-injected JWT for authenticating Instruqt API calls from within scripts |
| `INSTRUQT_TRACK_SLUG` | Slug of the current track (e.g., `getting-started-with-terraform`) |
| `INSTRUQT_TRACK_ID` | UUID of the current track |
| `INSTRUQT_PARTICIPANT_ID` | Unique identifier for the current track session |

### Sandbox Variables

| Variable | Description |
|----------|-------------|
| `_SANDBOX_ID` | Unique sandbox instance identifier |
| `_SANDBOX_DNS` | DNS suffix for the sandbox network |

**`_SANDBOX_ID` is NOT in the agent variable store.** It exists only as an environment variable on each host. To share it across hosts, bridge it explicitly:

```bash
# In setup on host A:
agent variable set SANDBOX_ID "$_SANDBOX_ID"

# In scripts on host B:
SANDBOX_ID=$(agent variable get SANDBOX_ID)
```

**Sandbox-public URL pattern:** Services exposed on sandbox hosts are reachable externally at:

```
https://<hostname>-<port>-${_SANDBOX_ID}.env.play.instruqt.com
```

This pattern is useful for constructing URLs in assignment text via variable interpolation, or for cross-host API calls within scripts.

### User Variables

| Variable | Description |
|----------|-------------|
| `INSTRUQT_USER_EMAIL` | Email address of the current learner |
| `INSTRUQT_USER_ID` | Unique identifier of the current learner |

> [!WARNING]
> `INSTRUQT_USER_EMAIL` and `INSTRUQT_USER_ID` are **not guaranteed** in all play contexts. They are empty during:
> - Hot-started sandbox provisioning (track-level setup scripts)
> - Anonymous embed plays
> - Certain API-triggered invocations
>
> Always provide a fallback:
>
> ```bash
> EMAIL="${INSTRUQT_USER_EMAIL:-anonymous@instruqt.com}"
> ```

## Common Patterns

### docker compose vs docker-compose

Docker Compose v2 (the `docker compose` CLI plugin) is now dominant. Prefer the v2 syntax over the legacy `docker-compose` standalone binary:

```bash
# Preferred (v2 plugin)
docker compose up -d
docker compose exec app bash
docker compose down

# Legacy (v1 standalone) — avoid unless the image only has v1
docker-compose up -d
```

### systemctl enable --now

Combine enable and start in a single command instead of two separate calls:

```bash
# One command
systemctl enable --now nginx

# Equivalent to:
systemctl enable nginx
systemctl start nginx
```

### Indirect Variable Expansion

Use `${!VAR}` for dynamic environment variable names. Useful when variable names are constructed at runtime or differ across provider renames:

```bash
# Construct the variable name dynamically
VAR_NAME="INSTRUQT_AWS_ACCOUNT_${ACCOUNT_NUM}_ID"
ACCOUNT_ID="${!VAR_NAME}"

# Portable across renames — if Instruqt renames an env var,
# only the name string needs updating
```

### cmd: vs shell: in Tab Configuration

`shell:` in `config.yml` sets the default shell for all terminal tabs on a host. `cmd:` on individual tabs in `assignment.md` runs a specific command instead of a shell.

| Setting | Scope | Example | Notes |
|---------|-------|---------|-------|
| `shell:` | Host-wide (config.yml) | `shell: /bin/bash` | Default for all terminal tabs on that host |
| `shell: su - user` | Host-wide | `shell: su - user` | Supported on VMs, not containers |
| `cmd:` | Per-tab (assignment.md) | `cmd: ssh remote-host` | Overrides shell for one tab |
| `cmd:` | Per-tab | `cmd: screen -xRR session` | Attach to a shared screen session |

Use `shell:` for the common case (what shell learners land in). Use `cmd:` for special-purpose tabs that run a specific command (SSH tunnel, REPL, screen session).

### Cross-Host State Sharing

For simple string values, use agent variables:

```bash
# Host A setup:
agent variable set DB_PASSWORD "$(openssl rand -hex 16)"

# Host B setup:
DB_PASSWORD=$(agent variable get DB_PASSWORD)
```

For structured data that multiple VMs consume, use the JSON-broker sidecar pattern or cross-VM environment passthrough. A leader host runs a lightweight HTTP server (Python, socat, or netcat) that serves a JSON payload. Other hosts curl it during setup:

```bash
# Leader host — serve structured state on port 8888
cat > /tmp/state.json <<EOF
{"cluster_ip": "$CLUSTER_IP", "join_token": "$JOIN_TOKEN", "ca_hash": "$CA_HASH"}
EOF
python3 -m http.server 8888 --directory /tmp &

# Worker host — consume structured state
STATE=$(curl -s http://leader:8888/state.json)
CLUSTER_IP=$(echo "$STATE" | jq -r '.cluster_ip')
JOIN_TOKEN=$(echo "$STATE" | jq -r '.join_token')
```

See `decision-frameworks/agent-variable-vs-json-broker.md` for the full decision tree.

## Windows PowerShell Scripts

Windows VMs use PowerShell for lifecycle scripts. No shebang is needed — the platform detects the host OS automatically.

### Conventions

- Use `$ErrorActionPreference = 'Stop'` as the PowerShell equivalent of `set -e`
- Scripts run as Administrator by default
- Common operations: `Register-ScheduledTask`, `Rename-Computer`, `Install-WindowsFeature`, `Add-WindowsFeature`
- Use `Restart-Computer -Force` with care — pair with a scheduled task for post-reboot continuation

```powershell
$ErrorActionPreference = 'Stop'

# Rename the computer
Rename-Computer -NewName "lab-dc01" -Force

# Install a Windows feature
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Schedule a task for post-reboot configuration
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-File C:\setup\post-reboot.ps1"
$trigger = New-ScheduledTaskTrigger -AtStartup
Register-ScheduledTask -TaskName "PostReboot" -Action $action -Trigger $trigger -RunLevel Highest
```

### PowerShell Check Scripts

PowerShell check scripts follow the same pattern as bash — exit 0 on success, exit 1 on failure:

```powershell
# Check if the AD feature is installed
if (-not (Get-WindowsFeature AD-Domain-Services).Installed) {
    fail-message "Active Directory Domain Services is not installed. Use Install-WindowsFeature to add it."
    exit 1
}

exit 0
```

### Native Terminal Tabs for Windows

Windows VMs support native PowerShell terminal tabs without needing Guacamole RDP. This is sufficient for CLI-focused Windows workflows. Use Guacamole RDP only when learners need a graphical desktop.

## Examples

### Track-Level Setup with Cloud Bootstrap

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

export DEBIAN_FRONTEND=noninteractive

# Clone lab repository
git clone https://github.com/example/lab-repo.git /root/lab

# Store a variable for later scripts
agent variable set LAB_VERSION "2.1.0"
```

### Check with Multiple Assertions

```bash
#!/bin/bash

# Check 1: Pod exists
POD_COUNT=$(kubectl get pods -n app -l app=web --no-headers 2>/dev/null | wc -l)
if [ "$POD_COUNT" -lt 1 ]; then
  fail-message "No 'web' pods found in the 'app' namespace. Create the deployment as described in the instructions."
  exit 1
fi

# Check 2: Pod is running
POD_STATUS=$(kubectl get pods -n app -l app=web -o jsonpath='{.items[0].status.phase}' 2>/dev/null)
if [ "$POD_STATUS" != "Running" ]; then
  fail-message "The web pod exists but is not running (status: $POD_STATUS). Check the pod logs for errors."
  exit 1
fi

# Check 3: Service exposed
SVC=$(kubectl get svc -n app web-svc --no-headers 2>/dev/null)
if [ -z "$SVC" ]; then
  fail-message "Service 'web-svc' not found in the 'app' namespace. Expose the deployment with a ClusterIP service."
  exit 1
fi

exit 0
```

### Solve with Full Prerequisite Chain

```bash
#!/bin/bash
set -euxo pipefail

# Defensive: ensure namespace exists
kubectl create namespace app 2>/dev/null || true

# Defensive: remove any partial deployment
kubectl delete deployment web -n app 2>/dev/null || true

# Create the expected state
kubectl create deployment web --image=nginx:latest -n app
kubectl expose deployment web --name=web-svc --port=80 -n app 2>/dev/null || true

# Wait for readiness
kubectl rollout status deployment/web -n app --timeout=60s
```
