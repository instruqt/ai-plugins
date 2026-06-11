# Environment Configuration

Evaluates how setup scripts configure the learner's shell environment, including environment variables, prompt customization, working directory, and shell completions.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Environment variables set only in one location (e.g., only in .bashrc); terminal prompt left at default with root@hostname; no working directory set |
| 2 | Below Standard | Env vars set in .bashrc but not /etc/profile.d/, causing them to be missing in some shell contexts; prompt customized but includes ANSI escape codes |
| 3 | Adequate | Env vars available in interactive shells; working directory set; but some shells (login vs interactive) still miss variables |
| 4 | Good | Env vars written to both .bashrc and /etc/profile.d/ for full coverage; working directory set via set-workdir; prompt is clean without username/hostname (production baseline) |
| 5 | Excellent | Branded, clean terminal experience with minimal PS1, kubectl/CLI completions configured, aliases set, and no visual noise from system messages |

## Guidance

The learner interacts with the environment through Instruqt's terminal tab, which may open as a login shell or an interactive non-login shell depending on context. Environment variables must be available in both.

### Dual-write pattern for environment variables

Good -- writing to both .bashrc and /etc/profile.d/:

```bash
# Write to /etc/profile.d/ for login shells
cat >> /etc/profile.d/instruqt-env.sh << 'EOF'
export APP_URL="http://app.localhost:8080"
export API_KEY="demo-key-12345"
EOF

# Write to .bashrc for interactive shells
cat >> /root/.bashrc << 'EOF'
export APP_URL="http://app.localhost:8080"
export API_KEY="demo-key-12345"
EOF
```

### PS1 prompt customization

The terminal prompt should be clean and minimal. Do not include the username or hostname -- the learner does not need to know they are root on a randomly named host. Do not use ANSI escape codes in PS1, as they cause rendering issues in the Instruqt terminal.

Good -- clean prompt:

```bash
cat >> /root/.bashrc << 'SHELL'
PS1="\W \$ "
SHELL
```

Good -- prompt with just the working directory:

```bash
cat >> /root/.bashrc << 'SHELL'
PS1="[\w] $ "
SHELL
```

Bad -- default prompt with user and hostname:

```bash
# Leaving PS1 as default: root@abc123xyz:~#
# This exposes infrastructure details to the learner
```

Bad -- ANSI escape codes in PS1:

```bash
PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "
# ANSI codes cause rendering artifacts in Instruqt terminal
```

### Working directory

Use `set-workdir` to configure which directory the terminal opens in:

```bash
set-workdir /root/project
```

### Shell completions and aliases

Good -- kubectl completion and alias:

```bash
kubectl completion bash > /etc/bash_completion.d/kubectl
echo 'alias k=kubectl' >> /root/.bashrc
echo 'complete -o default -F __start_kubectl k' >> /root/.bashrc
```

Good -- suppressing login noise:

```bash
# Suppress "last login" and motd messages
touch /root/.hushlogin
```

### Cloud resource name uniquification

When creating cloud resources (S3 buckets, GCS buckets, Azure resource groups, DNS entries) that require globally unique names, use `_SANDBOX_ID` to avoid collisions between concurrent track sessions:

```bash
# In setup script
BUCKET_NAME="lab-data-${_SANDBOX_ID}"
gsutil mb "gs://${BUCKET_NAME}"

# Store for use in check/solve scripts and assignment interpolation
agent variable set BUCKET_NAME "$BUCKET_NAME"
```

```bash
# In profile.d for learner use
cat >> /etc/profile.d/instruqt-env.sh <<EOF
export BUCKET_NAME="lab-data-${_SANDBOX_ID}"
EOF

cat >> /root/.bashrc <<EOF
export BUCKET_NAME="lab-data-${_SANDBOX_ID}"
EOF
```

Without uniquification, concurrent track sessions attempt to create resources with the same name, causing failures for all but the first.

## What to Watch For

- Variables set only in `.bashrc` are missing when the terminal opens as a login shell (and vice versa for `/etc/profile.d/`)
- The `set-workdir` command must be called in setup, not in the assignment notes
- PS1 must not contain `\u` (username) or `\h` (hostname) -- these expose infrastructure details
- ANSI color codes in PS1 cause cursor positioning bugs in the web terminal
- Shell completions that depend on the tool being installed must run after the tool installation step
- The `/etc/profile.d/` file must be sourced by the shell, so it needs a `.sh` extension
- Cloud resources with hardcoded names will collide when multiple learners run the track simultaneously — always uniquify with `_SANDBOX_ID`
