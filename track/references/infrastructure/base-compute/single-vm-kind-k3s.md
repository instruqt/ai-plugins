# Single VM with Kind or k3s

One virtual machine running a Kubernetes cluster via Kind or k3s. The standard pattern for any Kubernetes workshop that does not need a managed cloud cluster (EKS, AKS, GKE).

See `modifiers/kind-k3s-on-vm.md` for Kubernetes setup details, Kind vs k3s trade-offs, and cluster configuration patterns.

## Config.yml Example

### k3s (recommended default)

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/k3s-v1-33-4
    machine_type: n1-standard-4
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

The `instruqt/k3s-v1-33-4` image ships with k3s running and ready -- `kubectl` works immediately in the first challenge with no setup script needed.

### Kind (when you need throwaway or multi-version clusters)

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

Kind requires Docker, so use a Docker-ready image. Create the cluster in setup:

```bash
# Install Kind
curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
chmod +x /usr/local/bin/kind

# Create cluster
kind create cluster --name lab --wait 120s

# kubectl is configured automatically by Kind
kubectl cluster-info
```

## Kind vs k3s

| | Kind | k3s |
|---|------|-----|
| **What it is** | Kubernetes nodes as Docker containers | Lightweight single-binary Kubernetes distribution |
| **Best for** | Throwaway clusters, testing multiple K8s versions, multi-node simulation on one VM | Persistent clusters, production-like single-node, tracks where the cluster should feel "real" |
| **Startup** | 30-60 seconds to create a cluster | Instant with `instruqt/k3s-v1-33-4` pre-baked image |
| **Cluster recreation** | Fast -- `kind delete cluster && kind create cluster` | Slower -- requires reinstalling or resetting k3s |
| **Multi-node** | Simulates multi-node via Docker containers on one VM | Requires actual additional VMs for multi-node |
| **Default choice** | Use when the track creates/destroys clusters as part of the exercise | Use for everything else |

## When to Use

- Track teaches Kubernetes concepts (pods, deployments, services, ingress, RBAC, namespaces).
- Track teaches Helm charts, Kustomize, or Kubernetes operators.
- Track teaches tools that run on Kubernetes (service meshes, GitOps controllers, monitoring stacks).
- Track needs a Kubernetes cluster but not a specific cloud provider's managed offering.

## When NOT to Use

- Track teaches EKS/AKS/GKE-specific features (managed node pools, cloud IAM integration, cloud load balancers) -- use `managed-kubernetes/` patterns with a cloud account instead.
- Track does not need Kubernetes at all -- use a simpler pattern (`single-vm.md`, `single-vm-docker.md`).
- Track needs a multi-node cluster with actual separate hosts -- use `multi-vm.md` with k3s server + agent nodes.

## Common Pitfalls

- **Undersized VM**. Kubernetes has non-trivial base resource requirements. `n1-standard-4` (4 vCPU, 15 GB RAM) is the minimum. Workloads with multiple pods, operators, or monitoring stacks need `n1-standard-8`.
- **Missing `nested_virtualization: true`**. Both Kind (needs Docker) and k3s (on Instruqt VMs) require this flag.
- **Not waiting for cluster readiness**. Even with the pre-baked k3s image, verify readiness in setup before deploying workloads: `until kubectl get nodes | grep -q ' Ready'; do sleep 2; done`.
- **Ingress/NodePort accessibility**. For service tabs to reach Kubernetes services, use NodePort or HostPort mappings. With k3s, the Traefik ingress controller is included by default. With Kind, you need `extraPortMappings` in the Kind config or `kubectl port-forward`.
- **Image pull times inside the cluster**. Pulling container images from within Kubernetes adds another layer of latency. For production tracks, pre-load images into the cluster (Kind: `kind load docker-image`, k3s: `ctr images import`) or use a custom Packer image with images pre-cached (see `modifiers/custom-image-packer.md`).
- **Storage class differences**. k3s includes local-path-provisioner by default. Kind does not provision persistent volumes without extra setup. If the track uses PVCs, verify the storage class exists.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Kubernetes fundamentals (pods, deployments, services, ConfigMaps, Secrets)
- Helm chart development and deployment
- Kubernetes operator development
- Service mesh workshops (Istio, Linkerd, Consul Connect)
- GitOps with Argo CD or Flux
- Kubernetes security (RBAC, network policies, pod security standards)
- Monitoring and observability on Kubernetes (Prometheus, Grafana, OpenTelemetry)
