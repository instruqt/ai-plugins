---
severity: advisory
---
# Snap Install for Standard Tools

Using `snap install` to install standard CLI tools in setup scripts when direct binary downloads or apt packages are available.

## Why It's Bad

Snap adds significant overhead to the setup process:

- The `snapd` daemon must be running, which requires full systemd support. Container-based sandboxes may not have this.
- Snap packages use loopback-mounted squashfs images, adding disk I/O and startup latency.
- Snap's automatic background refresh can interfere with track operations.
- First-run snap installs are noticeably slower than direct binary downloads.

## What to Do Instead

Download static binaries directly from the tool's release page with version pinning. For tools available via apt, use apt with version pinning. Reserve snap for tools that are only distributed as snaps.

## Examples

Bad -- snap install for tools with direct binaries:

```bash
snap install yq
snap install jq
snap install terraform --classic
```

Good -- direct binary download:

```bash
YQ_VERSION="v4.40.5"
curl -sSL -o /usr/local/bin/yq \
  "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64"
chmod +x /usr/local/bin/yq
```

Good -- apt with version pinning:

```bash
apt-get update && apt-get install -y jq=1.6-2.1ubuntu3
```
