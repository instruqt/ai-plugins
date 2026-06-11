# Container Plus GCP

Cloud-client container paired with a sandboxed GCP project. The learner uses gcloud CLI, SDK, or Terraform from a lightweight container while Instruqt manages the project lifecycle (creation, service account injection, API enablement, cleanup).

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
      - roles/storage.objectAdmin
    admin_roles:
      - roles/owner
```

### Variant: GKE track

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - container.googleapis.com
      - storage.googleapis.com
      - iam.googleapis.com
      - cloudresourcemanager.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/container.admin
      - roles/compute.viewer
      - roles/iam.serviceAccountUser
    admin_roles:
      - roles/owner
```

### Variant: Cloud Functions and Pub/Sub

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

gcp_projects:
  - name: sandbox
    services:
      - cloudfunctions.googleapis.com
      - pubsub.googleapis.com
      - storage.googleapis.com
      - cloudbuild.googleapis.com
      - artifactregistry.googleapis.com
      - run.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/cloudfunctions.developer
      - roles/pubsub.editor
      - roles/storage.objectAdmin
      - roles/iam.serviceAccountUser
    admin_roles:
      - roles/owner
```

## Auto-Injected Environment Variables

The `{NAME}` segment is derived from the `name` field: uppercased, hyphens replaced by underscores. For `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID` | GCP project ID |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_EMAIL` | Learner user email |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_PASSWORD` | Learner user password |
| `INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY` | Learner service account key (base64-encoded JSON) |
| `INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_EMAIL` | Learner service account email |
| `INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY` | Admin service account key (base64-encoded JSON) |

## gcloud Authentication in Setup Scripts

The service account key is base64-encoded. Decode and activate it in the setup script:

```bash
#!/bin/bash

# Decode and activate the service account
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/sa-key.json
gcloud auth activate-service-account --key-file=/tmp/sa-key.json

# Set the project
gcloud config set project "$INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID"

# Clean up the key file
rm -f /tmp/sa-key.json
```

### Admin credentials for setup scripts

Use admin credentials in setup scripts to pre-create resources with elevated permissions:

```bash
#!/bin/bash

echo "$INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/admin-key.json
gcloud auth activate-service-account --key-file=/tmp/admin-key.json
gcloud config set project "$INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID"

# Create resources with admin permissions
gcloud compute networks create lab-network --subnet-mode=custom
gcloud compute networks subnets create lab-subnet \
  --network=lab-network \
  --region=us-central1 \
  --range=10.0.0.0/24

rm -f /tmp/admin-key.json
```

## API Service Enablement

GCP requires APIs to be enabled before you can use them. The `services:` list in config.yml enables these APIs when the sandbox project is created.

You MUST list every API your track uses. Common APIs:

| API | What it covers |
|-----|---------------|
| `compute.googleapis.com` | VMs, disks, networks, firewalls |
| `container.googleapis.com` | GKE clusters |
| `storage.googleapis.com` | Cloud Storage buckets |
| `iam.googleapis.com` | IAM policy management |
| `cloudresourcemanager.googleapis.com` | Project-level operations |
| `cloudfunctions.googleapis.com` | Cloud Functions |
| `run.googleapis.com` | Cloud Run |
| `pubsub.googleapis.com` | Pub/Sub topics and subscriptions |
| `cloudbuild.googleapis.com` | Cloud Build |
| `artifactregistry.googleapis.com` | Artifact Registry |
| `sqladmin.googleapis.com` | Cloud SQL |
| `bigquery.googleapis.com` | BigQuery |
| `logging.googleapis.com` | Cloud Logging |
| `monitoring.googleapis.com` | Cloud Monitoring |

## IAM Role Bindings

- `roles:` -- bound to the learner service account. Keep narrow to prevent learners from breaking the sandbox.
- `admin_roles:` -- bound to the admin service account used by lifecycle scripts. Typically `roles/owner`.

Role names must use the full `roles/` prefix and match exactly. GCP roles are hierarchical:

- `roles/viewer` -- read-only across all services
- `roles/editor` -- read-write across most services (too broad for most tracks)
- `roles/storage.objectAdmin` -- full control of Cloud Storage objects only

## When to Use

- Track teaches GCP services via CLI, SDK, or Terraform and the learner does not need Docker, systemd, or kernel features on the workstation.
- Track provisions GCP infrastructure (Compute Engine, GKE, Cloud Storage, Cloud SQL) as exercises.
- Track uses Terraform to deploy GCP resources.
- Track teaches GCP security, networking, or IAM topics.

## When NOT to Use

- Track needs Docker on the workstation (building images, running local containers) -- use `vm-plus-gcp.md`.
- Track needs systemd or kernel features on the workstation -- use `vm-plus-gcp.md`.
- Track needs both local infrastructure (VM with services) and GCP resources -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **Using non-existent IAM roles**. GCP does not have generic roles like `roles/writer` or `roles/admin`. Use the correct predefined role names: `roles/storage.objectCreator`, `roles/compute.instanceAdmin.v1`, `roles/container.admin`, etc. Check available roles with `gcloud iam roles list`.
- **Missing API enablement**. If an API is not listed under `services:`, gcloud commands fail with `API [foo.googleapis.com] not enabled on project`. This looks like a permissions error but is an enablement error.
- **Forgetting dependent APIs**. Some services require multiple APIs. Cloud Functions needs `cloudfunctions.googleapis.com`, `cloudbuild.googleapis.com`, and `artifactregistry.googleapis.com`. GKE needs `container.googleapis.com` and `compute.googleapis.com`.
- **Not authenticating in setup scripts**. gcloud does not auto-authenticate on cloud-client. You must decode the service account key and call `gcloud auth activate-service-account` in every setup script that uses gcloud.
- **Leaving key files on disk**. After activating the service account, remove the key file (`rm -f /tmp/sa-key.json`). Learners should not be able to find admin key files.
- **Terraform and GCP credentials**. Set `GOOGLE_CREDENTIALS` or `GOOGLE_APPLICATION_CREDENTIALS` for Terraform. The simplest approach: `export GOOGLE_APPLICATION_CREDENTIALS=/tmp/sa-key.json` before running Terraform, or use `export GOOGLE_CREDENTIALS=$(echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d)`.
- **Role scope confusion**. `roles/editor` is extremely broad and lets learners modify almost anything in the project. Prefer service-specific roles like `roles/storage.admin` or `roles/compute.instanceAdmin.v1`.
- **API enablement timing**. APIs can take 30-60 seconds to become fully available after enablement. If a setup script immediately tries to use a newly enabled API, it may fail. The `services:` list in config.yml handles this before the sandbox starts, so prefer listing APIs there rather than enabling them in setup scripts.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Cloud Storage operations (create buckets, upload objects, set IAM policies)
- GKE cluster management (create cluster, deploy workloads, configure autoscaling)
- Compute Engine lifecycle (create VMs, configure firewalls, attach disks)
- Terraform on GCP (provision VPCs, subnets, Compute Engine, Cloud SQL)
- Cloud Functions development (create, deploy, test, configure triggers)
- BigQuery data analysis (create datasets, run queries, manage access)
