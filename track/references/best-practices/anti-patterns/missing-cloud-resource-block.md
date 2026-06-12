---
severity: blocking
---
# Missing Cloud Resource Block

Referencing cloud environment variables like `INSTRUQT_AWS_ACCOUNT_*`, `INSTRUQT_AZURE_SUBSCRIPTION_*`, or `INSTRUQT_GCP_PROJECT_*` without a matching resource block in config.yml.

## Why It's Bad

These environment variables receive empty strings if the corresponding cloud resource block is absent from config.yml. Scripts proceed with empty credentials, causing silent authentication failures that are difficult to debug. The track appears to start normally but cloud operations fail deep in the setup flow.

## What to Do Instead

Always declare the matching resource block in config.yml before referencing cloud environment variables in lifecycle scripts.

## Examples

Bad -- referencing GCP variables without a resource block:

```bash
# setup-sandbox
gcloud config set project "${INSTRUQT_GCP_PROJECT_PROJECT_ID}"
# INSTRUQT_GCP_PROJECT_PROJECT_ID is empty -- no resource block in config.yml
```

Good -- declaring the resource block in config.yml:

```yaml
# config.yml
gcp_projects:
  - id: project
    roles:
      - roles/editor
```

```bash
# setup-sandbox
gcloud config set project "${INSTRUQT_GCP_PROJECT_PROJECT_ID}"
# Variable is populated from the provisioned sandbox project
```
