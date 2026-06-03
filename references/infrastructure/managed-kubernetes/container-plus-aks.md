# Cloud-Client Container + Azure Subscription + AKS Cluster

Standard pattern for Azure Kubernetes workshops. A cloud-client container provides kubectl, helm, and the Azure CLI. A sandboxed Azure subscription hosts an AKS cluster provisioned during setup.

See `cloud-accounts/container-plus-azure.md` for Azure subscription configuration details.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
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

## Setup Script Pattern

AKS provisioning takes approximately 5-10 minutes, faster than EKS but still significant.

### Basic AKS cluster

```bash
#!/bin/bash

RESOURCE_GROUP="lab-rg"
CLUSTER_NAME="lab-aks"
LOCATION="eastus"

# Create resource group
az group create --name "$RESOURCE_GROUP" --location "$LOCATION"

# Create AKS cluster
az aks create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys \
  --enable-managed-identity \
  --no-wait

# Wait for cluster to be ready
az aks wait \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --created \
  --timeout 600

# Retrieve kubeconfig
az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing

kubectl get nodes
```

### With Azure Container Registry (ACR) integration

```bash
#!/bin/bash

RESOURCE_GROUP="lab-rg"
CLUSTER_NAME="lab-aks"
ACR_NAME="labacr${RANDOM}"
LOCATION="eastus"

az group create --name "$RESOURCE_GROUP" --location "$LOCATION"

# Create ACR
az acr create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACR_NAME" \
  --sku Basic

# Create AKS and attach ACR
az aks create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys \
  --enable-managed-identity \
  --attach-acr "$ACR_NAME"

az aks get-credentials \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --overwrite-existing

kubectl get nodes
```

### Attaching ACR to an existing cluster

```bash
az aks update \
  --resource-group "$RESOURCE_GROUP" \
  --name "$CLUSTER_NAME" \
  --attach-acr "$ACR_NAME"
```

## Cleanup Script

The cleanup script must delete the resource group, which cascades to all resources within it (AKS cluster, ACR, networking, disks).

```bash
#!/bin/bash

az group delete --name "lab-rg" --yes --no-wait
```

## When to Use

- Track teaches AKS-specific features (managed identity, Azure CNI, AKS monitoring, virtual nodes).
- Track teaches tools that integrate with AKS (Azure Application Gateway Ingress Controller, Azure Key Vault CSI driver, Azure Monitor for containers).
- Track teaches Kubernetes on Azure for a customer whose platform is AKS.
- Track needs ACR integration for container image workflows.

## When NOT to Use

- Track teaches generic Kubernetes concepts with no Azure-specific requirements -- use `base-compute/single-vm-kind-k3s.md` instead. It is faster to provision and cheaper to run.
- Track only needs Azure services without Kubernetes -- use `cloud-accounts/container-plus-azure.md` instead.
- Track needs a local development cluster, not a managed cloud cluster -- use `base-compute/single-vm-kind-k3s.md`.

## Common Pitfalls

- **Missing resource providers**. AKS requires `Microsoft.ContainerService` registered on the subscription. Without it, `az aks create` fails. The `resource_providers` list in config.yml ensures these are registered before setup runs. Also register `Microsoft.OperationsManagement` if the track uses monitoring or Log Analytics.
- **Missing `User Access Administrator` role**. The default `Contributor` role cannot assign roles. If the track uses managed identities, workload identity, or ACR integration, the sandbox user needs `User Access Administrator` in addition to `Contributor`.
- **ACR name uniqueness**. ACR names must be globally unique. Append a random suffix (`${RANDOM}` or the sandbox ID) to avoid collisions.
- **Not using `--no-wait` and `az aks wait`**. Running `az aks create` synchronously blocks the shell for 5-10 minutes with no progress output. Use `--no-wait` plus `az aks wait --created` for cleaner behavior and timeout control.
- **Kubeconfig overwrite**. `az aks get-credentials` merges into `~/.kube/config`. In a fresh sandbox this is fine, but use `--overwrite-existing` to avoid merge conflicts if the script runs more than once.
- **Resource group cleanup cascade**. Deleting the resource group deletes everything inside it. This is the correct cleanup pattern for AKS -- do not try to delete individual resources.
- **Node VM size availability**. Not all VM sizes are available in all regions. `Standard_DS2_v2` in `eastus` is a safe default. If the track needs GPU nodes or specific sizes, verify availability in the chosen region.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- AKS cluster provisioning and management
- Azure Container Registry build and deploy pipelines
- AKS workload identity and Azure Key Vault integration
- Azure Application Gateway Ingress Controller setup
- Monitoring AKS with Azure Monitor and Container Insights
- AKS networking deep dive (Azure CNI, kubenet, network policies)
- Deploying microservices to AKS with Helm
