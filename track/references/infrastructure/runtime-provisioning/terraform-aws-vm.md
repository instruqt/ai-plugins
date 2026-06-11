# Terraform AWS VM (Runtime Provisioned)

Container + AWS account + Terraform-provisioned EC2 instance. A cloud-client container runs `terraform apply` during setup to create an EC2 instance that Instruqt cannot provide natively (ARM architecture, GPU instances, custom AMIs, specific OS images). The learner accesses the VM via SSH through a `cmd:` terminal tab or via a service tab for web UIs running on the instance.

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

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/AmazonEC2ReadOnlyAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
```

### Tab configuration

SSH to the provisioned VM via `cmd:` terminal tab:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: VM
    type: terminal
    hostname: shell
    cmd: ssh -o StrictHostKeyChecking=no root@${EC2_PUBLIC_IP}
```

For a web UI running on the EC2 instance, use an SSH tunnel and a service tab:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: Web UI
    type: service
    hostname: shell
    port: 8080
```

## Terraform Configuration

Store Terraform files in the container at `/root/terraform/`. Either bake them into a custom image or write them via heredoc in the setup script.

### main.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {}

data "aws_ami" "ubuntu_arm" {
  most_recent = true
  owners      = ["099720109477"]   # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-arm64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "tls_private_key" "ssh" {
  algorithm = "ED25519"
}

resource "aws_key_pair" "lab" {
  key_name   = "instruqt-lab-${random_id.suffix.hex}"
  public_key = tls_private_key.ssh.public_key_openssh
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_security_group" "lab" {
  name = "instruqt-lab-${random_id.suffix.hex}"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "lab" {
  ami                    = data.aws_ami.ubuntu_arm.id
  instance_type          = "t4g.micro"               # ARM; swap for p3.2xlarge for GPU
  key_name               = aws_key_pair.lab.key_name
  vpc_security_group_ids = [aws_security_group.lab.id]

  root_block_device {
    volume_size = 30
  }

  tags = {
    Name = "instruqt-lab-${random_id.suffix.hex}"
  }
}

output "instance_id" {
  value = aws_instance.lab.id
}

output "public_ip" {
  value = aws_instance.lab.public_ip
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

# Ensure a default VPC exists in the region -- some sandbox regions lack one
aws ec2 create-default-vpc 2>/dev/null || true

# Write Terraform files (or use pre-baked files in a custom image)
mkdir -p /root/terraform
cat > /root/terraform/main.tf <<'EOF'
# ... (Terraform config from above)
EOF

cd /root/terraform
terraform init -input=false
terraform apply -input=false -auto-approve

# Extract outputs
INSTANCE_ID=$(terraform output -raw instance_id)
PUBLIC_IP=$(terraform output -raw public_ip)
terraform output -raw private_key > /root/.ssh/lab_key
chmod 600 /root/.ssh/lab_key

# Store for cleanup and cross-challenge access
agent variable set INSTANCE_ID "$INSTANCE_ID"
agent variable set EC2_PUBLIC_IP "$PUBLIC_IP"

# Configure SSH alias for the learner
cat > /root/.ssh/config <<SSHEOF
Host lab-vm
  HostName ${PUBLIC_IP}
  User ubuntu
  IdentityFile /root/.ssh/lab_key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
SSHEOF

# Wait for SSH to become available
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

# Fallback: terminate the instance directly if terraform state is corrupted
INSTANCE_ID=$(agent variable get INSTANCE_ID 2>/dev/null || true)
if [ -n "$INSTANCE_ID" ]; then
  aws ec2 terminate-instances --instance-ids "$INSTANCE_ID" 2>/dev/null || true
fi
```

## When to Use

- Track needs ARM-based EC2 instances (Graviton: `t4g`, `m7g`, `c7g` families) not available as Instruqt-native VMs.
- Track needs GPU instances (`p3`, `p4d`, `g5` families) for ML/AI workloads.
- Track needs a specific AMI (custom OS, pre-configured software, marketplace image).
- Track teaches EC2 operations where provisioning is the exercise itself.
- Track needs instance types or configurations beyond what Instruqt VMs offer (e.g., multiple network interfaces, placement groups).

## When NOT to Use

- A standard x86 Linux VM is sufficient -- use Instruqt-native `virtualmachines:` instead. It is faster, cheaper, and automatically cleaned up.
- The track only needs cloud CLI access to AWS services (S3, Lambda, DynamoDB) -- use a container + AWS account without Terraform.
- The learner does not need SSH access to the instance -- if the exercise is purely API-driven, Terraform overhead is unnecessary.

## Common Pitfalls

- **No default VPC**. Instruqt sandbox AWS accounts in some regions lack a default VPC. Always run `aws ec2 create-default-vpc 2>/dev/null || true` before `terraform apply`, or explicitly create a VPC in your Terraform config.
- **Cleanup failure leaves running instances**. If `terraform destroy` fails (state corruption, provider errors), the instance keeps running and incurring costs. Always include a fallback `aws ec2 terminate-instances` using the stored instance ID.
- **SSH key not persisted**. The Terraform-generated SSH key exists only in Terraform state and the local file. If the setup script does not write it to disk, later challenges cannot SSH to the VM. Write the key to `/root/.ssh/` and store the IP via `agent variable set`.
- **AMI data source returns wrong architecture**. If using a dynamic `data "aws_ami"` lookup, always filter on architecture (`arm64` or `x86_64`) to avoid launching an x86 AMI on an ARM instance type or vice versa.
- **Security group too open**. Opening `0.0.0.0/0` on all ports is a common shortcut that risks abuse. Open only the ports the lab needs (SSH, application ports).
- **Instance not ready when setup completes**. EC2 instances take 30-90 seconds to pass status checks and accept SSH. The setup script must poll for SSH readiness before returning, or the learner hits a broken terminal tab.
- **Terraform state lost between challenges**. The Terraform state file in `/root/terraform/` persists across challenges on the container. Do not delete it or the cleanup script cannot destroy resources.
- **Region quota limits**. GPU and large instance types have low default quotas. The sandbox account may hit limits. Use common regions (`us-west-2`, `us-east-1`) and smaller instance types where possible.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- ARM architecture migration workshop (deploying and testing on Graviton instances)
- GPU-accelerated ML inference lab (provisioning `p3.2xlarge` with NVIDIA drivers)
- Custom AMI deployment exercise (launching from a pre-built golden image)
- EC2 networking deep-dive (multiple instances with ENIs and security groups)
- Performance benchmarking lab (comparing instance families on a real workload)
