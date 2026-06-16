# Hybrid Plus Cloud

Container + VM + cloud account in a single track. The container serves as the learner's CLI workstation (cloud-client), the VM runs local services (Docker, databases, Kubernetes), and a cloud account provides real cloud resources. Combines the lightweight CLI experience of a container with the OS-level capabilities of a VM and the real-world resources of a cloud provider.

## Config.yml Example (AWS)

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
      APP_HOST: appserver

virtualmachines:
  - name: appserver
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
      - rds
      - iam
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::aws:policy/AmazonS3FullAccess
      - arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess
    admin_managed_policies:
      - arn:aws:iam::aws:policy/AdministratorAccess
```

### Variant: container + VM + Azure

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

virtualmachines:
  - name: devserver
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true

azure_subscriptions:
  - name: sandbox
    services:
      - Microsoft.Compute
      - Microsoft.Network
      - Microsoft.Storage
      - Microsoft.ContainerService
      - Microsoft.ContainerRegistry
    regions:
      - eastus
    roles:
      - Contributor
    admin_roles:
      - Owner
```

### Variant: container + VM + GCP

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

virtualmachines:
  - name: devserver
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - container.googleapis.com
      - storage.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/container.admin
      - roles/storage.admin
    admin_roles:
      - roles/owner
```

### Tab configuration

Terminal on the container, service tabs on the VM:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Application
    type: service
    hostname: appserver
    port: 8080
    path: /
```

## Environment Variable Injection

Cloud account environment variables are injected into ALL sandbox hosts -- both the container and the VM. This means:

- The container (workstation) has cloud credentials for CLI/Terraform work.
- The VM (appserver) also has cloud credentials, which setup scripts can use to configure the local application to connect to cloud resources.

For example, a setup script on the VM can read `INSTRUQT_AWS_ACCOUNT_SANDBOX_AWS_ACCESS_KEY_ID` to configure a local application's connection to S3.

## Cross-Host Communication

Containers and VMs share a private network. Each host's `name` is its hostname:

```bash
# From workstation (container): reach the VM
curl http://appserver:8080

# From appserver (VM): reach the container
curl http://workstation:8080
```

Use `agent variable` for runtime-generated values (tokens, dynamic config):

```bash
# On the VM (setup-appserver):
agent variable set DB_CONNECTION_STRING "postgres://app:secret@appserver:5432/mydb"

# On the container (setup-workstation):
until agent variable get DB_CONNECTION_STRING 2>/dev/null | grep -q .; do
  sleep 3
done
export DB_CONNECTION_STRING=$(agent variable get DB_CONNECTION_STRING)
```

## When to Use

- Track teaches deploying a local application to the cloud (e.g., local Docker app pushes images to a cloud registry, then deploys to a cloud container service).
- Track needs both local infrastructure (Docker, Kubernetes, databases on a VM) AND cloud resources (S3 buckets, managed databases, cloud networking).
- Track simulates a real development workflow: code locally on the VM, test locally, then deploy to cloud via CLI from the container.
- Track teaches hybrid or multi-tier architectures (local frontend + cloud backend, or local dev environment + cloud staging).

## When NOT to Use

- Track only needs cloud CLI access with no local services -- use `container-plus-aws.md` / `container-plus-azure.md` / `container-plus-gcp.md`.
- Track needs Docker/systemd on the workstation itself but no separate service host -- use `vm-plus-aws.md` / `vm-plus-azure.md` / `vm-plus-gcp.md`.
- All services run in the cloud and the workstation is CLI-only -- a container + cloud account is sufficient.

## Common Pitfalls

- **Setup script race conditions**. The container and VM start setup scripts concurrently. Containers boot in seconds; VMs take 30-90 seconds. The container setup must poll for VM readiness before depending on VM services.
- **`agent variable` timing**. The container setup must poll with a loop -- `agent variable get` returns empty if the VM has not set the value yet.
- **Tab targeting confusion**. Terminal tabs should point to the container (the learner's workstation). Service tabs for web UIs should point to the VM. Swapping them is a common mistake.
- **Cloud credential assumptions on the VM**. The VM has cloud env vars but not cloud CLIs. If the VM's setup script needs cloud CLIs (e.g., to pull an image from a cloud registry), install them first.
- **Memory and cost**. This pattern uses the most resources: a container (256-512 MB), a VM (n1-standard-2 or larger), and a cloud account. Keep compute sizes as small as practical.
- **Port exposure on the container**. If the container serves a web UI via a service tab, the port must be listed under `ports:`.
- **Cloud-client auth on the container vs. VM**. On cloud-client containers, AWS credentials are auto-configured. On the VM, you must explicitly set up authentication. See the provider-specific files for auth patterns.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Local development to cloud deployment (Docker Compose app on VM, deploy to ECS/AKS/Cloud Run from workstation)
- Hybrid architecture (local database on VM + cloud object storage for backups)
- CI/CD simulation (Jenkins/GitLab Runner on VM, deploying to cloud from workstation CLI)
- Kubernetes migration (local k3s cluster on VM, migrating workloads to managed cloud Kubernetes)
- Infrastructure as Code development (edit Terraform on workstation, local test environment on VM, deploy to cloud)
- Cloud-native application development (local dev server on VM, cloud services for auth/storage/messaging)
