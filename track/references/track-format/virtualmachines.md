# Virtual Machines

Virtual machines defined in `config.yml` under the `virtualmachines` key. Each VM becomes a hostname accessible within the sandbox and can host terminal tabs, services, and graphical applications.

## Structure

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      ROLE: primary
      K3S_CONTROL_PLANE_HOSTNAME: workstation
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

## Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Unique name, becomes hostname |
| `image` | string | required | GCP image family or custom image path |
| `machine_type` | string | — | GCP machine-type string (preferred over memory/cpus) |
| `memory` | int (MB) | — | Alternative to machine_type |
| `cpus` | int | — | Alternative to machine_type |
| `shell` | string | — | Default shell for terminal tabs |
| `environment` | map | — | Key-value environment variables |
| `allow_external_ingress` | list | — | Network access types: `http`, `https`, `high-ports` |
| `nested_virtualization` | bool | false | Required for Docker/Kind running on the VM |
| `provision_ssl_certificate` | bool | false | Provision HTTPS certificate for the VM |

## Machine types

`machine_type` is the preferred way to size a VM. It takes a GCP machine-type string.

### Available machine types

| Family | Types |
|--------|-------|
| **n1-standard** | n1-standard-1, n1-standard-2, n1-standard-4, n1-standard-8 |
| **n1-highcpu** | n1-highcpu-8, n1-highcpu-16 |
| **n1-highmem** | n1-highmem-8 |
| **n2-standard** | n2-standard-2, n2-standard-4, n2-standard-8, n2-standard-16 |
| **n2-highcpu** | n2-highcpu-2, n2-highcpu-32 |
| **n2-highmem** | n2-highmem-16 |
| **n2d** | n2d-highcpu-32, n2d-standard-8 |
| **c2** | c2-standard-8 |
| **e2** | e2-small, e2-micro, e2-standard-2, e2-standard-4, e2-standard-8, e2-standard-16, e2-highcpu-16 |
| **f1/g1** | f1-micro, g1-small |
| **t2a (Arm64)** | t2a-standard-1 |

Use `machine_type` instead of `memory`/`cpus` fields. The latter are a legacy alternative.

## Images

### Stock cloud images

| Image | Description |
|-------|-------------|
| `ubuntu-os-cloud/ubuntu-2204-lts` | Ubuntu 22.04 LTS |
| `ubuntu-os-cloud/ubuntu-2404-lts-amd64` | Ubuntu 24.04 LTS |
| `centos-cloud/centos-stream-9` | CentOS Stream 9 |

### Instruqt-published images

| Image | Description |
|-------|-------------|
| `instruqt/k3s-*` | K3s Kubernetes images |
| `instruqt/docker-*` | Docker-enabled images |
| `instruqt/windows-server` | Windows Server |

### Image naming conventions

Stock cloud images use two naming conventions:

- **Family slug** (recommended): `ubuntu-os-cloud/ubuntu-2204-lts` — always resolves to the latest image in the family. Updates automatically when the cloud provider publishes a new image.
- **Pinned date format**: `ubuntu-os-cloud/ubuntu-2204-lts-v20240101` — pins to a specific image version. Use when reproducibility matters more than getting security patches.

Prefer the family slug for most tracks. Pin to a specific date only when a known image regression needs to be avoided.

### Custom images

Custom images from vendor GCP projects can be referenced by their full project/image path.

## Shell

The `shell` field controls the default shell for all terminal tabs connected to this VM.

To drop every terminal tab into a non-root user session:

```yaml
shell: su - student
```

This runs `su - student` as the shell command, giving learners a non-root experience in every terminal.

## Environment variables

The `environment` block sets environment variables available in lifecycle scripts and terminal sessions. Useful for cross-VM coordination:

```yaml
virtualmachines:
  - name: control-plane
    image: instruqt/k3s-control-plane
    machine_type: n1-standard-2
    environment:
      K3S_CONTROL_PLANE_HOSTNAME: control-plane

  - name: worker
    image: instruqt/k3s-worker
    machine_type: n1-standard-2
    environment:
      K3S_CONTROL_PLANE_HOSTNAME: control-plane
```

The `K3S_CONTROL_PLANE_HOSTNAME` env var enables worker nodes to auto-join the control plane.

## External ingress

`allow_external_ingress` controls what network access is allowed from outside the sandbox:

| Value | Description |
|-------|-------------|
| `http` | Allow HTTP (port 80) traffic |
| `https` | Allow HTTPS (port 443) traffic |
| `high-ports` | Allow traffic on high ports (1024+) |

Required when learners need to access web applications running on the VM via service or website tabs.

## Nested virtualization

Set `nested_virtualization: true` when the VM needs to run its own containers or VMs:
- Docker / Docker Compose
- Kind (Kubernetes in Docker)
- Minikube with VM driver
- Any workload requiring `/dev/kvm`

## SSL certificates

Set `provision_ssl_certificate: true` to get an HTTPS certificate provisioned for the VM. Used with service tabs or website tabs that require HTTPS.

## Windows VMs

When using Windows VM images (`instruqt/windows-server`):
- Lifecycle scripts (setup, check, solve, cleanup) are PowerShell, not Bash
- Terminal tabs open a native PowerShell session
- File paths use Windows conventions

### Windows via nested Docker (alternative)

For tracks that need a Windows environment but cannot use a Windows VM image, run a Windows Docker container inside a Linux VM and expose it via Guacamole RDP:

```yaml
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-4
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

The setup script pulls a Windows container image and runs it with Docker, then configures Guacamole to connect to its RDP port. This is heavier than a native Windows VM but avoids Windows image licensing constraints.

### Editor limitations

> [!NOTE]
> **Cursor (the editor)** has no viable self-hosted web mode. It cannot be run as a browser-based IDE like code-server or VS Code. If a track needs a browser-based code editor, use code-server instead.

## Examples

### Basic Linux VM

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: /bin/bash
```

### K3s cluster with control plane and worker

```yaml
version: "3"
virtualmachines:
  - name: control-plane
    image: instruqt/k3s-control-plane
    machine_type: n1-standard-4
    shell: /bin/bash
    environment:
      K3S_CONTROL_PLANE_HOSTNAME: control-plane
    allow_external_ingress:
      - http
      - https
    nested_virtualization: true

  - name: worker-1
    image: instruqt/k3s-worker
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      K3S_CONTROL_PLANE_HOSTNAME: control-plane
    nested_virtualization: true
```

### VM with Docker and web app

```yaml
version: "3"
virtualmachines:
  - name: docker-host
    image: instruqt/docker-ubuntu-2204
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
    environment:
      APP_PORT: "8080"
```

### Non-root user experience

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: su - developer
    environment:
      HOME: /home/developer
```
