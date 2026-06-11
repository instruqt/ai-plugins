# Resource Teardown

Evaluates whether cleanup scripts properly delete all external and costed resources that were provisioned during setup or challenge execution.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No cleanup script exists; external resources (cloud VMs, SaaS accounts, API keys) are orphaned after the track ends |
| 2 | Below Standard | Some resources cleaned up but others missed; resource IDs hardcoded instead of retrieved dynamically |
| 3 | Adequate | All known resources have cleanup logic, but IDs are not retrieved from agent variables; cleanup assumes all resources exist |
| 4 | Good | Every provisioned external resource has cleanup; resource IDs retrieved via agent variable get and deleted (production baseline) |
| 5 | Excellent | Cleanup handles partial provisioning gracefully -- checks whether each resource exists before deleting, uses || true to prevent failures from blocking subsequent deletions |

## Guidance

Any resource created outside the Instruqt sandbox (cloud resources, SaaS accounts, API keys) must be explicitly cleaned up. The sandbox itself is destroyed after the track, but external resources persist and accrue cost.

### Retrieving resource IDs from agent variables

Resources provisioned during setup should store their IDs in agent variables so cleanup can retrieve them:

Good -- setup stores the resource ID:

```bash
# In setup script
INSTANCE_ID=$(gcloud compute instances create gpu-worker \
  --zone=us-central1-a --machine-type=n1-standard-8 \
  --accelerator=count=1,type=nvidia-tesla-t4 \
  --format='value(name)')
agent variable set INSTANCE_ID "$INSTANCE_ID"
```

Good -- cleanup retrieves and deletes:

```bash
#!/bin/bash
# Retrieve the resource ID
INSTANCE_ID=$(agent variable get INSTANCE_ID)

# Delete the resource
if [ -n "$INSTANCE_ID" ]; then
  gcloud compute instances delete "$INSTANCE_ID" \
    --zone=us-central1-a --quiet || true
fi
```

### Common external resource patterns

Good -- AWS account subscription cleanup:

```bash
SUBSCRIPTION_ID=$(agent variable get AWS_SUBSCRIPTION_ID)
if [ -n "$SUBSCRIPTION_ID" ]; then
  curl -sf -X DELETE "https://api.internal/subscriptions/${SUBSCRIPTION_ID}" \
    -H "Authorization: Bearer ${API_TOKEN}" || true
fi
```

Good -- Elastic Serverless project cleanup:

```bash
PROJECT_ID=$(agent variable get ELASTIC_PROJECT_ID)
if [ -n "$PROJECT_ID" ]; then
  curl -sf -X DELETE "https://api.elastic-cloud.com/api/v1/serverless/projects/${PROJECT_ID}" \
    -H "Authorization: ApiKey ${EC_API_KEY}" || true
fi
```

Good -- LiteLLM API key cleanup:

```bash
KEY_ID=$(agent variable get LITELLM_KEY_ID)
if [ -n "$KEY_ID" ]; then
  curl -sf -X POST "https://litellm.internal/key/delete" \
    -H "Authorization: Bearer ${LITELLM_MASTER_KEY}" \
    -d "{\"keys\": [\"${KEY_ID}\"]}" || true
fi
```

Bad -- no cleanup for provisioned resources:

```bash
#!/bin/bash
# Cleanup script is empty or missing
# The GPU VM created in setup will keep running and costing money
```

Bad -- hardcoded resource ID:

```bash
#!/bin/bash
# DON'T hardcode; the ID changes every run
gcloud compute instances delete "gpu-worker-abc123" --quiet
```

Bad -- cleanup fails on first error, skipping subsequent resources:

```bash
#!/bin/bash
set -e  # First failure stops the script
gcloud compute instances delete "$INSTANCE_ID" --quiet  # If this fails...
curl -X DELETE "https://api/subscriptions/${SUB_ID}"     # ...this never runs
```

## What to Watch For

- Every resource provisioned in setup or during challenges must have a corresponding cleanup step
- Resource IDs must be stored in agent variables during provisioning and retrieved during cleanup
- Never use `set -e` in cleanup scripts without `|| true` on delete commands -- one failure should not block cleanup of other resources
- Check that the resource ID is non-empty before attempting deletion to handle partial provisioning
- API calls to delete resources should use `|| true` to prevent cleanup script failure if the resource was already deleted or never created
- Order cleanup from most expensive to least expensive resources so cost-critical items are cleaned first
