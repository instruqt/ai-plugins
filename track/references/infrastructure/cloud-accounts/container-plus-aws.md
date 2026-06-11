# Container Plus AWS

Cloud-client container paired with a sandboxed AWS account. The most common cloud pattern -- the learner uses AWS CLI, SDK, or Terraform from a lightweight container while Instruqt manages the AWS account lifecycle (creation, credential injection, cleanup).

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

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
      - iam
      - lambda
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/AmazonEC2ReadOnlyAccess
      - arn:aws:iam::policy/AmazonS3FullAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
```

### Variant: dual-policy with SCP guardrails

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
  - name: sandbox
    services:
      - ec2
      - s3
      - rds
      - iam
    regions:
      - us-west-2
      - us-east-1
    iam_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "s3:*",
              "ec2:Describe*",
              "ec2:RunInstances",
              "ec2:TerminateInstances"
            ],
            "Resource": "*"
          }
        ]
      }
    admin_iam_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
          }
        ]
      }
    scp_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Action": [
              "organizations:*",
              "account:*",
              "ec2:*Host*"
            ],
            "Resource": "*"
          }
        ]
      }
```

### Variant: Terraform with IAM role configuration

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
    environment:
      AWS_DEFAULT_REGION: us-west-2

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
      - iam
      - dynamodb
      - sts
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/PowerUserAccess
      - arn:aws:iam::policy/IAMFullAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
```

## Auto-Injected Environment Variables

On `gcr.io/instruqt/cloud-client:2` containers, both prefixed and unprefixed forms are available.

The `{NAME}` segment is derived from the `name` field: uppercased, hyphens replaced by underscores. For `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_ACCOUNT_ID` | AWS account ID |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_USERNAME` | IAM username |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_PASSWORD` | IAM password |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_ACCESS_KEY_ID` | Access key for learner role |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_SECRET_ACCESS_KEY` | Secret key for learner role |

On `gcr.io/instruqt/cloud-client:2`, these unprefixed aliases are also set automatically:

| Variable | Description |
|----------|-------------|
| `AWS_ACCESS_KEY_ID` | Learner access key |
| `AWS_SECRET_ACCESS_KEY` | Learner secret key |
| `AWS_DEFAULT_REGION` | First region from the `regions:` list |

The unprefixed aliases mean `aws` CLI commands work immediately on cloud-client containers without any setup script.

### Admin credentials in lifecycle scripts

Lifecycle scripts (setup, check, solve) run as the admin role. The admin credentials are available via:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_ADMIN_AWS_ACCESS_KEY_ID` | Admin access key |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_ADMIN_AWS_SECRET_ACCESS_KEY` | Admin secret key |

Use admin credentials in setup scripts to pre-create resources the learner should not be able to create themselves.

## SCP Policies

Service Control Policies (SCPs) are account-level guardrails that apply to ALL principals in the account, including admin. Use them to prevent learners (or buggy scripts) from doing things that break the sandbox or incur unexpected costs:

- Deny launching expensive instance types (e.g., `p3`, `p4d`, dedicated hosts)
- Deny region usage outside allowed regions
- Deny organization-level actions

SCPs are deny-only overlays. They cannot grant permissions -- they restrict what IAM policies can do.

## AMI Sharing

If your track uses a custom AMI, it must be shared with the Instruqt sandbox account. The sandbox account ID is not known until runtime, so AMI sharing is configured through the Instruqt platform, not in config.yml.

## When to Use

- Track teaches AWS services via CLI, SDK, or Terraform and the learner does not need Docker, systemd, or kernel features on the workstation.
- Track provisions AWS infrastructure (EC2, S3, Lambda, RDS, DynamoDB, ECS clusters) as part of exercises.
- Track uses Terraform, CloudFormation, or CDK to deploy AWS resources.
- Track teaches IAM, security, or compliance topics in a sandboxed AWS account.

## When NOT to Use

- Track needs Docker on the workstation (building container images, running local services) -- use `vm-plus-aws.md`.
- Track needs systemd services running alongside AWS CLI work -- use `vm-plus-aws.md`.
- Track only teaches AWS console navigation with no CLI -- cloud accounts alone may suffice, but consider whether a container workstation adds value.
- Track needs both local infrastructure (VM) and AWS -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **Referencing AWS env vars without the `aws_accounts:` block**. The `INSTRUQT_AWS_ACCOUNT_*` variables are only injected when `aws_accounts:` is present in config.yml. Without it, all AWS env vars are empty and CLI commands fail with authentication errors.
- **Using `gcr.io/instruqt/shell` instead of `cloud-client`**. The shell image does not include the AWS CLI. It also does not set the unprefixed `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` aliases -- only the long `INSTRUQT_AWS_ACCOUNT_*` forms are available.
- **Overly broad learner policies**. Giving the learner `AdministratorAccess` lets them delete resources created by setup scripts, break IAM, or launch expensive instances. Use narrow learner policies and broad admin policies.
- **Missing services in the `services:` list**. If the track uses an AWS service not listed under `services:`, it may not be available in the sandbox account.
- **Region mismatch**. If the `regions:` list says `us-west-2` but a setup script creates resources in `us-east-1`, the learner policy may not cover that region, or the SCP may block it.
- **SCP blocking admin operations**. SCPs apply to all principals including admin. If an SCP denies an action the setup script needs, the setup will fail. Test SCPs carefully.
- **Terraform state and credentials**. Terraform picks up `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` automatically on cloud-client. Do not hardcode credentials in Terraform provider blocks.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- S3 bucket operations (create, upload, configure policies, enable versioning)
- EC2 instance lifecycle (launch, connect, terminate, security groups)
- Lambda function deployment (create, test, configure triggers)
- Terraform on AWS (provision VPCs, subnets, EC2, RDS)
- IAM fundamentals (create users, roles, policies, test permissions)
- CloudFormation stack deployment and management
