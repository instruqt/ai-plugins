# Single VM with Docker

One virtual machine running one or a few Docker containers via `docker run`. The VM provides systemd alongside the Docker daemon, so you can mix native services with containerized ones. Not enough services to warrant Docker Compose -- just individual `docker run` commands in setup scripts.

See `modifiers/docker-on-vm.md` for Docker setup details and image pull strategies.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-2
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

### Common Images

| Image | What it provides |
|-------|-----------------|
| `instruqt/docker-28-3` | Ubuntu with Docker CE 28.3 pre-installed, ready to `docker run` immediately |
| `ubuntu-os-cloud/ubuntu-2404-lts-amd64` | Bare Ubuntu 24.04 -- requires installing Docker in setup (slower, use only if you need a specific Docker version) |

## When to Use

- Track needs both a native systemd service AND a containerized dependency (e.g., a database container alongside a natively-installed application).
- Track teaches Docker fundamentals (`docker run`, `docker build`, `docker exec`, volumes, networking).
- Track needs the Docker socket for tool integration (CI runners, container scanning tools, image building).
- Only one or two containers are needed -- not enough to justify a `docker-compose.yml`.

## When NOT to Use

- Track needs multiple cooperating containers with service discovery -- use `single-vm-docker-compose.md` instead.
- Track is CLI-only with no Docker or OS-level dependencies -- use `single-container.md` for faster startup.
- Track needs Kubernetes -- use `single-vm-kind-k3s.md` instead.
- Track has no need for VM-level features (systemd, kernel modules) and just runs one container -- use `single-container.md`.

## Common Pitfalls

- **Missing `nested_virtualization: true`**. Docker on Instruqt VMs requires `/dev/kvm`. Without this flag, the Docker daemon will not start.
- **Slow image pulls in setup**. Large Docker images (1+ GB) can take 30-60 seconds to pull. Pre-pull images in a custom Packer image for production tracks (see `modifiers/custom-image-packer.md`), or use the `instruqt/docker-28-3` base which already has Docker ready.
- **Forgetting `allow_external_ingress`**. Without it, service tabs cannot reach container ports mapped to the host. Include `http`, `https`, and `high-ports`.
- **Port mapping confusion**. Containers need `-p HOST_PORT:CONTAINER_PORT` to be reachable from service tabs. The tab URL targets the VM's hostname and the host port.
- **Container lifecycle across challenges**. Containers started in one challenge's setup survive into subsequent challenges unless explicitly stopped. Plan container lifecycle across the full track.
- **Forgetting `--restart unless-stopped`**. If a container should survive VM reboots or persist across challenges, use a restart policy.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Docker fundamentals (pulling images, running containers, building Dockerfiles)
- Application development with a containerized database (PostgreSQL/MySQL container + natively-installed app)
- Container security scanning and image hardening
- CI/CD runner setup (Jenkins or GitLab runner container on a VM)
- Log collection with a containerized agent (Fluentd/Filebeat container collecting host logs)
