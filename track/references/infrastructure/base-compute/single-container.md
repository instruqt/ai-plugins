# Single Container

Simplest infrastructure pattern. One Docker container for CLI-only workshops where no VM-level features (systemd, Docker, nested virtualization) are needed.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/shell        # minimal shell; swap for cloud-client if you need cloud CLIs
    ports:
      - 8080                             # expose if the challenge serves a web UI
    memory: 256                          # MB; raise to 512 for heavier workloads
    shell: /bin/bash
```

### Variant: cloud-client

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2   # includes gcloud, aws, az, kubectl, helm, terraform
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
    environment:
      AWS_DEFAULT_REGION: us-west-2
```

## When to Use

- Track teaches CLI tools, scripting, or language fundamentals with no OS-level dependencies.
- No need for systemd, Docker, or kernel features.
- Fastest sandbox startup time of any pattern.

## When NOT to Use

- Track needs to run Docker commands (use a VM -- containers have no Docker socket).
- Track needs systemd services (use a VM).
- Track needs a GUI desktop (use a VM with Guacamole/KasmVNC).
- Track needs more than ~1 GB of memory for heavy workloads (VM may be more appropriate).

## Common Pitfalls

- **No cloud CLIs on `gcr.io/instruqt/shell`**. It is a bare Alpine/Debian shell. If the track uses `aws`, `gcloud`, `az`, `kubectl`, `helm`, or `terraform`, use `gcr.io/instruqt/cloud-client:2` instead.
- **No display server**. Containers cannot run GUI applications, VNC, or RDP. Use a VM pattern for anything graphical.
- **No systemd**. Services that expect systemd (e.g., `systemctl enable`) will not work. Use entrypoint scripts or supervisord if you need background processes.
- **No Docker-in-Docker**. Containers do not have Docker socket access. If the track runs `docker` commands, use a VM with `nested_virtualization: true`.
- **Memory sizing**. The default 256 MB is tight. Node.js, JVM, or Python ML workloads often need 512 MB or more. Watch for OOM kills during testing.
- **Port exposure**. Only ports listed under `ports:` are reachable from tabs and other sandbox hosts. Forgetting to list a port is a common cause of "service tab shows nothing."

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- REST API fundamentals (curl exercises against a local server)
- Cloud CLI exercises (cloud-client + cloud account)
- Database migration tool workshops (Liquibase, Flyway)
- Shell scripting tutorials
- Git workflow exercises
- Package manager and dependency management tutorials
