# Bake Credentials Into Images

Embedding long-lived credentials (API keys, private keys, tokens) into custom VM or container images.

## Why It's Bad

Custom images get cached, copied, and shared across environments. Credentials baked into them leak to anyone with image access. They also end up in image layer history and are nearly impossible to rotate without rebuilding every image.

## What to Do Instead

Use the `secrets:` block in `config.yml` and reference secrets at setup time. Write credentials to disk only during the setup lifecycle script, never at image build time.

## Examples

Bad -- baking a CA private key into a custom image's Dockerfile:

```dockerfile
COPY ca-private-key.pem /etc/ssl/private/ca-key.pem
```

Good -- storing the key as an Instruqt secret and writing it at setup time:

```yaml
# config.yml
secrets:
  - name: SDE_CA_PRIVATE_KEY
    key: SDE_CA_PRIVATE_KEY
```

```bash
# setup-sandbox
echo "${SDE_CA_PRIVATE_KEY}" > /etc/ssl/private/ca-key.pem
chmod 600 /etc/ssl/private/ca-key.pem
```
