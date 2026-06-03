# Track-Level vs Per-Challenge Setup Placement

Evaluates whether setup logic is correctly placed in track-level setup (runs once at sandbox start) versus per-challenge setup (runs before each challenge).

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | All setup logic crammed into track-level setup or all into per-challenge setup; no thought given to placement |
| 2 | Below Standard | Some logic misplaced -- e.g., per-challenge state resets in track setup, or one-time infrastructure provisioning repeated every challenge |
| 3 | Adequate | Most logic correctly placed, but a few items are in the wrong tier causing unnecessary delays or missing state |
| 4 | Good | Correct placement: one-time provisioning in track setup, progressive state changes in per-challenge setup (production baseline) |
| 5 | Excellent | Per-challenge setup is defensive and re-creates prior state if missing, supporting learners who skip challenges or experience partial failures |

## Guidance

### Track-level setup (setup-<host>)

Runs once when the sandbox is created. Use for:

- **Bootstrap sentinel waiting** -- polling for host-bootstrap-completed
- **Infrastructure provisioning** -- cluster creation, cloud resource setup, image registration
- **Repository cloning** -- cloning starter repos into the working directory
- **Tool installation** -- installing binaries, CLI tools, language runtimes
- **Helper script seeding** -- placing reusable scripts the learner will use across challenges
- **Shared certificate/key generation** -- TLS certs, SSH keys, API tokens
- **Docker image pre-pulling** -- pulling images needed across multiple challenges
- **Environment variable configuration** -- .bashrc and /etc/profile.d/ setup

### Per-challenge setup (setup-<host> inside a challenge directory)

Runs before each challenge starts. Use for:

- **State reset** -- cleaning up previous challenge artifacts to start fresh
- **Progressive data seeding** -- adding data or config that this challenge needs
- **Configuration changes** -- modifying files, enabling features for this challenge's scenario
- **Triggering outages/failures** -- injecting faults the learner must diagnose and fix
- **Service restarts** -- restarting services in a known state for this challenge

Good -- track-level setup for one-time work:

```bash
#!/bin/bash
# Track-level setup: runs once
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

export DEBIAN_FRONTEND=noninteractive
apt-get update -qq
apt-get install -y -qq kubectl helm

git clone https://github.com/example/starter-repo.git /root/project
set-workdir /root/project
```

Good -- per-challenge setup for progressive state:

```bash
#!/bin/bash
# Per-challenge setup: seed the broken config for this challenge
cat > /root/project/config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  log_level: "FATAL"  # Learner must fix this
EOF

kubectl apply -f /root/project/config.yaml
```

Good -- defensive per-challenge setup (supports skipping):

```bash
#!/bin/bash
# Ensure the namespace exists even if the learner skipped the previous challenge
kubectl get namespace workshop > /dev/null 2>&1 || kubectl create namespace workshop

# Apply the base resources if they're missing
kubectl get deployment -n workshop web > /dev/null 2>&1 || \
  kubectl apply -f /root/project/base-deployment.yaml -n workshop

# Now apply this challenge's specific changes
kubectl apply -f /root/project/challenge-03-broken-service.yaml -n workshop
```

Bad -- infrastructure provisioning in per-challenge setup (runs every challenge):

```bash
#!/bin/bash
# Per-challenge setup: DON'T do this here
apt-get install -y kubectl  # Already installed, wastes time
git clone https://github.com/example/repo.git /root/project  # Already cloned
```

Bad -- per-challenge state setup in track-level (not available for later challenges):

```bash
#!/bin/bash
# Track-level setup: DON'T put challenge-specific state here
cat > /root/broken-config.yaml << 'EOF'
...
EOF
# This config is for challenge 3, but the learner modifies it in challenge 2
```

## What to Watch For

- Track-level setup that takes too long because it includes per-challenge work
- Per-challenge setup that repeats expensive operations already done in track setup
- Progressive tracks where challenge N's setup assumes challenge N-1 completed successfully -- this breaks if the learner skips or the previous check was lenient
- Defensive per-challenge setup adds resilience but should not re-run truly expensive operations (e.g., cluster creation)
- Agent variable get/set calls to pass state between track-level and per-challenge setup
