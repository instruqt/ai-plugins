# VM Plus GCP

Full VM workstation paired with a sandboxed GCP project. Use when the track needs OS-level features (Docker, systemd, nested virtualization) alongside GCP cloud resources. The GCP environment variables are injected into the VM just as they would be into a container.

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

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - container.googleapis.com
      - storage.googleapis.com
      - artifactregistry.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/compute.instanceAdmin.v1
      - roles/container.admin
      - roles/storage.admin
      - roles/artifactregistry.writer
    admin_roles:
      - roles/owner
```

### Variant: Docker-based workflow with Artifact Registry

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

gcp_projects:
  - name: sandbox
    services:
      - compute.googleapis.com
      - container.googleapis.com
      - storage.googleapis.com
      - artifactregistry.googleapis.com
      - cloudbuild.googleapis.com
      - run.googleapis.com
    regions:
      - us-central1
    roles:
      - roles/container.admin
      - roles/artifactregistry.writer
      - roles/run.developer
      - roles/iam.serviceAccountUser
    admin_roles:
      - roles/owner
```

### Setup script: install gcloud CLI on the VM

```bash
#!/bin/bash

# Install gcloud CLI
curl -fsSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz \
  -o /tmp/gcloud.tar.gz
tar -xzf /tmp/gcloud.tar.gz -C /opt
/opt/google-cloud-sdk/install.sh --quiet --path-update true
ln -s /opt/google-cloud-sdk/bin/gcloud /usr/local/bin/gcloud
ln -s /opt/google-cloud-sdk/bin/gsutil /usr/local/bin/gsutil
rm -f /tmp/gcloud.tar.gz

# Authenticate with the service account
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/sa-key.json
gcloud auth activate-service-account --key-file=/tmp/sa-key.json
gcloud config set project "$INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID"
rm -f /tmp/sa-key.json
```

### Setup script: Docker + Artifact Registry login

```bash
#!/bin/bash

# Install Docker
apt-get update
apt-get install -y docker.io
systemctl enable --now docker

# Install gcloud
curl -fsSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz \
  -o /tmp/gcloud.tar.gz
tar -xzf /tmp/gcloud.tar.gz -C /opt
/opt/google-cloud-sdk/install.sh --quiet --path-update true
ln -s /opt/google-cloud-sdk/bin/gcloud /usr/local/bin/gcloud
rm -f /tmp/gcloud.tar.gz

# Authenticate
echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d > /tmp/sa-key.json
gcloud auth activate-service-account --key-file=/tmp/sa-key.json
gcloud config set project "$INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID"

# Configure Docker to use gcloud for Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev --quiet

rm -f /tmp/sa-key.json
```

## Auto-Injected Environment Variables

The same GCP environment variables are injected into the VM as into a container. For `name: sandbox`:

| Variable | Description |
|----------|-------------|
| `INSTRUQT_GCP_PROJECT_SANDBOX_PROJECT_ID` | GCP project ID |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_EMAIL` | Learner user email |
| `INSTRUQT_GCP_PROJECT_SANDBOX_USER_PASSWORD` | Learner user password |
| `INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY` | Learner SA key (base64-encoded JSON) |
| `INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_EMAIL` | Learner SA email |
| `INSTRUQT_GCP_PROJECT_SANDBOX_ADMIN_SERVICE_ACCOUNT_KEY` | Admin SA key (base64-encoded JSON) |

Note: unlike cloud-client containers, VMs do not have gcloud pre-installed or pre-configured. You must install gcloud and run `gcloud auth activate-service-account` manually.

## When to Use

- Track needs Docker on the workstation (building container images, pushing to Artifact Registry, running local containers alongside GCP services).
- Track needs systemd services running alongside gcloud CLI work.
- Track teaches GKE workflows where the learner builds images locally and deploys to a GKE cluster.
- Track needs heavy tooling not available in cloud-client (custom SDKs, large binaries, GUI tools).
- Track needs nested virtualization for complex local environments that also interact with GCP.

## When NOT to Use

- Track only needs CLI/SDK/Terraform access to GCP -- use `container-plus-gcp.md` for faster startup and lower resource usage.
- Track needs both a VM (for services) and a lightweight CLI workstation -- use `hybrid-plus-cloud.md`.

## Common Pitfalls

- **No gcloud pre-installed**. VMs are bare OS images. The gcloud CLI installation takes 1-2 minutes. For tracks that need fast startup, pre-bake gcloud into a custom Packer image.
- **Forgetting to authenticate**. Even with env vars injected, gcloud does not auto-authenticate. You must decode the base64 key and call `gcloud auth activate-service-account` in every setup script.
- **Leaving key files on disk**. Always `rm -f /tmp/sa-key.json` after activating the service account. Learners should not find admin credentials lying around.
- **Using non-existent IAM roles**. GCP does not have generic roles like `roles/writer`. Use the correct predefined role names: `roles/storage.objectCreator`, `roles/compute.instanceAdmin.v1`, etc.
- **Missing API enablement**. Every API the track uses must be listed in `services:`. Missing APIs cause `API [foo.googleapis.com] not enabled` errors.
- **Long setup times**. Installing Docker + gcloud + other tools can take 3-5 minutes on a bare VM. Custom Packer images significantly reduce this.
- **Docker not running**. Always `systemctl enable --now docker` after installation.
- **Terraform credentials**. Set `GOOGLE_APPLICATION_CREDENTIALS` or `GOOGLE_CREDENTIALS` for Terraform: `export GOOGLE_CREDENTIALS=$(echo "$INSTRUQT_GCP_PROJECT_SANDBOX_SERVICE_ACCOUNT_KEY" | base64 -d)`.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Container image build and push to Artifact Registry (Docker build + push + Cloud Run deploy)
- GKE local development (build images locally, deploy to GKE cluster)
- Cloud Build pipeline development (local source + Cloud Build triggers)
- Hybrid application deployment (local Docker Compose app connecting to Cloud SQL/Cloud Storage)
- Terraform module development with local testing and GCP deployment
- Packer image creation for GCP Compute Engine
