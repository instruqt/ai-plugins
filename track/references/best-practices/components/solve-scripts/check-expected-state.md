# Check-Expected State

Solve must produce the EXACT state that check validates. Not "close enough" — if check greps for "3 rows", solve must produce exactly 3 rows. Solve and check are duals of each other.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Solve produces state that check does not validate, or check validates state that solve does not produce — solve+check fails. |
| 2 | Below Standard | Solve produces approximately correct state, but minor mismatches cause intermittent check failures (e.g., trailing whitespace, different casing). |
| 3 | Adequate | Solve produces state that check validates in the happy path, but edge cases (ordering, formatting) can cause failures. |
| 4 | Good | Every check assertion has a corresponding solve action that produces exactly the expected state. Solve followed by check always passes. Production baseline. |
| 5 | Excellent | Solve and check are verifiably dual — for every assertion in check, there is a traceable action in solve that produces the exact expected value, and this correspondence is documented or structurally obvious. |

## Guidance

### The duality principle

A check script is a set of assertions. A solve script is a set of actions. For the track to work correctly, these must be exact duals:

```
For every assertion A in check:
  There exists an action S in solve such that:
    After S runs, A passes.
```

If this property does not hold, `instruqt track test` will fail.

### Common mismatches

#### Row count mismatch

```bash
# Check expects exactly 3
ROW_COUNT=$(psql -U postgres -d lab -tAc "SELECT count(*) FROM users;")
if [[ "$ROW_COUNT" != "3" ]]; then
  fail-message "Expected 3 rows but found $ROW_COUNT."
  exit 1
fi
```

```bash
# Bad solve — inserts 3 but does not clear existing rows.
# On re-run or if setup pre-seeded data, count could be 4, 6, etc.
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES
  ('alice', 'alice@example.com'),
  ('bob', 'bob@example.com'),
  ('carol', 'carol@example.com');"
```

```bash
# Good solve — ensures exactly 3 rows
psql -U postgres -d lab -c "TRUNCATE users;"
psql -U postgres -d lab -c "INSERT INTO users (name, email) VALUES
  ('alice', 'alice@example.com'),
  ('bob', 'bob@example.com'),
  ('carol', 'carol@example.com');"
```

#### String matching mismatch

```bash
# Check greps for exact string
if ! grep -q 'server_name lab.example.com;' /etc/nginx/sites-enabled/default; then
```

```bash
# Bad solve — extra whitespace
echo "  server_name  lab.example.com ;" >> /etc/nginx/sites-enabled/default
```

```bash
# Good solve — produces the exact string check expects
cat > /etc/nginx/sites-enabled/default <<'EOF'
server {
    listen 80;
    server_name lab.example.com;
    root /var/www/html;
}
EOF
```

#### File permissions mismatch

```bash
# Check expects 600
PERMS=$(stat -c '%a' /etc/app/secret.key)
if [[ "$PERMS" != "600" ]]; then
```

```bash
# Bad solve — creates file with default umask (likely 644)
echo "secret" > /etc/app/secret.key

# Good solve — sets exact permissions
echo "secret" > /etc/app/secret.key
chmod 600 /etc/app/secret.key
```

#### Process name mismatch

```bash
# Check uses pgrep -x (exact match)
if ! pgrep -x nginx &>/dev/null; then
```

```bash
# Bad solve — starts nginx via a wrapper script whose process name is "start-nginx.sh"
/opt/scripts/start-nginx.sh

# Good solve — starts nginx directly so the process name matches
systemctl start nginx
```

### Verification approach

The most reliable way to verify solve/check duality is to run them in sequence:

```bash
# During development, on the sandbox:
bash solve-sandbox && bash check-sandbox
```

If check passes after solve, the duality holds for this run. Test multiple times to catch non-deterministic issues (ordering, timing).

### Structural correspondence (score 5)

For a score-5 solve script, the correspondence between solve actions and check assertions is structurally obvious. One approach is to mirror the check structure:

```bash
# solve-sandbox
set -euxo pipefail

# Corresponds to check assertion 1: nginx installed
apt-get install -y nginx

# Corresponds to check assertion 2: nginx running
systemctl start nginx

# Corresponds to check assertion 3: server_name configured
cat > /etc/nginx/sites-enabled/default <<'EOF'
server {
    listen 80;
    server_name lab.example.com;
    root /var/www/html;
}
EOF
systemctl reload nginx

# Corresponds to check assertion 4: HTTP 200
# (ensured by assertions 1-3 above)
```

## What to Watch For

- Check uses exact string match but solve produces the value with different whitespace, casing, or quoting.
- Check counts rows but solve appends without truncating — count grows on re-run.
- Check uses `pgrep -x` (exact name match) but solve starts the process via a wrapper.
- Check reads a specific file path but solve writes to a different path.
- Check validates JSON structure but solve produces YAML (or vice versa).
- Check uses `systemctl is-active` but solve uses `service start` (usually fine, but verify).
- Solve creates state asynchronously (e.g., `kubectl apply` with a deployment) but check runs before the state converges — solve needs a `rollout status` or equivalent wait.
