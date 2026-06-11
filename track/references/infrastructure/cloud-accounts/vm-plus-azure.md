# VM Plus Azure

Full VM workstation paired with a sandboxed Azure subscription. Use when the track needs OS-level features (Docker, systemd, nested virtualization) alongside Azure cloud resources. The Azure environment variables are injected into the VM just as they would be into a container.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ContainerService
      - Microsoft.ContainerRegistry
    regions:
      - eastus
    roles:
      - Contributor
      - AcrPush
    admin_roles:
      - Owner
```

### Variant: Docker-based workflow with ACR

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.ContainerRegistry
      - Microsoft.ContainerService
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ManagedIdentity
      - Microsoft.OperationalInsights
    regions:
      - eastus
    roles:
      - Contributor
      - AcrPush
      - Azure Kubernetes Service Cluster User Role
    admin_roles:
      - Owner
```

### Setup script: install Azure CLI on the VM

```bash
#!/bin/bash

# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Log in with the injected service principal
az login --service-principal \
  --username "$ARM_CLIENT_ID" \
  --password "$ARM_CLIENT_SECRET" \
  --tenant "$ARM_TENANT_ID"

az account set --subscription "$ARM_SUBSCRIPTION_ID"
```

### Setup script: Docker + ACR login

```bash
#!/bin/bash

# Install Docker
apt-get update
apt-get install -y docker.io
systemctl enable --now docker

# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Log in to Azure
az login --service-principal \
  --username "$ARM_CLIENT_ID" \
  --password "$ARM_CLIENT_SECRET" \
  --tenant "$ARM_TENANT_ID"
az account set --subscription "$ARM_SUBSCRIPTION_ID"

# Create and log into ACR (assuming setup creates the registry)
ACR_NAME="lab$(cat /dev/urandom | tr -dc 'a-z0-9' | head -c 8)"
az acr create --resource-group "$RESOURCE_GROUP" --name "$ACR_NAME" --sku Basic
az acr login --name "$ACR_NAME"

# Store ACR name for later challenges
agent variable set ACR_NAME "$ACR_NAME"
```

## Auto-Injected Environment Variables

The same Azure environment variables are injected into the VM as into a container:

| Variable | Description |
|----------|-------------|
| `ARM_CLIENT_ID` | Service principal client ID |
| `ARM_CLIENT_SECRET` | Service principal client secret |
| `ARM_TENANT_ID` | Azure AD tenant ID |
| `ARM_SUBSCRIPTION_ID` | Subscription ID |
| `AZURE_LOCATION` | Default Azure region |

The `ARM_*` variables are the standard names used by Terraform's Azure provider, so Terraform works without extra configuration.

## When to Use

- Track needs Docker on the workstation (building container images, pushing to ACR, running local containers alongside Azure services).
- Track needs systemd services running alongside Azure CLI work.
- Track teaches AKS workflows where the learner builds images locally and pushes to ACR.
- Track needs heavy tooling not available in cloud-client.
- Track needs nested virtualization for complex local environments that also interact with Azure.

## When NOT to Use

- Track only needs CLI/SDK/Terraform access to Azure -- use `container-plus-azure.md` for faster startup and lower resource usage.
- Track needs both a VM (for services) and a lightweight CLI workstation -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **No Azure CLI pre-installed**. VMs are bare OS images. You must install the Azure CLI in the setup script. The `curl -sL https://aka.ms/InstallAzureCLIDeb | bash` one-liner works on Ubuntu but takes 1-2 minutes.
- **Forgetting `az login` in setup**. Even though `ARM_*` variables are set, the Azure CLI does not auto-authenticate. You must explicitly call `az login --service-principal` in every setup script that uses `az` commands.
- **Forgetting resource providers**. The `services:` list must include every `Microsoft.*` provider the track uses. Missing providers cause `The resource provider 'Microsoft.Foo' is not registered` errors at runtime.
- **Long setup times**. Installing Docker + Azure CLI + other dependencies takes 2-4 minutes. Consider a custom Packer image for tracks with heavy dependency stacks.
- **Docker not running**. Always `systemctl enable --now docker` after installation.
- **Resource group discovery**. Instruqt creates a default resource group with a dynamic name. Use `az group list --query '[0].name' -o tsv` to discover it, or create your own in the setup script.
- **Terraform provider authentication**. Terraform picks up `ARM_*` variables automatically. Do not hardcode credentials in provider blocks. If you need a specific provider version, pin it in the Terraform configuration.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Container image build and push to ACR (Docker build + ACR push + AKS deploy)
- AKS local development (build images locally, deploy to AKS cluster)
- Azure DevOps pipeline simulation (local agent deploying to Azure)
- Hybrid application deployment (local Docker Compose app connecting to Azure SQL/Storage)
- Bicep/ARM template development with local testing
- Azure IoT Edge development (local Docker runtime + Azure IoT Hub)
