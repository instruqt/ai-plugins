# Idempotent Operations

Solve scripts must be idempotent and safe to re-run. Running a solve script twice must produce the same end state as running it once, with no errors on the second run.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Re-running the solve script fails — duplicate key errors, "already exists" errors, or conflicting state. |
| 2 | Below Standard | Some operations are idempotent but others fail on re-run (e.g., `CREATE TABLE` without `IF NOT EXISTS`). |
| 3 | Adequate | Most operations are idempotent, but edge cases (e.g., file append without dedup) cause drift on re-run. |
| 4 | Good | All operations use create-if-not-exists patterns. No failures on re-run. Production baseline. |
| 5 | Excellent | Solve detects current state and applies only the missing delta — skips steps already completed, minimizing execution time and side effects. |

## Guidance

### Why idempotency matters for solve scripts

Solve scripts run in several contexts:

- **Instruqt test mode**: the platform runs solve then check in sequence for each challenge. If a test run is retried, solve runs again on the same environment.
- **skipping_enabled: true**: when a learner skips a challenge, the solve script runs. If they skip back and forward, it may run multiple times.
- **Manual debugging**: track authors run solve scripts manually during development.

In all cases, a non-idempotent solve that fails on re-run causes test failures and broken skip behavior.

### Idempotent patterns by resource type

#### Files

```bash
# Good — overwrite, not append
cat > /etc/app/config.yaml <<'EOF'
max_connections: 100
log_level: info
EOF

# Bad — append creates duplicates on re-run
echo "max_connections: 100" >> /etc/app/config.yaml
```

If you must append a line to an existing file, guard it:

```bash
grep -q '^max_connections: 100' /etc/app/config.yaml 2>/dev/null || \
  echo "max_connections: 100" >> /etc/app/config.yaml
```

#### Databases

```bash
# Good — IF NOT EXISTS
psql -U postgres -c "CREATE DATABASE lab;" 2>/dev/null || true
psql -U postgres -d lab -c "CREATE TABLE IF NOT EXISTS users (id SERIAL PRIMARY KEY, name TEXT, email TEXT);"

# Good — upsert
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES ('admin', 'admin@example.com') ON CONFLICT (name) DO UPDATE SET email = EXCLUDED.email;"

# Bad — fails on re-run
psql -U postgres -c "CREATE DATABASE lab;"
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES ('admin', 'admin@example.com');"
```

#### Kubernetes

```bash
# Good — apply is inherently idempotent
kubectl apply -f /opt/manifests/deployment.yaml

# Bad — create fails if resource exists
kubectl create -f /opt/manifests/deployment.yaml
```

#### Docker

```bash
# Good — remove if exists, then create
docker rm -f app 2>/dev/null || true
docker run -d --name app nginx:1.25

# Bad — fails if container already exists
docker run -d --name app nginx:1.25
```

#### System packages

```bash
# Good — apt-get install is idempotent (already-installed packages are skipped)
apt-get install -y nginx

# Good — systemctl enable + start is idempotent
systemctl enable --now nginx
```

#### Directories

```bash
# Good — mkdir -p is idempotent
mkdir -p /opt/app/data

# Bad — fails if directory exists
mkdir /opt/app/data
```

### State-aware delta pattern (score 5)

```bash
#!/bin/bash
set -euxo pipefail

# Only install if not already present
if ! command -v nginx &>/dev/null; then
  apt-get update && apt-get install -y nginx
fi

# Only write config if it differs
EXPECTED_HASH="a1b2c3d4..."
ACTUAL_HASH=$(sha256sum /etc/nginx/sites-enabled/default 2>/dev/null | awk '{print $1}')
if [[ "$ACTUAL_HASH" != "$EXPECTED_HASH" ]]; then
  cp /opt/solve/nginx-default.conf /etc/nginx/sites-enabled/default
  systemctl reload nginx
fi

# Only insert rows if count is wrong
ROW_COUNT=$(psql -U postgres -d lab -tAc "SELECT count(*) FROM users;" 2>/dev/null)
if [[ "$ROW_COUNT" != "3" ]]; then
  psql -U postgres -d lab -c "TRUNCATE users; INSERT INTO users (name, email) VALUES ('alice','a@ex.com'),('bob','b@ex.com'),('carol','c@ex.com');"
fi
```

## What to Watch For

- `CREATE TABLE` without `IF NOT EXISTS`.
- `INSERT` without `ON CONFLICT` / `INSERT IGNORE` / prior `TRUNCATE`.
- `docker run` without prior `docker rm -f`.
- `kubectl create` instead of `kubectl apply`.
- File appends (`>>`) without dedup guards.
- `mkdir` without `-p`.
- `useradd` without checking `id username` first.
- `git clone` into a directory that may already exist.
