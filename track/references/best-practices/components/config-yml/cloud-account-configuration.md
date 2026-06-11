# Cloud Account Configuration

Evaluates whether cloud provider resource blocks (AWS, Azure, GCP) are correctly configured with appropriate IAM policies, services, and region settings.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Cloud accounts missing when the track requires cloud resources, or IAM policies grant unrestricted admin access to learners |
| 2 | Below Standard | Cloud accounts present but policies are too broad, services list is incomplete, or region selection causes availability issues |
| 3 | Adequate | Policies exist but use a single role for both learner and admin operations, or services list covers the basics but misses secondary dependencies |
| 4 | Good | Dual IAM policy pattern used -- learner gets narrow permissions, admin/setup gets full access; all needed services listed; region appropriate (production baseline) |
| 5 | Excellent | Least-privilege policies with explicit deny where needed; services list complete including transitive dependencies; BYOC considered where applicable |

## Guidance

Cloud account blocks in config.yml provision temporary cloud credentials for learners and setup/check scripts. The critical design pattern is the **dual IAM policy**: one narrow policy for the learner (what they should be able to do) and one broader policy for administrative operations (what setup and check scripts need).

### Dual IAM Policy Pattern

The learner policy restricts actions to exactly what the exercise requires. The admin policy (used by setup/check/solve scripts via a separate credential set) has broader access to provision resources, validate state, and clean up.

Good -- AWS with dual policies:

```yaml
aws_accounts:
- name: sandbox
  managed: true
  iam_policy:
    Version: "2012-10-17"
    Statement:
    - Effect: Allow
      Action:
      - s3:ListBucket
      - s3:GetObject
      - s3:PutObject
      Resource: "*"
  admin_iam_policy:
    Version: "2012-10-17"
    Statement:
    - Effect: Allow
      Action: "*"
      Resource: "*"
  aws_services:
  - s3
  - iam
  - cloudformation
```

Good -- GCP with scoped roles:

```yaml
gcp_projects:
- name: project
  managed: true
  iam_policy:
    roles:
    - roles/storage.objectViewer
    - roles/container.developer
  admin_iam_policy:
    roles:
    - roles/owner
  services:
  - container.googleapis.com
  - storage.googleapis.com
  - iam.googleapis.com
```

Bad -- single broad policy for both learner and admin:

```yaml
aws_accounts:
- name: sandbox
  managed: true
  iam_policy:
    Version: "2012-10-17"
    Statement:
    - Effect: Allow
      Action: "*"
      Resource: "*"
```

Bad -- services list missing dependencies:

```yaml
# Track uses EKS but only lists eks, missing ec2, iam, ecr
aws_accounts:
- name: sandbox
  managed: true
  aws_services:
  - eks
```

### Services List Completeness

Cloud services must be explicitly enabled. Missing a service causes API calls to fail during setup or learner exercises. Always include transitive dependencies:

- **AWS EKS** needs: eks, ec2, iam, ecr, elasticloadbalancing, autoscaling
- **AWS Lambda** needs: lambda, iam, logs, s3 (if using S3 triggers)
- **GCP GKE** needs: container.googleapis.com, compute.googleapis.com, iam.googleapis.com
- **Azure AKS** needs: Microsoft.ContainerService, Microsoft.Network, Microsoft.Compute

### Region Selection

Choose regions based on:
- Service availability (not all services are in all regions)
- Proximity to learner base (reduces latency for web-based exercises)
- Cost (some regions are more expensive)

Common defaults: `us-west-1` / `us-central1` / `westus2` for general tracks.

### BYOC (Bring Your Own Cloud)

For tracks where customers use their own cloud accounts, the config.yml does not include managed cloud blocks. Instead, the track expects pre-configured credentials passed via environment variables or secrets. Document this clearly in track metadata.

## What to Watch For

- Learner IAM policies that grant `*` actions -- learners should only have permissions the exercise requires
- Missing `admin_iam_policy` -- setup and check scripts may fail when they need broader permissions
- Incomplete services lists causing "service not enabled" errors during track launch
- Region hardcoded in policies or scripts but not matching the cloud account's configured region
- Cloud accounts defined but no corresponding setup script that uses the credentials
- BYOC tracks that accidentally include managed cloud blocks -- these provision unnecessary (and costly) accounts
