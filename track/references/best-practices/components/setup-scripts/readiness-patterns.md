# Readiness Patterns

Evaluates how setup scripts wait for services, APIs, and dependencies to become available before proceeding, distinguishing between naive sleep-based waits and robust polling with backoff and jitter.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Uses fixed `sleep` commands to wait for services; no polling or retry logic; setup fails intermittently when timing assumptions are wrong |
| 2 | Below Standard | Basic polling loop exists (e.g., `until curl ...`) but with no timeout, no backoff, and no failure message — hangs indefinitely on failure |
| 3 | Adequate | Polling with a fixed retry count and sleep interval; exits on timeout; but no exponential backoff or jitter, causing thundering-herd on multi-host setups |
| 4 | Good | Retry helper with exponential backoff and a maximum attempt count; HTTP status-code checks use specific codes rather than just "non-empty response"; local vs remote services use appropriate polling strategies (production baseline) |
| 5 | Excellent | Dual readiness probes (sentinel file + port/HTTP check); jitter added to prevent synchronized retries across hosts; different backoff strategies for local vs SaaS dependencies; helper function is reusable across scripts |

## Guidance

Setup scripts frequently need to wait for services that start asynchronously — databases accepting connections, APIs returning healthy status, cloud resources completing provisioning. The approach should match the target.

### Retry helper with exponential backoff

A reusable function that retries a command with exponential backoff and a maximum attempt count:

```bash
retry_command() {
  local max_attempts=${1}
  local delay=${2}
  local cmd="${@:3}"
  local attempt=1

  until eval "$cmd"; do
    if [ "$attempt" -ge "$max_attempts" ]; then
      echo "FATAL: command failed after $max_attempts attempts: $cmd"
      return 1
    fi
    echo "Attempt $attempt/$max_attempts failed, retrying in ${delay}s..."
    sleep "$delay"
    delay=$((delay * 2))
    attempt=$((attempt + 1))
  done
}

# Usage: wait up to ~5 min for a service with 1s initial backoff
retry_command 10 1 "curl -sf http://localhost:8080/health"
```

### Retry with jitter

In multi-host setups where several VMs poll the same service, add jitter to prevent synchronized retries from overwhelming the target:

```bash
retry_with_jitter() {
  local max_attempts=${1}
  local base_delay=${2}
  local cmd="${@:3}"
  local attempt=1

  until eval "$cmd"; do
    if [ "$attempt" -ge "$max_attempts" ]; then
      return 1
    fi
    # Add random jitter: base_delay * 2^attempt + random(0..base_delay)
    local exp_delay=$((base_delay * (2 ** (attempt - 1))))
    local jitter=$(shuf -i 0-"$base_delay" -n 1)
    sleep $((exp_delay + jitter))
    attempt=$((attempt + 1))
  done
}
```

### HTTP status-code polling

When waiting for a web service, check for specific HTTP status codes rather than just "any response". Some services return 500 during startup but 200 when ready. Others return 302 (redirect to login) or 401 (auth required) to signal they are up:

```bash
# Wait for HTTP 200 specifically
retry_command 15 2 "curl -so /dev/null -w '%{http_code}' http://localhost:3000 | grep -q '200'"

# Accept 200, 302, or 401 as "service is up" (redirect or auth-wall means the app is running)
retry_command 15 2 "curl -so /dev/null -w '%{http_code}' http://localhost:8443 | grep -qE '(200|302|401)'"

# Wait for a specific JSON field in the response
retry_command 10 3 "curl -sf http://localhost:8080/api/status | jq -e '.ready == true'"
```

### Dual readiness probe

For critical dependencies, combine a sentinel file check with a live port/HTTP probe. The sentinel confirms the process started; the probe confirms it is accepting connections:

```bash
# Wait for bootstrap sentinel AND agent port
until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Sentinel exists, but agent may not be ready yet
retry_command 10 1 "curl -sf http://localhost:15779/health"
```

### Local vs remote polling strategies

Different targets warrant different polling strategies:

| Target | Strategy | Rationale |
|--------|----------|-----------|
| Localhost service (nginx, postgres) | Flat retry, 1-2s interval, 30 attempts | Local startup is fast; exponential backoff wastes time |
| Same-sandbox remote host | Exponential backoff, 2s base, 15 attempts | Network + service startup; moderate variability |
| Cloud API / SaaS endpoint | Exponential backoff + jitter, 5s base, 12 attempts | High variability; thundering-herd risk; rate limits |
| Cloud resource provisioning | Exponential backoff, 10s base, 30 attempts | Minutes-scale operations; provider-side queuing |

```bash
# Local service — flat retry, fast cadence
for i in $(seq 1 30); do
  curl -sf http://localhost:5432/ && break
  sleep 2
done

# SaaS dependency — exponential backoff with jitter
retry_with_jitter 12 5 "curl -sf https://api.example.com/health"
```

## What to Watch For

- **Fixed `sleep 30` before a service check** — brittle; fails when the service takes 31 seconds or wastes 29 seconds when it starts in 1 second
- **`until` loop with no maximum attempt count** — hangs the setup script indefinitely, eventually hitting the 55-minute timeout with no diagnostic output
- **Polling a port that is open but not ready** — TCP connect succeeds before the HTTP server is initialized; use HTTP-level checks, not just `nc -z`
- **Same backoff strategy for local and remote** — exponential backoff for localhost adds unnecessary delay; flat retry for cloud APIs risks rate limiting
- **No jitter in multi-host setups** — three VMs all polling a leader at exactly 2s, 4s, 8s intervals create synchronized load spikes
- **Polling with `set -e` and no error suppression** — the first failed curl exits the script; use `|| true` inside retry logic or wrap in a function
