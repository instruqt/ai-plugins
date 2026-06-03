# Tool Installation

Evaluates the correctness and robustness of tool installation patterns in setup scripts, including version pinning, platform detection, and installation speed optimization.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Tools installed without version pinning using curl-pipe-bash patterns from untrusted sources; installations fail silently |
| 2 | Below Standard | Tools installed with some version awareness but using latest tags or unpinned apt packages; no checksum verification |
| 3 | Adequate | Most tools version-pinned; correct package repos added; but installation order is sequential and slow |
| 4 | Good | Installations are version-pinned where it matters, use the correct method for the image OS, and repos are added with proper GPG keys (production baseline) |
| 5 | Excellent | Slow installations are backgrounded with & and joined with wait; pre-built images used where possible; checksums verified for direct downloads |

## Guidance

Different tools require different installation strategies. Choose the right pattern for each tool type.

### APT repository pattern (HashiCorp, Docker, etc.)

Good -- adding a repo with signed key and pinned version:

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive

# Add HashiCorp repo
curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
  > /etc/apt/sources.list.d/hashicorp.list

apt-get update -qq
apt-get install -y -qq terraform=1.7.5-1
```

### Direct binary download pattern

Good -- version-pinned binary with platform detection:

```bash
VAULT_VERSION="1.15.4"
ARCH=$(dpkg --print-architecture)
curl -fsSL "https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_${ARCH}.zip" \
  -o /tmp/vault.zip
unzip -o /tmp/vault.zip -d /usr/local/bin/
rm /tmp/vault.zip
chmod +x /usr/local/bin/vault
```

### Python tool installation

Good -- using pip with version pin:

```bash
pip3 install ansible==2.16.2
```

### Helm chart from OCI registry

Good -- version-pinned Helm install:

```bash
helm install argocd oci://ghcr.io/argoproj/argo-helm/argo-cd --version 5.51.6 -n argocd --create-namespace
```

### Parallel installation for speed

Good -- backgrounding independent installations:

```bash
# Start slow installs in parallel
apt-get install -y -qq openjdk-17-jdk &
PID_JAVA=$!

curl -fsSL "https://go.dev/dl/go1.22.0.linux-amd64.tar.gz" -o /tmp/go.tar.gz &
PID_GO=$!

# Wait for all background installs
wait $PID_JAVA || { echo "Java install failed"; exit 1; }
wait $PID_GO || { echo "Go download failed"; exit 1; }

tar -C /usr/local -xzf /tmp/go.tar.gz
```

Bad -- unpinned version (breaks when upstream updates):

```bash
apt-get install -y terraform  # Gets whatever version is latest
```

Bad -- curl-pipe-bash from unknown source:

```bash
curl -s https://random-site.com/install.sh | bash  # Unverified, unpinned
```

Bad -- using latest tag for container images:

```bash
docker pull nginx:latest  # Unpredictable content
```

## What to Watch For

- Version pinning is critical for reproducibility; unpinned installs cause intermittent track failures when upstream releases change
- Use `curl -fsSL` (fail silently on HTTP errors, show errors, follow redirects) not bare `curl`
- GPG keys should go in `/usr/share/keyrings/` not the deprecated `apt-key add` method
- Architecture detection (`dpkg --print-architecture` or `uname -m`) is needed for multi-arch images
- When backgrounding installations, each `wait $PID` must be checked for failure, especially under `set -e`
- Pre-pulling Docker images (`docker pull image:tag &`) in track-level setup avoids pull delays during challenges
- If a tool is needed on every track, consider baking it into a custom sandbox image instead of installing at runtime
