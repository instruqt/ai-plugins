---
severity: blocking
---
# Credentials in Scripts

Embedding credentials (passwords, tokens, API keys) directly in lifecycle scripts -- whether as literal values, in git clone URLs, or exported to shell dotfiles.

## Why It's Bad

Credentials in scripts are visible in source control, shell history, process listings, and the filesystem. Even in ephemeral Instruqt sandboxes, scripts are committed to git repositories that may be public or shared across teams. Three common vectors:

**Git clone URLs.** Tokens in `git clone https://user:TOKEN@github.com/...` persist in `.git/config` of the cloned repo, in `ps` output during the clone, and in shell history.

**Hardcoded passwords.** Literal passwords in `docker run -e PASSWORD=secret` or `echo 'export PASSWORD="secret"' >> ~/.bashrc` are visible to anyone reading the script.

**Dotfile exports.** Appending secrets to `.bashrc` or `.profile` persists them in the filesystem where any process or user can read them.

## What to Do Instead

Use Instruqt's secrets mechanism (`secrets:` block in config.yml) for credentials that must be injected into the sandbox. Use `agent variable set/get` for values generated during setup that need to be shared across scripts. For git authentication, use credential helpers or separate header configuration.

## Examples

Bad -- token in git clone URL:

```bash
git clone https://x-access-token:ghp_XXXXXXXXXXXX@github.com/org/repo.git
```

Good -- token via environment variable and credential helper:

```bash
git -c "http.extraheader=Authorization: token ${GITHUB_TOKEN}" clone https://github.com/org/repo.git
```

Bad -- password exported to dotfile:

```bash
echo "export DB_PASSWORD='SuperSecret123'" >> /root/.bashrc
```

Good -- password via Instruqt secrets:

```yaml
# config.yml
secrets:
- name: DB_PASSWORD
```

```bash
# setup script -- DB_PASSWORD is available as an environment variable
mysql -u admin -p"${DB_PASSWORD}" -e "CREATE DATABASE lab;"
```

Bad -- hardcoded password in docker run:

```bash
docker run -e POSTGRES_PASSWORD=admin -d postgres:15
```

Good -- password from agent variable or Instruqt secret:

```bash
docker run -e POSTGRES_PASSWORD="${DB_PASSWORD}" -d postgres:15
```
