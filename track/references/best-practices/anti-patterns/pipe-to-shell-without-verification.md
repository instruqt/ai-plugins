---
severity: blocking
---
# Pipe to Shell Without Verification

Downloading install scripts and piping them directly to a shell interpreter without checksum verification or version pinning.

## Why It's Bad

`curl https://example.com/install.sh | bash` combines two problems:

1. **No integrity verification.** If the upstream script is modified (intentionally or via compromise), the track silently installs different software. There is no checksum to detect the change.
2. **No version pinning.** The script at that URL may change between track creation and track execution. A track that works today can break tomorrow because the install script changed its behavior or dependencies.

The Instruqt sandbox is ephemeral, but a compromised install script can exfiltrate secrets (cloud account credentials, API tokens) that are injected into the environment.

## What to Do Instead

Download a specific versioned binary or archive, verify its SHA256 checksum, then install. If you must use an install script, pin it to a specific commit hash and verify the download.

## Examples

Bad -- piping an unverified, unpinned script to bash:

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Good -- downloading a pinned version with checksum verification:

```bash
HELM_VERSION="v3.14.0"
curl -fsSL -o helm.tar.gz "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz"
echo "f43e1c3387f3571f1bbd62085cac9b32a73ed1e18f0627eb01cb0b1145b98e95  helm.tar.gz" | sha256sum -c -
tar xzf helm.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
rm -rf helm.tar.gz linux-amd64/
```

Acceptable -- pinned commit hash if an install script is the only option:

```bash
COMMIT="abc123def456"
curl -fsSL "https://raw.githubusercontent.com/org/repo/${COMMIT}/install.sh" -o install.sh
echo "expected_sha256  install.sh" | sha256sum -c -
bash install.sh
```
