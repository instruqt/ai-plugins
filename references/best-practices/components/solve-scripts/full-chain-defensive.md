# Full-Chain Defensive

Solve scripts must work independently at any challenge. Each solve script checks and re-creates the entire prerequisite chain from scratch, making `skipping_enabled: true` safe.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Solve assumes all previous challenges were completed — fails if a learner skips to this challenge. |
| 2 | Below Standard | Solve handles the immediate predecessor's state but not the full chain — fails if multiple challenges are skipped. |
| 3 | Adequate | Solve re-creates most prerequisites but misses one or two (e.g., assumes a package is installed or a service is running). |
| 4 | Good | Solve checks and re-creates the entire prerequisite chain — service running, database exists, table exists, data present. Works from a clean skip. Production baseline. |
| 5 | Excellent | Solve is structured as a declarative state-convergence script — each block ensures one layer of the stack is correct before moving to the next, with clear separation between layers. |

## Guidance

### Why full-chain matters

When `skipping_enabled: true` is set on a track, learners can jump to any challenge. Instruqt runs the solve script for each skipped challenge in order. If challenge 5's solve assumes that challenges 1-4's setup and solve scripts created the prerequisite state, it will fail when run on a fresh environment.

Each solve script must be self-sufficient: it should verify and establish every prerequisite it needs, regardless of what ran before it.

### The prerequisite chain

For a typical challenge that checks "the users table has 3 rows in the lab database on a running PostgreSQL server", the full chain is:

1. PostgreSQL is installed
2. PostgreSQL is running
3. The `lab` database exists
4. The `users` table exists with the correct schema
5. The table contains exactly 3 rows with the correct data

The solve script for this challenge must ensure ALL of these, not just step 5.

### Example — full chain

```bash
#!/bin/bash
set -euxo pipefail

# Layer 1: PostgreSQL is installed
if ! command -v psql &>/dev/null; then
  apt-get update && apt-get install -y postgresql
fi

# Layer 2: PostgreSQL is running
if ! systemctl is-active --quiet postgresql; then
  systemctl start postgresql
fi

# Layer 3: Database exists
psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname='lab'" | grep -q 1 || \
  psql -U postgres -c "CREATE DATABASE lab;"

# Layer 4: Table exists with correct schema
psql -U postgres -d lab -c "CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE
);"

# Layer 5: Correct data
psql -U postgres -d lab -c "TRUNCATE users;"
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES
  ('alice', 'alice@example.com'),
  ('bob', 'bob@example.com'),
  ('carol', 'carol@example.com');"
```

### Bad — assumes prerequisites exist

```bash
#!/bin/bash
set -euxo pipefail

# Assumes PostgreSQL is installed and running, database and table exist
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES
  ('alice', 'alice@example.com'),
  ('bob', 'bob@example.com'),
  ('carol', 'carol@example.com');"
```

This fails with "connection refused" if PostgreSQL is not running, "database does not exist" if the database was not created, or "relation does not exist" if the table is missing.

### Kubernetes example — full chain

```bash
#!/bin/bash
set -euxo pipefail

# Layer 1: Namespace exists
kubectl create namespace app 2>/dev/null || true

# Layer 2: ConfigMap exists
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: app
data:
  DB_HOST: "postgres.app.svc.cluster.local"
  DB_NAME: "lab"
EOF

# Layer 3: Deployment exists with correct spec
kubectl apply -f /opt/manifests/deployment.yaml

# Layer 4: Service exists
kubectl apply -f /opt/manifests/service.yaml

# Layer 5: Wait for rollout (this challenge's specific requirement)
kubectl rollout status deployment/web -n app --timeout=60s
```

### Structuring layered solve scripts

For complex challenges, organize the solve script into clearly labeled layers:

```bash
#!/bin/bash
set -euxo pipefail

#--- Layer 1: Infrastructure ---
# ...

#--- Layer 2: Application ---
# ...

#--- Layer 3: Configuration ---
# ...

#--- Layer 4: Data ---
# ...

#--- Layer 5: This challenge's specific state ---
# ...
```

This makes it obvious what each section ensures and makes the script easier to maintain when prerequisites change.

## What to Watch For

- Solve scripts that start with the challenge-specific action without checking prerequisites.
- Missing service-start commands (assuming setup already started the service).
- Missing package installation (assuming setup or a previous solve installed it).
- Database operations without first ensuring the database and tables exist.
- Kubernetes operations without ensuring the namespace exists.
- File operations without ensuring parent directories exist (`mkdir -p`).
- Tracks with `skipping_enabled: true` where solve scripts have not been tested in isolation.
