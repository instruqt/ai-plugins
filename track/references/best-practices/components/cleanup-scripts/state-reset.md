# State Reset

Evaluates whether per-challenge cleanup scripts correctly reset environment state between challenges, providing a clean slate for the next challenge's setup and instructions.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No per-challenge cleanup; state from previous challenges leaks into subsequent ones causing confusion or false passes |
| 2 | Below Standard | Some state reset exists but is incomplete; leftover files, running processes, or installed tools bleed between challenges |
| 3 | Adequate | Most state is reset, but edge cases remain (e.g., environment variables set by the learner persist, background processes still running) |
| 4 | Good | Clean slate for next challenge: files removed, services stopped, Helm releases uninstalled, binaries removed for progressive curricula (production baseline) |
| 5 | Excellent | Idempotent cleanup that works regardless of the learner's current state -- handles cases where the learner completed the challenge, partially completed it, or did nothing |

## Guidance

Per-challenge cleanup runs after the learner clicks "Check" and passes (or skips). It prepares the environment for the next challenge. This is distinct from resource teardown (track-level cleanup), which deletes external resources.

### File cleanup for progressive curricula

Good -- removing files the learner created so the next challenge starts fresh:

```bash
#!/bin/bash
# Remove the config file the learner created in this challenge
rm -f /root/project/config.yaml

# Remove binaries installed during this challenge
rm -f /usr/local/bin/mytool
```

### Helm release teardown

Good -- uninstalling a release between challenges:

```bash
#!/bin/bash
# Clean up the Helm release the learner deployed
helm uninstall my-release -n workshop --wait 2>/dev/null || true

# Ensure the namespace is clean for the next challenge
kubectl delete pods --all -n workshop --grace-period=0 --force 2>/dev/null || true
```

### Process cleanup

Good -- stopping background processes:

```bash
#!/bin/bash
# Kill any port-forward the learner started
pkill -f "kubectl port-forward" || true

# Stop a service the learner started
systemctl stop myapp 2>/dev/null || true
```

### Idempotent cleanup pattern

Good -- cleanup that works regardless of learner state:

```bash
#!/bin/bash
# These all use || true or conditional checks because the learner
# may not have reached the step that creates these resources

# Remove file if it exists
rm -f /root/project/deployment.yaml

# Delete k8s resources if they exist
kubectl delete deployment web -n workshop 2>/dev/null || true
kubectl delete service web -n workshop 2>/dev/null || true

# Uninstall Helm release if it exists
helm uninstall web -n workshop 2>/dev/null || true

# Kill process if running
pkill -f "node server.js" || true
```

Bad -- cleanup assumes the learner completed everything:

```bash
#!/bin/bash
# Fails if the learner didn't create the deployment
kubectl delete deployment web -n workshop  # Error: not found -> script fails
helm uninstall web -n workshop             # Error: release not found -> script fails
```

Bad -- no cleanup between challenges (state leak):

```bash
#!/bin/bash
# Empty cleanup script
# The deployment from challenge 2 is still running when challenge 3 starts
# This may cause the learner's check in challenge 3 to pass prematurely
```

Bad -- overly aggressive cleanup that breaks the environment:

```bash
#!/bin/bash
# DON'T wipe everything -- only clean up challenge-specific state
rm -rf /root/*  # Destroys helper scripts, git repos, and shared config
kubectl delete namespace workshop  # Destroys shared infrastructure
```

## What to Watch For

- Every delete/remove/stop command in cleanup must tolerate the resource not existing (`|| true`, `2>/dev/null`, `-f` flag)
- Cleanup should only remove challenge-specific state, not shared infrastructure set up at the track level
- For progressive curricula (each challenge builds on the last), cleanup must remove exactly what the learner added so the next setup can seed the starting state
- Background processes started by the learner (port-forwards, watches, servers) should be killed in cleanup
- Environment variables set by the learner in the terminal persist across challenges; cleanup cannot unset them from the learner's shell, but setup can re-export correct values
- Cleanup scripts run with a timeout; avoid long-running cleanup operations like waiting for pod termination
