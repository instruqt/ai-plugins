# Common Check Patterns

A reference catalog of reusable patterns for check scripts. This is not a scoring rubric — it is a pattern library for common validation scenarios.

## File Existence and Content

### File exists

```bash
if [[ ! -f /home/user/app/config.yaml ]]; then
  fail-message "The file /home/user/app/config.yaml does not exist. Create it as described in the assignment."
  exit 1
fi
```

### File contains specific content

```bash
if ! grep -q 'max_connections: 100' /etc/app/config.yaml; then
  fail-message "The max_connections setting in /etc/app/config.yaml is not set to 100."
  exit 1
fi
```

### File has correct permissions

```bash
PERMS=$(stat -c '%a' /etc/app/secret.key 2>/dev/null)
if [[ "$PERMS" != "600" ]]; then
  fail-message "Expected /etc/app/secret.key to have permissions 600 but found ${PERMS:-file missing}."
  exit 1
fi
```

## Process Running

### Check by process name

```bash
if ! pgrep -x nginx &>/dev/null; then
  fail-message "nginx is not running. Start it with: systemctl start nginx"
  exit 1
fi
```

### Check by systemd service status

```bash
if ! systemctl is-active --quiet nginx; then
  fail-message "The nginx service is not active. Run: systemctl start nginx"
  exit 1
fi
```

### Check that a process is listening on a port

```bash
if ! ss -tlnp | grep -q ':8080 '; then
  fail-message "No process is listening on port 8080. Make sure the application is running."
  exit 1
fi
```

## HTTP Status

### Check HTTP response code

```bash
STATUS=$(curl -s -o /dev/null -w '%{http_code}' --connect-timeout 3 --max-time 5 http://localhost:8080/ 2>/dev/null)
if [[ "$STATUS" != "200" ]]; then
  fail-message "Expected HTTP 200 from http://localhost:8080/ but got ${STATUS:-no response}. Check that the server is running and configured correctly."
  exit 1
fi
```

### Check HTTP response body

```bash
BODY=$(curl -s --connect-timeout 3 --max-time 5 http://localhost:8080/api/status 2>/dev/null)
if ! echo "$BODY" | grep -q '"healthy":true'; then
  fail-message "The /api/status endpoint did not return a healthy response. Verify the application configuration."
  exit 1
fi
```

### Check HTTPS with self-signed certificate

```bash
STATUS=$(curl -sk -o /dev/null -w '%{http_code}' --connect-timeout 3 --max-time 5 https://localhost:443/ 2>/dev/null)
if [[ "$STATUS" != "200" ]]; then
  fail-message "Expected HTTP 200 from https://localhost:443/ but got ${STATUS:-no response}."
  exit 1
fi
```

## Database Query

### Row count

```bash
ROW_COUNT=$(timeout --kill-after=3 10 psql -U postgres -d lab -tAc "SELECT count(*) FROM users;" 2>/dev/null)
if [[ "$ROW_COUNT" != "3" ]]; then
  fail-message "Expected 3 rows in the users table but found ${ROW_COUNT:-0}. Insert the required user records."
  exit 1
fi
```

### Specific value exists

```bash
RESULT=$(timeout --kill-after=3 10 mysql -u root -sN -e "SELECT email FROM lab.users WHERE name='admin';" 2>/dev/null)
if [[ "$RESULT" != "admin@example.com" ]]; then
  fail-message "The admin user's email should be admin@example.com but found '${RESULT:-not set}'."
  exit 1
fi
```

### Table or database exists

```bash
if ! timeout --kill-after=3 10 psql -U postgres -tAc "SELECT 1 FROM pg_database WHERE datname='lab';" 2>/dev/null | grep -q 1; then
  fail-message "The database 'lab' does not exist. Create it with: createdb -U postgres lab"
  exit 1
fi
```

## Bash History Grep

Validates that the learner ran a specific command (useful for verifying process, not just outcome).

```bash
if ! grep -q 'terraform apply' /root/.bash_history 2>/dev/null; then
  fail-message "It looks like you haven't run 'terraform apply' yet. Run the command to apply the configuration."
  exit 1
fi
```

Note: history-based checks are fragile — the learner might have used a different shell, cleared history, or used a slight variation. Use as a supplement, not a primary check. Prefer outcome-based checks when possible.

## System / Query Counter

### Check a metric or counter value

```bash
COUNTER=$(curl -s --max-time 5 http://localhost:9090/api/v1/query?query=http_requests_total 2>/dev/null | jq -r '.data.result[0].value[1]' 2>/dev/null)
if [[ -z "$COUNTER" ]] || [[ "$COUNTER" -lt 10 ]]; then
  fail-message "Expected at least 10 HTTP requests recorded in Prometheus but found ${COUNTER:-none}. Generate traffic to the application."
  exit 1
fi
```

## Kubernetes State

### Pod is running

```bash
PHASE=$(timeout --kill-after=3 10 kubectl get pod -n app -l app=web -o jsonpath='{.items[0].status.phase}' 2>/dev/null)
if [[ "$PHASE" != "Running" ]]; then
  fail-message "The web pod in the app namespace is not running. Current status: ${PHASE:-not found}."
  exit 1
fi
```

### Deployment has correct replica count

```bash
REPLICAS=$(timeout --kill-after=3 10 kubectl get deployment web -n app -o jsonpath='{.spec.replicas}' 2>/dev/null)
if [[ "$REPLICAS" != "3" ]]; then
  fail-message "Expected the web deployment to have 3 replicas but found ${REPLICAS:-not set}. Scale with: kubectl scale deployment web -n app --replicas=3"
  exit 1
fi
```

### Service exists and has correct type

```bash
SVC_TYPE=$(timeout --kill-after=3 10 kubectl get svc web -n app -o jsonpath='{.spec.type}' 2>/dev/null)
if [[ "$SVC_TYPE" != "LoadBalancer" ]]; then
  fail-message "The web service should be type LoadBalancer but is ${SVC_TYPE:-not found}."
  exit 1
fi
```

### Ingress or resource exists

```bash
if ! timeout --kill-after=3 10 kubectl get ingress web -n app &>/dev/null; then
  fail-message "The ingress 'web' does not exist in the app namespace. Create it as described in the assignment."
  exit 1
fi
```

## Live-Truth Comparison

Compares actual state against an expected reference to detect drift or verify correctness.

```bash
# Compare running config against expected
ACTUAL=$(timeout --kill-after=3 10 kubectl get configmap app-config -n app -o jsonpath='{.data.DB_HOST}' 2>/dev/null)
EXPECTED="postgres.app.svc.cluster.local"
if [[ "$ACTUAL" != "$EXPECTED" ]]; then
  fail-message "The DB_HOST in the app-config ConfigMap should be '$EXPECTED' but is '${ACTUAL:-not set}'."
  exit 1
fi
```

```bash
# Compare file hash against known-good
EXPECTED_HASH="a1b2c3d4e5f6..."
ACTUAL_HASH=$(sha256sum /etc/app/config.yaml 2>/dev/null | awk '{print $1}')
if [[ "$ACTUAL_HASH" != "$EXPECTED_HASH" ]]; then
  fail-message "The config file has been modified from the expected state. Re-read the assignment and update /etc/app/config.yaml."
  exit 1
fi
```

## Docker / Container State

### Container is running

```bash
if ! docker ps --format '{{.Names}}' | grep -q '^app$'; then
  fail-message "The 'app' container is not running. Start it with: docker start app"
  exit 1
fi
```

### Container image is correct

```bash
IMAGE=$(docker inspect app --format '{{.Config.Image}}' 2>/dev/null)
if [[ "$IMAGE" != "nginx:1.25" ]]; then
  fail-message "Expected the app container to use image nginx:1.25 but found ${IMAGE:-container not found}."
  exit 1
fi
```

## General Tips

- Always use `2>/dev/null` on commands that produce noisy stderr to keep the check output clean.
- Always set `--max-time` or use `timeout` on network commands — check scripts have a 60-second hard limit.
- Use `${VAR:-default}` in fail-messages to avoid empty strings when a variable is unset.
- Prefer exact matches over partial matches when the expected value is known.
- Order assertions from prerequisite to specific — check that the service exists before checking its configuration.
