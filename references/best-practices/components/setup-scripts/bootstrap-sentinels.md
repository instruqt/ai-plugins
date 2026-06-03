# Bootstrap Sentinel Waiting

Evaluates whether setup scripts correctly wait for host bootstrap to complete before performing any configuration, ensuring the environment is fully ready before setup logic runs.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No sentinel wait at all; setup runs immediately and fails intermittently due to race conditions with bootstrap |
| 2 | Below Standard | Sentinel wait exists but uses the wrong file path, or only some setup scripts that need it include the wait |
| 3 | Adequate | Correct sentinel wait present but placed after commands that depend on bootstrap, or uses a fixed sleep instead of polling |
| 4 | Good | All track-level setup scripts wait for the correct sentinel file before any setup logic; cloud-client hosts use gcp-bootstrap-completed (production baseline) |
| 5 | Excellent | Sentinel wait is combined with a secondary readiness probe (agent port 15779 responding) to guard against edge cases where the file exists but the agent is not yet ready |

## Guidance

Track-level setup scripts (setup-<host>) run as soon as the sandbox is allocated, but the host may still be bootstrapping. The bootstrap sentinel file signals that the host is fully ready.

The sentinel path depends on the host image:

- **Standard hosts** (container, VM): `/opt/instruqt/bootstrap/host-bootstrap-completed`
- **cloud-client hosts** (GCP shell): `/opt/instruqt/bootstrap/gcp-bootstrap-completed`

Good -- standard host sentinel wait at the top of setup:

```bash
#!/bin/bash
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# All setup logic goes here
apt-get update
```

Good -- cloud-client sentinel wait:

```bash
#!/bin/bash
until [ -f /opt/instruqt/bootstrap/gcp-bootstrap-completed ]; do
  sleep 1
done

# GCP setup logic goes here
gcloud config set project "$INSTRUQT_GCP_PROJECT"
```

Good -- dual readiness probe (sentinel + agent port):

```bash
#!/bin/bash
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Wait for agent to be fully responsive
until curl -sf http://localhost:15779/health > /dev/null 2>&1; do
  sleep 1
done

# Setup logic goes here
```

Bad -- no sentinel wait:

```bash
#!/bin/bash
apt-get update  # May fail if bootstrap hasn't finished
```

Bad -- fixed sleep instead of polling:

```bash
#!/bin/bash
sleep 30  # Fragile; bootstrap may take longer or shorter
apt-get update
```

Bad -- sentinel wait placed after commands that depend on bootstrap:

```bash
#!/bin/bash
kubectl get nodes  # Fails because bootstrap hasn't finished
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done
```

## What to Watch For

- The sentinel wait must be the very first logic in every track-level setup script
- Per-challenge setup scripts do not need the sentinel wait (bootstrap is complete by then)
- Using the wrong sentinel file for the host type (e.g., host-bootstrap-completed on a cloud-client)
- Using `sleep N` as a substitute for polling the sentinel file
- Using `test -f` instead of a loop -- the file may not exist yet when the script starts
