# VM Plus AWS

Full VM workstation paired with a sandboxed AWS account. Use when the track needs OS-level features (Docker, systemd, nested virtualization) alongside AWS cloud resources. The AWS environment variables are injected into the VM just as they would be into a container.

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

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
      - ecs
      - ecr
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::aws:policy/AmazonEC2FullAccess
      - arn:aws:iam::aws:policy/AmazonS3FullAccess
      - arn:aws:iam::aws:policy/AmazonECS_FullAccess
    admin_managed_policies:
      - arn:aws:iam::aws:policy/AdministratorAccess
```

### Variant: Docker-based workflow with ECR

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

aws_accounts:
  - name: sandbox
    services:
      - ecr
      - ecs
      - ec2
      - iam
      - sts
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
      - arn:aws:iam::aws:policy/AmazonECS_FullAccess
    admin_managed_policies:
      - arn:aws:iam::aws:policy/AdministratorAccess
```

### Setup script: install AWS CLI on the VM

VMs do not come with cloud CLIs pre-installed. Install in the setup script:

```bash
#!/bin/bash

# Install AWS CLI v2
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip
unzip -q /tmp/awscliv2.zip -d /tmp
/tmp/aws/install
rm -rf /tmp/aws /tmp/awscliv2.zip

# Verify
aws --version

# Configure default region
aws configure set default.region "$AWS_DEFAULT_REGION"
```

### Setup script: Docker + ECR login

```bash
#!/bin/bash

# Install Docker
apt-get update
apt-get install -y docker.io
systemctl enable --now docker

# Install AWS CLI
curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip
unzip -q /tmp/awscliv2.zip -d /tmp
/tmp/aws/install
rm -rf /tmp/aws /tmp/awscliv2.zip

# Log into ECR
ACCOUNT_ID="$INSTRUQT_AWS_ACCOUNT_SANDBOX_ACCOUNT_ID"
REGION="us-west-2"
aws ecr get-login-password --region "$REGION" | \
  docker login --username AWS --password-stdin "${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"
```

## Auto-Injected Environment Variables

The same AWS environment variables are injected into the VM as would be injected into a container. For `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_ACCOUNT_ID` | AWS account ID |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_USERNAME` | IAM username |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_PASSWORD` | IAM password |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_ACCESS_KEY_ID` | Access key for learner role |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_SECRET_ACCESS_KEY` | Secret key for learner role |

Note: unlike `gcr.io/instruqt/cloud-client:2`, VMs do NOT automatically set unprefixed `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`. If your track expects the unprefixed form, export them in the setup script:

```bash
echo "export AWS_ACCESS_KEY_ID=$INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_ACCESS_KEY_ID" >> /root/.bashrc
echo "export AWS_SECRET_ACCESS_KEY=$INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_SECRET_ACCESS_KEY" >> /root/.bashrc
echo "export AWS_DEFAULT_REGION=us-west-2" >> /root/.bashrc
```

## When to Use

- Track needs Docker on the workstation (building AMIs with Packer, running ECS task definitions locally, building and pushing container images to ECR).
- Track needs systemd services running alongside AWS CLI work (e.g., a local application server that deploys to AWS).
- Track needs heavy tooling not available in cloud-client (custom compilers, large SDKs, GUI tools).
- Track teaches ECS/EKS workflows where the learner builds images locally and pushes to ECR.
- Track needs nested virtualization (e.g., running VMs inside the sandbox VM for AMI building).

## When NOT to Use

- Track only needs CLI/SDK/Terraform access to AWS -- use `container-plus-aws.md` for faster startup and lower resource usage.
- Track needs both a VM (for services) and a lightweight CLI workstation -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **No cloud CLIs pre-installed**. Unlike cloud-client containers, VMs are bare OS images. You must install the AWS CLI (and any other tools) in the setup script. Forgetting this causes `aws: command not found`.
- **Missing unprefixed env vars**. VMs only get `INSTRUQT_AWS_ACCOUNT_*` variables, not the short `AWS_ACCESS_KEY_ID` form. Tools that expect the standard `AWS_*` variables (Terraform, boto3, AWS CLI) need them exported manually.
- **Long setup times**. Installing Docker, the AWS CLI, and other dependencies can take 2-3 minutes on a bare VM. Consider a custom Packer image with dependencies pre-installed (see `modifiers/custom-image-packer.md`).
- **Docker not running**. If the setup script installs Docker but does not start it (`systemctl enable --now docker`), Docker commands fail. Always start Docker immediately after installation.
- **VM startup time**. VMs take 30-90 seconds to boot, plus time for the setup script. The first challenge should account for this delay.
- **Forgetting `nested_virtualization: true`**. If the track runs Docker or needs kernel features, this flag must be set. Without it, Docker may fail to start.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Container image build and push to ECR (Docker build + ECR push + ECS deploy)
- AMI creation with Packer (Packer build on the VM, register AMI in AWS)
- ECS/EKS local development (build images locally, deploy to AWS container services)
- CI/CD pipeline simulation (local Jenkins/GitLab Runner deploying to AWS)
- Hybrid application deployment (local Docker Compose app connecting to AWS RDS/S3)
- AWS CDK development (Node.js/Python app that synthesizes and deploys CloudFormation)
