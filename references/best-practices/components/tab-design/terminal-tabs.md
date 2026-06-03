# Terminal Tab Design

Evaluates whether terminal tabs are configured with the correct user context, working directory, and startup commands so the learner lands in a ready-to-use shell.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Terminal tabs open with wrong user, wrong directory, or fail to start due to malformed cmd/shell values |
| 2 | Below Standard | Terminal opens but learner must manually cd or switch users before they can follow the assignment |
| 3 | Adequate | Correct user and directory, but no startup command when one would meaningfully improve the experience |
| 4 | Good | Terminal opens in the correct directory with the correct user context; cmd or shell is used appropriately for the host type (production baseline) |
| 5 | Excellent | cmd enhances UX beyond basic correctness -- auto-displays setup output, sets a branded prompt, pre-attaches a screen/tmux session, or runs a background watcher |

## Guidance

Terminal tabs connect learners to a shell on a sandbox host. The configuration differs by host type.

For **containers**, use `cmd` to set the startup command:

```yaml
- title: Terminal
  type: terminal
  hostname: workstation
  cmd: su - developer
  workdir: /home/developer/project
```

For **VMs**, use `shell` instead of `cmd` to specify the shell binary:

```yaml
- title: Terminal
  type: terminal
  hostname: vm
  shell: /bin/zsh
  workdir: /root/project
```

For **Windows VMs**, PowerShell is native -- no shell override needed:

```yaml
- title: PowerShell
  type: terminal
  hostname: windows-vm
```

The `workdir` attribute sets the starting directory. Use it whenever the assignment assumes the learner is in a specific path.

Good -- switch to non-root user and land in their project directory:

```yaml
- title: Terminal
  type: terminal
  hostname: workstation
  cmd: su - student
  workdir: /home/student/app
```

Good -- pre-attach a screen session so learner sees a running process:

```yaml
- title: Application
  type: terminal
  hostname: workstation
  cmd: screen -xRR app
```

Good -- start an interactive shell with a startup script displayed:

```yaml
- title: Terminal
  type: terminal
  hostname: workstation
  cmd: bash -c "cat /etc/motd && exec bash"
  workdir: /root
```

Bad -- using cmd on a VM (use shell instead):

```yaml
- title: Terminal
  type: terminal
  hostname: vm
  cmd: /bin/zsh
```

Bad -- no workdir when the assignment says "run the following in /opt/app":

```yaml
- title: Terminal
  type: terminal
  hostname: workstation
```

Bad -- using shell on a container (use cmd instead):

```yaml
- title: Terminal
  type: terminal
  hostname: workstation
  shell: /bin/bash
```

## What to Watch For

- Containers use `cmd`, VMs use `shell` -- mixing them up causes silent failures or unexpected behavior
- `workdir` should match where the assignment tells the learner to work; mismatches force the learner to cd manually
- `cmd: su - user` is the standard pattern for non-root container shells; `su user` (without the dash) does not load the user's profile
- Windows VMs use PowerShell natively; do not add `shell: powershell` unless a specific version is needed
- `cmd: screen -xRR session` lets the learner attach to a detached process started in setup; useful for showing running services
- Avoid `cmd: bash -c "npm start"` if the process blocks the shell -- the learner cannot type; use screen or background the process instead
- When multiple terminals target the same host, give each a descriptive title so the learner knows which is which
