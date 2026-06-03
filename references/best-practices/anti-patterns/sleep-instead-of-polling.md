# Sleep Instead of Polling

Using `sleep N` as a substitute for infrastructure readiness polling.

## Why It's Bad

Blind sleep is fragile in both directions:

- If provisioning takes longer than expected, the lab breaks because the script proceeds before the resource is ready.
- If provisioning is faster, learner time is wasted waiting for a sleep that is no longer needed.

Infrastructure timing varies across cloud regions, sandbox load, and resource types. A fixed sleep is a guess that will eventually be wrong.

## What to Do Instead

Use a polling loop that checks for actual readiness. Reserve blind sleep only for inherently undetectable waits (e.g., DNS TTL propagation) where there is no condition to poll.

## Examples

Bad -- blind sleep hoping Kubernetes is ready:

```bash
# setup-sandbox
sleep 120
kubectl apply -f /opt/manifests/
```

Good -- polling for readiness:

```bash
# setup-sandbox
until kubectl get nodes -o jsonpath='{.items[0].status.conditions[?(@.type=="Ready")].status}' 2>/dev/null | grep -q True; do
  sleep 10
done
kubectl apply -f /opt/manifests/
```

Bad -- sleeping for a database to start:

```bash
sleep 30
mysql -u root -e "CREATE DATABASE lab;"
```

Good -- polling the database:

```bash
until mysqladmin ping -u root --silent 2>/dev/null; do
  sleep 5
done
mysql -u root -e "CREATE DATABASE lab;"
```
