---
severity: blocking
---
# Skip Cleanup for Costed Resources

Omitting cleanup scripts for tracks that provision external or costed resources.

## Why It's Bad

Resources like Elastic Serverless projects, AppCo service accounts, LiteLLM keys, GCP GPU VMs, and similar billable infrastructure accumulate if not torn down. Without cleanup, abandoned resources run up costs indefinitely. This is especially dangerous for tracks run at scale during workshops or events.

## What to Do Instead

Every setup that creates an external or costed resource MUST have a corresponding cleanup script that:

1. Retrieves the resource ID via `agent variable get`.
2. Deletes or deprovisions the resource.

Store resource identifiers as agent variables during setup so cleanup can always find them.

## Examples

Bad -- creating an Elastic Serverless project without cleanup:

```bash
# setup-sandbox
PROJECT_ID=$(curl -s -X POST https://api.elastic.co/projects \
  -H "Authorization: ApiKey ${ELASTIC_API_KEY}" \
  -d '{"name": "lab-project"}' | jq -r '.id')
agent variable set ELASTIC_PROJECT_ID "$PROJECT_ID"
```

Good -- adding the matching cleanup script:

```bash
# cleanup-sandbox
PROJECT_ID=$(agent variable get ELASTIC_PROJECT_ID)
if [ -n "$PROJECT_ID" ]; then
  curl -s -X DELETE "https://api.elastic.co/projects/${PROJECT_ID}" \
    -H "Authorization: ApiKey ${ELASTIC_API_KEY}"
fi
```
