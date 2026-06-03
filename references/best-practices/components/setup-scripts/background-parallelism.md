# Background Parallelism

Evaluates whether setup scripts run independent slow operations in parallel to minimize total setup time while maintaining reliability.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | All operations run sequentially; setup takes 3-5x longer than necessary; no use of backgrounding |
| 2 | Below Standard | Some operations backgrounded but without capturing PIDs or checking exit codes; failures go unnoticed |
| 3 | Adequate | Background operations used with wait, but error handling is missing or incorrect under set -e |
| 4 | Good | Independent slow operations run in parallel with &, PIDs captured, and each joined with wait checking for failure (production baseline) |
| 5 | Excellent | Setup time is minimized through parallel execution, Docker image pre-pulls, and dependency-aware ordering; no reliability sacrificed |

## Guidance

Sandbox setup has a time limit. Running independent operations in parallel can dramatically reduce total setup time. The key is to background only truly independent operations and handle failures explicitly.

### Basic parallel pattern

Good -- backgrounding independent installs with failure handling:

```bash
#!/bin/bash
set -e

# Start independent operations in parallel
apt-get install -y -qq openjdk-17-jdk &
PID_APT=$!

curl -fsSL "https://go.dev/dl/go1.22.0.linux-amd64.tar.gz" -o /tmp/go.tar.gz &
PID_CURL=$!

helm repo add bitnami https://charts.bitnami.com/bitnami &
PID_HELM=$!

# Wait for each and check results
wait $PID_APT || { echo "ERROR: apt install failed"; exit 1; }
wait $PID_CURL || { echo "ERROR: Go download failed"; exit 1; }
wait $PID_HELM || { echo "ERROR: Helm repo add failed"; exit 1; }

# Sequential step that depends on the download above
tar -C /usr/local -xzf /tmp/go.tar.gz
```

### Docker image pre-pull pattern

Good -- pulling images in parallel during track setup:

```bash
# Pre-pull images needed across challenges
docker pull nginx:1.25 &
docker pull redis:7-alpine &
docker pull postgres:16-alpine &
wait  # Wait for all background jobs
```

### Dependency-aware ordering

Good -- parallelizing independent branches, sequencing dependencies:

```bash
#!/bin/bash
set -e

# Branch 1: Kubernetes tooling (sequential within branch)
(
  curl -fsSL -o /usr/local/bin/kubectl "https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl"
  chmod +x /usr/local/bin/kubectl
  kubectl completion bash > /etc/bash_completion.d/kubectl
) &
PID_K8S=$!

# Branch 2: App dependencies (sequential within branch)
(
  apt-get install -y -qq nodejs npm
  npm install -g yarn@1.22.21
) &
PID_NODE=$!

# Branch 3: Clone and prepare repo
(
  git clone https://github.com/example/app.git /root/app
  cd /root/app && npm install
) &
PID_REPO=$!

# Join all branches
wait $PID_K8S || { echo "ERROR: K8s tooling setup failed"; exit 1; }
wait $PID_NODE || { echo "ERROR: Node setup failed"; exit 1; }
wait $PID_REPO || { echo "ERROR: Repo setup failed"; exit 1; }
```

Bad -- no parallelism (unnecessarily slow):

```bash
apt-get install -y openjdk-17-jdk    # 30 seconds
curl -fsSL -o /tmp/go.tar.gz ...     # 10 seconds
helm repo add bitnami ...             # 5 seconds
# Total: 45 seconds sequential (could be ~30 seconds parallel)
```

Bad -- backgrounding without capturing PIDs:

```bash
apt-get install -y openjdk-17-jdk &
curl -fsSL -o /tmp/go.tar.gz ... &
wait  # Waits for all, but if one fails you don't know which
```

Bad -- backgrounding dependent operations:

```bash
curl -fsSL -o /tmp/go.tar.gz ... &
tar -C /usr/local -xzf /tmp/go.tar.gz &  # Races with the download!
wait
```

### Handling set -e with wait

Under `set -e`, a failed `wait $PID` will exit the script immediately. If you need to collect results from multiple background jobs before deciding what to do, temporarily disable errexit:

```bash
set -e

some_slow_command &
PID1=$!
another_slow_command &
PID2=$!

# Collect results without exiting on first failure
set +e
wait $PID1; RC1=$?
wait $PID2; RC2=$?
set -e

if [ $RC1 -ne 0 ] || [ $RC2 -ne 0 ]; then
  echo "ERROR: Background job failed (rc1=$RC1, rc2=$RC2)"
  exit 1
fi
```

## What to Watch For

- Only background truly independent operations; dependencies must remain sequential
- Always capture PIDs and check exit codes -- silent failures cause confusing downstream errors
- `wait` with no arguments waits for all background jobs but gives no per-job failure info
- Backgrounded apt-get operations can conflict if they share the dpkg lock; only one apt-get process at a time
- Docker pulls can be safely parallelized; they use independent layer downloads
- Subshells `( ... ) &` are useful for grouping sequential steps into a single background branch
- Under `set -e`, a failing `wait` exits immediately; use the set +e pattern if you need to collect all results
