# Secrets Management

Evaluates whether sensitive values (API keys, tokens, SSH keys, passwords) are properly handled using the secrets: block and never leaked through environment variables, scripts, or images.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Secrets hardcoded in scripts, environment blocks, or baked into container images |
| 2 | Below Standard | Some secrets in the secrets: block but others leaked in environment: or embedded in setup scripts as literal values |
| 3 | Adequate | All secrets in the secrets: block, but no documentation of their purpose or missing CRLF sanitization for multi-line values |
| 4 | Good | All sensitive values stored in the secrets: block; scripts reference them correctly; no secrets in environment: or script bodies (production baseline) |
| 5 | Excellent | Secrets documented with inline comments in config.yml explaining their purpose; multi-line values (SSH keys) properly formatted; CRLF sanitization applied where needed |

## Guidance

The `secrets:` block in config.yml is the only safe place to store sensitive values. Secrets defined here are injected as environment variables into the sandbox at runtime and are not visible in track source code or the Instruqt UI.

### Where Secrets Belong

Good -- secrets in the secrets: block:

```yaml
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
  secrets:
  - name: API_TOKEN
    value: sk-live-abc123def456
  - name: DB_PASSWORD
    value: "p@ssw0rd!complex"
```

Bad -- secrets in the environment: block (visible in track config):

```yaml
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
  environment:
    API_TOKEN: sk-live-abc123def456
```

Bad -- secrets hardcoded in setup scripts:

```bash
#!/bin/bash
export API_TOKEN="sk-live-abc123def456"
echo "$API_TOKEN" > /root/.config/token
```

Bad -- secrets baked into a container image:

```dockerfile
# Never do this
ENV API_TOKEN=sk-live-abc123def456
```

### SSH Key Formatting

SSH private keys are multi-line values that require careful handling. YAML multi-line scalars can introduce unwanted whitespace or newline changes.

Good -- SSH key with proper YAML formatting:

```yaml
secrets:
- name: SSH_PRIVATE_KEY
  value: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gt
    ZWQyNTUxOQAAACBhbGljZUBleGFtcGxlLmNvbQAAAABhbGljZUBleGFtcGxlLmNvbQ==
    -----END OPENSSH PRIVATE KEY-----
```

### CRLF Sanitization

Values pasted from Windows environments or web UIs may contain CRLF line endings (\r\n) instead of LF (\n). This breaks SSH keys, certificates, and other PEM-formatted values. Setup scripts should sanitize:

```bash
#!/bin/bash
# Sanitize CRLF from SSH key
echo "$SSH_PRIVATE_KEY" | tr -d '\r' > /root/.ssh/id_rsa
chmod 600 /root/.ssh/id_rsa
```

### Documenting Secrets

Good -- secrets with purpose comments:

```yaml
secrets:
# GitHub personal access token for cloning private repos
- name: GITHUB_TOKEN
  value: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# SSH key for accessing the staging server
- name: SSH_PRIVATE_KEY
  value: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----
```

## What to Watch For

- Any literal secret value appearing in `environment:` blocks -- these are not encrypted or hidden
- Secrets hardcoded directly in setup, check, or solve scripts instead of referenced as environment variables
- SSH private keys without CRLF sanitization in the setup script that consumes them
- Multi-line secrets (keys, certificates) using incorrect YAML scalar style (avoid `>` folded style for PEM data -- use `|` literal style)
- Secrets referenced in scripts but not defined in the secrets: block -- the variable will be empty at runtime
- Passwords or tokens visible in git history even if later moved to secrets: -- the exposure already happened
