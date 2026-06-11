# VM with Kind Cluster + Azure Subscription + AKS Cluster

Multi-cluster pattern combining a local Kind cluster on a VM with a remote AKS cluster in a sandboxed Azure subscription. Both clusters are accessible from the VM via separate kubeconfig contexts.

See `cloud-accounts/vm-plus-azure.md` for Azure subscription configuration details. See `base-compute/single-vm-kind-k3s.md` for Kind setup details.

## Config.yml Example

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
azure_subscriptions:
  - name: sandbox
    managed: true
    roles:
      - Contributor
      - "User Access Administrator"
    resource_providers:
      - Microsoft.ContainerService
      - Microsoft.OperationsManagement
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ContainerRegistry
      - Microsoft.ManagedIdentity
```

The VM needs `nested_virtualization: true` and a Docker-ready image for Kind. The Azure CLI and kubectl must be installed on the VM in setup if not on the base image.

## Setup Script Pattern

Kind creates in ~30-60 seconds, AKS in ~5-10 minutes. Start AKS first with `--no-wait`, create Kind in parallel, then wait for AKS readiness.

```bash
#!/bin/bash

RESOURCE_GROUP="lab-rg"
CLUSTER_NAME="production"
LOCATION="eastus"

# Install tools on the VM
curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
chmod +x /usr/local/bin/kind

# Install Azure CLI if not on the base image
if ! command -v az &>/dev/null; then
  curl -sL https://aka.ms/InstallAzureCLIDeb | bash
fi

# Create resource group and start AKS provisioning
az group create --name "$RESOURCE_GROUP" --location "$LOCATION"

az aks create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys \
  --enable-managed-identity \
  --no-wait

# Create local Kind cluster while AKS provisions
kind create cluster --name local --wait 120s

# Rename the Kind context for clarity
kubectl config rename-context kind-local local

# Wait for AKS to finish
az aks wait \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --created \
  --timeout 600

# Retrieve AKS kubeconfig (merges into ~/.kube/config)
az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing

# Rename the AKS context for clarity
kubectl config rename-context "$CLUSTER_NAME" production

# Verify both clusters
kubectl --context local get nodes
kubectl --context production get nodes
```

## Dual Kubeconfig Management

Both clusters share `~/.kube/config` with separate contexts. Teach learners to switch between them:

```bash
# List available contexts
kubectl config get-contexts

# Switch to the local Kind cluster
kubectl config use-context local

# Switch to the remote AKS cluster
kubectl config use-context production

# Run a command against a specific context without switching
kubectl --context production get pods -A
```

## Cleanup Script

```bash
#!/bin/bash

# Delete the Azure resource group (cascades to AKS and all resources)
az group delete --name "lab-rg" --yes --no-wait

# Delete Kind cluster
kind delete cluster --name local
```

## When to Use

- Track teaches multi-cluster scenarios (federation, multi-cluster service mesh, failover) on Azure.
- Track teaches GitOps with a local development cluster and a remote AKS production cluster.
- Track teaches tools that bridge local and cloud clusters (Loft vCluster, Telepresence, Skupper) in an Azure environment.
- Track teaches service mesh across clusters where one cluster is AKS.
- Track compares local Kubernetes development with Azure cloud deployment.

## When NOT to Use

- Track only needs one Kubernetes cluster -- use `container-plus-aks.md` for AKS or `base-compute/single-vm-kind-k3s.md` for local-only.
- Track does not need a managed cloud cluster -- use `base-compute/single-vm-kind-k3s.md` with multi-node Kind config instead.
- Track needs AWS as the cloud provider -- use `vm-kind-plus-eks.md` instead.
- Track needs multiple AKS clusters -- use `container-plus-aks.md` and provision multiple clusters in setup.

## Common Pitfalls

- **Resource requirements**. The VM runs Kind locally while also managing connections to AKS. Use at least `n1-standard-4`. Bump to `n1-standard-8` if Kind runs multiple nodes or heavy workloads.
- **Azure credentials on the VM**. Instruqt injects Azure credentials into the sandbox environment. Verify they are available on the VM. The Azure CLI should be able to run `az account show` without additional login steps.
- **Tool installation**. Unlike the cloud-client container, the VM base image does not include the Azure CLI, kubectl, or helm. Install everything needed in setup.
- **Missing resource providers**. AKS requires `Microsoft.ContainerService` registered on the subscription. The `resource_providers` list in config.yml handles this, but if the track dynamically creates additional Azure resources, ensure their providers are also listed.
- **Context name confusion**. Default context names from Kind (`kind-local`) and AKS (the cluster name) are inconsistent. Rename them in setup to clear names like `local` and `production`.
- **Kind networking is VM-local**. Kind clusters use Docker networking internal to the VM. Services in Kind are not directly reachable from AKS. Cross-cluster communication requires a service mesh, `kubectl port-forward`, or an explicit networking bridge.
- **Resource group cleanup**. Deleting the resource group cascades to all Azure resources. This is the correct and simplest cleanup pattern. Do not attempt to delete individual AKS resources.
- **AKS `--no-wait` behavior**. Unlike EKS with a backgrounded process, `az aks create --no-wait` returns immediately and AKS provisions asynchronously. Use `az aks wait --created` to block until the cluster is ready before retrieving credentials.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Multi-cluster service mesh with Linkerd (local + AKS)
- GitOps workflow: develop locally, deploy to AKS with Flux
- Kubernetes federation across local and cloud clusters on Azure
- Telepresence: bridge local development to a remote AKS cluster
- Multi-cluster observability with local Kind and Azure Monitor
