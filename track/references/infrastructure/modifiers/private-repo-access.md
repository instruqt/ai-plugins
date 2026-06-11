# Private Repository Access

Securely cloning private Git repositories from sandbox hosts. Use when a track needs source code, configuration, or exercise files from a private GitHub, GitLab, or Bitbucket repository.

## What It Provides

A pattern for injecting Git credentials into the sandbox at runtime using Instruqt `secrets:`, so lifecycle scripts can clone private repositories without hardcoding tokens in track source code.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

secrets:
  - name: GITHUB_TOKEN                            # value set in Instruqt UI
```

### HTTPS Clone with Personal Access Token

```bash
#!/bin/bash
set -euxo pipefail

git clone "https://${GITHUB_TOKEN}@github.com/myorg/workshop-exercises.git" /root/exercises
```

### SSH Deploy Key Approach

```yaml
secrets:
  - name: DEPLOY_KEY_B64                          # base64-encoded SSH private key
```

```bash
#!/bin/bash
set -euxo pipefail

mkdir -p /root/.ssh
echo "$DEPLOY_KEY_B64" | base64 -d > /root/.ssh/id_ed25519
chmod 600 /root/.ssh/id_ed25519
ssh-keyscan github.com >> /root/.ssh/known_hosts 2>/dev/null

git clone git@github.com:myorg/workshop-exercises.git /root/exercises
```

## Common Pitfalls

- **Hardcoded tokens in scripts or config.yml**. Tokens committed to the track repository end up in git history and are visible to anyone with repo access. Always use `secrets:` and reference the environment variable.
- **Token scope too broad**. Use a fine-grained PAT or a read-only deploy key scoped to the specific repository. Never use a token with write or admin access unless the track explicitly requires it.
- **SSH key newline mangling**. Private keys pasted into the Instruqt UI often lose their newlines. Store the key base64-encoded and decode at runtime (see example above).
- **Forgetting ssh-keyscan**. Without adding the remote host to `known_hosts`, the SSH clone fails with a host verification prompt that hangs the setup script.
- **Rate limits on GitHub PATs**. If many sandbox sessions clone simultaneously, a single PAT may hit GitHub's API rate limit. Consider a GitHub App installation token or a machine user for high-traffic tracks.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- DevOps pipeline workshop cloning a private sample application repo
- Security scanning lab pulling a repository with intentional vulnerabilities
- GitOps tutorial where learners work with a pre-configured private Flux/Argo repo
- Microservices debugging exercise with multiple private service repositories
