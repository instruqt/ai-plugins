# Cloud Provider Selection

Choosing the right cloud account configuration when a track needs real cloud resources. Each provider has different auth mechanisms, CLI tools, IAM models, and setup requirements.

## Provider Comparison

| Aspect | AWS | Azure | GCP |
|--------|-----|-------|-----|
| Account type | AWS account | Azure subscription | GCP project |
| Config key | `aws_accounts:` | `azure_subscriptions:` | `gcp_projects:` |
| Auth mechanism | Access key + secret key | Service principal (ARM_* vars) | Service account key (base64) |
| CLI pre-installed | Yes (cloud-client) | Yes (cloud-client) | Yes (cloud-client) |
| CLI auth in setup | Automatic (env vars) | Requires `az login --service-principal` | Requires key decode + `gcloud auth activate-service-account` |
| IAM model | IAM policies + SCP guardrails | Role assignments + resource providers | IAM roles + API enablement |
| Common gotcha | SCP too restrictive | Resource provider not registered | Wrong IAM role name |

## Decision Criteria

The cloud provider is usually dictated by what the track teaches. When there is a choice:

### Use AWS when:
- Teaching AWS-specific services (EC2, S3, EKS, Lambda, CloudFormation).
- Learner's target audience primarily uses AWS.
- Track needs AMI-based custom images.

### Use Azure when:
- Teaching Azure-specific services (AKS, Azure Functions, Azure DevOps).
- Track needs Active Directory integration.
- Learner's target audience primarily uses Azure.

### Use GCP when:
- Teaching GCP-specific services (GKE, Cloud Run, BigQuery).
- Track needs GPU VMs (GCP has straightforward GPU attachment).
- Track uses IAP tunnels for secure access to private VMs.

### Use multiple providers when:
- Track teaches multi-cloud orchestration or comparison.
- Track needs resources from more than one provider simultaneously.
- See `cloud-accounts/multi-cloud.md`.

## Container vs VM with Cloud Accounts

| Scenario | Recommended pattern |
|----------|-------------------|
| CLI/SDK exercises against cloud APIs | Container (cloud-client) + cloud account |
| Terraform with no local Docker | Container (cloud-client) + cloud account |
| Docker builds + push to cloud registry | VM + cloud account |
| Kubernetes with local Kind + cloud EKS/AKS/GKE | VM + cloud account |
| Hybrid (CLI workstation + heavy local services + cloud) | Container + VM + cloud account |

## Provider-Specific Pitfalls

### AWS
- **SCP policies too restrictive.** Overly tight SCPs block learner actions with cryptic "Access Denied" errors. Test the full learner workflow against the SCP before publishing.
- **Default VPC missing.** Some regions lack a default VPC. Use `aws ec2 create-default-vpc || true` in setup if provisioning EC2 instances.
- **AMI region mismatch.** AMIs are region-specific. If the track hardcodes an AMI ID, it only works in that region.

### Azure
- **Resource provider not registered.** The most common Azure failure. Every Azure service requires its resource provider to be registered in the subscription. List all needed providers in `resource_providers:`.
- **`az login` required.** Unlike AWS (auto-configured), Azure CLI requires explicit `az login --service-principal` in setup using the injected ARM_* variables.
- **Resource group discovery.** Azure creates a resource group for the sandbox. Discover it with `az group list` rather than hardcoding.

### GCP
- **Invalid role names.** `roles/writer` does not exist. Use specific roles like `roles/storage.objectCreator`. Check the GCP predefined roles list.
- **API not enabled.** GCP APIs must be explicitly enabled. List required APIs in the `services:` field of the `gcp_projects:` block.
- **Service account activation.** The base64-encoded key must be decoded and activated with `gcloud auth activate-service-account` before gcloud commands work.
