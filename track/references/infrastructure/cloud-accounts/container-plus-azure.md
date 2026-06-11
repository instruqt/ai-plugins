# Container Plus Azure

Cloud-client container paired with a sandboxed Azure subscription. The learner uses Azure CLI, SDK, or Terraform from a lightweight container while Instruqt manages the subscription lifecycle (creation, service principal injection, cleanup).

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

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
    regions:
      - eastus
    roles:
      - Contributor
    admin_roles:
      - Owner
```

### Variant: AKS track with container registry

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ContainerService
      - Microsoft.ContainerRegistry
      - Microsoft.OperationalInsights
      - Microsoft.OperationsManagement
      - Microsoft.ManagedIdentity
    regions:
      - eastus
    roles:
      - Contributor
      - AcrPush
      - Azure Kubernetes Service Cluster User Role
    admin_roles:
      - Owner
```

### Variant: narrow learner with read-only access

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
    regions:
      - eastus
    roles:
      - Reader
    admin_roles:
      - Owner
```

## Auto-Injected Environment Variables

These variables are injected into ALL sandbox hosts (containers and VMs alike):

| Variable | Description |
|----------|-------------|
| `ARM_CLIENT_ID` | Service principal client ID |
| `ARM_CLIENT_SECRET` | Service principal client secret |
| `ARM_TENANT_ID` | Azure AD tenant ID |
| `ARM_SUBSCRIPTION_ID` | Subscription ID |
| `AZURE_LOCATION` | Default Azure region (first from `regions:` list) |

The `ARM_*` variables are the standard names used by Terraform's Azure provider, so Terraform works without any provider configuration.

### Azure CLI authentication in setup scripts

The cloud-client image includes the `az` CLI. Authenticate using the injected service principal:

```bash
#!/bin/bash

az login --service-principal \
  --username "$ARM_CLIENT_ID" \
  --password "$ARM_CLIENT_SECRET" \
  --tenant "$ARM_TENANT_ID"

az account set --subscription "$ARM_SUBSCRIPTION_ID"
```

## Resource Provider Registration

Azure requires explicit registration of resource providers before you can create resources of that type. The `services:` list in config.yml registers these providers when the sandbox starts.

You MUST list every resource provider namespace your track uses. Common providers:

| Provider | What it covers |
|----------|---------------|
| `Microsoft.Compute` | VMs, disks, availability sets |
| `Microsoft.Network` | VNets, subnets, NSGs, load balancers, public IPs |
| `Microsoft.Storage` | Storage accounts, blobs, files |
| `Microsoft.ContainerService` | AKS clusters |
| `Microsoft.ContainerRegistry` | ACR registries |
| `Microsoft.Sql` | Azure SQL databases |
| `Microsoft.DBforPostgreSQL` | PostgreSQL flexible servers |
| `Microsoft.Web` | App Service, Function Apps |
| `Microsoft.KeyVault` | Key Vault |
| `Microsoft.ManagedIdentity` | User-assigned managed identities |
| `Microsoft.OperationalInsights` | Log Analytics workspaces |
| `Microsoft.OperationsManagement` | Monitoring solutions |
| `Microsoft.Authorization` | Role assignments, policy definitions |

## Azure Role Assignments

- `roles:` -- assigned to the learner service principal. Use narrow roles (e.g., `Reader`, `Contributor` scoped to specific resource types).
- `admin_roles:` -- assigned to the admin service principal used by lifecycle scripts. Typically `Owner` for full control.

Role names must match Azure's built-in role names exactly (case-sensitive). Custom role definitions are not supported in config.yml.

## When to Use

- Track teaches Azure services via CLI, SDK, or Terraform and the learner does not need Docker, systemd, or kernel features on the workstation.
- Track provisions Azure infrastructure (VMs, AKS, storage accounts, databases) as exercises.
- Track uses Terraform or Bicep to deploy Azure resources.
- Track teaches Azure security, networking, or identity topics.

## When NOT to Use

- Track needs Docker on the workstation (building images, running local containers) -- use `vm-plus-azure.md`.
- Track needs systemd or kernel features on the workstation -- use `vm-plus-azure.md`.
- Track needs both local infrastructure (VM with services) and Azure resources -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **Forgetting to list resource providers**. This is the single most common Azure sandbox failure. If a resource provider is not listed under `services:`, any `az` command or Terraform resource that uses it fails with: `The resource provider 'Microsoft.Foo' is not registered`. The error looks like a permissions problem but is actually a registration problem.
- **Incomplete provider chains**. Some Azure services depend on multiple providers. For example, AKS needs `Microsoft.ContainerService`, `Microsoft.Network`, `Microsoft.Compute`, and often `Microsoft.ManagedIdentity`. Missing any one of them causes failures partway through deployment.
- **Not authenticating in setup scripts**. Unlike AWS where cloud-client sets up credentials automatically, Azure requires an explicit `az login` call in setup scripts. Without it, `az` commands fail.
- **Role name typos**. Azure role names are case-sensitive and must match exactly. `contributor` does not work -- it must be `Contributor`. Check the exact name with `az role definition list`.
- **Resource group creation**. Instruqt creates a default resource group, but its name is dynamic. Use `az group list` in setup to discover it, or create your own resource group in the setup script.
- **Region availability**. Not all Azure services are available in all regions. If the track uses a specialized service (e.g., certain AI/ML services), verify it is available in the region listed in `regions:`.
- **Subscription-level quotas**. Sandboxed subscriptions have default quotas. Deploying many VMs or large SKUs may hit quota limits. Keep resource requirements modest.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Azure VM deployment (create VMs, configure NSGs, attach disks)
- AKS cluster operations (create cluster, deploy workloads, scale)
- Azure Storage fundamentals (blob operations, access tiers, lifecycle policies)
- Terraform on Azure (provision resource groups, VNets, VMs)
- Azure networking (VNet peering, load balancers, DNS zones)
- Azure security (Key Vault, managed identities, RBAC)
