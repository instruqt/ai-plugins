# Container Plus VM

Hybrid pattern: a lightweight Instruqt container as the learner's CLI workstation, paired with one or more VMs running heavier services. The container is where the learner types commands; the VM runs databases, application servers, Docker, Kubernetes, or anything requiring systemd or kernel features. Saves resources compared to using a VM for the workstation.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
    environment:
      DB_HOST: server
      KUBECONFIG: /root/.kube/config

virtualmachines:
  - name: server
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      ROLE: server
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

### Tab configuration

Terminal on the container, service tab on the VM:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Dashboard
    type: service
    hostname: server
    port: 8443
    path: /
```

### Variant: cloud-client workstation + Kubernetes VM

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
    environment:
      KUBECONFIG: /root/.kube/config

virtualmachines:
  - name: k8s
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

**setup-k8s** (install k3s on the VM):

```bash
#!/bin/bash

curl -sfL https://get.k3s.io | sh -

# Wait for k3s to be ready
until kubectl get nodes 2>/dev/null | grep -q " Ready"; do
  sleep 2
done

# Export the kubeconfig for the workstation to pick up
agent variable set KUBECONFIG_B64 "$(cat /etc/rancher/k3s/k3s.yaml | sed 's/127.0.0.1/k8s/g' | base64 -w 0)"
```

**setup-workstation** (retrieve kubeconfig from the VM):

```bash
#!/bin/bash

# Wait for the VM to publish the kubeconfig
until agent variable get KUBECONFIG_B64 2>/dev/null | grep -q .; do
  echo "Waiting for Kubernetes cluster..."
  sleep 3
done

agent variable get KUBECONFIG_B64 | base64 -d > /root/.kube/config
chmod 600 /root/.kube/config

# Verify connectivity
until kubectl get nodes 2>/dev/null | grep -q " Ready"; do
  sleep 2
done
```

### Variant: workstation + database VM

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 256
    shell: /bin/bash
    entrypoint: bash
    environment:
      PGHOST: database
      PGUSER: postgres
      PGPASSWORD: instruqt
      PGDATABASE: app

virtualmachines:
  - name: database
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
```

## Cross-Host Communication

Containers and VMs in the same sandbox share a private network. Each host's `name` is its hostname:

- From `workstation` (container): `curl http://server:8080` reaches the VM.
- From `server` (VM): `curl http://workstation:8080` reaches the container.
- No special network configuration needed.

### Passing dynamic values between hosts

Static values (hostnames, roles) go in the `environment:` block. For dynamic values generated at runtime (tokens, kubeconfigs, generated passwords), use `agent variable`:

```bash
# On the VM (setup-server):
agent variable set JOIN_TOKEN "$(cat /var/lib/rancher/k3s/server/node-token)"

# On the container (setup-workstation):
TOKEN=$(agent variable get JOIN_TOKEN)
```

The workstation setup must poll until the variable is available, since both hosts start concurrently.

## When to Use

- The learner only needs CLI access (no GUI, no Docker on the workstation) but the services require VM features (systemd, Docker, nested virtualization, kernel modules).
- Track teaches Kubernetes, Docker, or container orchestration where the learner uses `kubectl`/`docker` from a CLI workstation pointed at a remote cluster/daemon.
- Track teaches database administration where the DB runs as a systemd service on the VM and the learner connects via CLI tools from the container.
- Saving resources: a container workstation uses ~256-512 MB vs 2+ GB for a VM workstation.
- Track needs cloud CLIs (`gcr.io/instruqt/cloud-client:2`) alongside a VM running the target service.

## When NOT to Use

- The learner needs Docker or systemd on the workstation itself -- use `multi-vm.md` with a dedicated workstation VM.
- The learner needs a GUI desktop -- containers cannot run display servers.
- Both hosts are lightweight and CLI-only -- use `multi-container-multi-tier.md` instead (faster startup, lower cost).
- The learner needs to SSH between hosts as part of the exercise -- while possible, it adds complexity. Consider whether `multi-vm.md` is cleaner for SSH-focused tracks.

## Common Pitfalls

- **Setup script race conditions**. The container and VM start setup scripts concurrently. The container's setup will almost always finish first (containers boot in seconds, VMs take longer). The container must poll for the VM's readiness: `until nc -z server 6443; do sleep 2; done`.
- **`agent variable` timing**. If the container's setup calls `agent variable get` before the VM's setup has called `agent variable set`, it returns empty. Always wrap in a poll loop.
- **kubeconfig server address**. When copying a kubeconfig from a VM to a container, replace `127.0.0.1` or `localhost` with the VM's hostname. Otherwise `kubectl` on the container tries to connect to itself.
- **Missing `ports:` on the container**. If the container runs a web server for a service tab, the port must be listed. Containers do not expose all ports by default.
- **Tab targeting**. Terminal tabs should target the container (where the learner types). Service tabs for web UIs should target the VM (where the service runs). Mixing these up is a common mistake.
- **Cloud CLI availability**. `gcr.io/instruqt/shell` does not include cloud CLIs. If the track uses `kubectl`, `aws`, `gcloud`, or `terraform`, use `gcr.io/instruqt/cloud-client:2` for the workstation container.
- **VM startup time**. VMs take 30-90 seconds to boot. The first challenge's setup script on the VM should install dependencies and start services. Consider custom Packer images for heavy dependency stacks.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Kubernetes operations (cloud-client workstation + k3s/kind VM)
- Docker fundamentals (shell workstation + VM with Docker daemon)
- Database administration (shell workstation + VM running PostgreSQL/MySQL as systemd service)
- Infrastructure as Code (cloud-client workstation + VM as deployment target)
- CI/CD pipeline configuration (workstation with CLI tools + VM running Jenkins/GitLab Runner)
