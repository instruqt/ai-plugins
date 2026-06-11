# Secrets

Named secrets injected as environment variables into all lifecycle scripts. Values are stored in the Instruqt platform UI and never committed to source code.

## Structure

```yaml
# In config.yml
secrets:
  - name: <ENV_VAR_NAME>
  - name: <ENV_VAR_NAME>
```

Each entry declares a secret by name. The actual value is set in the Instruqt web UI at the organization or track level.

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Environment variable name exposed to lifecycle scripts. Convention is UPPER_SNAKE_CASE. |

## Availability

- Available in **all** lifecycle scripts: setup, check, solve, cleanup (both track-level and per-challenge).
- Accessed as a standard environment variable: `$SECRET_NAME`.

## Use Cases

- API keys and tokens
- SSH private keys
- License keys
- Container registry credentials
- Cloud provider credentials not covered by native integrations

## Handling Gotchas

### SSH Private Keys

Private keys pasted into the UI often lose their newlines. Reformat in the setup script:

```bash
echo "$SSH_PRIVATE_KEY" | perl -pe 's/\\n/\n/g' > /root/.ssh/id_rsa
chmod 600 /root/.ssh/id_rsa
```

Or store the key base64-encoded and decode at runtime:

```bash
echo "$SSH_PRIVATE_KEY_B64" | base64 -d > /root/.ssh/id_rsa
chmod 600 /root/.ssh/id_rsa
```

### CRLF Sanitization

Secrets copied from Windows hosts may carry carriage returns. Strip them:

```bash
CLEAN_TOKEN=$(echo "$API_TOKEN" | tr -d '\r\n')
```

## Anti-Patterns

- **Never** put secret values in `config.yml` `environment:` blocks.
- **Never** bake secrets into custom VM images.
- **Never** commit secret values to the track repository.

## Examples

### Typical Config

```yaml
secrets:
  - name: ARANGO_PWD
  - name: REGISTRY_TOKEN
  - name: LICENSE_KEY
```

### Using a Secret in a Setup Script

```bash
#!/bin/bash
set -euxo pipefail

# Secret injected by Instruqt
curl -H "Authorization: Bearer $API_TOKEN" https://api.example.com/setup
```
