# config.yml

Defines the sandbox infrastructure for a track: containers, virtual machines, virtual browsers, cloud accounts, and secrets. Located at the track root alongside `track.yml`.

## Structure

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 256
    shell: /bin/bash
    entrypoint: bash
    environment:
      MY_VAR: my-value
    private: false

virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      ROLE: primary
    allow_external_ingress:
      - http
      - https
    nested_virtualization: true
    provision_ssl_certificate: true

virtualbrowsers:
  - name: web-app
    urls:
      - url: https://app-${_SANDBOX_ID}.instruqt.io
        label: Application

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::policy/ReadOnlyAccess
    iam_policy: |
      {"Version":"2012-10-17","Statement":[...]}
    admin_managed_policies:
      - arn:aws:iam::policy/AdministratorAccess
    admin_iam_policy: |
      {"Version":"2012-10-17","Statement":[...]}
    scp_policy: |
      {"Version":"2012-10-17","Statement":[...]}

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
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
    regions:
      - us-central1
    roles:
      - roles/viewer
    admin_roles:
      - roles/owner

secrets:
  - name: API_KEY
    value: secret-value
```

## Fields

### version

Always `"3"`. Required.

### containers

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Unique name, used as hostname and in tab references |
| `image` | string | required | Docker image reference |
| `ports` | list of int | — | Exposed ports |
| `memory` | int (MB) | 256 | Memory allocation |
| `shell` | string | `/bin/sh` | Default shell for terminal tabs |
| `entrypoint` | string | — | Overrides image ENTRYPOINT. Commonly `"bash"` alongside `shell: /bin/bash` |
| `environment` | map | — | Key-value environment variables |
| `private` | bool | false | `true` marks container as not directly accessible to learners (backend-only, no tabs) |

**Common images:**
- `gcr.io/instruqt/shell` — Lightweight, no cloud CLIs. Only `INSTRUQT_AWS_ACCOUNT_*` prefixed env vars available.
- `gcr.io/instruqt/cloud-client` (or `:2`) — Includes AWS, Azure, and GCP CLIs. Unprefixed AWS env vars available (`AWS_ACCESS_KEY_ID`, etc.).
- Custom images from private Artifact Registries or upstream registries (`node:22-bookworm`, etc.)

**Constraints:**
- Containers do NOT support virtual display servers (X11, Xvfb). KasmVNC must run in a VM.
- code-server runs fine in containers (web server, not GUI app).

### virtualmachines

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Unique name, used as hostname |
| `image` | string | required | GCP image family or custom image path |
| `machine_type` | string | — | GCP machine-type string (preferred over memory/cpus) |
| `memory` | int (MB) | — | Alternative to machine_type |
| `cpus` | int | — | Alternative to machine_type |
| `shell` | string | — | Default shell for terminal tabs |
| `environment` | map | — | Key-value environment variables |
| `allow_external_ingress` | list | — | `[http, https, high-ports]` for web apps |
| `nested_virtualization` | bool | false | Required for Docker/Kind on VM |
| `provision_ssl_certificate` | bool | false | Provision HTTPS certificate |

### virtualbrowsers

Virtual browser tabs pointing to URLs within the sandbox.

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique name, referenced in tab definitions |
| `urls` | list | Array of `{url, label}` objects |

Supports `${_SANDBOX_ID}` interpolation in URLs.

### aws_accounts

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Account name, used in env var prefix |
| `services` | list | AWS service names to enable |
| `regions` | list | Allowed AWS regions |
| `managed_policies` | list | IAM managed policy ARNs for learner role |
| `admin_managed_policies` | list | IAM managed policy ARNs for admin/setup role |
| `iam_policy` | string | Inline IAM policy JSON for learner role |
| `admin_iam_policy` | string | Inline IAM policy JSON for admin/setup role |
| `scp_policy` | string | Service Control Policy JSON |

### azure_subscriptions

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Subscription name |
| `services` | list | Resource provider namespaces (e.g. `Microsoft.Compute`) |
| `regions` | list | Allowed Azure regions |
| `roles` | list | Azure roles for learner |
| `admin_roles` | list | Azure roles for admin/setup |

### gcp_projects

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Project name, used in env var prefix |
| `services` | list | API names (e.g. `compute.googleapis.com`) |
| `regions` | list | Allowed GCP regions |
| `roles` | list | IAM roles for learner |
| `admin_roles` | list | IAM roles for admin/setup |

### secrets

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Secret name, injected as environment variable |
| `value` | string | Secret value |

Named secrets are injected as environment variables into all containers and VMs.

### Custom Resources

Terraform modules configured in the Instruqt Web UI, not in `config.yml`. These are managed separately from the track source.

## Examples

### Container-only track

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    shell: /bin/bash
    environment:
      TUTORIAL_MODE: "true"
```

### VM with cloud account

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
    nested_virtualization: true

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
```

### Multi-container with private backend

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    shell: /bin/bash
  - name: database
    image: postgres:16
    ports:
      - 5432
    environment:
      POSTGRES_PASSWORD: instruqt
    private: true
```
