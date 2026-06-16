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
git -c "http.extraheader=Authorization: token ${GITHUB_TOKEN}" clone https://github.com/acme/private-repo.git /opt/repo
```

Avoid putting the token in the clone URL (`https://${GITHUB_TOKEN}@github.com/...`) — it persists in the cloned repo's `.git/config`. The `http.extraheader` form keeps it out. See `best-practices/anti-patterns/credentials-in-scripts.md`.
