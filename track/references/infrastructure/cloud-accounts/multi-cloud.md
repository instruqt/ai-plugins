# Multi-Cloud

Multiple cloud providers in a single track. Instruqt can provision sandboxed accounts from different providers simultaneously -- for example, an AWS account and an Azure subscription in the same sandbox. Each provider's environment variables are injected into ALL sandbox hosts. This is a rare but powerful pattern for cross-cloud scenarios.

## Config.yml Example (AWS + Azure)

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

aws_accounts:
  - name: aws
    services:
      - s3
      - ec2
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/AmazonS3FullAccess
      - arn:aws:iam::policy/AmazonEC2ReadOnlyAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess

azure_subscriptions:
  - name: azure
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

### Variant: AWS + GCP

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

aws_accounts:
  - name: aws
    services:
      - s3
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/AmazonS3FullAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess

gcp_projects:
  - name: gcp
    services:
      - storage.googleapis.com
      - compute.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/storage.admin
      - roles/compute.viewer
    admin_roles:
      - roles/owner
```

### Variant: all three providers

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 1024
    shell: /bin/bash

aws_accounts:
  - name: aws
    services:
      - s3
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/AmazonS3FullAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess

azure_subscriptions:
  - name: azure
    services:
      - Microsoft.Storage
      - Microsoft.Compute
      - Microsoft.Network
    regions:
      - eastus
    roles:
      - Contributor
    admin_roles:
      - Owner

gcp_projects:
  - name: gcp
    services:
      - storage.googleapis.com
      - compute.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/storage.admin
    admin_roles:
      - roles/owner
```

## Environment Variables

All cloud provider environment variables are injected into every sandbox host simultaneously. On a cloud-client container with the AWS + Azure example above:

### AWS variables (name: aws)

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AWS_ACCOUNT_AWS_ACCOUNT_ID` | AWS account ID |
| `INSTRUQT_AWS_ACCOUNT_AWS_AWS_ACCESS_KEY_ID` | AWS access key |
| `INSTRUQT_AWS_ACCOUNT_AWS_AWS_SECRET_ACCESS_KEY` | AWS secret key |
| `AWS_ACCESS_KEY_ID` | Unprefixed alias (cloud-client only) |
| `AWS_SECRET_ACCESS_KEY` | Unprefixed alias (cloud-client only) |

### Azure variables (name: azure)

| Variable | Description |
|----------|-------------|
| `ARM_CLIENT_ID` | Azure service principal client ID |
| `ARM_CLIENT_SECRET` | Azure service principal secret |
| `ARM_TENANT_ID` | Azure AD tenant ID |
| `ARM_SUBSCRIPTION_ID` | Azure subscription ID |

### GCP variables (name: gcp)

| Variable | Description |
|----------|-------------|
| `INSTRUQT_GCP_PROJECT_GCP_PROJECT_ID` | GCP project ID |
| `INSTRUQT_GCP_PROJECT_GCP_SERVICE_ACCOUNT_KEY` | GCP SA key (base64) |
| `INSTRUQT_GCP_PROJECT_GCP_SERVICE_ACCOUNT_EMAIL` | GCP SA email |

All of these coexist in the same environment without conflicts because each provider uses distinct variable name prefixes.

## Setup Script Pattern

In a multi-cloud setup, authenticate each provider separately:

```bash
#!/bin/bash

# --- AWS ---
# AWS CLI auto-authenticates on cloud-client via AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY
aws sts get-caller-identity

# --- Azure ---
az login --service-principal \
  --username "$ARM_CLIENT_ID" \
  --password "$ARM_CLIENT_SECRET" \
  --tenant "$ARM_TENANT_ID"
az account set --subscription "$ARM_SUBSCRIPTION_ID"

# --- GCP ---
echo "$INSTRUQT_GCP_PROJECT_GCP_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/sa-key.json
gcloud auth activate-service-account --key-file=/tmp/sa-key.json
gcloud config set project "$INSTRUQT_GCP_PROJECT_GCP_PROJECT_ID"
rm -f /tmp/sa-key.json
```

## Terraform with Multiple Providers

Terraform can target multiple cloud providers in a single configuration. The standard env vars are picked up automatically:

```hcl
provider "aws" {
  region = "us-west-2"
  # AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY from env
}

provider "azurerm" {
  features {}
  # ARM_CLIENT_ID, ARM_CLIENT_SECRET, ARM_TENANT_ID, ARM_SUBSCRIPTION_ID from env
}

provider "google" {
  project = var.gcp_project_id
  region  = "us-central1"
  # GOOGLE_CREDENTIALS or GOOGLE_APPLICATION_CREDENTIALS from env
}
```

## When to Use

- Track teaches multi-cloud orchestration (e.g., Terraform managing resources across AWS and Azure).
- Track compares equivalent services across providers (S3 vs. Blob Storage vs. Cloud Storage).
- Track teaches cross-cloud networking (VPN/peering between providers).
- Track teaches cloud-agnostic tools (Pulumi, Crossplane, Terraform) deployed to multiple providers.
- Track demonstrates migration between cloud providers.

## When NOT to Use

- Track only uses one cloud provider -- use the single-provider patterns instead.
- Track mentions multiple clouds conceptually but only provisions resources in one -- do not add unused cloud accounts.
- The multi-cloud angle is not central to the learning objectives. Each additional cloud account adds sandbox startup time and cost.

## Common Pitfalls

- **Sandbox startup time**. Each cloud account adds to provisioning time. A three-provider sandbox can take significantly longer to start than a single-provider one. Account for this in the learner experience.
- **Script complexity**. Multi-cloud setup scripts are longer and harder to debug. Authenticate each provider separately and verify each one works before proceeding.
- **Variable naming in scripts**. When writing scripts that interact with multiple providers, use clear variable names. Avoid ambiguous names like `$ACCOUNT_ID` -- use `$AWS_ACCOUNT_ID` and `$GCP_PROJECT_ID` to make it clear which provider you mean.
- **GCP auth is manual**. Even on cloud-client, GCP requires decoding the base64 key and calling `gcloud auth activate-service-account`. AWS and Azure credentials are picked up more automatically.
- **Terraform state management**. With multiple providers, Terraform state contains resources from all providers. A failed `terraform destroy` may leave orphaned resources in one provider.
- **Cost**. Multiple cloud accounts means multiple sets of cloud resources. Keep exercises focused and resource requirements minimal.
- **Learner cognitive load**. Multi-cloud tracks are inherently more complex. Ensure the learning objectives justify the added complexity -- do not add a second cloud provider just because you can.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Multi-cloud Terraform deployment (provision equivalent resources in AWS and Azure)
- Cloud storage comparison (create and manage objects in S3, Blob Storage, and Cloud Storage side by side)
- Cross-cloud VPN setup (site-to-site VPN between AWS VPC and Azure VNet)
- Cloud migration simulation (move workloads from one provider to another)
- Cloud-agnostic tool training (Pulumi or Crossplane deploying to multiple providers)
- Multi-cloud monitoring (unified observability across providers)
