# Cloud-Client Container + GCP Project + GKE Cluster

Standard pattern for GCP Kubernetes workshops. A cloud-client container provides kubectl, helm, and gcloud. A sandboxed GCP project hosts a GKE cluster provisioned during setup.

See `cloud-accounts/container-plus-gcp.md` for GCP project configuration details.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
gcp_projects:
  - name: sandbox
    managed: true
    roles:
      - roles/container.admin
      - roles/compute.admin
      - roles/iam.serviceAccountUser
      - roles/iam.serviceAccountAdmin
      - roles/storage.admin
    services:
      - container.googleapis.com
      - compute.googleapis.com
      - containerregistry.googleapis.com
      - artifactregistry.googleapis.com
      - iam.googleapis.com
      - cloudresourcemanager.googleapis.com
```

## Setup Script Pattern

GKE provisioning takes approximately 5-10 minutes. GKE Autopilot clusters can take slightly longer on initial workload scheduling.

### Standard GKE cluster

```bash
#!/bin/bash

CLUSTER_NAME="lab"
ZONE="us-central1-a"
PROJECT=$(gcloud config get-value project)

gcloud container clusters create "$CLUSTER_NAME" \
  --zone "$ZONE" \
  --num-nodes 2 \
  --machine-type e2-standard-2 \
  --enable-ip-alias \
  --no-enable-autoupgrade \
  --quiet

# gcloud container clusters create automatically configures kubeconfig
kubectl get nodes
```

### GKE Autopilot cluster

```bash
#!/bin/bash

CLUSTER_NAME="lab"
REGION="us-central1"

gcloud container clusters create-auto "$CLUSTER_NAME" \
  --region "$REGION" \
  --quiet

gcloud container clusters get-credentials "$CLUSTER_NAME" \
  --region "$REGION"

kubectl get nodes
```

### Kubeconfig retrieval (if cluster already exists)

```bash
# Zonal cluster
gcloud container clusters get-credentials lab --zone us-central1-a

# Regional / Autopilot cluster
gcloud container clusters get-credentials lab --region us-central1
```

## Cleanup Script

```bash
#!/bin/bash

gcloud container clusters delete lab \
  --zone us-central1-a \
  --quiet \
  --async
```

Use `--async` in cleanup to avoid blocking on the delete operation. The sandboxed project gets destroyed at track end regardless.

## When to Use

- Track teaches GKE-specific features (Autopilot, Config Connector, Workload Identity, GKE Ingress).
- Track teaches tools that integrate with GKE (Anthos, Cloud Run for Anthos, Traffic Director, Binary Authorization).
- Track teaches Kubernetes on GCP for a customer whose platform is GKE.
- Track needs Artifact Registry or Container Registry integration for image workflows.

## When NOT to Use

- Track teaches generic Kubernetes concepts with no GCP-specific requirements -- use `base-compute/single-vm-kind-k3s.md` instead. It is faster to provision and cheaper to run.
- Track only needs GCP services without Kubernetes -- use `cloud-accounts/container-plus-gcp.md` instead.
- Track needs a local development cluster, not a managed cloud cluster -- use `base-compute/single-vm-kind-k3s.md`.

## Common Pitfalls

- **Missing API enablement**. GKE requires `container.googleapis.com` and `compute.googleapis.com` enabled on the project. The `services` list in config.yml ensures these are enabled before setup runs. Without them, `gcloud container clusters create` fails with a "API not enabled" error.
- **Missing `roles/iam.serviceAccountUser`**. GKE nodes run as a service account. Without this role, the cluster creation fails because the sandbox user cannot act as the default compute service account.
- **Zone vs region**. Standard clusters are zonal (`--zone us-central1-a`). Autopilot and regional clusters use `--region us-central1`. Using the wrong flag causes "not found" errors when retrieving credentials.
- **Autopilot scheduling delay**. Autopilot clusters do not pre-provision nodes. The first workload deployment triggers node provisioning, which adds 1-3 minutes. Account for this in setup if deploying initial workloads.
- **`--no-enable-autoupgrade`**. Without this flag, GKE may start upgrading nodes during the lab, causing disruption. Always disable auto-upgrade for lab clusters.
- **Kubeconfig auto-configuration**. `gcloud container clusters create` automatically runs `get-credentials`. If setup does anything that disrupts kubeconfig after creation (e.g., switching projects), run `gcloud container clusters get-credentials` again.
- **Quota limits**. GCP projects have default quotas for CPUs, IP addresses, and other resources. Two-node clusters with `e2-standard-2` are within default quotas. Larger clusters or GPU workloads may hit quota limits.
- **Artifact Registry vs Container Registry**. Container Registry (`gcr.io`) is in maintenance mode. Prefer Artifact Registry (`artifactregistry.googleapis.com`) for new tracks. Enable both if the track references legacy `gcr.io` paths.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- GKE cluster provisioning and management
- GKE Autopilot vs Standard cluster comparison
- Workload Identity configuration on GKE
- Config Connector for managing GCP resources from Kubernetes
- Binary Authorization and supply chain security on GKE
- Anthos Service Mesh setup and traffic management
- Deploying applications to GKE with Cloud Build and Artifact Registry
