# Terraform Azure VM (Runtime Provisioned)

Container + Azure subscription + Terraform-provisioned VM in Azure. A cloud-client container runs `terraform apply` during setup to create an Azure VM for workloads that require specific VM sizes (ARM-based `Dps_v5` series, GPU `NC` series), custom images, or Azure-specific networking. The learner accesses the VM via SSH through a `cmd:` terminal tab or via a service tab for web UIs.

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

### Tab configuration

SSH to the provisioned VM:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: VM
    type: terminal
    hostname: shell
    cmd: ssh -o StrictHostKeyChecking=no azureuser@${VM_PUBLIC_IP}
```

## Terraform Configuration

### main.tf

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
  # ARM_CLIENT_ID, ARM_CLIENT_SECRET, ARM_TENANT_ID, ARM_SUBSCRIPTION_ID
  # are auto-injected by Instruqt
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "azurerm_resource_group" "lab" {
  name     = "instruqt-lab-${random_id.suffix.hex}"
  location = "eastus"
}

resource "azurerm_virtual_network" "lab" {
  name                = "lab-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
}

resource "azurerm_subnet" "lab" {
  name                 = "lab-subnet"
  resource_group_name  = azurerm_resource_group.lab.name
  virtual_network_name = azurerm_virtual_network.lab.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "lab" {
  name                = "lab-pip"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_security_group" "lab" {
  name                = "lab-nsg"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name

  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "HTTP"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "8080"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface" "lab" {
  name                = "lab-nic"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.lab.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.lab.id
  }
}

resource "azurerm_network_interface_security_group_association" "lab" {
  network_interface_id      = azurerm_network_interface.lab.id
  network_security_group_id = azurerm_network_security_group.lab.id
}

resource "tls_private_key" "ssh" {
  algorithm = "ED25519"
}

resource "azurerm_linux_virtual_machine" "lab" {
  name                = "instruqt-lab-vm"
  resource_group_name = azurerm_resource_group.lab.name
  location            = azurerm_resource_group.lab.location
  size                = "Standard_D2ps_v5"            # ARM; swap for Standard_NC6s_v3 for GPU
  admin_username      = "azureuser"

  network_interface_ids = [
    azurerm_network_interface.lab.id,
  ]

  admin_ssh_key {
    username   = "azureuser"
    public_key = tls_private_key.ssh.public_key_openssh
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 30
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "ubuntu-24_04-lts"
    sku       = "server-arm64"                        # use "server" for x86_64
    version   = "latest"
  }
}

output "resource_group_name" {
  value = azurerm_resource_group.lab.name
}

output "public_ip" {
  value = azurerm_public_ip.lab.ip_address
}

output "private_key" {
  value     = tls_private_key.ssh.private_key_openssh
  sensitive = true
}
```

## Setup Script (setup-shell)

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Write Terraform files (or use pre-baked files in a custom image)
mkdir -p /root/terraform
cat > /root/terraform/main.tf <<'EOF'
# ... (Terraform config from above)
EOF

cd /root/terraform
terraform init -input=false
terraform apply -input=false -auto-approve

# Extract outputs
RG_NAME=$(terraform output -raw resource_group_name)
PUBLIC_IP=$(terraform output -raw public_ip)
terraform output -raw private_key > /root/.ssh/lab_key
chmod 600 /root/.ssh/lab_key

# Store for cleanup and cross-challenge access
agent variable set RESOURCE_GROUP "$RG_NAME"
agent variable set VM_PUBLIC_IP "$PUBLIC_IP"

# Configure SSH alias
cat > /root/.ssh/config <<SSHEOF
Host lab-vm
  HostName ${PUBLIC_IP}
  User azureuser
  IdentityFile /root/.ssh/lab_key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
SSHEOF

# Wait for SSH to become available (Azure VMs can take 60-120 seconds)
until ssh -o ConnectTimeout=5 lab-vm true 2>/dev/null; do
  echo "Waiting for VM SSH..."
  sleep 5
done
```

## Cleanup Script (cleanup-shell)

```bash
#!/bin/bash
set -euxo pipefail

cd /root/terraform

# Terraform destroy handles all created resources
terraform destroy -input=false -auto-approve || true

# Fallback: delete the entire resource group if Terraform state is corrupted
RG_NAME=$(agent variable get RESOURCE_GROUP 2>/dev/null || true)
if [ -n "$RG_NAME" ]; then
  az group delete --name "$RG_NAME" --yes --no-wait 2>/dev/null || true
fi
```

## When to Use

- Track needs ARM-based Azure VMs (`Dps_v5`, `Dpds_v5` series) not available as Instruqt-native VMs.
- Track needs GPU VMs (`NC`, `ND`, `NV` series) for ML/AI or visualization workloads.
- Track needs a specific Azure Marketplace image or custom managed image.
- Track teaches Azure VM operations where provisioning is the exercise itself.
- Track needs Azure-specific VM features (proximity placement groups, availability zones, Ultra Disks, trusted launch).

## When NOT to Use

- A standard x86 Linux VM is sufficient -- use Instruqt-native `virtualmachines:` instead. It is faster and automatically cleaned up.
- The track only needs Azure CLI access to services (Storage, App Service, AKS) -- use a container + Azure subscription without Terraform.
- The VM size is available in Instruqt's standard machine types -- runtime provisioning adds 1-3 minutes of startup latency.

## Common Pitfalls

- **Resource group not deleted in cleanup**. Azure resources inside a resource group keep incurring costs until the group is deleted. Always include `az group delete` as a fallback in case `terraform destroy` fails.
- **Slow VM provisioning**. Azure VMs can take 2-4 minutes to provision, especially GPU SKUs. The setup script must account for this in the total sandbox startup time budget. Consider showing a progress message to the learner.
- **Azure credential environment variables**. The azurerm provider reads `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, and `ARM_SUBSCRIPTION_ID` automatically. These are injected by Instruqt. Do not hardcode credentials in the Terraform config.
- **VM size not available in region**. Not all VM sizes are available in all Azure regions. ARM VMs (`Dps_v5`) and GPU VMs (`NC` series) have limited regional availability. Check availability before choosing a region.
- **NSG blocks SSH**. Azure VMs require an explicit Network Security Group rule to allow inbound SSH. Forgetting the NSG or the SSH rule means the `cmd:` terminal tab cannot connect.
- **SSH key format**. Azure requires RSA or ED25519 keys. The `tls_private_key` resource generates them correctly, but manually created keys must be in the right format.
- **Terraform state lost between challenges**. The state file at `/root/terraform/terraform.tfstate` persists on the container across challenges. Do not delete it or the cleanup script loses track of created resources.
- **Subscription quota limits**. Azure sandbox subscriptions may have low vCPU quotas, especially for GPU SKUs. The `terraform apply` fails with an opaque quota error. Use smaller VM sizes and common regions (`eastus`, `westus2`).

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- ARM architecture migration on Azure (deploying to Ampere-based `Dps_v5` VMs)
- GPU-accelerated ML training lab (provisioning `NC` series with NVIDIA drivers)
- Azure VM operations deep-dive (managed disks, extensions, availability sets)
- Custom image deployment workshop (launching from a shared image gallery)
- Azure networking lab (VNet peering, NSGs, and load balancers with Terraform-provisioned VMs)
