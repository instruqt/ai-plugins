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

- Linux: `#!/bin/bash` or `#!/usr/bin/env bash`
- Windows VMs: no shebang (PowerShell assumed)

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
```

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
| Working directory | `/root` by default on Linux VMs |
| Environment | All `config.yml` `environment:` vars, all secrets, Instruqt built-in vars |
| Timeout (check) | 1 minute |
| Exit codes (check) | 0 = pass, 1 = fail |

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
