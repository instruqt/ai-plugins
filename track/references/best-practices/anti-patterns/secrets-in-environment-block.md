---
severity: blocking
---
# Secrets in Environment Block

Placing SSH keys, API keys, or other secret material in config.yml's `environment:` block.

## Why It's Bad

Two failure modes:

1. **Git history exposure**: config.yml is committed to git. Secrets in the environment block live in repository history forever, even if later removed.
2. **Debug log visibility**: The `environment:` block contents are visible in Instruqt's debug log, exposing secrets to anyone with log access.

## What to Do Instead

Use the `secrets:` block in config.yml. Secrets referenced this way are injected at runtime, never stored in git, and redacted from logs.

## Examples

Bad -- API key in the environment block:

```yaml
# config.yml
virtualmachines:
  - name: sandbox
    environment:
      ELASTIC_API_KEY: "base64encodedkey=="
      SSH_PRIVATE_KEY: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIEpAIBAAKCAQEA...
```

Good -- using the secrets block:

```yaml
# config.yml
virtualmachines:
  - name: sandbox
    secrets:
      - name: ELASTIC_API_KEY
        key: ELASTIC_API_KEY
      - name: SSH_PRIVATE_KEY
        key: SSH_PRIVATE_KEY
```
