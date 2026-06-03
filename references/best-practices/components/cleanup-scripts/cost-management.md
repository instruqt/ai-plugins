# Cost Management

Evaluates whether cleanup scripts prevent cost leakage by ensuring all billable external resources are deleted when the track ends, with defensive patterns that handle partial or failed provisioning.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No cost awareness; cloud resources, SaaS subscriptions, and API keys left running after track ends; direct financial impact |
| 2 | Below Standard | Some costed resources cleaned up but high-cost items (GPU VMs, cloud subscriptions) missed or cleanup is fragile |
| 3 | Adequate | All known costed resources have cleanup entries, but cleanup fails silently if a resource was partially provisioned or already deleted |
| 4 | Good | All costed resources cleaned up with resource IDs retrieved from agent variables (production baseline) |
| 5 | Excellent | Cleanup is defensive (|| true on every delete call), logs what it deleted for audit, and handles every partial-provisioning edge case |

## Guidance

Costed resources are anything that accrues charges outside the Instruqt sandbox. These must be cleaned up in the track-level cleanup script (not per-challenge cleanup). The track-level cleanup runs when the track ends, whether the learner completed it, abandoned it, or timed out.

### Defensive deletion pattern

Every delete call should be guarded so that one failure does not prevent subsequent cleanups:

Good -- defensive cleanup with logging:

```bash
#!/bin/bash

# Retrieve all resource IDs stored during setup
GPU_INSTANCE=$(agent variable get GPU_INSTANCE_NAME)
ELASTIC_PROJECT=$(agent variable get ELASTIC_PROJECT_ID)
LITELLM_KEY=$(agent variable get LITELLM_KEY_ID)
AZURE_SUB=$(agent variable get AZURE_SUBSCRIPTION_ID)

# Delete GPU VM
if [ -n "$GPU_INSTANCE" ]; then
  echo "Deleting GPU instance: $GPU_INSTANCE"
  gcloud compute instances delete "$GPU_INSTANCE" \
    --zone=us-central1-a --quiet || true
else
  echo "No GPU instance to clean up"
fi

# Delete Elastic Serverless project
if [ -n "$ELASTIC_PROJECT" ]; then
  echo "Deleting Elastic project: $ELASTIC_PROJECT"
  curl -sf -X DELETE \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/${ELASTIC_PROJECT}" \
    -H "Authorization: ApiKey ${EC_API_KEY}" || true
else
  echo "No Elastic project to clean up"
fi

# Revoke LiteLLM API key
if [ -n "$LITELLM_KEY" ]; then
  echo "Revoking LiteLLM key: $LITELLM_KEY"
  curl -sf -X POST "https://litellm.internal/key/delete" \
    -H "Authorization: Bearer ${LITELLM_MASTER_KEY}" \
    -d "{\"keys\": [\"${LITELLM_KEY}\"]}" || true
else
  echo "No LiteLLM key to clean up"
fi

# Cancel Azure subscription
if [ -n "$AZURE_SUB" ]; then
  echo "Cancelling Azure subscription: $AZURE_SUB"
  az account subscription cancel --id "$AZURE_SUB" --yes || true
else
  echo "No Azure subscription to clean up"
fi
```

### SaaS tenant deprovisioning

Good -- cleaning up a service account created for the learner:

```bash
SERVICE_ACCOUNT_ID=$(agent variable get APPCO_SERVICE_ACCOUNT_ID)
if [ -n "$SERVICE_ACCOUNT_ID" ]; then
  echo "Deleting service account: $SERVICE_ACCOUNT_ID"
  curl -sf -X DELETE "https://api.appco.io/v1/service-accounts/${SERVICE_ACCOUNT_ID}" \
    -H "Authorization: Bearer ${APPCO_ADMIN_TOKEN}" || true
fi
```

Bad -- no || true (first failure blocks everything):

```bash
#!/bin/bash
gcloud compute instances delete "$GPU_INSTANCE" --zone=us-central1-a --quiet
# If the above fails, nothing below runs
curl -X DELETE "https://api.elastic-cloud.com/api/v1/serverless/projects/${PROJECT_ID}"
az account subscription cancel --id "$AZURE_SUB" --yes
```

Bad -- no empty-variable guard:

```bash
#!/bin/bash
# If GPU_INSTANCE is empty, this deletes... nothing, but the gcloud command
# may error or behave unexpectedly with an empty argument
gcloud compute instances delete "$GPU_INSTANCE" --zone=us-central1-a --quiet || true
```

Bad -- no logging (impossible to audit what was cleaned up):

```bash
#!/bin/bash
gcloud compute instances delete "$GPU_INSTANCE" --zone=us-central1-a --quiet || true
curl -sf -X DELETE "https://api/projects/${PROJECT_ID}" || true
# If costs leak, there's no record of what cleanup attempted
```

Bad -- cleanup in per-challenge script instead of track-level:

```bash
#!/bin/bash
# Per-challenge cleanup: DON'T put resource teardown here
# This only runs if the learner passes the check; if they abandon
# the track, per-challenge cleanup never runs for remaining challenges
gcloud compute instances delete "$GPU_INSTANCE" --quiet
```

## What to Watch For

- Track-level cleanup is the only reliable place for cost-critical teardown; per-challenge cleanup does not run for challenges the learner never reaches
- Every delete/cancel/revoke call must have `|| true` to prevent cascading failures
- Always check that the resource ID variable is non-empty before attempting deletion
- Log the resource ID being deleted so that cost leaks can be investigated from track logs
- Order deletions from most expensive to least expensive; if the script times out, the highest-cost items are already handled
- Cloud provider CLI commands need `--quiet` or `--yes` flags to suppress interactive confirmation prompts
- API tokens used for cleanup (admin tokens, master keys) should be available as sandbox environment variables, not hardcoded
