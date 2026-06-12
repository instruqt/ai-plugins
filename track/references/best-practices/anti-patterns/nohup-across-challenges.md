---
severity: blocking
---
# Nohup Across Challenges

Relying on `nohup` to keep background processes alive across challenge boundaries.

## Why It's Bad

Instruqt's runner does NOT keep nohup jobs alive between challenges. Async warmups, deferred installs, and background daemons started with nohup die at the challenge boundary. The process simply disappears when the next challenge begins, causing silent failures.

## What to Do Instead

Use nohup only within a single challenge where you need a background process for that challenge's duration. For work that must persist, run it synchronously in the prior challenge's setup script so it completes before the learner advances. Warm caches, pre-pull images, and install packages synchronously.

For services that genuinely need to run continuously across challenges (web servers, databases, monitoring agents), create a systemd unit file instead:

```bash
# setup-sandbox (track-level setup)
cat > /etc/systemd/system/lab-service.service <<'EOF'
[Unit]
Description=Lab Service
After=network.target

[Service]
ExecStart=/usr/local/bin/lab-service --listen :8080
Restart=always
Environment=LOG_LEVEL=info

[Install]
WantedBy=multi-user.target
EOF
systemctl enable --now lab-service
```

Systemd manages the process lifecycle, restarts on crashes, and survives challenge boundaries.

## Examples

Bad -- starting an async warmup in challenge 1's setup hoping it finishes by challenge 2:

```bash
# setup-sandbox (challenge 1)
nohup /usr/local/bin/warm-cache.sh &
```

Good -- running the warmup synchronously in the setup where it's needed:

```bash
# setup-sandbox (challenge 1)
/usr/local/bin/warm-cache.sh
```

If the warmup is for challenge 2, run it synchronously in challenge 2's setup script instead.
