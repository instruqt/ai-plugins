# Docker on VM

Docker daemon running on a virtual machine. Use when the track needs to run containers alongside VM-level features (systemd, kernel modules, nested virtualization) or when only one or two containers are needed and a full Compose setup is overkill.

## Requirements

- `nested_virtualization: true` in config.yml -- Docker requires `/dev/kvm` on Instruqt VMs.
- `allow_external_ingress` with `http`, `https`, and `high-ports` if any container exposes a web UI.
- Minimum `n1-standard-2` for light workloads. Bump to `n1-standard-4` if pulling large images or running memory-hungry containers.

## Setup Example

Using an Instruqt pre-built Docker image (fastest startup):

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

```bash
# track_scripts/setup-workstation
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Pull images during setup so learners don't wait
docker pull nginx:alpine
docker pull redis:7
```

Installing Docker on a bare Ubuntu image (when you need a specific Docker version or distro):

```bash
# track_scripts/setup-workstation
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Install Docker CE
curl -fsSL https://get.docker.com | sh

# Pre-pull images
docker pull postgres:16-alpine
```

## Config.yml Implications

| Field | Why |
|-------|-----|
| `nested_virtualization: true` | Docker cannot start without it |
| `allow_external_ingress: [http, https, high-ports]` | Required for service tabs reaching container ports |
| `provision_ssl_certificate: true` | Avoids certificate errors on HTTPS service tabs |

## Common Pitfalls

- **Docker fails to start without nested virtualization**. This is the most common misconfiguration. If `nested_virtualization: true` is missing, `dockerd` will not start and `docker` commands will error with "Cannot connect to the Docker daemon."
- **Slow setup from image pulls**. Large images (multi-GB) can add minutes to track startup. Use `instruqt/docker-*` pre-built images and pull only the specific tags you need in setup, not `latest`.
- **Container port not reachable from service tabs**. The container port must be published (`docker run -p 8080:8080`) AND the VM must have `allow_external_ingress` with `high-ports`. Both are required.
- **Docker socket permissions**. Lifecycle scripts run as root, so Docker commands work. If the track uses a non-root shell (`shell: su - student`), add that user to the `docker` group in setup.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Running a single containerized web application for learners to inspect and configure
- Docker CLI fundamentals (build, run, exec, logs, inspect)
- Container security scanning workshops
- Database administration with a containerized database engine
