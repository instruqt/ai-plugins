---
severity: blocking
---
# Unpinned Latest Release Downloads

Downloading tools from GitHub's `/releases/latest/download/` URL, which changes whenever the upstream project cuts a new release.

## Why It's Bad

The URL `https://github.com/org/repo/releases/latest/download/binary` resolves to whichever version is currently tagged as "latest." This means:

- A track that works today can break tomorrow when a new release changes CLI flags, output format, or dependencies.
- Two learners running the same track on different days may get different tool versions, making troubleshooting inconsistent.
- There is no way to reproduce a past track state because the version that was installed is not recorded.

## What to Do Instead

Pin to a specific version tag in the download URL. Store the version in a variable at the top of the script for easy updating.

## Examples

Bad -- unpinned latest:

```bash
curl -sSL -o /usr/local/bin/yq \
  https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
chmod +x /usr/local/bin/yq
```

Good -- pinned version:

```bash
YQ_VERSION="v4.40.5"
curl -sSL -o /usr/local/bin/yq \
  "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64"
chmod +x /usr/local/bin/yq
```

Bad -- unpinned with no version variable:

```bash
curl -L -o /usr/local/bin/argocd \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

Good -- pinned with version variable:

```bash
ARGOCD_VERSION="v2.10.1"
curl -L -o /usr/local/bin/argocd \
  "https://github.com/argoproj/argo-cd/releases/download/${ARGOCD_VERSION}/argocd-linux-amd64"
chmod +x /usr/local/bin/argocd
```
