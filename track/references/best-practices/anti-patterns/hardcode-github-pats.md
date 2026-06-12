---
severity: blocking
---
# Hardcode GitHub PATs

Hardcoding GitHub Personal Access Tokens or repository tokens directly in lifecycle scripts.

## Why It's Bad

Script files are committed to git. A PAT in a script file means it lives in repository history permanently, accessible to anyone who can clone the repo. Rotating the token requires rewriting git history or accepting the leak.

## What to Do Instead

Store the token as an Instruqt secret and reference it as an environment variable in the script.

## Examples

Bad -- PAT hardcoded in setup script:

```bash
# setup-sandbox
git clone https://ghp_abc123def456@github.com/acme/private-repo.git /opt/repo
```

Good -- using an Instruqt secret:

```yaml
# config.yml
virtualmachines:
  - name: sandbox
    secrets:
      - name: GITHUB_TOKEN
        key: GITHUB_TOKEN
```

```bash
# setup-sandbox
git clone "https://${GITHUB_TOKEN}@github.com/acme/private-repo.git" /opt/repo
```
