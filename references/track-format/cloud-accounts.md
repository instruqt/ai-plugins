# Cloud Accounts

Cloud provider accounts configured in `config.yml` to provision sandboxed AWS, Azure, or GCP environments for learners. Each provider supports a dual-role pattern: narrow learner permissions and broader admin/setup permissions.

## Structure

```yaml
version: "3"

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/ReadOnlyAccess
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
    iam_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:ListBucket"],
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
            "Action": ["organizations:*"],
            "Resource": "*"
          }
        ]
      }

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

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - container.googleapis.com
      - storage.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/viewer
      - roles/container.developer
    admin_roles:
      - roles/owner
```

## AWS Accounts

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Account name, used in environment variable prefix |
| `services` | list | AWS service names to enable |
| `regions` | list | Allowed AWS regions |
| `managed_policies` | list | IAM managed policy ARNs for the learner role |
| `admin_managed_policies` | list | IAM managed policy ARNs for the admin/setup role |
| `iam_policy` | string | Inline IAM policy JSON for the learner role (narrow) |
| `admin_iam_policy` | string | Inline IAM policy JSON for the admin/setup role (full access) |
| `scp_policy` | string | Service Control Policy JSON applied to the account |

### Dual policy pattern

- `iam_policy` / `managed_policies` — Applied to the learner role. Keep narrow to prevent learners from breaking the sandbox.
- `admin_iam_policy` / `admin_managed_policies` — Applied to the admin role used by setup/check/solve lifecycle scripts. Typically has full access.

### Auto-injected environment variables

The `{NAME}` segment is derived from the `name` field: uppercased with hyphens replaced by underscores.

For a config with `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_ACCOUNT_ID` | AWS account ID |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_USERNAME` | IAM username |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_PASSWORD` | IAM password |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_ACCESS_KEY_ID` | Access key for learner role |
| `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_SECRET_ACCESS_KEY` | Secret key for learner role |

On `gcr.io/instruqt/cloud-client` containers, unprefixed versions are also available:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`

On `gcr.io/instruqt/shell`, only the prefixed `INSTRUQT_AWS_ACCOUNT_*` form is available.

### BYOC (Bring Your Own Credentials)

Alternative to Instruqt-managed accounts. Use secrets for direct credential injection:

```yaml
secrets:
  - name: AWS_ACCESS_KEY_ID
    value: AKIA...
  - name: AWS_SECRET_ACCESS_KEY
    value: ...
  - name: AWS_DEFAULT_REGION
    value: us-west-2
```

## Azure Subscriptions

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Subscription name |
| `services` | list | Resource provider namespaces (e.g. `Microsoft.Compute`, `Microsoft.Network`) |
| `regions` | list | Allowed Azure regions |
| `roles` | list | Azure roles for the learner |
| `admin_roles` | list | Azure roles for admin/setup |

### Auto-injected environment variables

| Variable | Description |
|----------|-------------|
| `ARM_CLIENT_ID` | Service principal client ID |
| `ARM_CLIENT_SECRET` | Service principal client secret |
| `ARM_TENANT_ID` | Azure AD tenant ID |
| `ARM_SUBSCRIPTION_ID` | Subscription ID |
| `AZURE_LOCATION` | Default Azure region |

## GCP Projects

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Project name, used in environment variable prefix |
| `services` | list | API names to enable (e.g. `compute.googleapis.com`, `container.googleapis.com`) |
| `regions` | list | Allowed GCP regions |
| `roles` | list | IAM roles for the learner |
| `admin_roles` | list | IAM roles for admin/setup |

### Auto-injected environment variables

The `{NAME}` segment is derived from the `name` field: uppercased with hyphens replaced by underscores.

For a config with `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID` | GCP project ID |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_EMAIL` | Learner user email |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_PASSWORD` | Learner user password |
| `INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY` | Learner SA key (base64-encoded) |
| `INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY` | Admin SA key (base64-encoded) |

The service account key is base64-encoded. Decode with:

```bash
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/sa-key.json
gcloud auth activate-service-account --key-file=/tmp/sa-key.json
```

### Name-to-variable mapping

The `{NAME}` segment in all cloud provider env vars follows this rule:
1. Take the `name` field value from config.yml
2. Convert to uppercase
3. Replace hyphens with underscores

Example: `name: my-project` becomes `MY_PROJECT` in variable names like `INSTRUQT_GCP_PROJECT_MY_PROJECT_PROJECT_ID`.

## Examples

### AWS with dual policies

```yaml
aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
      - lambda
      - dynamodb
    regions:
      - us-west-2
      - us-east-1
    managed_policies:
      - arn:aws:iam::policy/AmazonEC2ReadOnlyAccess
      - arn:aws:iam::policy/AmazonS3ReadOnlyAccess
    iam_policy: |
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "lambda:CreateFunction",
              "lambda:InvokeFunction",
              "lambda:GetFunction"
            ],
            "Resource": "*"
          }
        ]
      }
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
```

### Multi-cloud track

```yaml
aws_accounts:
  - name: aws
    services:
      - s3
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
    regions:
      - us-central1
    roles:
      - roles/storage.admin
    admin_roles:
      - roles/owner
```

### Azure with resource providers

```yaml
azure_subscriptions:
  - name: azure
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ContainerService
    regions:
      - eastus
      - westus2
    roles:
      - Contributor
      - AcrPush
    admin_roles:
      - Owner
```
