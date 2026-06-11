# Terminal Tab

Shell access to a sandbox host. The default learner interface -- every track has at least one. Supports both standard interactive shells and custom startup commands (SSH tunnels, REPLs, screen/tmux sessions).

## When to Use

- **Always.** Every challenge needs at least one terminal tab so learners can run commands.
- **Multiple terminals** when the track has multiple hosts (one terminal per host) or when learners need parallel sessions (e.g., one terminal running a server, another for `curl`/`kubectl` commands).
- **cmd variant** when the default shell is not what the learner needs: SSH to an external host, a language REPL, a pre-attached tmux/screen session, or `su -` to a non-root user.

## Config.yml Example

Standard shell terminal:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
```

Custom command (SSH to a remote host provisioned by the track):

```yaml
tabs:
  - title: Remote Host
    type: terminal
    hostname: workstation
    cmd: ssh -o StrictHostKeyChecking=no user@remote-server
```

Language REPL:

```yaml
tabs:
  - title: Python
    type: terminal
    hostname: workstation
    cmd: python3
    workdir: /home/developer/project
```

Multi-host track:

```yaml
tabs:
  - title: Control Plane
    type: terminal
    hostname: controlplane
  - title: Worker Node
    type: terminal
    hostname: worker01
```

## Pairs With

All base compute patterns -- containers, VMs, Kubernetes clusters. Terminal tabs connect to any sandbox host defined in `config.yml`.

## Common Patterns

- **Terminal + Editor side by side.** Learner edits code in an editor tab, runs it in the terminal. The most common two-tab layout.
- **Terminal + Service tab.** Learner runs commands in the terminal, observes results in a web UI (Grafana, a custom app, etc.).
- **Multiple terminals for split roles.** A Kubernetes track with `controlplane` and `worker01` hosts gives each its own terminal tab.
- **cmd for user context.** Use `cmd: su - developer` when the learner should operate as a non-root user on a container host.
- **Pre-attached multiplexer.** Use `cmd: screen -xRR` or `cmd: tmux attach` when setup scripts start a background session the learner should see.

## Common Pitfalls

- **Too many terminal tabs.** If every challenge exposes terminals for hosts the learner never touches, the tab bar becomes cluttered. Only add terminal tabs for hosts the learner actively uses in that challenge.
- **Missing workdir.** Without `workdir`, the terminal opens in `/root` (VMs) or `/` (containers). If the exercise assumes the learner is in a project directory, set `workdir` explicitly.
- **cmd that exits immediately.** If the `cmd` process exits (e.g., a script that finishes), the terminal shows a disconnected state. Use `cmd` only for long-running or interactive processes.

## When NOT to Use

Terminal tabs are appropriate for virtually all tracks. The only case where you might omit them is a pure browser-based track where learners interact exclusively through a virtual browser and never need a shell -- but this is rare.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- CLI tool fundamentals (single terminal, single host)
- Kubernetes multi-node cluster administration (terminal per node)
- Database administration with SSH access to a managed instance (cmd variant with SSH)
- Full-stack development workshop (terminal + editor + service tab)
