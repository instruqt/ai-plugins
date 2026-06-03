# Terraform GCP VM (Runtime Provisioned)

Container + GCP project + Terraform or gcloud-provisioned VM in GCP. A cloud-client container provisions a Compute Engine instance during setup for workloads requiring GPU attachment, custom images from other projects, specific machine series, or IAP-tunneled SSH access to instances without public IPs. Covers both the Terraform approach and direct `gcloud compute` commands.

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

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - iap.googleapis.com                             # required for IAP tunnel SSH
    roles:
      - roles/viewer
    admin_roles:
      - roles/owner
    regions:
      - us-central1
```

### Tab configuration

SSH to the provisioned VM (public IP):

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: VM
    type: terminal
    hostname: shell
    cmd: ssh -o StrictHostKeyChecking=no root@${VM_PUBLIC_IP}
```

SSH via IAP tunnel (no public IP on the VM):

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: VM
    type: terminal
    hostname: shell
    cmd: gcloud compute ssh lab-vm --zone=us-central1-a --tunnel-through-iap --project=${INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID}
```

## Approach 1: Terraform

### main.tf

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 6.0"
    }
  }
}

variable "project_id" {
  type = string
}

provider "google" {
  project = var.project_id
  region  = "us-central1"
  zone    = "us-central1-a"
}

resource "tls_private_key" "ssh" {
  algorithm = "ED25519"
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "google_compute_firewall" "lab_ssh" {
  name    = "instruqt-lab-ssh-${random_id.suffix.hex}"
  network = "default"

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["instruqt-lab"]
}

resource "google_compute_firewall" "lab_http" {
  name    = "instruqt-lab-http-${random_id.suffix.hex}"
  network = "default"

  allow {
    protocol = "tcp"
    ports    = ["8080"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["instruqt-lab"]
}

resource "google_compute_instance" "lab" {
  name         = "instruqt-lab-${random_id.suffix.hex}"
  machine_type = "t2a-standard-1"                     # ARM (Tau T2A); swap for n1-standard-4 + GPU
  zone         = "us-central1-a"

  tags = ["instruqt-lab"]

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2404-lts-arm64"  # use ubuntu-2404-lts-amd64 for x86
      size  = 30
    }
  }

  network_interface {
    network = "default"
    access_config {}                                   # gives a public IP; omit for IAP-only
  }

  metadata = {
    ssh-keys = "root:${tls_private_key.ssh.public_key_openssh}"
  }
}

output "instance_name" {
  value = google_compute_instance.lab.name
}

output "public_ip" {
  value = google_compute_instance.lab.network_interface[0].access_config[0].nat_ip
}

output "private_key" {
  value     = tls_private_key.ssh.private_key_openssh
  sensitive = true
}
```

### GPU variant

Replace the instance resource for GPU workloads:

```hcl
resource "google_compute_instance" "lab" {
  name         = "instruqt-lab-${random_id.suffix.hex}"
  machine_type = "n1-standard-4"
  zone         = "us-central1-c"                       # GPU availability varies by zone

  tags = ["instruqt-lab"]

  boot_disk {
    initialize_params {
      image = "deeplearning-platform-release/common-cu124-v20250101"
      size  = 100
    }
  }

  guest_accelerator {
    type  = "nvidia-tesla-t4"
    count = 1
  }

  scheduling {
    on_host_maintenance = "TERMINATE"                  # required for GPU instances
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata = {
    ssh-keys = "root:${tls_private_key.ssh.public_key_openssh}"
  }
}
```

### IAP-only variant (no public IP)

Omit `access_config` so the VM has no external IP. SSH access goes through IAP:

```hcl
resource "google_compute_instance" "lab" {
  name         = "instruqt-lab-${random_id.suffix.hex}"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

  tags = ["instruqt-lab"]

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2404-lts-amd64"
      size  = 30
    }
  }

  network_interface {
    network = "default"
    # No access_config -- no public IP
  }

  metadata = {
    ssh-keys = "root:${tls_private_key.ssh.public_key_openssh}"
  }
}
```

### Setup Script -- Terraform (setup-shell)

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

PROJECT_ID=$(agent variable get INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID)

# Activate admin service account for Terraform
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/admin-sa.json
gcloud auth activate-service-account --key-file=/tmp/admin-sa.json
export GOOGLE_APPLICATION_CREDENTIALS=/tmp/admin-sa.json

# Ensure default network exists (some sandbox projects may lack it)
gcloud compute networks describe default --project="$PROJECT_ID" 2>/dev/null || \
  gcloud compute networks create default --subnet-mode=auto --project="$PROJECT_ID"

mkdir -p /root/terraform
cat > /root/terraform/main.tf <<'EOF'
# ... (Terraform config from above)
EOF

cd /root/terraform
terraform init -input=false
terraform apply -input=false -auto-approve -var="project_id=${PROJECT_ID}"

# Extract outputs
INSTANCE_NAME=$(terraform output -raw instance_name)
PUBLIC_IP=$(terraform output -raw public_ip)
terraform output -raw private_key > /root/.ssh/lab_key
chmod 600 /root/.ssh/lab_key

# Store for cleanup and cross-challenge access
agent variable set INSTANCE_NAME "$INSTANCE_NAME"
agent variable set VM_PUBLIC_IP "$PUBLIC_IP"

# Configure SSH alias
cat > /root/.ssh/config <<SSHEOF
Host lab-vm
  HostName ${PUBLIC_IP}
  User root
  IdentityFile /root/.ssh/lab_key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
SSHEOF

# Wait for SSH
until ssh -o ConnectTimeout=5 lab-vm true 2>/dev/null; do
  echo "Waiting for VM SSH..."
  sleep 5
done
```

### Cleanup Script -- Terraform (cleanup-shell)

```bash
#!/bin/bash
set -euxo pipefail

PROJECT_ID=$(agent variable get INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID 2>/dev/null || true)

# Re-activate admin credentials for cleanup
if [ -f /tmp/admin-sa.json ]; then
  export GOOGLE_APPLICATION_CREDENTIALS=/tmp/admin-sa.json
fi

cd /root/terraform

terraform destroy -input=false -auto-approve -var="project_id=${PROJECT_ID}" || true

# Fallback: delete the instance directly if Terraform state is corrupted
INSTANCE_NAME=$(agent variable get INSTANCE_NAME 2>/dev/null || true)
if [ -n "$INSTANCE_NAME" ]; then
  gcloud compute instances delete "$INSTANCE_NAME" \
    --zone=us-central1-a \
    --project="$PROJECT_ID" \
    --quiet 2>/dev/null || true
fi
```

## Approach 2: gcloud CLI

For simpler cases where a single VM is needed without complex networking, `gcloud compute` commands are more concise than Terraform.

### Setup Script -- gcloud (setup-shell)

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

PROJECT_ID=$(agent variable get INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID)

# Activate admin service account
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/admin-sa.json
gcloud auth activate-service-account --key-file=/tmp/admin-sa.json
gcloud config set project "$PROJECT_ID"

# Generate SSH key
ssh-keygen -t ed25519 -f /root/.ssh/lab_key -N "" -q
echo "root:$(cat /root/.ssh/lab_key.pub)" > /tmp/ssh-metadata.txt

# Create the instance
INSTANCE_NAME="instruqt-lab-$(head -c 4 /dev/urandom | xxd -p)"

gcloud compute instances create "$INSTANCE_NAME" \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=30GB \
  --metadata-from-file=ssh-keys=/tmp/ssh-metadata.txt \
  --tags=instruqt-lab \
  --quiet

# Open SSH port
gcloud compute firewall-rules create instruqt-lab-ssh \
  --allow=tcp:22 \
  --target-tags=instruqt-lab \
  --quiet 2>/dev/null || true

# Get the public IP
PUBLIC_IP=$(gcloud compute instances describe "$INSTANCE_NAME" \
  --zone=us-central1-a \
  --format='get(networkInterfaces[0].accessConfigs[0].natIP)')

agent variable set INSTANCE_NAME "$INSTANCE_NAME"
agent variable set VM_PUBLIC_IP "$PUBLIC_IP"

# Configure SSH alias
cat > /root/.ssh/config <<SSHEOF
Host lab-vm
  HostName ${PUBLIC_IP}
  User root
  IdentityFile /root/.ssh/lab_key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
SSHEOF

# Wait for SSH
until ssh -o ConnectTimeout=5 lab-vm true 2>/dev/null; do
  echo "Waiting for VM SSH..."
  sleep 5
done
```

### Setup Script -- gcloud with IAP tunnel (no public IP)

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

PROJECT_ID=$(agent variable get INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID)

echo "$INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/admin-sa.json
gcloud auth activate-service-account --key-file=/tmp/admin-sa.json
gcloud config set project "$PROJECT_ID"

INSTANCE_NAME="instruqt-lab-$(head -c 4 /dev/urandom | xxd -p)"

# Create instance without external IP
gcloud compute instances create "$INSTANCE_NAME" \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=30GB \
  --no-address \
  --quiet

# Allow IAP tunnel traffic
gcloud compute firewall-rules create instruqt-lab-iap \
  --allow=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --quiet 2>/dev/null || true

agent variable set INSTANCE_NAME "$INSTANCE_NAME"

# Wait for instance to be reachable via IAP
until gcloud compute ssh "$INSTANCE_NAME" \
  --zone=us-central1-a \
  --tunnel-through-iap \
  --command="true" \
  --quiet 2>/dev/null; do
  echo "Waiting for IAP tunnel..."
  sleep 10
done
```

### Cleanup Script -- gcloud (cleanup-shell)

```bash
#!/bin/bash
set -euxo pipefail

PROJECT_ID=$(agent variable get INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID 2>/dev/null || true)
INSTANCE_NAME=$(agent variable get INSTANCE_NAME 2>/dev/null || true)

if [ -f /tmp/admin-sa.json ]; then
  gcloud auth activate-service-account --key-file=/tmp/admin-sa.json
  gcloud config set project "$PROJECT_ID"
fi

if [ -n "$INSTANCE_NAME" ]; then
  gcloud compute instances delete "$INSTANCE_NAME" \
    --zone=us-central1-a \
    --quiet 2>/dev/null || true
fi

# Clean up firewall rules
gcloud compute firewall-rules delete instruqt-lab-ssh --quiet 2>/dev/null || true
gcloud compute firewall-rules delete instruqt-lab-iap --quiet 2>/dev/null || true
```

## When to Use

- Track needs GPU instances (NVIDIA T4, V100, A100) -- GCP attaches GPUs to standard machine types via `guest_accelerator`.
- Track needs ARM VMs (Tau T2A series) not available as Instruqt-native VMs.
- Track needs custom images from another GCP project (shared image galleries, Deep Learning VM images).
- Track needs IAP-tunneled SSH for exercises about private networking or zero-trust access.
- Track teaches GCP Compute Engine operations where provisioning is the exercise itself.
- Simple single-VM provisioning where `gcloud compute` is faster to write than Terraform.

## When NOT to Use

- A standard x86 Linux VM is sufficient -- use Instruqt-native `virtualmachines:` instead. It is faster, cheaper, and automatically cleaned up.
- The track only needs GCP API access (GKE, Cloud Run, BigQuery) -- use a container + GCP project without Terraform.
- The VM image and machine type are standard -- runtime provisioning adds 1-3 minutes of startup latency for no benefit.

## Common Pitfalls

- **Admin service account not activated**. The default Instruqt-injected GCP credentials use the learner role. Terraform and `gcloud compute instances create` need admin-level credentials. Always activate the admin service account key before provisioning.
- **No default network**. Some GCP sandbox projects lack a default VPC network. Creating instances that reference `network = "default"` will fail. Check for the default network and create it if missing.
- **GPU quota exhausted**. GPU quotas in GCP are low by default and region-specific. `nvidia-tesla-t4` is most widely available. If quota is hit, the instance creation fails with an opaque error. Use common zones (`us-central1-c`, `us-east1-c`).
- **`on_host_maintenance` not set for GPU**. GPU instances require `scheduling { on_host_maintenance = "TERMINATE" }`. Without it, Terraform apply fails with a validation error.
- **IAP tunnel latency**. IAP-tunneled SSH adds 2-5 seconds of connection latency per session. The learner experience is noticeably slower than direct SSH. Only use IAP when the exercise requires it (e.g., teaching private networking).
- **IAP firewall rule missing**. IAP tunnels originate from `35.235.240.0/20`. If no firewall rule allows this range on port 22, `gcloud compute ssh --tunnel-through-iap` hangs silently.
- **Custom image access**. Images from other GCP projects require explicit IAM sharing (`roles/compute.imageUser` on the image or project). Without it, the instance creation fails with a permission error.
- **Cleanup credential expiry**. The admin service account key file at `/tmp/admin-sa.json` must still be present during cleanup. If a challenge script deletes `/tmp/`, the cleanup script cannot authenticate. Store the key in a persistent location like `/root/`.
- **Terraform state lost between challenges**. The state file at `/root/terraform/terraform.tfstate` must persist. Do not delete `/root/terraform/` or the cleanup script cannot destroy resources.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- GPU-accelerated ML training lab (T4 or V100 with Deep Learning VM image)
- ARM workload migration workshop (Tau T2A instances)
- GCP private networking exercise (IAP-tunneled SSH, no public IPs)
- Custom image deployment pipeline (building and launching from shared images)
- Compute Engine operations deep-dive (live migration, preemptible VMs, instance groups)
- Multi-GPU inference serving lab (multiple T4s attached to a single instance)
