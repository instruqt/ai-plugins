---
severity: blocking
---
# Base64 Docker Credentials

Hardcoding dockerconfigjson credentials as base64 strings in setup scripts.

## Why It's Bad

base64 is an encoding, not encryption. Anyone who reads the script can decode the credentials instantly. Since setup scripts are committed to git, the registry credentials are permanently exposed in repository history.

## What to Do Instead

Use the `secrets:` block to deliver registry credentials at runtime. Construct the dockerconfigjson dynamically in the setup script using the injected secret.

## Examples

Bad -- base64-encoded credentials in a setup script:

```bash
# setup-sandbox
kubectl create secret docker-registry regcred \
  --from-file=.dockerconfigjson=<(echo "eyJhdXRocyI6eyJyZWdpc3RyeS5leGFtcGxlLmNvbSI6eyJ1c2VybmFtZSI6ImFkbWluIiwicGFzc3dvcmQiOiJzM2NyM3QifX19" | base64 -d)
```

Good -- using Instruqt secrets:

```yaml
# config.yml
virtualmachines:
  - name: sandbox
    secrets:
      - name: REGISTRY_USERNAME
        key: REGISTRY_USERNAME
      - name: REGISTRY_PASSWORD
        key: REGISTRY_PASSWORD
```

```bash
# setup-sandbox
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username="${REGISTRY_USERNAME}" \
  --docker-password="${REGISTRY_PASSWORD}"
```
