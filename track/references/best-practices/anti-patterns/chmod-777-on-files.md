---
severity: advisory
---
# chmod 777 on Files

Setting world-readable, world-writable, and world-executable permissions on files and directories in setup scripts.

## Why It's Bad

While Instruqt sandboxes are ephemeral, `chmod 777` teaches learners bad permission habits that carry over to production environments. Tracks are educational content -- the patterns learners see become the patterns they reproduce.

Common cases seen in real tracks: `chmod 777 /var/run/docker.sock` (grants any user full Docker daemon access), `chmod 777 /etc/mongod.conf` (database config world-writable), and `chmod 777 -R /mnt/pv/` (entire persistent volume world-writable).

## What to Do Instead

Use the minimum permissions needed. Set ownership with `chown` and permissions with appropriate modes:

- `755` for directories and executables
- `644` for configuration files
- `600` for private keys and secrets
- `666` only when genuinely needed (e.g., a shared pipe)

## Examples

Bad -- blanket 777 on a sensitive file:

```bash
chmod 777 /var/run/docker.sock
chmod 777 /etc/app/config.yml
```

Good -- appropriate ownership and permissions:

```bash
chown root:docker /var/run/docker.sock
chmod 660 /var/run/docker.sock
usermod -aG docker student

chown root:root /etc/app/config.yml
chmod 644 /etc/app/config.yml
```
