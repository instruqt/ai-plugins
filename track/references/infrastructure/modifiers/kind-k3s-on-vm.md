# Kubernetes on VM (Kind / k3s)

Kubernetes cluster running on a single VM via Kind or k3s. Use for any Kubernetes workshop that does not require a managed cloud cluster (EKS, AKS, GKE).

## Kind vs k3s

| | Kind | k3s |
|---|------|-----|
| **What it is** | Kubernetes nodes as Docker containers | Lightweight single-binary Kubernetes |
| **Best for** | Throwaway clusters, testing multiple versions, multi-node simulation | Persistent lightweight clusters, production-like single-node |
| **Startup time** | ~60-90s from scratch | ~30s from scratch, instant on pre-built images |
| **Multi-node** | Yes (multiple Docker containers) | Yes (multiple VMs with agent join) |
| **Disk usage** | Heavier (Docker layers per node) | Lighter (single binary + containerd) |
| **Pre-built image** | None -- install in setup | `instruqt/k3s-v1-33-4` (cluster running on boot) |

Use k3s with the Instruqt pre-built image when fast startup matters. Use Kind when the track needs multiple clusters, specific Kubernetes versions, or disposable cluster-per-challenge patterns.

## Setup Example: k3s (pre-built image)

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/k3s-v1-33-4
    machine_type: n1-standard-4
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

# k3s is already running on the pre-built image; wait for readiness
until kubectl get nodes | grep -q ' Ready'; do
  sleep 2
done

# Pre-load container images to avoid pull delays during challenges
ctr images pull docker.io/library/nginx:alpine
ctr images pull docker.io/library/redis:7

# Deploy baseline resources
kubectl apply -f /root/manifests/
```

## Setup Example: Kind (installed in setup)

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
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

# Install Kind
curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-amd64
chmod +x /usr/local/bin/kind

# Install kubectl
curl -Lo /usr/local/bin/kubectl "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x /usr/local/bin/kubectl

# Create cluster
kind create cluster --name workshop --wait 120s

# Pre-load images into the Kind cluster
docker pull nginx:alpine
kind load docker-image nginx:alpine --name workshop
```

## Config.yml Implications

| Field | Recommended |
|-------|-------------|
| `machine_type` | `n1-standard-4` minimum (4 vCPU, 15 GB RAM). Use `n1-standard-8` for multi-node clusters or heavy workloads. |
| `nested_virtualization` | `true` (required -- k3s uses containerd, Kind uses Docker, both need it) |
| `allow_external_ingress` | Include `high-ports` for NodePort services and ingress controllers |
| `provision_ssl_certificate` | `true` if exposing Kubernetes dashboards or ingress via service tabs |

## Common Pitfalls

- **Under-sized VM**. Kubernetes has baseline overhead. `n1-standard-2` will OOM under real workloads. Always use `n1-standard-4` or larger.
- **Cluster not ready when learner starts**. The setup script must wait for the node to reach `Ready` status and for core pods (CoreDNS, kube-proxy) to be running. A bare `kind create cluster` or k3s install returns before the cluster is fully usable.
- **Image pull timeouts during challenges**. Large container images pulled on-demand during a challenge add wait time and can fail on slow networks. Pre-load images in setup: `ctr images pull` for k3s, `kind load docker-image` for Kind.
- **kubectl not configured**. k3s writes kubeconfig to `/etc/rancher/k3s/k3s.yaml` and symlinks kubectl. Kind sets the current context automatically. If the track uses a non-root shell, copy the kubeconfig to the user's home directory.
- **NodePort not reachable from service tabs**. NodePort services listen on high ports (30000-32767). The VM needs `allow_external_ingress: [high-ports]` and the service tab must target the NodePort, not the ClusterIP port.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Kubernetes fundamentals (pods, deployments, services, ingress)
- Helm chart development and testing
- Kubernetes operator workshops
- GitOps with Flux or Argo CD on a local cluster
- Service mesh introduction (Istio, Linkerd on Kind)
- RBAC and network policy exercises
